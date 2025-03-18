# 19 Serverless: DynamoDB

**Traditional applications leverage RDBMS databases**, which have the SQL query language that imposes strong requirements about how the data should be modeled to provide the ability to do query joins, aggregations, complex computations, etc.

These applications follow a traditional architecture:

![Traditional Architecture](/assets/aws-certified-developer-associate/traditional_architecture.png "Traditional Architecture")

In such applications, vertical scaling happens via getting more powerful CPU/RAM/IO, while horizontal scaling is achieved by increasing reading capability by adding EC2/RDS read replicas (limited by the number of read replicas that can be added).

**NoSQL databases** (e.g., MongoDB, DynamoDB) **are non-relational databases and are distributed**. They:
- Do not support query joins (or just limited support), so all the data that is needed for a query is present in one "row".
- Do not perform aggregations such as `SUM`, `AVG`, etc.
- **Scale horizontally** thanks to these design choices.

## 19.1 Overview

It is a fully managed NoSQL database, highly available with replication across multiple AZs.

It **scales to massive workloads and is fully distributed**, meaning it can handle millions of requests per seconds, trillions of rows, 100s of TB of storage, etc., while providing **fast and consistent performance with low latency on retrieval**. It:
- Is integrated with IAM for security, authorization and administration.
- Enables event-driven programming with DynamoDB Streams.
- Offers Low cost and auto-scaling capabilities.
- Provides *Standard & Infrequent Access (IA)* table class for different storage tiers.
