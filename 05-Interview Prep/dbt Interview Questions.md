---
title: dbt Interview Questions & Skills
aliases:
  - dbt Interview Prep
  - Data Build Tool Interview
tags:
  - interview
  - dbt
  - data-engineering
  - analytics-engineering
  - ETL
date: 2024-02-19
source: Tech with Abhishek
author: Abhishek Kumar Gupta
status: active
---

# dbt Interview Questions & Skills for Analytics Engineers

> [!abstract] Overview
> Comprehensive guide covering all core, advanced, scenario, and behavioural questions for dbt data engineering roles. Each with direct, practical answers and real-world tips for both candidates and interviewers.

---

## dbt Fundamentals & Core Concepts

### Q1: What is dbt and how does it fit in the data engineering workflow?

> [!tip] Answer
> **dbt (Data Build Tool)** is an open-source framework for transforming, testing, and documenting data inside your data warehouse using SQL. It fits into **ELT (Extract, Load, Transform)** processes by performing transformations *after* loading raw data into the data warehouse, unlike traditional ETL tools that usually transform before loading.

---

### Q2: What's the difference between `ref()` and `source()`?

> [!info] Key Differences
>
> | Function | Purpose | Usage |
> |----------|---------|-------|
> | `ref('model_name')` | References other dbt models | Creates managed dependencies; ensures correct build order and lineage |
> | `source('source_name', 'table_name')` | References raw data sources | Tracks external tables not owned by dbt but consumed as source inputs |

---

### Q3: What is the function of `dbt_project.yml`?

> [!tip] Answer
> Central config file controlling project structure, default materializations, target schema, and test/default settings. It ensures all contributors use a unified setup and simplifies collaboration.

**Example:**
```yaml
name: my_dbt_project
version: 1.0.0
models:
  my_dbt_project:
    materialized: table
target-path: target/
```

---

### Q4: Explain dbt model materializations (table, view, incremental, ephemeral)

> [!success] Materialization Types
>
> | Type | Description | Use Case |
> |------|-------------|----------|
> | `table` | Create/replace a persistent table | Final models, frequently queried data |
> | `view` | Create/replace a dynamic (virtual) view | Lightweight transformations |
> | `incremental` | Update only new/changed records | Large datasets, saving compute |
> | `ephemeral` | No database artifact — SQL logic used as CTE | Helper logic, DRY code |

---

### Q5: How do you run dbt models and what are key CLI commands?

> [!note] Essential CLI Commands
>
> | Command | Purpose |
> |---------|---------|
> | `dbt run` | Builds all models |
> | `dbt test` | Runs schema/data tests |
> | `dbt seed` | Loads seed data from CSVs |
> | `dbt run --select model_name` | Runs only the specified model |
> | `dbt docs generate` | Builds documentation with lineage |

---

## Data Modeling, Project Setup & Configuration

### Q6: How do you structure a dbt project for clarity and scalability?

> [!tip] Best Practice Structure
>
> **Layered folder structure:**
> - `staging/` — Raw clean-up (mirror source tables)
> - `intermediate/` — Business logic and joins
> - `marts/` — Analytics-ready tables (fact, dimension)
>
> **Naming conventions:** `stg_`, `int_`, `dim_`, `fct_` for clarity
>
> **Meta fields** for tracking owner, data domain, sensitivity, etc.

**Project Structure Example:**
```
models/
  staging/
  marts/
  intermediate/
  schema.yml
```

---

## Advanced Data Engineering & Optimization

### Q7: How do you design and use incremental models in dbt?

> [!example] Incremental Model Pattern
> Incremental models process only new or updated data after initial run, using `materialized='incremental'` and a unique key (such as `updated_at`).

```sql
{{ config(materialized='incremental', unique_key='order_id') }}

select *
from {{ source('raw', 'orders') }}
where
  {% if is_incremental() %}
    updated_at > (select max(updated_at) from {{ this }})
  {% endif %}
```

> [!success] Benefit
> Dramatically reduces compute/cost for big tables.

---

### Q8: How do you optimize slow-performing dbt models?

