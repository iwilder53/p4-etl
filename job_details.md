````markdown
# AWS Glue Job: JFE Inventory Data ETL Process

## Overview

This AWS Glue job performs ETL (Extract, Transform, Load) on inventory data stored in CSV files on Amazon S3 and synchronizes it with DynamoDB tables. It reads schema definitions, applies data transformations, compares schema differences, and writes updated data back to DynamoDB and S3 (in Parquet format).

---

## Architecture Diagram

```plaintext
+---------------------+        +-----------------+       +----------------+
|                     |        |                 |       |                |
|    Amazon S3        +------->|   AWS Glue Job  +------>|  DynamoDB      |
| (CSV files & schemas)|        | (ETL & Transform)|      | (Target Tables)|
|                     |        |                 |       |                |
+---------------------+        +-----------------+       +----------------+
        |                            |
        |                            v
        |                   +-----------------+
        |                   | Amazon S3       |
        |                   | (Output Parquet)|
        |                   +-----------------+
        |
        v
  +-----------------+
  | S3 Error Logs   |
  +-----------------+
```
````

---

## Detailed Steps

### 1. Read Input Files

- Loads CSV schema definitions from S3.
- Loads inventory CSV data from S3.
- Detects and handles file encoding using `chardet` to ensure proper reading.

### 2. DynamoDB Table Handling

- Lists DynamoDB tables using `boto3`.
- Identifies the target table based on a configured prefix.

### 3. Data Transformation

- Converts AWS Glue DynamicFrames to Spark DataFrames.
- Applies schema transformations by casting columns to the types defined in the schema.
- Detects schema differences between existing data and DynamoDB data.
- Generates compound keys for unique identification if necessary.
- Manages timestamps: updates `updated_at` based on `created_at` or current timestamp.

### 4. Writing Output

- Writes updated records back to DynamoDB.
- Writes data as Parquet files to S3 with Snappy compression.
- Logs errors and exceptions to S3 for monitoring.

### 5. Job Commit

- Commits the Glue job to finalize the execution and resource cleanup.

---

## Key Functions

- **read_csv(name)**
  Reads CSV files from S3 with automatic encoding detection and returns a pandas DataFrame.

- **apply_schema(df2, schema_df)**
  Casts DataFrame columns to the data types defined in the schema DataFrame.

- **schema_diff(spark, df_1, df_2)**
  Compares two DataFrames to detect schema differences and creates compound keys for new or modified records.

- **getTableName(prefix)**
  Finds and returns the DynamoDB table name that matches the given prefix.

---

## Notes

- Utilizes pandas and PySpark for flexible and efficient data processing.
- Implements error handling with try-except blocks and logs exceptions to S3.
- Uses `boto3` for interacting with AWS services such as S3 and DynamoDB.
- Supports batch processing for multiple tables (loop implementation is present but commented out).
- Designed for scalable and maintainable ETL pipelines within AWS Glue.

---

## How to Run

1. Ensure IAM roles assigned to the Glue job have proper permissions for S3 and DynamoDB access.
2. Upload CSV data files and schema definitions to designated S3 buckets.
3. Configure job parameters such as prefixes, S3 paths, and DynamoDB table names as required.
4. Schedule or manually trigger the AWS Glue job.
5. Monitor logs and output in S3.

---

## Contact

For further assistance or questions, please contact Mr Takeuchi or Yash.
