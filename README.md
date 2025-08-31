
## Overview of Project ☁️
Build a serverless data pipeline that continuously streams stock market data from Yahoo Finance to AWS Kinesis Data Streams. The pipeline fetches real-time stock prices, calculates price changes, and streams the data for downstream analytics and monitoring. 

## Services Used 🛠
- **AWS Kinesis Data Streams**: Ingests and processes real-time stock data streams [Data Streaming]
- **Yahoo Finance API (yfinance)**: Provides real-time stock market data [Data Source]
- **Amazon DynamoDB**: Stores processed stock data for low-latency querying
- **AWS Lambda**: Processes data and detects anomalies
- **Amazon S3**: Stores raw stock data for long-term analytics
- **Amazon Athena**: Querying historical data
- **Amazon SNS**: Sends real-time stock trend alerts using AWS Lambda
- **AWS IAM**: Manages secure access permissions for Kinesis operations [Security]
- **Python Virtual Environment**: Isolates project dependencies [Development]

## Prerequisites 📋
- Python 3.9+
- AWS CLI configured with appropriate permissions
- AWS account with Kinesis access


## Architecture 🏗️

<img width="1538" height="750" alt="image" src="https://github.com/user-attachments/assets/8033df09-3702-4db1-94c8-c53e8218bf2d" />

## ➡️ Final Result

A fully functional near real-time stock analytics pipeline built using AWS services, featuring:

Event-driven architecture with Amazon Kinesis for real-time data ingestion
Lambda-based anomaly detection and stock trend evaluation
Low-latency storage in DynamoDB for fast access to processed data
Historical data archiving in Amazon S3 and querying via Athena
Real-time alerts via Amazon SNS (Email/SMS) for significant stock movements
Secure and cost-optimized design using IAM and serverless technologies
This project demonstrates how to build a scalable, alert-driven financial data pipeline using modern AWS cloud-native tools.

## Setup Instructions

### 1. Clone the Repository
```bash
git clone https://github.com/Johnholli/stock-market-streaming.git
cd stock-market-streaming
```

### 2. Create Virtual Environment
```bash
python3 -m venv stock_env
source stock_env/bin/activate  # On Windows: stock_env\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure AWS Credentials
```bash
aws configure
# Enter your AWS Access Key ID, Secret Access Key, Region (us-east-1), and output format (json)
```

### 5. Create Kinesis Stream
```bash
aws kinesis create-stream --stream-name Stock-market-stream --shard-count 1
```

### 6. Verify Stream Status
```bash
aws kinesis describe-stream --stream-name Stock-market-stream --query 'StreamDescription.StreamStatus'
```
Wait until status shows `"ACTIVE"` before proceeding.

### 7. Write the Python Script to stream stock data
Now that Kinesis is ready and our local environment is set up, we need a Python script to fetch stock data and continuously send it to Kinesis.

1. Create a New Python File
Open any code editor (VS Code, PyCharm, or Notepad++).
Create a new file and save it as `stream_stock_data.py`.

2. Copy and Paste the Script Below

<img width="696" height="664" alt="Screenshot 2025-08-30 at 5 45 56 PM" src="https://github.com/user-attachments/assets/10960f53-e4ea-4c4a-a6dc-d6876a54cca7" />

<img width="697" height="608" alt="Screenshot 2025-08-30 at 5 48 50 PM" src="https://github.com/user-attachments/assets/78faa1ed-ef7d-45a2-9a3a-2fb8e2c2fbe5" />


📝 Remember to replace <YOUR_DATA_STREAM_NAME> with the actual name of your Kinesis Data Stream!

What This Code Does

1. Fetches Stock Data from Yahoo Finance
Retrieves the latest Open, High, Low, Close, Volume, and Previous Close for `AAPL`.
Calculates Change and Change Percentage based on previous close.

2. Formats the Data into a JSON Object
Converts stock data into a structured dictionary with a timestamp.

3. Streams Data to AWS Kinesis
Sends the JSON-encoded stock data to an AWS Kinesis Data Stream every 30 seconds.
Uses AAPL as the `PartitionKey` for better ordering.

4. Handles Errors & Retries
If API fails or data is missing, the script waits and retries instead of crashing.

5. Ensures Debugging & Logging
Prints each sent record and Kinesis response to the terminal for monitoring.

### 8. Run the Streaming Application
```bash
python stream_stock_data.py
```
Expected Output:
Every 30 seconds, you should see something like:

<img width="1396" height="146" alt="image" src="https://github.com/user-attachments/assets/aae5752d-9d5a-45e8-a814-560fb821e92e" />


## Configuration ⚙️

### Environment Variables
Create a `.env` file (optional):
```env
AWS_REGION=us-east-1
STREAM_NAME=Stock-market-stream
STOCK_SYMBOL=AAPL
DELAY_SECONDS=30
```

### Modify Stock Symbol
Edit `stream_stock_data.py` to change the stock symbol:
```python
STOCK_SYMBOL = "TSLA"  # Change to any valid stock symbol
```


```

