# Snowflake Real-Time Data Warehouse Project

This project demonstrates how to build a scalable, cloud-native data warehouse using **Snowflake**, integrated with **AWS S3** for data storage, **Snowpipe** for real-time data ingestion, and **AWS QuickSight** for visualization. The project processes **Tesla stock market data** (`TSLA.csv`) and **customer data** (`customer_detail.csv`) to create a robust data pipeline, showcasing Snowflake's architecture, data loading methods, and advanced features like Time Travel.
![ProjectArchitectureDiagram](https://github.com/user-attachments/assets/1c1df414-dbe1-4545-aaff-3d0b8acbd25e)

## Table of Contents
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Step-by-Step Execution](#step-by-step-execution)
  - [Step 1: Set Up AWS S3 Bucket](#step-1-set-up-aws-s3-bucket)
  - [Step 2: Set Up Snowflake Account](#step-2-set-up-snowflake-account)
  - [Step 3: Install and Configure SnowSQL](#step-3-install-and-configure-snowsql)
  - [Step 4: Create Database and Tables](#step-4-create-database-and-tables)
  - [Step 5: Load Data Using SnowSQL (Local Files)](#step-5-load-data-using-snowsql-local-files)
  - [Step 6: Load Data Using AWS S3](#step-6-load-data-using-aws-s3)
    - [Approach A: Direct Credentials](#approach-a-direct-credentials)
    - [Approach B: Storage Integration (IAM Role)](#approach-b-storage-integration-iam-role)
  - [Step 7: Set Up Snowpipe for Real-Time Data Loading](#step-7-set-up-snowpipe-for-real-time-data-loading)
  - [Step 8: Visualize Data with AWS QuickSight](#step-8-visualize-data-with-aws-quicksight)
  - [Step 9: Explore Time Travel](#step-9-explore-time-travel)
  - [Step 10: Optimize and Monitor](#step-10-optimize-and-monitor)
- [Snowflake Architecture](#snowflake-architecture)
- [Security Features](#security-features)
- [Cost Management](#cost-management)
- [Potential Challenges and Solutions](#potential-challenges-and-solutions)
- [Sample Visualization](#sample-visualization)


## Project Overview
This project builds a real-time data warehouse to process and analyze Tesla stock data and customer data. Key objectives include:
- Understanding Snowflake’s cloud-native architecture (separation of compute and storage).
- Loading data using multiple methods: SnowSQL (local files), AWS S3 (cloud storage), and Snowpipe (real-time ingestion).
- Visualizing stock price trends using AWS QuickSight.
- Leveraging Snowflake features like Time Travel for data recovery and auditing.
- Optimizing performance and managing costs.

The pipeline ingests data from an S3 bucket, loads it into Snowflake tables, automates ingestion with Snowpipe, and creates a dashboard for business insights.

## Prerequisites
Before starting, ensure you have:
- **AWS Account**: For S3 and QuickSight (sign up at `https://aws.amazon.com`).
- **Snowflake Account**: Standard Edition recommended (sign up at `https://snowflakecomputing.com`).
- **SnowSQL**: Command-line client for Snowflake (download from Snowflake’s website).
- **Data Files**: `TSLA.csv` (Tesla stock data) and `customer_detail.csv` (customer data), available in the project’s `data/` folder.
- **Naming Convention**: Use consistent names (e.g., `TEST_DB`, `TESLA_DATA`) for resources.
- **Cost Awareness**: Enable auto-suspend and delete unused resources to minimize charges.

## Tech Stack
- **Languages**: SQL
- **Services**: Snowflake, SnowSQL, AWS S3, Snowpipe, AWS QuickSight
- **Cloud Provider**: AWS

## Project Structure
```
snowflake-data-warehouse/
├── data/
│   ├── TSLA.csv
│   ├── customer_detail.csv
├── scripts/
│   ├── load_from_s3.sql
│   ├── snowpipe.sql
│   ├── snowsql_load.sql
├── README.md
```

- `data/`: Contains `TSLA.csv` (Tesla stock data) and `customer_detail.csv` (customer data).
- `scripts/`: SQL scripts for data loading and Snowpipe setup.
- `README.md`: This file.

## Step-by-Step Execution

### Step 1: Set Up AWS S3 Bucket
1. **Log in to AWS Console**:
   - Access `https://console.aws.amazon.com` with IAM or root credentials.
   - Ensure all services are in the same region (e.g., US East - N. Virginia) to avoid latency/costs.

2. **Create S3 Bucket**:
   - Navigate to S3 > “Create Bucket.”
   - Name the bucket (e.g., `snowflakecomputingpro`).
   - Enable “Block Public Access” for security.

3. **Create Input Folder**:
   - Open the bucket and click “Create Folder.”
   - Name it `Input`.

4. **Upload Data**:
   - Upload `TSLA.csv` to the `Input` folder.
   - Verify the file path: `s3://snowflakecomputingpro/Input/TSLA.csv`.

### Step 2: Set Up Snowflake Account
1. **Sign Up**:
   - Go to `https://snowflakecomputing.com` and create a trial account.
   - Select **Standard Edition** for cost efficiency.
   - Choose **AWS** as the cloud provider.
   - Activate the account via the confirmation email.

2. **Access UI**:
   - Log in at `https://<account_identifier>.snowflakecomputing.com`.
   - Explore sections: **Data** (databases/schemas), **Worksheets** (SQL queries), **Monitoring** (query history), **Admin** (users/roles).

3. **Configure Warehouse**:
   - Set auto-suspend to 60 seconds for the default warehouse (`compute_wh`):
     ```sql
     ALTER WAREHOUSE compute_wh SET AUTO_SUSPEND = 60;
     ```

### Step 3: Install and Configure SnowSQL
1. **Download SnowSQL**:
   - Visit Snowflake’s Downloads page and select the installer for your OS (Windows: `.msi`, macOS: `.pkg`/`.dmg`, Linux: `.tar.gz`).
   - Install following standard procedures.

2. **Configure SnowSQL**:
   - Locate the config file:
     - Windows: `C:\Users\<username>\.snowsql\config`
     - macOS/Linux: `~/.snowsql/config`
   - Add credentials:
     ```ini
     [connections]
     accountname = <account_identifier>
     username = <your_username>
     password = <your_password>
     ```
   - Replace placeholders with your Snowflake account details.

3. **Test Connection**:
   - Run in terminal:
     ```bash
     snowsql -a <account_identifier> -u <your_username>
     ```
   - Enter password to start an interactive session.

### Step 4: Create Database and Tables
1. **Create Database**:
   - In SnowSQL or Snowflake worksheet, run:
     ```sql
     CREATE DATABASE TEST_DB;
     USE DATABASE TEST_DB;
     ```

2. **Create Tesla Data Table**:
   - Create a table for Tesla stock data:
     ```sql
     CREATE TABLE TESLA_DATA (
         Date DATE,
         Open_value DOUBLE,
         High_value DOUBLE,
         Low_value DOUBLE,
         Close_value DOUBLE,
         Adj_Close DOUBLE,
         Volume BIGINT
     );
     ```

3. **Create Customer Detail Table**:
   - Create a table for customer data:
     ```sql
     CREATE TABLE customer_detail (
         first_name VARCHAR,
         last_name VARCHAR,
         address VARCHAR,
         city VARCHAR,
         state VARCHAR
     );
     ```

4. **Verify Tables**:
   - Check table creation:
     ```sql
     SELECT * FROM TESLA_DATA;
     SELECT * FROM customer_detail;
     ```

### Step 5: Load Data Using SnowSQL (Local Files)
1. **Create File Format**:
   - Define a pipe-delimited file format:
     ```sql
     CREATE OR REPLACE FILE FORMAT PIPE_FORMAT_CLI
         TYPE = 'CSV'
         FIELD_DELIMITER = '|'
         SKIP_HEADER = 1;
     ```

2. **Create Stage**:
   - Create an internal stage:
     ```sql
     CREATE OR REPLACE STAGE PIPE_CLI_STAGE
         FILE_FORMAT = PIPE_FORMAT_CLI;
     ```

3. **Upload File**:
   - From terminal, upload `customer_detail.csv`:
     ```bash
     snowsql -a <account_identifier> -u <your_username> -q "PUT file://<path_to_file>/customer_detail.csv @PIPE_CLI_STAGE AUTO_COMPRESS=TRUE;"
     ```
   - Replace `<path_to_file>` with the local file path.

4. **List Stage Contents**:
   - Verify upload:
     ```sql
     LIST @PIPE_CLI_STAGE;
     ```

5. **Resume Warehouse** (if auto-resume is disabled):
   - Run:
     ```sql
     ALTER WAREHOUSE compute_wh RESUME;
     ```

6. **Load Data**:
   - Load into `customer_detail`:
     ```sql
     COPY INTO customer_detail
         FROM @PIPE_CLI_STAGE
         FILE_FORMAT = (FORMAT_NAME = PIPE_FORMAT_CLI)
         ON_ERROR = 'SKIP_FILE';
     ```

7. **Verify Data**:
   - Query:
     ```sql
     SELECT * FROM customer_detail;
     ```

### Step 6: Load Data Using AWS S3
#### Approach A: Direct Credentials
1. **Create File Format**:
   - Define a comma-delimited file format:
     ```sql
     CREATE OR REPLACE FILE FORMAT CSV_FORMAT
         TYPE = 'CSV'
         FIELD_DELIMITER = ','
         SKIP_HEADER = 1
         NULL_IF = ('NULL', '')
         EMPTY_FIELD_AS_NULL = TRUE
         TRIM_SPACE = TRUE;
     ```

2. **Create Stage**:
   - Create an external stage with AWS credentials:
     ```sql
     CREATE OR REPLACE STAGE BULK_COPY_TESLA_STAGE
         URL = 's3://snowflakecomputingpro/TSLA.csv'
         CREDENTIALS = (AWS_KEY_ID = '<your_aws_key_id>' AWS_SECRET_KEY = '<your_aws_secret_key>')
         FILE_FORMAT = CSV_FORMAT;
     ```
   - Obtain credentials from AWS IAM > Users > Create Access Key.

3. **List Stage Contents**:
   - Verify:
     ```sql
     LIST @BULK_COPY_TESLA_STAGE;
     ```

4. **Load Data**:
   - Load into `TESLA_DATA`:
     ```sql
     COPY INTO TESLA_DATA
         FROM @BULK_COPY_TESLA_STAGE
         FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = ',' SKIP_HEADER = 1);
     ```

5. **Verify Data**:
   - Query:
     ```sql
     SELECT * FROM TESLA_DATA;
     ```

#### Approach B: Storage Integration (IAM Role)
1. **Create IAM Policy**:
   - In AWS IAM, create a policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": [
                     "s3:PutObject",
                     "s3:GetObject",
                     "s3:GetObjectVersion",
                     "s3:DeleteObject",
                     "s3:DeleteObjectVersion"
                 ],
                 "Resource": "arn:aws:s3:::snowflakecomputingpro/Input/*"
             },
             {
                 "Effect": "Allow",
                 "Action": "s3:ListBucket",
                 "Resource": "arn:aws:s3:::snowflakecomputingpro",
                 "Condition": {
                     "StringLike": {
                         "s3:prefix": ["Input/*"]
                     }
                 }
             }
         ]
     }
     ```
   - Create an IAM role (`Testsnowflakerole`) and attach the policy.
   - Copy the role’s ARN (e.g., `arn:aws:iam::191015975005:role/Testsnowflakerole`).

2. **Grant Integration Permission**:
   - Run:
     ```sql
     GRANT CREATE INTEGRATION ON ACCOUNT TO ROLE sysadmin;
     ```

3. **Create Storage Integration**:
   - Create:
     ```sql
     CREATE STORAGE INTEGRATION S3_INTEGRATION
         TYPE = EXTERNAL_STAGE
         STORAGE_PROVIDER = S3
         STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::191015975005:role/Testsnowflakerole'
         ENABLED = TRUE
         STORAGE_ALLOWED_LOCATIONS = ('s3://snowflakecomputingpro/Input/');
     ```

4. **Describe Integration**:
   - Get the external ID and IAM user ARN:
     ```sql
     DESC INTEGRATION S3_INTEGRATION;
     ```

5. **Update IAM Role Trust Policy**:
   - In AWS IAM, edit the role’s trust policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Principal": {
                     "AWS": "<Storage_AWS_IAM_USER_ARN>"
                 },
                 "Action": "sts:AssumeRole",
                 "Condition": {
                     "StringEquals": {
                         "sts:ExternalId": "<external_id>"
                     }
                 }
             }
         ]
     }
     ```

6. **Create Stage**:
   - Create:
     ```sql
     CREATE OR REPLACE STAGE TESLA_DATA_STAGE
         URL = 's3://snowflakecomputingpro/Input/*.csv'
         STORAGE_INTEGRATION = S3_INTEGRATION
         FILE_FORMAT = CSV_FORMAT;
     ```

7. **Load Data**:
   - Load all CSVs:
     ```sql
     COPY INTO TESLA_DATA
         FROM @TESLA_DATA_STAGE
         PATTERN = '.*.csv';
     ```

8. **Verify Data**:
   - Query:
     ```sql
     SELECT * FROM TESLA_DATA;
     ```

### Step 7: Set Up Snowpipe for Real-Time Data Loading
1. **Use Existing Integration and Stage**:
   - Use `S3_INTEGRATION` and `TESLA_DATA_STAGE` from Step 6B.

2. **Create Snowpipe**:
   - Create:
     ```sql
     CREATE OR REPLACE PIPE tesla_pipe_test
         AUTO_INGEST = TRUE
         AS
         COPY INTO TESLA_DATA
         FROM @TESLA_DATA_STAGE
         FILE_FORMAT = CSV_FORMAT;
     ```

3. **Verify Pipe**:
   - Run:
     ```sql
     SHOW PIPES;
     ```

4. **Configure S3 Event Notification**:
   - Run `SHOW PIPES` to get the pipe’s notification channel ARN.
   - In AWS S3, go to the bucket’s “Properties” > “Create event notification.”
   - Select “All object create events (`s3:ObjectCreated:*`)”.
   - Specify the SQS queue using the ARN from `SHOW PIPES`.
   - Save the notification.

5. **Test Snowpipe**:
   - Upload a new CSV to `s3://snowflakecomputingpro/Input/`.
   - Query `TESLA_DATA` to verify automatic loading:
     ```sql
     SELECT * FROM TESLA_DATA;
     ```

### Step 8: Visualize Data with AWS QuickSight
1. **Access QuickSight**:
   - Log in to AWS Console > QuickSight.
   - Click “Datasets” or “New Analysis.”

2. **Create Data Source**:
   - Click “New Dataset” > Select Snowflake.
   - Enter:
     - **Data source name**: `tesla_snowflake_data`
     - **Server**: `<account_identifier>.snowflakecomputing.com`
     - **Database**: `TEST_DB`
     - **Warehouse**: `compute_wh`
     - **Schema**: `PUBLIC`
     - **Username/Password**: Snowflake credentials
   - Click “Create Data Source.”

3. **Select Table**:
   - Choose `TESLA_DATA` > Click “Select.”

4. **Create Visualizations**:
   - Click “Create Analysis.”
   - Drag fields:
     - X-axis: `Date`
     - Y-axis: `Close_value`
   - Select **Line Chart**.
   - Add filters or trend lines as needed.

5. **Publish Dashboard**:
   - Save and publish the analysis for sharing.

### Step 9: Explore Time Travel
1. **Test Dropping and Restoring**:
   - Drop `TESLA_DATA`:
     ```sql
     DROP TABLE TESLA_DATA;
     ```
   - Restore within retention period (1 day for Standard Edition):
     ```sql
     UNDROP TABLE TESLA_DATA;
     ```

2. **Query Historical Data**:
   - Query before a specific statement (replace with actual query ID):
     ```sql
     SELECT * FROM TESLA_DATA BEFORE (STATEMENT = '019e03c6-0000-1682-0000-000187721799');
     ```

### Step 10: Optimize and Monitor
1. **Enable Auto-Suspend**:
   - Set:
     ```sql
     ALTER WAREHOUSE compute_wh SET AUTO_SUSPEND = 60;
     ```

2. **Monitor Queries**:
   - Check query history in Snowflake’s Monitoring section.

3. **Manage Costs**:
   - Use Standard Edition ($2/credit).
   - Delete unused AWS/Snowflake resources after completion.

## Snowflake Architecture
- **Cloud Services Layer**: Handles authentication, optimization, and metadata.
- **Query Processing Layer**: Virtual warehouses execute queries.
- **Database Storage Layer**: Stores structured/semi-structured data.
- **Separation of Compute and Storage**: Enables scalability and cost efficiency.

## Security Features
- **Role-Based Access Control**: Use `sysadmin` or `securityadmin` roles, avoid `ACCOUNTADMIN`.
- **Encryption**: Data encrypted at rest and in transit.
- **Storage Integration**: Secure S3 access via IAM roles.

## Cost Management
- Use Standard Edition for cost efficiency ($2/credit).
- Enable auto-suspend (60 seconds) and auto-resume.
- Monitor Snowpipe usage to avoid excessive charges.
- Delete unused resources (S3 buckets, Snowflake databases).

## Potential Challenges and Solutions
- **File Format Mismatch**:
  - **Issue**: CSV structure doesn’t match table schema.
  - **Solution**: Validate CSV files; use `FORCE = TRUE` for testing only.
- **Snowpipe Notification Failure**:
  - **Issue**: SQS configuration errors prevent auto-ingest.
  - **Solution**: Verify ARN in S3 event notification.
- **Cost Overruns**:
  - **Issue**: Idle warehouses or frequent Snowpipe runs.
  - **Solution**: Set low auto-suspend timeout; monitor usage.
- **QuickSight Connection**:
  - **Issue**: Incorrect Snowflake credentials.
  - **Solution**: Verify server, database, and credentials.

## Sample Visualization
A line chart plotting `Close_value` over `Date` is recommended. Below is a sample Chart.js configuration for reference:

```json
{
    "type": "line",
    "data": {
        "labels": ["2025-01-01", "2025-01-02", "2025-01-03", "2025-01-04", "2025-01-05"],
        "datasets": [{
            "label": "Tesla Close Price",
            "data": [400.5, 405.2, 398.7, 410.0, 415.3],
            "borderColor": "#1E90FF",
            "backgroundColor": "rgba(30, 144, 255, 0.2)",
            "fill": true,
            "tension": 0.4
        }]
    },
    "options": {
        "scales": {
            "x": {
                "title": {
                    "display": true,
                    "text": "Date"
                }
            },
            "y": {
                "title": {
                    "display": true,
                    "text": "Close Price ($)"
                },
                "beginAtZero": false
            }
        },
        "plugins": {
            "title": {
                "display": true,
                "text": "Tesla Stock Price Trend"
            }
        }
    }
}
```

In QuickSight, map `Date` to X-axis and `Close_value` to Y-axis, then publish the dashboard.

