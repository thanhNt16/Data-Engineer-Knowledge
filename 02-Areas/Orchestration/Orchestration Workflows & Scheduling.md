---
tags: [orchestration, airflow, dagster, workflow, scheduling]
date: 2026-02-15
status: learning
---

# Orchestration: Workflow & Scheduling

## Overview

Orchestration is the process of managing data workflows: scheduling, executing, and monitoring data pipelines and jobs.

**Goal**: Ensure reliable, scalable data pipeline execution.

---

## Orchestration Patterns

### 1. Directed Acyclic Graph (DAG)

**Definition**: A workflow with dependencies represented as a directed graph with no cycles.

**Example**:
```
Task A (Extract)
   ↓
Task B (Transform)
   ↓
Task C (Load)
```

**Key Properties**:
- **Direction**: Edges show task dependencies (A must complete before B)
- **Acyclic**: No circular dependencies (A→B→C→A would deadlock)
- **Parallelism**: Tasks with no dependencies run concurrently

**Anti-Pattern**: Diamond dependency (converging DAG) can cause resource contention.

### 2. Orchestration Strategies

| Strategy | Description | Pros | Cons |
|-----------|-------------|------|-------|
| **Scheduler-Based** | Cron-like scheduling | Simple | Limited flexibility |
| **Event-Driven** | Trigger on data arrival | Real-time | Hard to debug |
| **Workflow-Based** | Code-defined workflows | Flexible | More complex |

### 3. Airflow vs. Dagster Comparison

| Aspect | Apache Airflow | Dagster |
|---------|-----------------|----------|
| **Abstraction Level** | DAGs (imperative) | Assets (declarative) |
| **Code Location** | Separate DAG files | Asset functions (with code) |
| **Data Passing** | XCom (pull/push) | Function inputs/outputs |
| **State Management** | Manual | Built-in (Op/IO) |
| **Testing** | Hard (requires airflow run) | Easy (unit test assets) |
| **UI** | Airflow Web UI | Dagster UI (limited) |
| **Community** | Mature | Growing |

---

## Apache Airflow

### Core Concepts

#### 1. DAG (Directed Acyclic Graph)

```python
from airflow import DAG
from datetime import datetime

dag = DAG(
    'daily_etl_pipeline',
    default_args={'owner': 'data_engineering'},
    schedule_interval='@daily',  # Runs every day at midnight
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=['etl', 'daily']
)
```

#### 2. Operators

**Operators** are the building blocks of Airflow tasks.

| Operator Category | Examples | Use Case |
|------------------|----------|----------|
| **Database** | PostgresOperator, MySqlOperator, SnowflakeOperator | Run SQL, load data |
| **File** | FileSensor, BashOperator | Wait for files, run scripts |
| **Data Transfer** | S3ToRedshiftOperator, SFTPOperator | Move data |
| **Email** | EmailOperator | Send notifications |
| **HTTP** | SimpleHttpOperator, HttpOperator | API calls |
| **Sensor** | FileSensor, SqlSensor, TimeSensor | Wait for conditions |

#### 3. Task Dependencies

```python
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator

# Define tasks
extract_task = PythonOperator(
    task_id='extract_from_api',
    python_callable=extract_data,
    dag=dag
)

transform_task = BashOperator(
    task_id='transform_data',
    bash_command='python /scripts/transform.py',
    dag=dag
)

load_task = PostgresOperator(
    task_id='load_to_warehouse',
    postgres_conn_id='warehouse_db',
    sql='INSERT INTO fact_sales SELECT * FROM stg_sales',
    dag=dag
)

# Define dependencies (DAG structure)
extract_task >> transform_task >> load_task
```

### Advanced Airflow Patterns

#### 1. Dynamic DAGs

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
import os

def generate_dag(dataset_name):
    """Generate DAG dynamically for each dataset"""
    dag_id = f'process_{dataset_name}'

    with DAG(
        dag_id=dag_id,
        schedule_interval='@once',
        start_date=datetime(2024, 1, 1),
        catchup=False,
        tags=['dynamic', 'datasets']
    ) as dag:
        extract_task = PythonOperator(
            task_id=f'extract_{dataset_name}',
            python_callable=extract_from_s3,
            op_kwargs={'dataset': dataset_name},
            dag=dag
        )

        load_task = PythonOperator(
            task_id=f'load_{dataset_name}',
            python_callable=load_to_warehouse,
            op_kwargs={'dataset': dataset_name},
            dag=dag
        )

        extract_task >> load_task

    return dag

# Generate DAGs for all datasets
datasets = ['sales', 'customers', 'products']
for dataset in datasets:
    generate_dag(dataset)
```

#### 2. Branching

```python
from airflow.operators.python import BranchPythonOperator
from airflow.utils.trigger_rule import TriggerRule

# Task with branching (success/failure)
def check_data_quality(**context):
    records = context['ti'].xcom_pull(task_ids='validate_task')
    if records > 0:
        return 'process'  # Good quality, proceed
    else:
        return 'skip'  # Bad quality, skip processing

