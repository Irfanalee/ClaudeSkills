# Run Python Tests

Run pytest tests with optional coverage reporting.

## Arguments

- `$ARGUMENTS` - Optional: specific test file, test function, or pytest flags (e.g., `-v`, `-x`, `--cov`)

## Steps

1. Check if `pytest` is installed in the environment
2. Check if `tests/` directory exists
3. If no tests directory, offer to create test structure
4. Run pytest with provided arguments or defaults
5. If `--cov` flag provided or no args, run with coverage
6. Report test results summary

## Commands

```bash
# Run all tests (verbose)
pytest -v

# Run specific test file
pytest tests/test_example.py -v

# Run with coverage
pytest --cov=. --cov-report=term-missing

# Run and stop on first failure
pytest -x

# Run tests matching pattern
pytest -k "test_function_name"
```

## Test Structure Template

If no tests exist, create:

```
tests/
├── __init__.py
├── conftest.py      # Shared fixtures
└── test_example.py  # Example test file
```

### conftest.py template:
```python
"""Shared test fixtures"""
import pytest

@pytest.fixture
def sample_data():
    """Example fixture"""
    return {"key": "value"}
```

### test_example.py template:
```python
"""Example tests"""

def test_example(sample_data):
    """Example test using fixture"""
    assert sample_data["key"] == "value"
```

## Dev Dependencies

If pytest not installed, add:
```
pytest==8.3.4
pytest-mock==3.14.0
pytest-cov==6.0.0
```

## Pytest Configuration

Add to pyproject.toml:
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = "test_*.py"
addopts = "-v"

[tool.coverage.run]
source = ["."]
omit = ["tests/*", "venv/*"]
```
