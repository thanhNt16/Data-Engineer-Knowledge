# Unit Testing Interview Questions

> Essential testing concepts and practices for data engineers

## Fundamental Questions

### Q1: Explain the difference between unit testing and integration testing.

**Answer**:

- **Unit testing**: Testing individual components or functions in isolation. Tests for proper functions for an individual component.
- **Integration testing**: Focuses on testing how a section of code interacts within the whole system. Validates how well different components perform together.

---

### Q2: How are assertions used in unit testing?

**Answer**: Assertions are used to validate the expected behavior of a function. They compare the expected output and the actual output to evaluate for accuracy.

---

### Q3: What are ways you can improve the performance of your unit tests?

**Answer**:

- **Minimize dependencies**: Reduce external dependencies in tests
- **Use setUp() and tearDown() methods**: Efficiently manage test fixtures
- **Optimize your code**: Write efficient test code
- **Run tests in parallel**: Execute tests concurrently to save time

---

### Q4: What is continuous testing and why is it important in unit testing?

**Answer**: Continuous testing runs your unit tests automatically as part of the CI/CD pipeline. This helps:

- Reduce regression
- Identify issues early
- Ensure code quality throughout development
- Provide fast feedback to developers

---

### Q5: What are examples of external dependencies, and what are strategies to handle them in unit testing?

**Answer**: Examples of external dependencies include:

- **APIs**: External service endpoints
- **Databases**: Data storage systems
- **File systems**: External file resources

**Strategies to minimize dependencies**:
- **Mocking**: Use simulated and predefined versions of the data for unit tests
- **Dependency injection**: Pass dependencies as parameters
- **Interface abstraction**: Abstract external dependencies behind interfaces

---

### Q6: What is regression testing?

**Answer**: Regression testing is the process of testing your code to ensure it still works as expected over time as:

- Code changes occur
- Bug fixes are implemented
- New features are added

It ensures that recent changes haven't broken existing functionality.

---

### Q7: What is the purpose of test fixtures?

**Answer**: Test fixtures ensure your unit tests have:

- **Consistent environments**: Standard setup for each test
- **Stable operations**: Predictable starting conditions
- **Consistent data**: Known data states for testing
- **Reliable results**: Repeatable test outcomes

Fixtures can include:
- Test data setup
- Database connections
- Variable initialization
- Resource allocation

---

## Testing Frameworks in Python

### unittest

Python's built-in testing framework that provides:

- **Test cases**: Base class for creating tests
- **Test suites**: Grouping multiple tests
- **Test runner**: Executing tests
- **Assertions**: Methods for validation

```python
import unittest

class TestMathOperations(unittest.TestCase):
    def test_add(self):
        self.assertEqual(add(2, 3), 5)

if __name__ == '__main__':
    unittest.main()
```

### pytest

A more advanced testing framework with:

- **Simpler syntax**: No need for test classes
- **Fixtures**: Reusable test components
- **Parameterized tests**: Run tests with multiple inputs
- **Better error messages**: Detailed failure information

```python
def test_add():
    assert add(2, 3) == 5
```

---

## Advanced Testing Concepts

### Parameterized Testing

Running the same test with multiple inputs:

```python
@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```

### Performance and Stress Testing

Testing how code handles:

- **Large datasets**: Testing with high data volumes
- **Concurrent operations**: Multiple simultaneous operations
- **Resource limits**: Memory and CPU constraints

### Scenario Testing

Testing specific use cases and edge cases:

- **Happy path**: Normal operation scenarios
- **Error cases**: Invalid input handling
- **Boundary conditions**: Edge values and limits

---

## Testing Best Practices

### 1. Arrange-Act-Assert (AAA) Pattern

```python
def test_calculate_discount():
    # Arrange: Set up test data
    customer = Customer(type="premium")
    amount = 100

    # Act: Execute the function
    result = calculate_discount(customer, amount)

    # Assert: Verify the result
    assert result == 85  # 15% discount
```

### 2. Test Isolation

Each test should be:
- Independent of other tests
- Able to run in any order
- Not dependent on shared state

### 3. Descriptive Test Names

```python
# Bad
def test_1():
    pass

# Good
def test_calculate_discount_for_premium_customer_returns_15_percent():
    pass
```

### 4. Mock External Dependencies

```python
from unittest.mock import patch

@patch('module.api_call')
def test_process_data(mock_api):
    mock_api.return_value = {'status': 'success'}
    result = process_data()
    assert result == 'processed'
```

---

## Data Engineering Testing Scenarios

### Testing Data Pipelines

```python
def test_etl_pipeline():
    # Test data extraction
    raw_data = extract_data('source.csv')
    assert len(raw_data) > 0

    # Test data transformation
    cleaned_data = transform_data(raw_data)
    assert cleaned_data.isnull().sum().sum() == 0

    # Test data loading
    load_data(cleaned_data, 'target.db')
    assert verify_data_in_database('target.db')
```

### Testing Data Quality

```python
def test_data_quality_checks():
    data = load_data()

    # Check for nulls
    assert data['id'].notnull().all()

    # Check data types
    assert data['amount'].dtype == 'float64'

    # Check value ranges
    assert (data['age'] >= 0).all() & (data['age'] <= 120).all()
```

### Testing Schema Validation

```python
def test_schema_validation():
    data = load_data()

    expected_schema = {
        'id': 'int64',
        'name': 'object',
        'amount': 'float64',
        'date': 'datetime64[ns]'
    }

    for column, dtype in expected_schema.items():
        assert data[column].dtype == dtype
```

---

## Related Topics

- [[02-Areas/Python/Testing]]
- [[05-Interview Prep/Python Interview Questions]]
- [[05-Interview Prep/SQL Interview Questions]]
