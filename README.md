# Real-Time Stock Market Data Streaming Pipeline ⚡

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

## Next Steps 🚀
- Set up Kinesis Analytics for real-time processing
- Add Lambda functions for data transformation
- Store processed data in DynamoDB or S3
- Create CloudWatch dashboards for monitoring
- Implement multiple stock symbols streaming

## Cost Considerations 💰
- Kinesis Data Streams: ~$0.014 per shard hour + $0.014 per million records
- Minimal compute costs for the Python script
- Consider using spot instances for production deployments

## Security Best Practices 🔒
- Use IAM roles instead of hardcoded credentials
- Enable CloudTrail for API logging
- Implement least-privilege access policies
- Rotate AWS access keys regularly

---

**Author**: John Hollingsworth  
**Date**: August 2025  
**Version**: 1.0
