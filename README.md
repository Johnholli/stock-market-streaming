
This project implements a real-time stock data streaming pipeline that fetches live stock prices and streams them to AWS Kinesis for processing and analytics.

## Overview of Project ☁️
Build a serverless data pipeline that continuously streams stock market data from Yahoo Finance to AWS Kinesis Data Streams. The pipeline fetches real-time stock prices, calculates price changes, and streams the data for downstream analytics and monitoring.

## Architecture 🏗️
```
Yahoo Finance API → Python Script → AWS Kinesis Data Streams → [Analytics/Storage]
```

## Services Used 🛠
- **AWS Kinesis Data Streams**: Ingests and processes real-time stock data streams [Data Streaming]
- **Yahoo Finance API (yfinance)**: Provides real-time stock market data [Data Source]  
- **AWS IAM**: Manages secure access permissions for Kinesis operations [Security]
- **Python Virtual Environment**: Isolates project dependencies [Development]

## Prerequisites 📋
- Python 3.9+
- AWS CLI configured with appropriate permissions
- AWS account with Kinesis access

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

### 7. Run the Streaming Application
```bash
python stream_stock_data.py
```

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

## Data Schema 📊
Each streamed record contains:
```json
{
  "symbol": "AAPL",
  "open": 232.51,
  "high": 233.38,
  "low": 231.37,
  "price": 232.14,
  "previous_close": 232.56,
  "change": -0.42,
  "change_percent": -0.18,
  "volume": 39389400,
  "timestamp": "2025-08-30T21:50:25Z"
}
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

2. **Create S3 Bucket for Raw Data**
   - Bucket name: `stock-market-data-bucket-{unique-id}`
   - Store raw JSON data for historical analysis

3. **Deploy Lambda Function**
   - Function name: `ProcessStockData`
   - Runtime: Python 3.13
   - Trigger: Kinesis Data Stream
   - Purpose: Process and store data in both DynamoDB and S3

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

3. **Deploy Trend Analysis Lambda**
   - Function name: `StockTrendAnalysis`
   - Trigger: DynamoDB Streams
   - Purpose: Detect trend reversals using moving averages (SMA-5 vs SMA-20)

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
