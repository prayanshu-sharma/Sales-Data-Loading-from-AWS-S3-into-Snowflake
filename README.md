# Sales-Data-Loading-from-AWS-S3-into-Snowflake

# Sample Project: Sales Data Loading from AWS S3 into Snowflake

Project Goal-

Load a sample sales dataset stored in AWS S3 into Snowflake, then transform and query it for reporting.

Step 1: Create a Database and Schema

-- Create a database
CREATE DATABASE SALES_DB;
-- Create a schema
CREATE SCHEMA RAW_DATA;

Step 2: Create a Table

Let's assume our sales data has the following columns:
ORDER_ID (Number)
ORDER_DATE (Date)
CUSTOMER_NAME (String)
PRODUCT (String)
QUANTITY (Number)
PRICE (Number)

USE DATABASE SALES_DB;
USE SCHEMA RAW_DATA;

CREATE OR REPLACE TABLE SALES (
    ORDER_ID NUMBER,
    ORDER_DATE DATE,
    CUSTOMER_NAME STRING,
    PRODUCT STRING,
    QUANTITY NUMBER,
    PRICE NUMBER
);

Step 3: Prepare Sample Data & Upload to S3

Create a CSV file called sales_data.csv with sample records:

ORDER_ID,ORDER_DATE,CUSTOMER_NAME,PRODUCT,QUANTITY,PRICE
1001,2025-09-01,John Doe,Laptop,2,1200
1002,2025-09-02,Jane Smith,Mouse,5,25
1003,2025-09-03,David Lee,Keyboard,3,45
1004,2025-09-04,Amy Brown,Monitor,1,300

Upload it to your S3 bucket:    //this is your file location in s3 bucket

s3://my-snowflake-demo-bucket/sales_data.csv


Step 4: Create an External Stage for S3
CREATE OR REPLACE STAGE S3_STAGE
URL='s3://my-snowflake-demo-bucket'                                         //URL → points to your S3 bucket
CREDENTIALS=(AWS_KEY_ID='YOUR_AWS_KEY' AWS_SECRET_KEY='YOUR_AWS_SECRET')   //CREDENTIALS → AWS keys (use IAM Role if possible in production)
FILE_FORMAT = (TYPE=CSV FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);   //FILE_FORMAT → tells Snowflake how to read the file



Step 5: Validate Files in S3 and check which file are present in s3 bucket

LIST @S3_STAGE;                        //shows all file name are available in this stage
data looks like

+------------------------------------------+------+---------------------------------+---------------------+
| name                                     | size | md5                             | last_modified       |
+------------------------------------------+------+---------------------------------+---------------------+
| sales_data.csv                            |  123 | ...                             | 2025-09-20 12:34:56|
+------------------------------------------+------+---------------------------------+---------------------+


Step 6: Load Data into Table using copy into command

COPY INTO SALES                                                       //COPY INTO → loads data from external stage to table
FROM @S3_STAGE/sales_data.csv
FILE_FORMAT = (TYPE=CSV FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1)
ON_ERROR = 'CONTINUE';                                          //ON_ERROR='CONTINUE' → skips bad records (you can use 'ABORT_STATEMENT' in production)


Step 7: Verify Data is present in snowflake table or not 

SELECT * FROM SALES;

You should see:

ORDER_ID	ORDER_DATE	CUSTOMER_NAME	PRODUCT	QUANTITY	PRICE
1001	2025-09-01	John Doe	Laptop	2	1200
1002	2025-09-02	Jane Smith	Mouse	5	25
1003	2025-09-03	David Lee	Keyboard	3	45
1004	2025-09-04	Amy Brown	Monitor	1	300


Step 8: Transform & Query Data

Example: Calculate total sales by product
SELECT PRODUCT,
       SUM(QUANTITY * PRICE) AS TOTAL_SALES
FROM SALES
GROUP BY PRODUCT
ORDER BY TOTAL_SALES DESC;


Step 9: Automate with Snowpipe (Optional, Advanced)

If you want to make this real-time, you can create a Snowpipe to auto-ingest data from S3 whenever a new file is uploaded.

CREATE OR REPLACE PIPE SALES_PIPE
AS
COPY INTO SALES
FROM @S3_STAGE
FILE_FORMAT = (TYPE=CSV FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1)
ON_ERROR = 'CONTINUE';
You would then configure S3 event notifications to trigger this Snowpipe.



Conclusion-
Creating databases, schemas, tables in Snowflake
Creating stages to connect with S3
Using COPY INTO to load data
Running basic transformations (aggregation, grouping)
(Optional) Setting up Snowpipe for real-time ingestion
