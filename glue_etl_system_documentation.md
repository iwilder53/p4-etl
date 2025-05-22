
# 📄 System Documentation: AWS Glue ETL Jobs for `migration-glue-etl-yash`

## 📁 S3 Structure

Your system is centered around the S3 bucket:  
`s3://migration-glue-etl-yash/`

### Key Directories and Files:

| Path | Description |
|------|-------------|
| `glue-master/` | Primary working directory for source CSVs and intermediate data |
| `glue-master/JFE_Map.csv` | Source data for JFE ETL process |
| `glue-master/cumulative_map.csv` | Aggregated source map for journal ETL |
| `glue-master/journal.csv` | Source map for journal processing |
| `glue-master/mapping_data.csv` | Source for master data mapping |
| `glue-master/csv_schema/` | stores schema files in CSV  |
| `glue-master/newDBs/` |  output or staging area for new transformed data |
| `glue-master/oldDBs/` |  contains legacy source data or backups |
| `glue-master/table_defs/` |  stores Excel table definitions & metadata for individual tables |

## 🧩 Glue Job Breakdown

### Jobs Involved:

We are currently managing **six AWS Glue jobs**, categorized into **three main jobs** and **three staging jobs** :

| Job Name | Purpose | S3 Input Path | Manual Intervention |
|----------|---------|---------------|---------------------|
| `etl-master` | Handles master mapping data | `glue-master/mapping_data.csv` | Yes (some files) |
| `etl-journal` | Handles journal data | `glue-master/journal.csv` | Yes (some files) |
| `etl-JFE` | Handles JFE-specific data | `glue-master/JFE_Map.csv` | Yes (some files) |
| `etl-master-staging` | Staging version of master ETL | Likely same source or a variant | No |
| `etl-journal-staging` | Staging version of journal ETL | Likely same source or a variant | No |
| `etl-JFE-staging` | Staging version of JFE ETL | Likely same source or a variant | No |

### Common Behavior:

- **Shared Codebase**: All jobs use a common set of ETL functions (probably stored in a shared script like `etl_common.py` in future, atm they are hardcoded).
- **Manual File Exclusions**: Some files require manual review or edits before they are included in the ETL process.
- **Staging Jobs**: Run on automation pipelines, no manual file exclusions, used for pre-prod or test transformations.

## 🔄 Workflow Summary

### EXpectations For Production Jobs:

1. Data is dropped into `glue-master/` (e.g., `mapping_data.csv`, `journal.csv`, etc.).
2. Some files are manually excluded or preprocessed before running.
3. Glue ETL job (e.g., `etl-master`) reads the relevant file.
4. Job uses shared logic (e.g., CSV parsing, schema application, transformation).
5. Transformed data is written back to `glue-master/newDBs/` & another DDB target table.

### For Staging Jobs:

- Same flow but without manual exclusions.

## ⚙️ Technical Assumptions

- All Jobs are written in **Python (PySpark)**.
- Common transformations (e.g., schema enforcement, column renaming) are implemented in a **shared script** (like `etl_common.py`).
- Outputs may be stored as **Parquet** or CSV depending on destination(parquet are preffered for bigger tables).
- AWS Glue Crawlers or Athena may be used to query output data.

## 🛠 Manual Operations

- **Manual Review**: Certain input files are manually reviewed before ETL jobs run (e.g., edited locally or validated).
- **Exclusion List**: Files requiring manual intervention are not excluded from automation but the errors are logged.

## ✅ Opportunities for Improvement

- Create a **manifest file** to track manually processed files.
- Add metadata tagging (e.g., `processed_by`, `timestamp`) to output files.
- Introduce **unit test jobs** for staging runs.
- Add version control to ETL logic stored in `glue-scripts/`.
