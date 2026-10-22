# dbt Ecommerce Project

A small dbt project that turns raw ecommerce data (customers, orders, products, payments) into clean, analytics-ready tables in Snowflake. Built as a hands-on learning project.

## What this project does

- Loads raw CSV data into Snowflake as **seeds**
- Cleans and standardizes it in **staging** views
- Builds business-ready **marts** tables on top of the staging layer

## Tech stack

- [dbt](https://www.getdbt.com/) (built in dbt Studio)
- Snowflake
- GitHub

## Data flow

```
seeds (CSV)  ->  staging (views)  ->  marts (tables)
```

| Layer | Snowflake schema | Materialization | Reads from |
|---|---|---|---|
| Raw | `DBT_LEARN_SAMPLE_RAW` | table (seed) | CSV files in `seeds/` |
| Staging | `DBT_LEARN_SAMPLE_STAGING` | view | raw tables, via `source()` |
| Marts | `DBT_LEARN_SAMPLE_MARTS` | table | staging models, via `ref()` |

All schemas live in the `DBT_LEARN` database. dbt builds each schema name as `<target schema>_<custom schema>`, where the target schema is `dbt_learn_sample` and the custom schemas (`raw`, `staging`, `marts`) are set in `dbt_project.yml`.

## Project structure

```
.
├── dbt_project.yml        # project config, schema and materialization settings
├── seeds/                 # raw CSV data
├── models/
│   ├── staging/
│   │   ├── _sources.yml   # declares the raw tables
│   │   ├── _models.yml    # column tests for staging models
│   │   ├── stg_customers.sql
│   │   ├── stg_orders.sql
│   │   ├── stg_payments.sql
│   │   └── stg_products.sql
│   └── marts/             # business-level models
├── macros/
│   └── cents_to_dollars.sql
├── analyses/
├── snapshots/
└── tests/                 # singular data tests (SQL)
```

## Models

**Seeds:** `raw_customers`, `raw_orders`, `raw_products`, `raw_payments`, `order_status_mapping`

**Staging:** `stg_customers`, `stg_orders`, `stg_payments`, `stg_products`. One model per raw table, with renamed columns and light cleaning.

**Marts:** built from the staging models using `ref()`.

**Macro:** `cents_to_dollars` converts amounts stored in cents to dollars.

## Tests

Data quality is checked with `dbt test` (or as part of `dbt build`). The project uses two kinds of tests.

**Column tests on the staging models** (defined in the YAML file in `models/staging/`):

- **Customers:** every `customer_id` must be unique and not null, and every `email` must be unique, so no customer appears twice.
- **Orders:** every `order_id` must be unique and not null, and `order_status` must be one of completed, returned, shipped, processing or cancelled.
- **Products:** every `product_id` must be unique and not null.
- **Payments:** every `payment_id` must be unique and not null, and `payment_method` must be credit card, debit card or UPI.

**Custom data test** (in the `tests/` folder): checks the payments data for bad values by looking for successful payments with an amount of zero or less. The test passes only if no such payments exist.

## Getting started

### Prerequisites

- A Snowflake account with a database named `DBT_LEARN`
- A dbt project connected to Snowflake (dbt Studio or dbt Core)
- A development target schema of `dbt_learn_sample`

### Run the project

```bash
dbt seed      # load raw CSVs into DBT_LEARN_SAMPLE_RAW
dbt run       # build staging views and marts tables
dbt test      # run data quality tests
```

Or run everything in dependency order with:

```bash
dbt build
```

### Useful commands

```bash
dbt run --select staging       # build only the staging layer
dbt run --select marts         # build only the marts layer
dbt run --select +marts        # build marts and everything upstream
dbt run --select stg_orders+   # build a model and everything downstream
```

## Key concepts practiced

- Layered modeling: raw, staging, marts
- `source()` to read existing raw tables and `ref()` to build model dependencies
- Custom schemas and how dbt combines them with the target schema
- Folder-level configs for materialization in `dbt_project.yml`
- Reusable logic with macros
- Data quality checks with generic and singular tests
- Git workflow: feature branch, pull request, merge to `main`