## Monitoring 📈
- View streaming data in AWS Kinesis Console
- Monitor stream metrics in CloudWatch
- Check application logs for errors or connection issues

## Project Structure 📁
```
stock-market-streaming/
├── stream_stock_data.py    # Main streaming application
├── requirements.txt        # Python dependencies
├── README.md              # Project documentation
└── .gitignore            # Git ignore file
```

## Troubleshooting 🔧

### Common Issues:
1. **ResourceNotFoundException**: Stream not found
   - Verify stream name matches exactly (case-sensitive)
   - Check AWS region in both code and CLI

2. **ModuleNotFoundError**: Missing dependencies
   - Ensure virtual environment is activated
   - Run `pip install -r requirements.txt`

3. **AWS Credentials**: Access denied errors
   - Run `aws configure` to set up credentials
   - Verify IAM permissions for Kinesis

### Debug Commands:
```bash
# Check AWS identity
aws sts get-caller-identity

# List Kinesis streams
aws kinesis list-streams

# Test stream access
aws kinesis put-record --stream-name Stock-market-stream --data '{"test": "data"}' --partition-key "test"
```

## Advanced Pipeline Components

### Processing Data with AWS Lambda
1. **Create DynamoDB Table**
   - Table name: `stock-market-data`
   - Partition key: `symbol` (String)
   - Sort key: `timestamp` (String)
  
*Why Do We Need a DynamoDB Table?*
A NoSQL database like DynamoDB is ideal for handling real-time stock data due to its:
- Fast read/write operations – Ensuring low-latency querying.
- Flexible schema – Enabling easy adjustments to stock data fields.
- Scalability – Handling high-volume stock transactions efficiently.

*What Will Be Stored in DynamoDB?*
The Lambda function will process the following key stock data fields before storing them in DynamoDB:
<img width="608" height="568" alt="image" src="https://github.com/user-attachments/assets/9468544b-9931-4974-85b4-b8a01c2563ef" />


2. **Create S3 Bucket for Raw Data**
   - Bucket name: `stock-market-data-bucket-{unique-id}`
   - Store raw JSON data for historical analysis
  
3. **Create a New IAM Role for Lambda**
   Before creating the Lambda function, set up an IAM role with the necessary permissions:
   Open AWS Console → Navigate to IAM.
   Click Roles → Create Role.
   Trusted Entity Type: Select AWS Service.
   Use Case: Choose Lambda → Click Next.

   Attach Policies: Add the following managed policies:
  - 'AmazonKinesisFullAccess` (Read from Kinesis).
  - `AmazonDynamoDBFullAccess` (Write to DynamoDB).
  - `AWSLambdaBasicExecutionRole` (CloudWatch logging).
  - `AmazonS3FullAccess` (Write to S3).

Click Next → Name the role `Lambda_Kinesis_DynamoDB_Role` → Create Role.

4. **Deploy Lambda Function**
   - Function name: `ProcessStockData`
   - Runtime: Python 3.13
   - Trigger: Kinesis Data Stream
   - Purpose: Process and store data in both DynamoDB and S3

5. **Deploy the Lambda Code**
   
<img width="694" height="629" alt="Screenshot 2025-08-30 at 6 15 13 PM" src="https://github.com/user-attachments/assets/7f1de9ab-ec57-4ca1-a313-6e6dc4118be0" />

<img width="690" height="578" alt="Screenshot 2025-08-30 at 6 15 38 PM" src="https://github.com/user-attachments/assets/f6a2a64b-e22a-4bb0-9678-35d61ed4e547" />

