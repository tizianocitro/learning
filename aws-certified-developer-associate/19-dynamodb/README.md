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

## 19.3 Creating Tables and Items in DynamoDB

### 19.3.1 Creating Tables in DynamoDB

Go to the DynamoDB console and click on *Create Table* to **create a table**:

![DynamoDB Create Table](/assets/aws-certified-developer-associate/dynamodb_create_table.png "DynamoDB Create Table")

Give it a **name**:

![DynamoDB Table Name](/assets/aws-certified-developer-associate/dynamodb_table_name.png "DynamoDB Table Name")

Then, define the **primary key** and optionally the **sort key**:

![DynamoDB Primary Key](/assets/aws-certified-developer-associate/dynamodb_primary_key.png "DynamoDB Primary Key")

You can choose between using some default **settings** or customizing them yourself, we will customize them in this example:

![DynamoDB Settings](/assets/aws-certified-developer-associate/dynamodb_settings.png "DynamoDB Settings")

First thing is the **table class** (standard or standard & infraquent access):

![DynamoDB Table Class](/assets/aws-certified-developer-associate/dynamodb_table_class.png "DynamoDB Table Class")

Then, we need to define the **read/write capacity settings** by chosing between **provisioned** and **on-demand** modes:

![DynamoDB Read/Write Capacity](/assets/aws-certified-developer-associate/dynamodb_read_write_capacity.png "DynamoDB Read/Write Capacity")

And, if you select provisioned, you can set the **auto-scaling settings for read/write capacity** (more later on this):

![DynamoDB Auto-Scaling](/assets/aws-certified-developer-associate/dynamodb_auto_scaling.png "DynamoDB Auto-Scaling")

You get a **cost estimation** based on the settings you chose so far:

![DynamoDB Cost Estimation](/assets/aws-certified-developer-associate/dynamodb_cost_estimation.png "DynamoDB Cost Estimation")

Next are the settings for **encryption at rest**:

![DynamoDB Encryption at Rest](/assets/aws-certified-developer-associate/dynamodb_encryption_at_rest.png "DynamoDB Encryption at Rest")

Finally, you can optionally add **tags** and create the table. The table will appear in the **list of tables**:

![DynamoDB Table List](/assets/aws-certified-developer-associate/dynamodb_table_list.png "DynamoDB Table List")

### 19.3.2 Creating Items in DynamoDB

You can access the table by clicking on it in the table list, and you will see the **overview** of the table:

![DynamoDB Table Overview](/assets/aws-certified-developer-associate/dynamodb_table_overview.png "DynamoDB Table Overview")

You can also view and count the **items** in the table:

![DynamoDB Table Items](/assets/aws-certified-developer-associate/dynamodb_table_items.png "DynamoDB Table Items")

If you click on *View Items*, you can see the **items** in the table, **run queries**, **create items**, etc.:

![DynamoDB Table Items View](/assets/aws-certified-developer-associate/dynamodb_table_items_view.png "DynamoDB Table Items View")

Click on *Create Item* to **create an item** with as many attributes (and their values) as you want though the a UI form or JSON:

![DynamoDB Table Create Item](/assets/aws-certified-developer-associate/dynamodb_table_create_item.png "DynamoDB Table Create Item")

Create the item and it will appear in the **list of items**:

![DynamoDB Table Item Created](/assets/aws-certified-developer-associate/dynamodb_table_item_created.png "DynamoDB Table Item Created")

If add an item with a **duplicate primary key**, you will get an error:

![DynamoDB Table Item Error](/assets/aws-certified-developer-associate/dynamodb_table_item_error.png "DynamoDB Table Item Error")

Note how the attributes in this second item are not the same as the first one. Once you save it, the **missing attributes** will be added as `null` with no error:

![DynamoDB Table Item Null Attributes](/assets/aws-certified-developer-associate/dynamodb_table_item_null_attributes.png "DynamoDB Table Item Null Attributes")

### 19.3.3 Creating Tables and Items in DynamoDB with Sort Key

Create a new table `UserPosts` with partition key `user_id` and **sort key** `post_ts` (post timestamp) to store users' posts:

![DynamoDB Table UserPosts](/assets/aws-certified-developer-associate/dynamodb_table_userposts.png "DynamoDB Table UserPosts")

The sort key appears in the table's overview:

![DynamoDB Table UserPosts Overview](/assets/aws-certified-developer-associate/dynamodb_table_userposts_overview.png "DynamoDB Table UserPosts Overview")

Then, create new items and see that you will need to **enter both the partition key and the sort key**:

![DynamoDB Table UserPosts Create Item](/assets/aws-certified-developer-associate/dynamodb_table_userposts_create_item.png "DynamoDB Table UserPosts Create Item")

Keep creating items with the same partition key and different sort keys to see how they are **grouped by partition key and sorted by sort key**:

![DynamoDB Table UserPosts Items](/assets/aws-certified-developer-associate/dynamodb_table_userposts_items.png "DynamoDB Table UserPosts Items")

