# AWS-Event-Driven-E-Commerce-Order-Processing-Analytics-Platform

📌 Project Overview

This project demonstrates a hands-on, event-driven e-commerce architecture built on AWS.

The platform accepts customer orders through an API, processes them asynchronously using AWS Lambda and Amazon SQS, stores order data in Amazon RDS for MySQL, uses ElastiCache/Valkey for caching, sends notifications through Amazon SNS, and builds an analytics pipeline using Amazon S3, AWS Glue, and Amazon Athena.

The project also focuses on:

IAM least-privilege access
Secure credential management
KMS encryption
VPC networking
SQS failure handling
CloudWatch monitoring
Troubleshooting real AWS service failures
Cost-conscious resource management

<img width="1536" height="1024" alt="AWS-Event-Driven-E-Commerce-Order-Processing-Analytics-Platform" src="https://github.com/user-attachments/assets/469f6470-8960-479e-b2b3-528e5ce11bc4" />

🔄 Order Processing Flow

1. Order Submission

The client sends an order through the API Gateway endpoint.

Client
  ↓
API Gateway
  ↓
Order Receiver Lambda

The API returns an asynchronous success response:

HTTP 202
Order accepted

2. Queue Processing

The receiver Lambda publishes the order to Amazon SQS.

Lambda
   ↓
SQS Order Queue

This decouples the API request from the database processing.

3. Order Worker

The worker Lambda consumes messages from SQS.

SQS
 ↓
enterprise-order-worker

The worker processes the order and stores it in MySQL.

4. Database

Orders are persisted in:

Amazon RDS
    ↓
MySQL

The final dataset was validated with:

3,611 orders

5. Notification

After order processing, SNS is used for order notifications.

Order Worker
     ↓
SNS Topic
     ↓
Notification

6. Caching

ElastiCache/Valkey was used as the caching layer to demonstrate application-side caching and private network connectivity.

📊 Analytics Pipeline

The project also includes a serverless analytics pipeline.

Order Data
    ↓
Amazon S3
    ↓
AWS Glue Crawler
    ↓
Glue Data Catalog
    ↓
Amazon Athena
    ↓
SQL Analytics
Data Validation

The final analytics dataset contained:

Total Orders = 3,611

Validation query:

SELECT COUNT(*) AS total_orders
FROM orders_03acf00548cb7ec3c6159db1ddbc9737;

Result:

total_orders
------------
3611

Analytical Query

The Glue crawler generated generic column names for the final table:

col0 → order_id
col1 → customer_id
col2 → total_amount
col3 → status
col4 → created_at

Therefore the analytical query was:

SELECT
    col3 AS status,
    COUNT(*) AS order_count,
    SUM(col2) AS total_amount
    
FROM orders_03acf00548cb7ec3c6159db1ddbc9737
GROUP BY col3
ORDER BY order_count DESC;

🔐 Security Architecture

IAM Least Privilege

The Lambda execution role was configured with AWS-managed and customer-created policies.

Important application permissions were restricted to specific resources.

SNS
{
  "Effect": "Allow",
  "Action": "sns:Publish",
  "Resource": "arn:aws:sns:us-east-1:<ACCOUNT_ID>:enterprise-order-notifications"
}
Secrets Manager
{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "arn:aws:secretsmanager:us-east-1:<ACCOUNT_ID>:secret:enterprise/ecommerce/rds-<SUFFIX>"
}
KMS
{
  "Effect": "Allow",
  "Action": "kms:Decrypt",
  "Resource": "arn:aws:kms:us-east-1:<ACCOUNT_ID>:key/<KEY_ID>"
}

Sensitive values and account identifiers should be redacted before publishing screenshots or configuration files publicly.

🔑 Secrets Manager & KMS

Database credentials are stored in:

AWS Secrets Manager
enterprise/ecommerce/rds

The secret is encrypted using the customer-managed KMS key:

enterprise-ecommerce-key

The Lambda receives permission to retrieve the secret and decrypt it without exposing credentials directly in application configuration.

No passwords or secret values are stored in the repository.

🪣 S3 Security

The analytics bucket was configured with:

Block Public Access → ON

Default Encryption → SSE-S3

This prevents accidental public exposure of the stored data.

