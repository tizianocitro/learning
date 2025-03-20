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

## 19.2 Basics

DynamoDB is made of **tables**, each having a **primary key** (section [#19.2.1 Primary Keys](#1921-primary-keys)) that must be decided at creation time.

Each table can have an infinite number of **items** (also called rows), and each item has **attributes** that can be aded over time or be missing (null).
- Maximum size of an item is 400KB.
- Supported data types are:
    - Scalar types: `String`, `Number`, `Binary`, `Boolean`, `Null`.
    - Document types: `List`, `Map`.
    - Set types: `String Set`, `Number Set`, `Binary Set`.

### 19.2.1 Primary Keys

1. **Partition key** (also referred to as HASH strategy): the partition key must be unique for each item and diverse enough so that the data is distributed evenly across partitions.
    - For example, `User_ID` for a users table.

    ![DynamoDB Partition Key](/assets/aws-certified-developer-associate/dynamodb_partition_key.png "DynamoDB Partition Key")

2. **Partition key + sort key** (also referred to as HASH + RANGE strategy): the combination must be unique for each item and data is grouped by partition key (so it is crucial to choose a good partition key), then sorted by sort key.
    - For example, `User_ID` + `Game_ID` for a users table, so that all items for a user are grouped together and sorted by game.

    ![DynamoDB Partition Key + Sort Key](/assets/aws-certified-developer-associate/dynamodb_partition_key_sort_key.png "DynamoDB Partition Key + Sort Key")

### 19.2.2 Choosing the Right Partition Key

For instance, suppose that we are building a movie database with the following attributes:
- `movie_id`.
- `producer_name`.
- `leader_actor_name`.
- `movie_language`.

What could be the best partition key to maximize data distribution?
- `movie_id` has the highest cardinality so it is a good candidate.
- `movie_language` does not take many values and may be skewed towards English, so it is not a great choice for the partition key.
