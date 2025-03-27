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