📬 SQS Reliability

The order queue uses a Dead-Letter Queue for repeatedly unsuccessful messages.

Order Queue
    │
    ├── Successful → Processed
    │
    └── Failed
          ↓
       Retry
          ↓
       Retry
          ↓
       Retry
          ↓
        DLQ

Maximum receives before moving a message to the DLQ:

3

The visibility timeout was configured to:

60 seconds

This gives the Lambda sufficient time to process messages before they become visible again.

📈 Monitoring

Amazon CloudWatch was used for:

Lambda invocations
Lambda errors
Lambda duration
Concurrent executions
Throttles
SQS metrics
RDS metrics
CloudWatch Logs
Alarms

The project included testing of CloudWatch metrics and alarms as part of operational troubleshooting.

🧪 Troubleshooting Experience

One of the main goals of this project was learning to troubleshoot AWS infrastructure rather than simply deploying resources.

1. S3 AccessDenied

While uploading analytics data:

AccessDenied
s3:PutObject

The EC2 role did not initially have permission to upload to the required S3 location.

Resolution: IAM permissions were reviewed and the required access was granted.

2. S3 DeleteObject AccessDenied

An attempt to remove a test object resulted in:

AccessDenied
s3:DeleteObject

This demonstrated that S3 upload and delete permissions are separate IAM actions.

3. Glue Crawler Trust Policy

The first crawler failed with:

Service is unable to assume provided role.
Please verify role's TrustPolicy

The crawler IAM role trust relationship was investigated and corrected.

4. Athena Unsupported Format

Athena initially returned:

HIVE_UNSUPPORTED_FORMAT:
Unable to create input format

The Glue table/location configuration was investigated and corrected.

5. Athena Schema Issue

An analytical query initially failed:

COLUMN_NOT_FOUND:
Column 'status' cannot be resolved

Instead of guessing, the table schema was inspected:

DESCRIBE orders_03acf00548cb7ec3c6159db1ddbc9737;

The table contained:

col0
col1
col2
col3
col4

The query was then adapted to the actual schema.

6. API Gateway 404

An API request initially returned:

HTTP/2 404

The API ID, route, and stage were investigated.

The configured stage was identified as:

api-lamb

This demonstrated the importance of verifying the deployed API stage and route rather than assuming the endpoint URL.

7. CloudWatch No Data

CloudWatch initially displayed:

No data available

The metric namespace, function name, and time range were investigated.

The Lambda metrics were eventually confirmed:

Invocations
Errors
Duration
ConcurrentExecutions
Throttles

🧠 Key Technical Lessons

This project provided practical experience with:

Event-driven architecture
Asynchronous processing
Serverless architecture
AWS networking
IAM troubleshooting
Security Groups
SQS retry mechanisms
Dead-Letter Queues
Lambda troubleshooting
CloudWatch monitoring
S3 permissions
Glue catalog management
Athena SQL troubleshooting
Secrets Manager
KMS encryption
ElastiCache/Valkey
API Gateway routing
Cost-aware AWS resource management

Most importantly, troubleshooting followed a structured approach:

Problem
   ↓
Read the error
   ↓
Identify the AWS service
   ↓
Check logs / metrics
   ↓
Check IAM
   ↓
Check networking / Security Groups
   ↓
Test configuration
   ↓
Apply fix
   ↓
Retest

💰 Cost Management

This project was designed with cost awareness in mind.

After completing the implementation and documentation, temporary AWS resources were identified for cleanup to avoid unnecessary ongoing charges.

Resources should be deleted systematically after all screenshots and documentation are completed.

Particular attention should be given to potentially billable resources such as:

RDS
ElastiCache/Valkey
NAT Gateway
VPC endpoints
CloudWatch usage
S3 storage
Glue
KMS requests

🎯 Project Outcome

Successfully built and validated an AWS-based e-commerce processing and analytics platform with:

API Gateway
     ↓
Lambda
     ↓
SQS
     ↓
Lambda
     ↓
RDS + ElastiCache
     ↓
SNS

RDS / Data
     ↓
S3
     ↓
Glue
     ↓
Athena
     ↓
3,611 validated orders

The project demonstrates both AWS architecture implementation and hands-on troubleshooting across multiple AWS services.