6. **Test the Integration**

- Verify Data in DynamoDB
Open AWS Console → Navigate to DynamoDB.
Click on stock-market-data → Explore Table Items.
You should see processed stock data records appearing in real-time.

- Verify Data in S3
Open AWS Console → Navigate to S3.
You should see the raw stock data records appearing in real-time.

### Query Historical Stock Data using Amazon Athena

#### Create Glue Catalog Table
```bash
# 1. Create Glue Database
aws glue create-database --database-input Name=stock_data_db

# 2. Create Glue Table (replace YOUR-BUCKET-NAME)
aws glue create-table --database-name stock_data_db --table-input '{
  "Name": "stock_data_table",
  "StorageDescriptor": {
    "Columns": [
      {"Name": "symbol", "Type": "string"},
      {"Name": "open", "Type": "double"},
      {"Name": "high", "Type": "double"},
      {"Name": "low", "Type": "double"},
      {"Name": "price", "Type": "double"},
      {"Name": "previous_close", "Type": "double"},
      {"Name": "change", "Type": "double"},
      {"Name": "change_percent", "Type": "double"},
      {"Name": "volume", "Type": "bigint"},
      {"Name": "timestamp", "Type": "string"}
    ],
    "Location": "s3://YOUR-BUCKET-NAME/raw-data/",
    "InputFormat": "org.apache.hadoop.mapred.TextInputFormat",
    "OutputFormat": "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
    "SerdeInfo": {
      "SerializationLibrary": "org.openx.data.jsonserde.JsonSerDe"
    }
  }
}'
```
Query Data Using Athena

1. Open AWS Console → Navigate to Amazon Athena.
2. Click- Launch Query Editor. 
3. Select stock_data_db as the database.
4. You will get a pop-up like one given below asking to setup the query location in Amazon S3. Click on Edit Settings.
5. From the view option, select the currently created S3 bucket. 
6. Click Save.

#### Query Examples
```sql
-- Basic query
SELECT * FROM stock_data_table LIMIT 10;

-- Top 5 stocks with highest price change
SELECT symbol, price, previous_close,
       (price - previous_close) AS price_change
FROM stock_data_table
ORDER BY price_change DESC
LIMIT 5;

-- Average trading volume per stock
SELECT symbol, AVG(volume) AS avg_volume
FROM stock_data_table
GROUP BY symbol;

-- Anomalous stocks (Price change > 5%)
SELECT symbol, price, previous_close,
       ROUND(((price - previous_close) / previous_close) * 100, 2) AS change_percent
FROM stock_data_table
WHERE ABS(((price - previous_close) / previous_close) * 100) > 5;
```

### Stock Trend Alerts using SNS

#### Setup Steps
1. **Enable DynamoDB Streams**
   ```bash
   aws dynamodb update-table --table-name stock-market-data --stream-specification StreamEnabled=true,StreamViewType=NEW_IMAGE
   ```

2. **Create SNS Topic**
   ```bash
   aws sns create-topic --name Stock_Trend_Alerts
   # Subscribe to email alerts
   aws sns subscribe --topic-arn arn:aws:sns:us-east-1:YOUR-ACCOUNT:Stock_Trend_Alerts --protocol email --notification-endpoint your-email@example.com
   ```
3. **Create an IAM Role for Lambda**

   Before creating the Lambda function, we need to set up an IAM role with appropriate permissions:

   - Go to AWS IAM Console → Click Roles → Click Create Role.
   - Select Trusted Entity: `AWS Service`, and choose `Lambda`.
   - Attach Policies:
    'AmazonDynamoDBFullAccess` → Allows Lambda to read stock data.
    `AmazonSNSFullAccess` → Allows Lambda to publish alerts to SNS.
    `AWSLambdaBasicExecutionRole` → Allows CloudWatch Logs.
   - Click Next, name the role `StockTrendLambdaRole`, and create the role.

