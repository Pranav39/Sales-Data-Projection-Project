# Sales Data Projection — Real-Time AWS Data Pipeline

## Goal

The **Sales Data Projection** project is a real-time data ingestion and analytics pipeline using AWS. It simulates the generation of sales data for tech gadgets, processes the data using streaming and transformation services, and stores it in a queryable data lake for analysis using SQL. 

---

## Technologies & AWS Services

- Python 3.x
- `boto3` for AWS access
- AWS CLI with named profile
- AWS Services:
  - DynamoDB
  - IAM
  - EventBridge Pipes
  - Kinesis Data Streams
  - Kinesis Firehose
  - Lambda
  - S3
  - Glue Crawler
  - Glue Data Catalog
  - Athena
    
---

## Project Overview

- **Data Generation**: A Python script generates mock sales orders locally.
- **Ingestion**: Data is written to a DynamoDB table (`order_table`).
- **Streaming Pipeline**:
  - EventBridge Pipe routes from DynamoDB stream to Kinesis Data Stream.
  - Kinesis Firehose invokes a Lambda function for transformation.
  - Transformed records are written to Amazon S3.
- **Data Lake Setup**:
  - AWS Glue Crawler scans the S3 bucket.
  - Athena is used for querying structured results.

---

## Architecture Diagram
Python Script → DynamoDB → EventBridge Pipe → Kinesis Data Stream → Firehose (Lambda Transform) → S3 → Glue Crawler → Glue Data Catalog → Athena

---

## Workflow

This pipeline simulates and processes real-time sales data as follows:

### 1. Mock Data Generation (Local)
- The script `mock_data_generator_for_dynamodb.py` generates synthetic sales records.
- Each record includes: `orderid`, `product_name`, `quantity`, and `price`.
- Data is inserted every 3 seconds into a **DynamoDB** table named `order_table`.

### 2. EventBridge Pipe
- DynamoDB streams are enabled on `order_table`.
- **EventBridge Pipes** captures the `NewImage` from each stream event and sends it to **Kinesis Data Streams**.

### 3. Kinesis Data Firehose + Lambda Transformation

#### Lambda Function (Transformation Layer)
- **Input**: Raw stream record with DynamoDB's `NewImage` structure (type-annotated).
- **Process**:
  - Decode base64 and parse the JSON payload.
  - Extract values from `NewImage`:
    ```json
    {
      "orderid": {"S": "123"},
      "product_name": {"S": "Laptop"},
      "quantity": {"N": "2"},
      "price": {"N": "349.99"}
    }
    ```
  - Flatten and convert to clean JSON:
    ```json
    {
      "orderid": "123",
      "product_name": "Laptop",
      "quantity": 2,
      "price": 349.99
    }
    ```
  - Encode it back to base64 (required by Firehose).
- **Output**: Line-delimited base64-encoded JSON passed to Firehose.

### 4. Amazon S3 (Data Lake)
- Firehose writes the transformed data to an **S3 bucket** every 60 seconds.
- Data is stored as newline-delimited JSON for downstream processing.

### 5. AWS Glue Crawler
- A scheduled **Glue Crawler** scans the S3 location.
- Automatically creates or updates a **Glue Data Catalog Table** for Athena.

### 6. Amazon Athena (SQL Queries)
- You can run SQL queries directly on the transformed data in S3:
  ```sql
  SELECT * FROM sales_data_projection_db.sales_data
  WHERE price > 100
  ORDER BY orderid DESC
  LIMIT 10;

---

