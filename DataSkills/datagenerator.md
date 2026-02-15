---
name: datagenerator
description: Generate synthetic datasets from a natural language prompt. Use when asked to generate data, create sample data, make test data, build a dataset, produce fake data, generate CSV/JSON/Parquet data, or create mock data for testing, demos, or development. Triggers on phrases like "generate data", "create dataset", "sample data", "fake data", "mock data", "synthetic data", or "test data".
---

# Data Generator

Generate synthetic datasets from a natural language description. The user provides a prompt describing the data they need, and this skill produces realistic, well-structured output in the requested format.

## Workflow

1. **Parse the prompt** — Extract from the user's description:
   - Column/field names and their data types
   - Number of rows (default: 100 if not specified)
   - Output format (default: CSV if not specified)
   - Any constraints, distributions, or relationships between fields
   - Domain context (e.g. e-commerce, healthcare, IoT)

2. **Check dependencies** — Ensure the Python environment has the required packages:

```bash
pip install faker pandas pyarrow --quiet
```

3. **Generate a Python script** — Write a self-contained Python script named `generate_data.py` in the current directory that:
   - Uses `Faker` for realistic names, addresses, emails, dates, etc.
   - Uses `random` / `numpy` for numeric distributions
   - Uses `pandas` for structuring and exporting the data
   - Respects any constraints the user specified (ranges, categories, nullability, uniqueness)
   - Seeds the random generator (`seed=42`) so results are reproducible

4. **Run the script** — Execute and verify the output:

```bash
python generate_data.py
```

5. **Show a preview** — Display the first 10 rows and the shape of the generated dataset so the user can confirm it looks correct.

6. **Iterate if needed** — Ask the user if adjustments are required (more rows, different distributions, additional columns, format change).

## Supported Output Formats

| Format  | Extension | Notes |
|---------|-----------|-------|
| CSV     | `.csv`    | Default. Comma-separated, UTF-8 encoded. |
| JSON    | `.json`   | Array of objects (`records` orientation). |
| Parquet | `.parquet` | Requires `pyarrow`. Efficient for large datasets. |
| Excel   | `.xlsx`   | Requires `openpyxl`. |
| SQL INSERT | `.sql` | Generates INSERT statements for a given table name. |

## Supported Field Types

- **String**: names, emails, addresses, company names, free text, UUIDs, phone numbers
- **Numeric**: integers (with min/max), floats (with precision), normal/uniform distributions
- **Temporal**: dates (with range), timestamps, time deltas
- **Categorical**: pick from a fixed set of values with optional weights
- **Boolean**: true/false with configurable probability
- **Relational**: foreign-key references to other generated tables

## Arguments

- `prompt` (required) — Natural language description of the desired dataset.
- `rows` (optional) — Number of rows to generate. Default: `100`.
- `format` (optional) — Output format: `csv`, `json`, `parquet`, `xlsx`, `sql`. Default: `csv`.
- `output` (optional) — Output filename. Default: `generated_data.<format>`.
- `seed` (optional) — Random seed for reproducibility. Default: `42`.

## Example Prompts

**Simple:**
> Generate 500 rows of customer data with name, email, age (18-65), and signup date in the last year.

**With relationships:**
> Create two related tables: `orders` (1000 rows) with order_id, customer_id, product, quantity (1-10), and price (5.00-500.00); and `customers` (200 rows) with customer_id, name, email, and country. Each order should reference a valid customer_id.

**With distributions:**
> Generate 10000 rows of sensor data: timestamp (every 5 seconds starting 2025-01-01), temperature (normal distribution mean=22, std=3), humidity (uniform 30-80), and status (90% "ok", 8% "warning", 2% "error").

**Domain-specific:**
> Create a healthcare dataset with 300 patient records: patient_id, age (0-100), gender, blood_type (realistic distribution), diagnosis (from: diabetes, hypertension, asthma, none), admission_date (2024), and length_of_stay (1-30 days, right-skewed).

## Script Template

```python
import pandas as pd
from faker import Faker
import random
import numpy as np
from datetime import datetime, timedelta

fake = Faker()
random.seed(42)
np.random.seed(42)
Faker.seed(42)

NUM_ROWS = 100  # adjust per user request

def generate_data(n=NUM_ROWS):
    data = {
        # Add columns here based on the user prompt
        # Example:
        # "name": [fake.name() for _ in range(n)],
        # "email": [fake.email() for _ in range(n)],
        # "age": [random.randint(18, 65) for _ in range(n)],
        # "signup_date": [fake.date_between(start_date="-1y", end_date="today") for _ in range(n)],
    }
    return pd.DataFrame(data)

if __name__ == "__main__":
    df = generate_data()
    df.to_csv("generated_data.csv", index=False)  # change format as needed
    print(f"Generated {len(df)} rows x {len(df.columns)} columns")
    print(df.head(10).to_string())
```

## Dev Dependencies

```
faker
pandas
numpy
pyarrow
openpyxl
```