4. **Deploy Trend Analysis Lambda**
    1. Go to AWS Lambda Console → Create Function.
    2. Choose "Author from Scratch".
    3. Function Name: `StockTrendAnalysis`
    4. Runtime: Python 3.13.
    5. Permissions: Choose "Use an existing role" and select StockTrendLambdaRole.
    6. Click Create Function.
    7. In the function overview, Click on Add Trigger.
    8. Select DynamoDB as the source.
    9. Choose the created DynamoDB table (`stock-market-data`).
    10. Modify the Batch size to 2.
    11. Add the Lambda Code given below:
  
  <img width="692" height="663" alt="Screenshot 2025-08-30 at 6 37 05 PM" src="https://github.com/user-attachments/assets/866c5227-3a72-477e-91e5-df399194e083" />

  <img width="694" height="667" alt="Screenshot 2025-08-30 at 6 38 38 PM" src="https://github.com/user-attachments/assets/802b572c-9ccb-49a1-9556-b8575ceddd66" />

   📝 Make sure to replace <YOUR-DYNAMODB-TABLE-NAME> with your actual table name and <YOUR-SNS-TOPIC-ARN> with your actual SNS Topic ARN. 
  
  12. Click on Deploy.

Understanding the Lambda Code for Trend Analysis
Here’s a breakdown of how it works:

1. Fetching Recent Stock Data
The function retrieves stock prices from the last 5 minutes using `get_recent_stock_data()`.
It queries DynamoDB for stock records and sorts them by timestamp.

2. Calculating Moving Averages
The function computes SMA-5 (short-term) and SMA-20 (long-term) averages using `calculate_moving_average()`.
It also calculates the previous values of these SMAs to compare trends over time.

3. Detecting Trend Reversals
If SMA-5 crosses above SMA-20, it signals an uptrend (BUY opportunity).
If SMA-5 crosses below SMA-20, it signals a downtrend (SELL opportunity).

4. Sending Alerts via Amazon SNS
If a trend shift is detected, a notification is published to an SNS topic, alerting users about potential buying or selling opportunities.
The function handles possible SNS failures using a `try-except` block.

### Conclusion

This project successfully demonstrates how to build a near real-time stock market data analytics pipeline using AWSS fully managed, serverless services. By integrating Amazon Kinesis, AWS Lambda, Amazon DynamoDB, Amazon S3, Amazon Athena, and Amazon SNS, we've created a robust and scalable architecture capable of streaming, processing, storing, analyzing, and alerting on stock data with minimal operational overhead. This hands-on experience with in this project provides a strong foundation for building more advanced real-time analytics systems in production environments.

Key outcomes:
Real-time data ingestion and processing with event-driven architecture.
Anomaly detection and trend alerts using AWS Lambda and SNS.
Separation of raw and processed data for efficient storage and analytics.
Low-cost implementation suitable for learning and prototyping.
 
## Project Cleanup 🧹

**Warning**: Remember to clean up resources to avoid unexpected charges!

```bash
# Delete Kinesis Stream
aws kinesis delete-stream --stream-name Stock-market-stream

# Delete DynamoDB Table
aws dynamodb delete-table --table-name stock-market-data

# Delete SNS Topic
aws sns delete-topic --topic-arn arn:aws:sns:us-east-1:YOUR-ACCOUNT:Stock_Trend_Alerts

# Delete S3 Buckets (empty first)
aws s3 rm s3://your-bucket-name --recursive
aws s3 rb s3://your-bucket-name

# Delete Lambda Functions
aws lambda delete-function --function-name ProcessStockData
aws lambda delete-function --function-name StockTrendAnalysis

# Delete Glue Database
aws glue delete-database --name stock_data_db
```

## Cost Considerations 💰
- Kinesis Data Streams: ~$0.014 per shard hour + $0.014 per million records
- Lambda: First 1M requests free, then $0.20 per 1M requests
- DynamoDB: 25GB free storage, $0.25/GB beyond that
- S3: First 5GB free, then $0.023/GB
- Athena: $5 per TB of data scanned
- SNS: First 1,000 notifications free

## Security Best Practices 🔒
- Use IAM roles instead of hardcoded credentials
- Enable CloudTrail for API logging
- Implement least-privilege access policies
- Rotate AWS access keys regularly
- Enable encryption for data at rest and in transit

---

**Author**: John Holly  
**Date**: August 2025  
**Version**: 2.0  
**Note**: This implements a near real-time pipeline (30-second delays) optimized for learning and cost-effectiveness.