branch_task = BranchPythonOperator(
    task_id='branch_on_quality',
    python_callable=check_data_quality,
    trigger_rule=TriggerRule.SUCCESS_ONE,
    dag=dag
)

process_task = PythonOperator(
    task_id='process_data',
    python_callable=process_data,
    trigger_rule='process',  # Only runs if branch_task returns 'process'
    dag=dag
)

skip_task = PythonOperator(
    task_id='skip_processing',
    python_callable=skip_data,
    trigger_rule='skip',  # Only runs if branch_task returns 'skip'
    dag=dag
)

branch_task >> [process_task, skip_task]
```

#### 3. Custom Operators

```python
from airflow.models import BaseOperator
from airflow.utils.decorators import apply_defaults

class CustomFileOperator(BaseOperator):
    @apply_defaults
    def __init__(self, source_path, dest_path, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.source_path = source_path
        self.dest_path = dest_path

    def execute(self, context):
        source = self.source_path
        dest = self.dest_path

        # Custom logic
        with open(source, 'r') as f:
            data = f.read()
        # Transform data here
            processed_data = data.upper()

        with open(dest, 'w') as f:
            f.write(processed_data)

        # Log file metadata
        context['ti'].xcom_push('file_info', {
            'source': source,
            'dest': dest,
            'size': len(data)
        })
```

---

## Dagster

### Core Concepts

#### 1. Software-Defined Assets

**Definition**: Assets are the foundational objects in your data platform - tables, ML models, features, files, etc.

```python
from dagster import asset, Definitions

@asset
def orders_raw():
    """Raw orders from API"""
    return fetch_orders_from_api()

@asset
def orders_clean(orders_raw):
    """Clean and validate orders"""
    df = orders_raw.copy()

    # Remove nulls
    df = df.dropna()

    # Validate amounts
    df = df[df['amount'] > 0]

    # Validate status
    df = df[df['status'].isin(['completed', 'pending'])]

    return df

@asset
def daily_orders(orders_clean):
    """Daily aggregated orders"""
    df = orders_clean.copy()
    df['order_date'] = pd.to_datetime(df['created_at'])

    return df.groupby('order_date').agg({
        'total_revenue': ('amount', 'sum'),
        'order_count': ('order_id', 'count')
    })
```

#### 2. Asset Dependencies

```python
from dagster import asset, Definitions

# Assets have explicit dependencies
@asset(deps=[orders_clean])
def orders_enriched(orders_clean):
    """Enrich orders with user data"""
    users = fetch_users()

    return orders_clean.merge(
        users,
        on='user_id',
        how='left'
    )

@asset(deps=[orders_enriched])
def daily_revenue(orders_enriched):
    """Calculate daily revenue"""
    return orders_enriched.groupby('order_date')['total_revenue'].sum()
```

#### 3. Materialization

**Definition**: Process of storing asset data to persistent storage (warehouse, lakehouse).

```python
from dagster import materialize, Definitions

# Materialize to SQL warehouse
@materialize(
    write_to_metadata=True,
    io_manager="warehouse_io",
    mode="replace",
)
def materialize_orders(orders_clean):
    """Store orders in data warehouse"""
    return orders_clean

# Materialize to CSV (data lake)
@materialize(
    write_to_metadata=False,
    output_format="csv",
    path="s3://data-lake/orders/",
    mode="append",
)
def materialize_to_s3(orders_clean):
    """Append to S3 data lake"""
    return orders_clean
```

### Dagster vs. Airflow Code Comparison

| Operation | Airflow (DAG-based) | Dagster (Asset-based) |
|-----------|---------------------|---------------------|
| **Define** | DAG object in .py file | @asset function in .py file |
| **Dependencies** | A >> B >> C | deps=[asset1, asset2] |
| **Data Passing** | XCom (pull/push) | Function inputs/outputs |
| **Testing** | Run airflow, check logs | pytest test_assets() |
| **State** | Manual in operator | Built-in Op/IO |
| **Documentation** | Separate DAG docs | Docstrings in code |
| **UI** | Rich Airflow UI | Dagster UI (Dagit) |
| **Deployment** | Separate scheduler | Integrated scheduler |

---

## Orchestration Best Practices

### 1. Idempotency

**Definition**: Running the same job multiple times has the same result as running it once.

**Airflow**:
```python
# Idempotent operator (overwrite instead of append)
INSERT INTO target_table (user_id, name)
SELECT user_id, name FROM source
ON CONFLICT (user_id) DO NOTHING;
```

**Dagster**:
```python
# Idempotent asset (replaces instead of appends)
@asset
def load_data_to_db(context, upstream_data):
    context.log.info(f"Loading {len(upstream_data)} records")
    # Write replaces existing data
    write_to_db(upstream_data, mode='replace')
```

### 2. Error Handling

**Airflow**: Retries,sla_miss_timeout,donot_retry_on_down.

```python
load_task = PostgresOperator(
    task_id='load_data',
    retries=3,
    retry_delay=timedelta(minutes=5),
    sla_miss_timeout=timedelta(hours=1),
    donot_retry_on_down=True,
    dag=dag
)
```

**Dagster**: Built-in retry policies.

```python
from dagster import job, op

@op(retry_policy="exponential_backoff", max_retries=3)
def fetch_api():
    response = requests.get("https://api.example.com/data")
    response.raise_for_status()
    return response.json()
```

### 3. Backfilling

**Definition**: Reprocessing historical data.

**Airflow**:
```python
from airflow.operators.python import BashOperator

# Manual backfill command
backfill_command = """
airflow dags backfill -s 2024-01-01 -e 2024-01-31 \\
    --task_regex process_data
    --reset_dagruns True
"""

backfill_task = BashOperator(
    task_id='backfill_january_data',
    bash_command=backfill_command,
    dag=dag
)
```

**Dagster**:
```python
from dagster import backfill

# Backfill asset from Jan 1-31
backfill_result = backfill(
    assets=[daily_orders],
    start=PartitionKey("date", "2024-01-01"),
    end=PartitionKey("date", "2024-01-31"),
    partition_window=timedelta(days=7)  # Re-process last 7 days
)

print(f"Backfilled {backfill_result.success_count} partitions")
```

---

## Modern Orchestration: Temporal

### Workflow as Code

**Definition**: Define workflows in code (Python/Go/TypeScript) as functions.

**Example**:
```python
from temporalio import workflow, activity, Client

@workflow
def order_processing(order_id: str):
    """Order processing workflow"""
    # Activity 1: Reserve inventory
    @activity
    def reserve_inventory():
        # Call inventory service
        result = inventory_service.reserve(order_id)
        return result

    # Activity 2: Process payment
    @activity
    def process_payment():
        # Call payment service
        result = payment_service.process(order_id)
        return result

    # Activity 3: Ship order
    @activity
    def ship_order():
        # Call shipping service
        result = shipping_service.ship(order_id)
        return result

# Start workflow
client = Client()
result = client.execute_workflow(order_processing, "order-123")
print(f"Workflow result: {result}")
```

**Features**:
- Exactly-once semantics (built-in)
- Visibility into workflow execution
- Compensation actions (retries)
- Scheduling and timeouts

---

## Monitoring & Observability

### Airflow Metrics

- **DAG Runs**: Success/failure rates, duration
- **Task Duration**: Individual task performance
- **Operator Duration**: Custom operator performance
- **SLA Misses**: Timed out tasks

### Dagster Metrics

- **Asset Materializations**: When assets were updated
- **Runs**: Successful/failed runs
- **Op/IO**: Input/output sizes, execution time
- **Resource Usage**: CPU, memory per job

---

## Interview Questions

**Q1: What's the difference between DAG and DAG?**

**A1**: DAG stands for Directed Acyclic Graph. It's a workflow structure where tasks have dependencies. It's not an acronym for "DAG and DAG."

**Q2: How does Airflow handle task dependencies?**

**A2**: Airflow executes tasks in topological order (parents before children). The scheduler determines which tasks are ready to run based on their dependencies and available resources.

**Q3: What's the difference between XCom and Dagster Op/IO?**

**A3**: XCom (Airflow) is a mechanism for passing data between tasks - it's explicit pull/push (task A pushes, task B pulls). Op/IO (Dagster) is a built-in typed system where outputs are automatically passed to downstream inputs - it's type-safe and cleaner.

**Q4: How would you design an idempotent data pipeline?**

**A4**: Use `ON CONFLICT DO NOTHING` (PostgreSQL) or `REPLACE INTO` (MySQL) to avoid duplicates. For streaming, use upsert logic with unique keys. Always design tasks to be safe to re-run.

**Q5: What's a backfill and when would you use it?**

**A5**: Backfill is the process of re-processing historical data (e.g., re-running a pipeline for the last 6 months). Use when:
- Bug fixes in pipeline logic
- Schema changes need to be applied to historical data
- New data source added with historical data
- Downstream jobs need corrected data

---

## Related Notes

- [[System Design]] - Scalability, caching, load balancing
- [[Apache Flink - Real-Time Analytics]] - Stateful stream processing
- [[Apache Kafka - Event Streaming]] - Event-driven architecture
- [[Data Engineering Trends 2026]] - Modern orchestration trends

---

## Resources

- [Airflow Documentation](https://airflow.apache.org/docs/apache-airflow/stable/)
- [Dagster Documentation](https://docs.dagster.io/)
- [Temporal Documentation](https://temporal.io/docs/)
- [Prefect Documentation](https://docs.prefect.io/)
- [Orchestration Patterns](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491942454/part1.html)

---

**Progress**: 🟢 Learning (Airflow familiar, Dagster concepts understood)

**Next Steps**:
- [ ] Practice Airflow DAG patterns (dynamic, branching)
- [ ] Implement Dagster assets for ETL pipeline
- [ ] Set up monitoring and alerting
- [ ] Design backfill strategy
- [ ] Compare Prefect with Airflow (modern alternative)
