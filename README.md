# 📌 Project Overview

This project is an end-to-end AWS e-commerce platform designed to demonstrate how different AWS services work together to process orders, maintain high availability, secure application data, and perform analytics.

The application uses API Gateway and Lambda to receive and process orders, RDS PostgreSQL for transactional data, SQS and SNS for event-driven and asynchronous processing, and ElastiCache Valkey for caching. The infrastructure uses VPC, EC2, Application Load Balancer, and Auto Scaling to provide a highly available application environment.

For analytics, order data is stored in Amazon S3, cataloged using AWS Glue, and queried with Amazon Athena. Secrets Manager, KMS, IAM, and CloudWatch provide security, encryption, access control, monitoring, logging, and troubleshooting.

The project also involved real-world troubleshooting, including IAM AccessDenied errors, KMS permission issues, API Gateway 404 errors, CloudWatch metric issues, ElastiCache connectivity testing, and Athena schema problems.

Final analytics result: 3,611 orders processed and analyzed


# 🏗️ Architecture

<img width="1536" height="1024" alt="AWS-Event-Driven-E-Commerce-Order-Processing-Analytics-Platform" src="https://github.com/user-attachments/assets/1685427f-de42-4a91-8308-04267d3301ac" />

# ☁️ AWS Services

- **Amazon VPC** — Network architecture
- **Amazon EC2** — Application servers
- **Application Load Balancer** — Traffic distribution
- **Auto Scaling** — High availability and scaling
- **Amazon API Gateway** — API interface
- **AWS Lambda** — Serverless processing
- **Amazon SQS** — Asynchronous messaging
- **Amazon SNS** — Notifications
- **Amazon RDS** — PostgreSQL database
- **Amazon ElastiCache** — Valkey caching
- **Amazon S3** — Data storage
- **AWS Glue** — Data catalog
- **Amazon Athena** — SQL analytics
- **AWS Secrets Manager** — Credential management
- **AWS KMS** — Encryption
- **Amazon CloudWatch** — Monitoring and logging
- **AWS IAM** — Access control

## 🔄 Order Processing Flow

```text
Client
  │
  ▼
API Gateway
  │
  ▼
Lambda
  │
  ├──────────► RDS
  │
  ├──────────► SQS
  │              │
  │              ▼
  │         Lambda Worker
  │              │
  │              ▼
  │             SNS
  │
  └──────────► CloudWatch
```
  
## 📊 Data Validation

### SQL Query

```sql
SELECT COUNT(*) AS total_orders
FROM orders;
```

### Result

```text
total_orders
------------
3611
```

# 🛠️ Troubleshooting

Hands-on troubleshooting included:

S3 AccessDenied
Lambda/KMS permission failure
API Gateway 404
CloudWatch missing metrics
ElastiCache connectivity testing
Athena schema/column issue

Issues were investigated using IAM policies, CloudWatch logs, AWS CLI, service configuration, and connectivity tests.

## ✅ Validation

- API successfully returned **HTTP 202**
- RDS order insertion confirmed
- SNS notification published successfully
- ALB targets reported **healthy**
- Auto Scaling maintained **2 healthy instances**
- ElastiCache connectivity successfully verified
- Athena query returned **3,611 orders**
- CloudWatch logs and alarms successfully validated

```text
AWS-Event-Driven-E-Commerce-Order-Processing-Analytics-Platform/
│
├── 01-vpc/
├── 02-ec2/
├── 03-alb/
├── 04-auto-scaling/
├── 05-api-gateway/
├── 06-lambda/
├── 07-sqs/
├── 08-sns/
├── 09-rds/
├── 10-elasticache/
├── 11-s3/
├── 12-glue/
├── 13-athena/
├── 14-secrets-manager/
├── 15-kms/
├── 16-cloudwatch/
├── 17-iam/
│
└── README.md
```

## 🎯 Skills Demonstrated

AWS: VPC, EC2, ALB, Auto Scaling, API Gateway, Lambda, SQS, SNS, RDS, ElastiCache, S3, Glue, Athena, Secrets Manager, KMS, CloudWatch, IAM

Cloud Engineering: High Availability, Multi-AZ, Event-Driven Architecture, Serverless, Monitoring, Troubleshooting, IAM, Linux/AWS CLI, SQL Analytics