## 19.4 Read and Write Capacity Modes

They are the settings that **control how you manage your tables' capacity in terms of read/write throughput**.

There are two capacity modes and you can switch between them once every 24 hours.

**Provisioned mode (default)**: you specify the number of reads/writes per second.
- You need to **plan capacity beforehand** because you pay for the provisioned capacity even if you are not using it.
- Pay for provisioned read/write capacity units.

**On-demand mode**: read/writes automatically scale up/down based on your workloads.
- **No capacity planning** needed because DynamoDB accommodates your workloads.
- Pay for what you use but it is more expensive.

## 19.5 Provisioned Capacity Mode

Tables must have provisioned read and write capacity units:
- **Read capacity unit (RCU)**: throughput for reads.
- **Write capacity unit (WCU)**: throughput for writes.

You have the **option to setup auto-scaling of throughput to meet demand** and can also use **burst capacity to exceed the provisioned capacity temporarily**.
-  If burst capacity has been consumed, you will get the `ProvisionedThroughputExceededException` exception.
- In case you get this exception, you can retry the request with exponential backoff.

### 19.5.1 Write Capacity Units (WCU)

**One WCU represents one write per second for an item up to 1 KB in size**. If the **items are larger than 1 KB, more WCUs are consumed**.

A few examples:
- We write 10 items per second, with item size 2 KB => we need 20 WCUs because `10 * (2 KB / 1 KB) = 20`.
- We write 6 items per second, with item size 4.5 KB => we need 30 WCUs because `6 * (5 KB / 1 KB) = 30`, 4.5 gets rounded to the upper KB, so 5 KB.
- We write 120 items per minute, with item size 2 KB => we need 4 WCUs because `(120 / 60) * (2 KB / 1 KB) = 4`.

### 19.5.2 Strongly Consistent Reads vs Eventually Consistent Reads

With DynamoDB, you have an offering that behind the scenes is a distributed system:

![DynamoDB Behind the Scenes](/assets/aws-certified-developer-associate/dynamodb_behind_the_scenes.png "DynamoDB Behind the Scenes")

In such a system, you have to make a choice between:
- **Eventually consistent read (default)**: if we read just after a write, it is possible that we will get stale data because of replication.
- **Strongly consistent read**: if we read just after a write, we will get the correct data for sure.
    - For strongly consistent reads, you need to specify `ConsistentRead: true` in the API calls (e.g., `GetItem`, `BatchGetItem`, `Query`, `Scan`).
    - Strongly consistent reads **consume twice read capacity units**.
    - Strongly consistent reads **may have higher latency**.

The **choice between the two impacts DynamoDB read capacity units**.

### 19.5.3 Read Capacity Units (RCU)

**One RCU represents one strongly consistent read per second or two eventually consistent reads per second for an item up to 4KB in size**. If the **items are larger than 4 KB, more RCUs are consumed**, and the number of RCUs is rounded up to the nearest 4 KB.

A few examples:
- We do 10 strongly consistent reads per second, with item size 4 KB => we need `10 * (4 KB / 4 KB) = 10 RCUs`.
- We do 16 eventually consistent reads per second, with item size 12 KB => we need `(16 / 2) * (12 KB / 4 KB) = 24 RCUs`.
- We do 10 strongly consistent reads per second, with item size 6 KB => we need `10 * (8 KB / 4 KB) = 20 RCUs`, 6 KB gets rounded to the upper 4 KB, so 8 KB.

### 19.5.4 DynamoDB Partitions Internals

DynamoDB is made of tables, **each table has partitions, and partitions are copies of your data that live on different servers.**
- Data is distributed across partitions based on the partition key.
- Partition keys go through a hashing algorithm to know to which partition the data should go.

![DynamoDB Partitions](/assets/aws-certified-developer-associate/dynamodb_partitions.png "DynamoDB Partitions")

To compute the number of partitions you need to know a formula (which is not needed for the exam, as opposed to the RSU/WSU formulas that are needed for the exame):
- Number of partitions by capacity: `(RCUs / 3000) + (WCUs / 1000)`.
- Number of partitions by size: `(total data size / 10 GB)`.
- Number of partitions: `ceil(max(number of partitions by capacity + number of partitions by size))`.

**WCUs and RCUs are spread evenly across partitions**.

### 19.5.5 DynamoDB Throttling

If you exceed provisioned RCUs or WCUs at partition level, you get `ProvisionedThroughputExceededException`.

There are different possible **reasons for throttling**:
- **Hot keys**: one partition key is being read too many times (e.g., popular item).
- **Hot partitions**: one partition is being read too many times (e.g., popular partition).
- **Very large items**: RCU and WCU depends on size of items, so very large items can cause consumption of too many RCUs/WCUs.

Possible **solutions to avoid throttling**:
- Exponential backoff when exception is encountered (already done in SDK).
- Distribute partition keys as much as possible by choosing a good partition key.
- If RCU issue (e.g., hot partition), you can use *DynamoDB Accelerator* (DAX) to cache reads.