> [!warning] Optimization Strategies
>
> 1. **Refactor large SQL** — Split into manageable models or CTEs
> 2. **Use incremental materialization** — Process only new data
> 3. **Apply warehouse features** — Clustering, partitioning, result caching
> 4. **Avoid excessive CTE nesting** — Keep queries readable
> 5. **Materialize complex joins** — As persistent tables for reuse

---

### Q9: What is a dbt snapshot and when would you use it?

> [!info] Answer
> **Snapshot tracks history (SCD)** when you want to record how values in your table change over time (e.g., customer address). You define snapshot logic specifying unique/id columns and when to capture changes.

**Example:**
```yaml
snapshots:
  - name: customer_status_history
    unique_key: user_id
    strategy: timestamp
    updated_at: updated_on
```

---

### Q10: How do you manage dependencies in large dbt projects?

> [!tip] Dependency Management
>
> - Use `ref()` throughout for all model dependencies
> - Layer code: `stg_`, `int_`, `fct_`
> - Modular model approach (one purpose per model)
> - Use **exposures** to document BI/ML data consumers for traceability
> - Visualize the DAG using `dbt docs`

---

## Quality, Testing & CI/CD

### Q11: How do you write and enforce tests in dbt?

> [!example] Schema Tests (Built-in)
> `not_null`, `unique`, `accepted_values`, etc.

```yaml
columns:
  - name: order_id
    tests:
      - unique
      - not_null
```

> [!example] Custom Data Tests
> Write SQL queries returning failing rows.

```sql
-- tests/valid_email.sql
select * from {{ ref('users') }}
where not email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'
```

Add in YAML:
```yaml
- name: email
  tests:
    - valid_email
```

> [!warning] Enforcement
> Integrate `dbt test` in CI/CD pipelines. Block merges if tests fail.

---

### Q12: What are seeds in dbt and how are they used?

> [!note] Answer
> **CSV files loaded into warehouse as reference/lookups** (via `dbt seed`). Used for mapping codes/types to business names or creating small, managed reference tables.

---

### Q13: What is source freshness and why does it matter?

> [!info] Answer
> **Source freshness tests verify if raw data sources are up-to-date.**

**Example:**
```yaml
sources:
  - name: raw
    freshness:
      warn_after: {count: 24, period: hour}
      error_after: {count: 48, period: hour}
```

> [!tip] Why It Matters
> Helps avoid reporting on outdated data due to pipeline lags/failures.

---

### Q14: Describe a process for automated CI/CD with dbt

> [!success] CI/CD Workflow
>
> 1. **Version control with Git**
> 2. **Pull requests trigger:**
>    - `dbt seed` (to load lookup tables)
>    - `dbt run` (to build models)
>    - `dbt test` (run and enforce tests)
> 3. **Fail build if tests fail**
> 4. **Post-merge**, deploy models to production warehouse
> 5. **Optionally**, notify team of failures (Slack/email)

**Sample Workflow Snippet:**
```yaml
jobs:
  dbt-build:
    steps:
    - run: dbt test
    - run: dbt run
```

---

## Governance, Documentation & Lineage

### Q15: How does dbt support documentation and data lineage?

> [!info] Documentation Features
>
> - **Docs blocks** (in model/product YAML) let you document models, fields
> - Generate docs with `dbt docs generate` — gives live web docs including lineage DAG
> - **Exposures** connect dbt data models to BI tools, tracking downstream consumers

---

### Q16: How do you use meta tags in dbt YAML?

> [!example] Meta Tags for Governance
> Meta tags help with ownership, sensitivity, or lifecycle.

```yaml
columns:
  - name: ssn
    meta:
      pii: true
      owner: dpo-team
```

---

## Advanced: Macros, Packages, and Extensibility

### Q17: What are macros in dbt? Give an example.

> [!tip] Answer
> **Macros are Jinja-powered re-usable SQL snippets** — like SQL "functions." Used to DRY up logic, implement standards, build test templates.

