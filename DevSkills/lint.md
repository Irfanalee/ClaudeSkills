# Lint Python Project

Run linting and formatting checks on the Python project using Ruff.

## Steps

1. First, check if `pyproject.toml` exists with Ruff configuration
2. If not, offer to create a standard Ruff configuration
3. Check if `ruff` is installed in the environment
4. Run `ruff check .` to find linting issues
5. If there are auto-fixable issues, ask if user wants to apply fixes with `ruff check --fix .`
6. Run `ruff format --check .` to check formatting
7. If formatting issues exist, ask if user wants to auto-format with `ruff format .`
8. Report summary of results

## Ruff Configuration Template

If no pyproject.toml exists, create one with:

```toml
[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "W", "I", "UP"]
ignore = ["E501"]

[tool.ruff.format]
quote-style = "double"
```

## Commands to Run

```bash
# Check for issues
ruff check .

# Auto-fix issues
ruff check --fix .

# Check formatting
ruff format --check .

# Apply formatting
ruff format .
```

## Dev Dependencies

If ruff is not installed, add to requirements.txt or pyproject.toml:
```
ruff==0.9.2
```
