# Adventure_works
Medallion (raw -> bronze -> silver -> gold) pipeline for the Adventure Works sales data on Azure Databricks with Unity Catalog.

## Project structure

```
config/
  pipeline_config              widgets (catalog, storage_account, datasets), ADLS paths, schemas, dataset registry
utils/
  common_functions             parsing helpers, ingestion metadata, dedup, Delta writer, DataQualityChecker
notebooks/
  00_setup/
    01_setup_storage_and_schemas   external locations + bronze/silver/gold schemas (idempotent)
  01_bronze/
    01_raw_to_bronze               ADLS raw files -> bronze Delta tables (values kept as raw strings)
  02_silver/
    01_bronze_to_silver            bronze tables -> typed, cleaned, deduplicated silver tables
    02_silver_data_quality_checks  quality gate; results appended to silver.dq_check_results
  03_gold/
    01_build_dimensions            dim_product, dim_reseller, dim_sales_territory, dim_salesperson, dim_date, bridge_salesperson_territory
    02_build_facts                 fact_sales, fact_sales_target (validated against dimensions)
    03_business_aggregations       agg_* reporting tables (revenue reconciled to fact_sales)
00-sample_notebook             exploration scratch notebook (not part of the pipeline)
```

## Run order
`00_setup` (once) -> `01_raw_to_bronze` -> `01_bronze_to_silver` -> `02_silver_data_quality_checks` -> `01_build_dimensions` -> `02_build_facts` -> `03_business_aggregations`

Each notebook loads `config/pipeline_config` and `utils/common_functions` with `%run`. Override the widgets with job parameters of the same name, e.g. `catalog=adventure_works_catalog`, `storage_account=adventurework`, `datasets=sales,product`.

## Storage
* ADLS account `adventurework`, containers `raw`, `bronze`, `silver`, `gold`
* Tables are external Delta tables in `adventure_works_catalog.<layer>`, stored at `abfss://<layer>@adventurework.dfs.core.windows.net/<folder>`
* Raw files are tab-separated, with currency like `$2,024.99` and dates like `Friday, August 25, 2017`