**Example:**
```sql
-- macros/percentile.sql
{% macro get_percentile(column, percentile) %}
percentile_cont({{ percentile }}) within group (order by {{ column }})
{% endmacro %}

-- Usage in model:
select
  {{ get_percentile('amount', 0.9) }} as p90_amount
from {{ ref('payments') }}
```

---

### Q18: How does dbt integrate with orchestration tools (Airflow, Dagster, Prefect)?

> [!info] Answer
> dbt runs are triggered as tasks/operators within DAGs in orchestration tools (e.g., Airflow's `BashOperator`/`CliOperator`). This allows full pipeline automation and integration with upstream/downstream ETL jobs.

---

### Q19: How can you handle deployment to multiple environments (dev/stage/prod)?

> [!tip] Multi-Environment Strategy
>
> - Use `profiles.yml` with different targets (dev, staging, prod)
> - Parameterize schema/database via Jinja or environment variables

**Example:**
```yaml
schema: "{{ target.schema }}"
```

Switch environment with `dbt run --target prod`

---

## Behavioral & Communication

### Q20: How to explain a model change to a non-technical stakeholder?

> [!quote] Sample Answer
> "I'm updating the customer model so that every customer's status reflects recent purchase activity. This helps marketing accurately target users who haven't connected in over 6 months, directly improving campaign results."

---

### Q21: How do you resolve a failing production test that you did not create?

> [!success] Sample Answer
>
> 1. Review test details in dbt docs and run logs
> 2. Audit recent code changes and source data loads
> 3. Use stored test failure rows to diagnose root cause
> 4. Engage relevant teammates (original test owner, data source team)
> 5. Patch, write regression test, and retroactively run CI

---

## Scenario-Based & Expert-Level

### Q22: You discover models with circular dependencies via `ref()`. How do you fix this?

> [!warning] Solution
>
> 1. Re-architect layers — move common logic upstream
> 2. Modularize transformations to remove inter-model loops
> 3. Add intermediate/bridge models

---

### Q23: Your model is running slow after a recent increase in data volume. What do you do?

> [!tip] Performance Troubleshooting
>
> 1. Profile SQL, identify expensive joins/scans
> 2. Use partitioning/clustering in warehouse
> 3. Materialize as table instead of view
> 4. Apply incremental model pattern
> 5. Refactor code to avoid re-processing unchanged data

---

### Q24: How do you safely add a new column to a production model that many downstream assets depend on?

> [!success] Safe Deployment Steps
>
> 1. Add the column in upstream model SQL
> 2. Update schema YAML with docs/tests on the new column
> 3. Notify stakeholders and update dependent models if needed
> 4. Run tests in dev/stage. If all pass, deploy to prod
> 5. Monitor downstream jobs for errors

---

### Q25: Explain how you would manage sensitive data (PII) in dbt.

> [!danger] PII Management
>
> - Use **meta tags** for PII fields in YAML
> - **Mask or hash** sensitive fields in models
> - **Limit field exposure** in downstream marts
> - Apply **column-level permissions/masking** in the warehouse

---

### Q26: How is dbt version-controlled and maintained by teams?

> [!note] Version Control Best Practices
>
> - All code under Git (feature branches, PRs, code reviews)
> - dbt Cloud or CI automates tests on PRs pre-merge
> - Use standards on naming, macros, and documentation
> - Automated builds and deploys on a merge to main/master branch

---

## Quick Reference: Most Frequently Asked Questions

> [!abstract] Key Questions to Know
>
> 1. What is dbt and what is its core role?
> 2. Difference between `ref()` and `source()`?
> 3. Explain dbt model materializations
> 4. How to run tests and what makes a good test?
> 5. How do you manage incremental data loads?
> 6. Show a macro and its benefit
> 7. How do you structure layered dbt projects?
> 8. How do you optimize for performance?
> 9. How does documentation work in dbt?
> 10. How do you automate "test before deploy" workflows?

---

## Related Notes

- [[SQL Interview Questions]]
- [[Database Fundamentals Interview Questions]]
- [[System Design Framework]]

---

> [!quote] Closing Thought
> dbt interview success relies not just on theoretical knowledge, but real-world judgement, strong documentation, and sound engineering tradeoffs.
