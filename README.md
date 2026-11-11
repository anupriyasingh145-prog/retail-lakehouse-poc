# Retail Sales Lakehouse POC (Databricks, medallion architecture)

A small end-to-end proof of concept built to learn the Azure data engineering stack.

**Built and run:** Databricks Free Edition (serverless) - bronze/silver/gold, Delta Lake, Databricks Job.
**Designed only (not deployed):** Azure Data Factory orchestration and Logic App alerting - see
[docs/azure_integration_design.md](docs/azure_integration_design.md).

## Architecture

```
Synthetic messy sales data
   -> bronze_sales            (raw, all strings, includes duplicates and bad rows)
   -> silver_sales            (deduped, typed, cleaned)  + silver_sales_rejected (quarantine)
   -> gold_sales_by_region / _by_product / _by_month   (reporting aggregates)
Databricks Job chains the notebooks: 00 -> 01 -> 02
```

## Data quality handled in silver

- 50 duplicate orders removed (keep one row per `order_id`)
- Mixed date formats (`yyyy-MM-dd` and `dd/MM/yyyy`) normalised
- Text trimmed and standardised (region)
- Invalid rows (missing customer, missing/negative quantity) sent to `silver_sales_rejected`
  instead of being dropped silently

Result: 2,050 bronze rows -> 1,859 silver + 141 rejected (2,000 unique orders).

## Delta Lake features demonstrated

`DESCRIBE HISTORY`, time travel (`VERSION AS OF`), `DELETE`, `RESTORE` - see `notebooks/03_delta_demo.py`.

## Repo layout

| Path | Purpose |
|---|---|
| notebooks/00_setup_bronze.py | Schema + sample data -> bronze |
| notebooks/01_silver.py | Cleaning, typing, quarantine -> silver |
| notebooks/02_gold.py | SQL aggregates -> gold |
| notebooks/03_delta_demo.py | Delta history, time travel, restore (not in job) |
| sql/practice_queries.sql | SQL practice on the tables |
| docs/ | Design notes and screenshots |

## How to run

1. Sign up for Databricks Free Edition.
2. Import the notebooks from `notebooks/` (Workspace -> Import).
3. Run 00, 01, 02 in order, or create a Job with three dependent notebook tasks.

## Limitations / honest notes

- Free Edition is serverless only, so cluster sizing and tuning were not exercised.
- Sample data is synthetic. No real or commercial data is used.
- Azure Data Factory and Logic Apps were not built because no Azure subscription was available;
  the design notes describe the intended approach.

## Next steps

- Incremental load with `MERGE INTO` instead of overwrite
- Auto Loader for file ingestion
- Build the ADF + Logic App layer when an Azure subscription is available
