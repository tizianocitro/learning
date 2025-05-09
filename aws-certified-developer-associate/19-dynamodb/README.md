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

DynamoDB is made of **tables**, each having a **primary key** (section [19.2.1 Primary Keys](#1921-primary-keys)) that must be decided at creation time.

Each table can have an infinite number of **items** (also called rows), and each item has **attributes** that can be aded over time or be missing (null).
- **Maximum size of an item is 400KB**.
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

## 19.6 On-Demand Capacity Mode

Read/writes automatically scale up/down with your workloads, so no capacity planning is needed (no WCUs/RCUs). You get **unlimited WCUs/RCUs, no throttle, but it is more expensive**, around 2.5x more expensive than provisioned capacity.

You are charged for reads/writes that you use in terms of:
- **Read request units (RRUs)**: throughput for reads (same as RCU).
- **Write request units (WRUs)**: throughput for writes (same as WCU).

Use cases:
- Unknown workloads.
- Unpredictable application traffic.
- ...

## 19.7 Configuring RCUs and WCUs

To configure RCUs and WCUs, you can go to the table's details and search for the *Additional Settings* tab:

![DynamoDB Table Additional Settings](/assets/aws-certified-developer-associate/dynamodb_table_additional_settings.png "DynamoDB Table Additional Settings")

If you click on *Edit*, you can change the capacity settings (in this case, we are using on-demand mode):

![DynamoDB Table Edit Capacity](/assets/aws-certified-developer-associate/dynamodb_table_edit_capacity.png "DynamoDB Table Edit Capacity")

But we can **switch to provisioned mode**:

![DynamoDB Table Edit Capacity Provisioned](/assets/aws-certified-developer-associate/dynamodb_table_edit_capacity_provisioned.png "DynamoDB Table Edit Capacity Provisioned")

Set the **capacity settings** via the capacity calculator that also shows the **cost estimation**. In this settings, you also indicate the **read/write consistency**:

![DynamoDB Table Set Capacity](/assets/aws-certified-developer-associate/dynamodb_table_set_capacity.png "DynamoDB Table Set Capacity")

Next is to set **table capacity**, in this case **without auto-scaling**, so you need to **set the provisioned capacity units**:

![DynamoDB Table Set Table Capacity No Auto-Scaling](/assets/aws-certified-developer-associate/dynamodb_table_set_table_capacity_no_auto_scaling.png "DynamoDB Table Set Table Capacity No Auto-Scaling")

Or with **auto-scaling**, where you need to set the **minimum and maximum capacity units**, and the **target utilization**, which indicates that DynamoDB will scale to max capacity if the utilization is on average this value. However, if you are on low utilization, it will scale down up to but never lower than the minimum capacity.

![DynamoDB Table Set Table Capacity Auto-Scaling](/assets/aws-certified-developer-associate/dynamodb_table_set_table_capacity_auto_scaling.png "DynamoDB Table Set Table Capacity Auto-Scaling")

Finally, you can see the cost estimation for the settings you chose and update the table.

In the same tab, you can see **auto-scaling activities**:

![DynamoDB Table Auto-Scaling Activities](/assets/aws-certified-developer-associate/dynamodb_table_auto_scaling_activities.png "DynamoDB Table Auto-Scaling Activities")

## 19.8 DynamoDB Basic Operations APIs

### 19.8.1 Write APIs

The following APIs are important to know for the exam:

| API | Description |
| --- | ----------- |
| `PutItem` | **Creates a new item or replaces an old item** with a new item (same primary key). |
| `UpdateItem` | **Modifies an existing item's attributes or adds a new item if it does not exist**. It can be used to implement *atomic counters*: numeric attributes that are unconditionally incremented. |
| `ConditionalWrite` | **Writes/updates/deletes an item if a condition is met**, otherwise returns an error. It helps with concurrent access to items and has no performance impact. |

### 19.8.2 Read APIs

The following APIs are important to know for the exam:
- `GetItem`: retrieves a **single item by primary key** (primary key can be HASH or HASH + RANGE).
    - It performs eventually consistent reads by default.
    - It provides the option to use strongly consistent reads but will consume more RCUs and might take longer.
    - A `ProjectionExpression` can be specified to retrieve only certain attributes.

---

- `Query`: returns **items based on a key condition expression** (partition key and sort key) **and filter expression**.
    - `KeyConditionExpression`:
        - Partition key value is required and must be `=` operator.
        - Sort key value is optional (`=`,`<`, `<=`, `>`, `>=,` `Between`, `Begins with`, `Equals to`).
    - `FilterExpression`:
        - Additional filtering after the query operation (client-side), before data is returned.
        - Usable only with non-key attributes (does not allow HASH or RANGE attributes).
    - It returns the number of items specified in `Limit` or up to 1 MB of data, so either you reach the limit or the 1 MB.
    - It provides pagination on the results.
    - It can query a table, a local secondary index, or a global secondary index (more later).
    - In the console, you can use the query wizard to help you build the query:
    ![DynamoDB Query Wizard](/assets/aws-certified-developer-associate/dynamodb_query_wizard.png "DynamoDB Query Wizard")
---

- `Scan`: **reads the entire table** and then filter out data clinet-sider, so inefficient.
    - Scans return up to 1 MB of data and can be paginated, so use pagination to keep on reading.
    - It consumes a lot or RCU because it reads the entire table.
    - Limit the impact on other operations by using `Limit` or reduce the size of the result and pause.
    - For faster performance, use `Parallel Scan`, which uses **multiple workers to scan multiple data segments at the same time**.
        - It increases the throughput but also the consumed RCU.
        - Limit the impact of parallel scans just like you would for scans with `Limit`.
    - It can use `ProjectionExpression` and `FilterExpression` with no impact on RCU.
    - You can easily do a scan from the console:
    ![DynamoDB Scan Wizard](/assets/aws-certified-developer-associate/dynamodb_scan_wizard.png "DynamoDB Scan Wizard")

### 19.8.3 Delete APIs

The following APIs are important to know for the exam.

| API | Description |
| --- | ----------- |
| `DeleteItem` | **Deletes a single item by primary key** with the option to perform a conditional delete. |
| `DeleteTable` | **Deletes a table** and all its items. It provides a much quicker deletion than calling `DeleteItem` on all items. This is **important for the exam in case you are asked to delete all items**. |

### 19.8.4 Batch APIs

Batch operations allow you to save in latency by reducing the number of API calls.
- Batches are processed in parallel for better efficiency.
- Part of a batch can fail and, in that case, you need to try again for the failed items.

The following APIs are important to know for the exam:

- `BatchWriteItem`: **performs up to 25 `PutItem` and/or `DeleteItem` in one call**.
    - Up to 16 MB of data written and up to 400 KB of data per item.
    - It cannot update items (`UpdateItem`), only create or replace (`PutItem`) and delete (`DeleteItem`).
    - `UnprocessedItems` is used to return failed items to retry them with exponential backoff or add WCU.

- `BatchGetItem`: **retrieves multiple items from multiple tables** in a single API call.
    - Up to 100 items and up to 16 MB of data.
    - Items are retrieved in parallel to minimize latency.
    - `UnprocessedKeys` for failed items operations to retry them with exponential backoff or add RCU.

## 19.9 DynamoDB PartiQL Query Language

PartiQL is a **SQL-compatible query language for DynamoDB** that allows you to select, insert, update, and delete items using SQL-like syntax.
- You can run queries across multiple tables.
- It **does not support joins**.
- It supports batch operations.

You can run PartiQL queries from:
- DynamoDB APIs.
- NoSQL Workbench for DynamoDB.
- CLI.
- SDK.
- Console.

You can **use PartiQL to query both tables and indexes**:

```sql
-- Query a table
SELECT 'user_id', 'game_ts'
FROM 'users'
WHERE 'user_id' = '123' AND
      'game_ts' BETWEEN '2021-01-01' AND '2021-01-31'

-- Query an index
SELECT *
FROM 'users'.'game_id-index'
WHERE 'user_id' = '123' AND 'game_id' = '456'
```

In the console, you can use the **PartiQL editor to run queries**:

![DynamoDB PartiQL Editor](/assets/aws-certified-developer-associate/dynamodb_partiql_editor.png "DynamoDB PartiQL Editor")

## 19.10 Conditional Writes in DynamoDB

They are a feature for `PutItem`, `UpdateItem`, `DeleteItem`, and `BatchWriteItem`. The idea is that you can **specify a condition expression to determine which items should be modified**. You can use the following conditions:
- `attribute_exists`: checks if the attribute exists in the item.
- `attribute_not_exists`: checks if the attribute does not exist in the item.
- `attribute_type`: checks if the attribute exists and is of the specified type.
- `contains` (for string): checks if the attribute contains a substring.
- `begins_with` (for string): checks if the attribute begins with a substring.
- `size` (for string): checks if the attribute is of a certain size.
- `IN`: checks if the attribute is in the list of values, e.g., `ProductCategory IN ['Book', 'Movie']`.
- `Between`: checks if the attribute is between two values, e.g., `Price BETWEEN 100 AND 200`.

The difference between filter expressions and condition expressions is that **filter expressions are for read operations** (`Query`, `Scan`) and **condition expressions are for write operations** (`PutItem`, `UpdateItem`, `DeleteItem`, `BatchWriteItem`).

### 19.10.1 Conditional Writes to Not Overwrite Items

You can use `attribute_not_exists(partition_key)` to **prevent overwriting an item with the same partition key**. Using this condition, you can ensure that the item is created if the partition key does not exist, otherwise the operation will fail.

The same can be done with the combination of partition key and sort key, using `attribute_not_exists(partition_key) AND attribute_not_exists(sort_key)`.

### 19.10.2 Update Items with Conditional Writes

To update an item with a condition expression, you can use the following command:

```bash
aws dynamodb update-item \
    --table-name ProductCatalog \
    --key '{"Id": {"N": "456"}}' \
    --update-expression "SET Price = Price - :discount" \
    --condition-expression "Price > :limit" \
    --expression-attribute-values file://values.json
```

Where `values.json` contains:

```json
{
    ":discount": {"N": "150"},
    ":limit": {"N": "500"}
}
```

So, if we have the following item:

```json
{
    "Id": {"N": "456"},
    "Price": {"N": "650"}
}
```

the update will result in the following item:

```json
{
    "Id": {"N": "456"},
    "Price": {"N": "500"}
}
```

### 19.10.3 Delete Items with Conditional Writes

First example uses the `attribute_not_exists` condition, which will delete the item only if the attribute does not exist (no value). For example, the following command will delete the item if the `Price` is not set:

```bash
aws dynamodb delete-item \
    --table-name ProductCatalog \
    --key '{"Id": {"N": "456"}}' \
    --condition-expression "attribute_not_exists(Price)"
```

Second example uses the `attribute_exists` condition, which will delete the item only if the attribute exists (has a value). For example, the following command will delete the item if the it has at least a review with 1 star:

```bash
aws dynamodb delete-item \
    --table-name ProductCatalog \
    --key '{"Id": {"N": "456"}}' \
    --condition-expression "attribute_exists(ProductReviews.OneStar)"
```

### 19.10.4 Complex Conditional Writes

The following command deletes an item if the `Price` is between 100 and 200 and the `ProductCategory` is `Book` or `Movie`:

```bash
aws dynamodb delete-item \
    --table-name ProductCatalog \
    --key '{"Id": {"N": "456"}}' \
    --condition-expression \
    "Price BETWEEN :low AND :high AND ProductCategory IN (:cat1, :cat2)" \
    --expression-attribute-values file://values.json
```

Where `values.json` contains:

```json
{
    ":low": {"N": "500"},
    ":high": {"N": "600"},
    ":cat1": {"S": "Book"},
    ":cat2": {"S": "Movie"}
}
```

### 19.10.5 String Comparisons in Conditional Writes

The following command deletes an item if the attribute `Src` begins with `http://`:

```bash
aws dynamodb delete-item \
    --table-name ProductCatalog \
    --key '{"Id": {"N": "456"}}' \
    --condition-expression "begins_with(Src, :prefix)" \
    --expression-attribute-values '{":prefix": {"S": "http://"}}'
```

## 19.11 DynamoDB Indexes

There are two types of indexes in DynamoDB: local secondary indexes and global secondary indexes.

### 19.11.1 Local Secondary Indexes (LSI)

An **LSI provides an alternative (additional) sort key for a table**.
- The sort key consists of one scalar attribute (`String`, `Number`, or `Binary`).
- Up to 5 LSIs per table.
- They **must be defined at table creation time**.

LSIs can contain some or all of the attributes that are projected into the index (KEYS_ONLY, INCLUDE, ALL), so the attributes that are not projected into the index are fetched from the base table.

For instance, consider the following table:

![DynamoDB Table LSI](/assets/aws-certified-developer-associate/dynamodb_table_lsi.png "DynamoDB Table LSI")

We have a primary key `User_ID` and a sort key `Game_ID`, which we can use to query the table. However, we may want to query using the `Game_TS` and the  `User_ID` because we want to get all games played by a user in a certain time frame. We can create an LSI with the sort key `Game_TS` to achieve this, so that we do not need to scan the table and then filter it, which is inefficient.

### 19.11.2 Global Secondary Indexes (GSI)

A **GSI provides an alternative (additional) primary key (either HASH or HASH + RANGE)** for a table.
- Used to speed up queries on non-key attributes.
- The index key consists of scalar attributes (`String`, `Number`, or `Binary`)
- You can think of it like a new table with the same data but with a different primary key, so you need to provision RCUs and WCUs for the index.
- They **can be created/modified at table creation time or after**.

GSIs can contain some or all of the attributes that are projected into the index (KEYS_ONLY, INCLUDE, ALL), so the attributes that are not projected into the index are fetched from the base table.

For instance, consider the following table:

![DynamoDB Table GSI](/assets/aws-certified-developer-associate/dynamodb_table_gsi.png "DynamoDB Table GSI")

On this table, we can query only by `User_ID` and `Game_ID`. However, we may want to query by `Game_ID` and `Game_TS` to get all games played in a certain time frame. With the table above, we can only perform this query by scanning the table and then filtering, which is inefficient. To solve this, we can create a GSI with the primary key `Game_ID` and `Game_TS` which looks like a new table with the same data but with a different primary key:

![DynamoDB Table GSI Created](/assets/aws-certified-developer-associate/dynamodb_table_gsi_created.png "DynamoDB Table GSI Created")

### 19.11.3 Indexes and Throttling

- Global secondary indexes:
    - **If the writes are throttled on the GSI, then the main table will be throttled, even if the WCU on the main tables are fine**.
    - Choose GSI partition key carefully.
    - Assign WCU capacity carefully.
    - This is a **caviet to be aware of because the exam may ask you about this**: GSIs use an independent amount of RCUs/WCUs and if they are throttled due to insufficient capacity, then the main table will also be throttled.
- Local secondary indexes: they use the WCUs and RCUs of the main table, so there are no special throttling consideration.

## 19.12 Creating Indexes in DynamoDB

During table creation, scroll to **secondary indexes** where you can create both LSIs and GSIs:

![DynamoDB Table Create Indexes](/assets/aws-certified-developer-associate/dynamodb_table_create_indexes.png "DynamoDB Table Create Indexes")

You can see created indexes in the table's details at any time:

![DynamoDB Table Indexes](/assets/aws-certified-developer-associate/dynamodb_table_indexes.png "DynamoDB Table Indexes")

From the image above, you can confirm that LSIs cannot be modified after table creation, while GSIs can be modified/created.

### 19.12.1 Creating a Local Secondary Index

To **create a LSI**, you need to specify the **index name** and the **sort key**:

![DynamoDB Table Create LSI](/assets/aws-certified-developer-associate/dynamodb_table_create_lsi.png "DynamoDB Table Create LSI")

And it will appear in the table's secondary indexes:

![DynamoDB Table LSI Created](/assets/aws-certified-developer-associate/dynamodb_table_lsi_created.png "DynamoDB Table LSI Created")

LSIs can be created only at table creation time and cannot be modified after table creation.

### 19.12.2 Creating a Global Secondary Index

To **create a GSI**, you need to specify the **index name**, the **partition key**, and the **sort key**:

![DynamoDB Table Create GSI](/assets/aws-certified-developer-associate/dynamodb_table_create_gsi.png "DynamoDB Table Create GSI")

For GSIs, you also need to specify the **index capacity** (which you can also copy from the main table):

![DynamoDB Table GSI Capacity](/assets/aws-certified-developer-associate/dynamodb_table_gsi_capacity.png "DynamoDB Table GSI Capacity")

### 19.12.3 Querying Secondary Indexes

If you into the table's details, you **can query the table or the indexes**:

![DynamoDB Table Query Indexes](/assets/aws-certified-developer-associate/dynamodb_table_query_indexes.png "DynamoDB Table Query Indexes")

In this case, the `game_id-index` is the LSI we created, and we can query it by the sort key `game_id` and the partition key `user_id`. Instead, if we query the table, we can only use the partition key `user_id` and the sort key `game_ts`.

However, GSIs will appear in the dropwdown list of indexes to query, so you can query them as well. In that case, you can query by the partition key and the sort key of the GSI.

## 19.13 DynamoDB Optimistic Locking

DynamoDB has a feature called conditional writes, which provides a way to ensure an item has not changed before you update/delete it.

This **strategy is called optimistic locking** and the idea behind it is that you read an item, then you update it, but before you update it, you check if the item has not changed since you read it. If it has changed, you do not update it because you do not have the latest version of the item anymore. So, you need to get the latest version of the item and try again.

![DynamoDB Optimistic Locking](/assets/aws-certified-developer-associate/dynamodb_optimistic_locking.png "DynamoDB Optimistic Locking")

## 19.14 DynamoDB Accelerator (DAX)

DAX is a **fully managed, highly available, in-memory cache for DynamoDB** that delivers up to 10x performance improvement by caching the most frequently used data, thus offloading the heavy reads on hot keys of your DynamoDB table and preventing the `ProvisionedThroughputExceededException` exception.

DAX:
- Provides microseconds latency for cached reads and queries because it **stores the most popular items in memory**.
- **Solves the hot key problem** when you have too many reads on the same partition key.
- Is fully secure with encryption at rest with KMS, VPC, IAM, CloudTrail, CloudWatch, etc.

DAX **does not require application logic modification** because it is compatible with the DynamoDB API. You just need to point your application to the DAX cluster you provisioned.
- 5 minutes TTL for cache by default.
- Up to 10 nodes in the cluster with a recommended 3 nodes in 3 different AZs for production.

![DynamoDB DAX](/assets/aws-certified-developer-associate/dynamodb_dax.png "DynamoDB DAX")

DAX has no free tier, so you pay for the nodes you provision.

### 19.14.1 DAX vs ElastiCache

**DAX and ElastiCache can be used in conjunction**:
- DAX is to cache individual items or for queries and scan.
- Elisticache allows you to cache results of the results of whatever logic your application has, so not only for DynamoDB and its queries.

### 19.14.2 Creating a DAX Cluster

Go to *Clusters* in the DAX console and click on *Create Cluster*:

![DynamoDB DAX Create Cluster](/assets/aws-certified-developer-associate/dynamodb_dax_create_cluster.png "DynamoDB DAX Create Cluster")

Give the cluster a **name** (e.g., `demodax`) and optionally a **description**:

![DynamoDB DAX Cluster Name](/assets/aws-certified-developer-associate/dynamodb_dax_cluster_name.png "DynamoDB DAX Cluster Name")

Next is to configure the **node families** by choosing betweenn 3 options:
- `t-type`: for burstable workloads.
- `r-type`: for memory-optimized workloads.
- `All families`: for a mix of both.

![DynamoDB DAX Node Family](/assets/aws-certified-developer-associate/dynamodb_dax_node_family.png "DynamoDB DAX Node Family")

Then, select the **node type**:

![DynamoDB DAX Node Type](/assets/aws-certified-developer-associate/dynamodb_dax_node_type.png "DynamoDB DAX Node Type")

And the **number of nodes** (cluster size between 1 and 11):

![DynamoDB DAX Number of Nodes](/assets/aws-certified-developer-associate/dynamodb_dax_number_of_nodes.png "DynamoDB DAX Number of Nodes")

If you select less than 3 nodes, you will get a warning because it is not recommended for production due to reduced availability.

Next configure the **subnets and VPC**:

![DynamoDB DAX Subnets](/assets/aws-certified-developer-associate/dynamodb_dax_subnets.png "DynamoDB DAX Subnets")

![DynamoDB DAX Subnets 2](/assets/aws-certified-developer-associate/dynamodb_dax_subnets_2.png "DynamoDB DAX Subnets 2")

Configure **access control via security group**:

![DynamoDB DAX Security Group](/assets/aws-certified-developer-associate/dynamodb_dax_security_group.png "DynamoDB DAX Security Group")

And **AZ allocation**:

![DynamoDB DAX AZ Allocation](/assets/aws-certified-developer-associate/dynamodb_dax_az_allocation.png "DynamoDB DAX AZ Allocation")

Next thing is **security**. Start with IAM permissions, you need to create a new role or use an existing one to **allow DAX to access DynamoDB**:

![DynamoDB DAX Security](/assets/aws-certified-developer-associate/dynamodb_dax_security.png "DynamoDB DAX Security")

![DynamoDB DAX Security 2](/assets/aws-certified-developer-associate/dynamodb_dax_security_2.png "DynamoDB DAX Security 2")

And finish with **encryption** configuration:

![DynamoDB DAX Encryption](/assets/aws-certified-developer-associate/dynamodb_dax_encryption.png "DynamoDB DAX Encryption")

You have also some **advanced settings** to configure. The first is the **parameter group which defines settings like TTL and for items and queries**. You can set the following parameters in a parameter group:

![DynamoDB DAX Parameter Group](/assets/aws-certified-developer-associate/dynamodb_dax_parameter_group.png "DynamoDB DAX Parameter Group")

And then set it in the cluster configuration:

![DynamoDB DAX Parameter Group Set](/assets/aws-certified-developer-associate/dynamodb_dax_parameter_group_set.png "DynamoDB DAX Parameter Group Set")

Finally, you can set **tags** and the **maintenance window** for software updates and patches:

![DynamoDB DAX Maintenance Window](/assets/aws-certified-developer-associate/dynamodb_dax_maintenance_window.png "DynamoDB DAX Maintenance Window")

After the cluster is created, you can see it in the **list of clusters**:

![DynamoDB DAX Cluster List](/assets/aws-certified-developer-associate/dynamodb_dax_cluster_list.png "DynamoDB DAX Cluster List")

And by clicking on it, you can see the **details of the cluster** where you have crucial information like the **cluster endpoint that you need to point your applications to**:

![DynamoDB DAX Cluster Details](/assets/aws-certified-developer-associate/dynamodb_dax_cluster_details.png "DynamoDB DAX Cluster Details")

Or the **nodes in the cluster**:

![DynamoDB DAX Cluster Nodes](/assets/aws-certified-developer-associate/dynamodb_dax_cluster_nodes.png "DynamoDB DAX Cluster Nodes")

Which you can also add:

![DynamoDB DAX Cluster Add Nodes](/assets/aws-certified-developer-associate/dynamodb_dax_cluster_add_nodes.png "DynamoDB DAX Cluster Add Nodes")

But you can also check the events of your database and monitor metrics.

## 19.15 DynamoDB Streams

They are **ordered stream of item-level modifications (create/update/delete) in a table**. Stream records can be:
- Sent to Kinesis Data Streams.
- Processes by Kinesis Client Library applications.
- Processes by Lambda functions.

**Data retention is of up to 24 hours**, so you need to process them in this time frame or use Lambda or Kinesis to store them for longer.

Use cases:
- React to changes in real-time (e.g., welcome email to users).
- Analytics.
- Insert into derivative tables.
- Insert into OpenSearch.
- Implement cross-region replication.

The image below explain what you can do with DynamoDB Streams:

![DynamoDB Streams](/assets/aws-certified-developer-associate/dynamodb_streams.png "DynamoDB Streams")

### 19.15.1 Features of DynamoDB Streams

You have the ability to **choose the information that will be written to the stream**:
- `KEYS_ONLY`: only the key attributes of the modified item.
- `NEW_IMAGE`: the entire item, as it appears after it was modified.
- `OLD_IMAGE`: the entire item, as it appeared before it was modified.
- `NEW_AND_OLD_IMAGES`: both the new and the old images of the item.

**DynamoDB streams are made of shards**, just like Kinesis Data Streams.
- You do not provision shards, this is automated by AWS.

**Records are not retroactively populated in a stream after enabling it** but only after the stream is enabled. The **exam tries to trick you with this**.

### 19.15.2 DynamoDB Streams Integration with Lambda

You need to **define an event source mapping to read from a stream** and **ensure that the Lambda function has the appropriate permissions to pull from the stream**.
- The Lambda function is invoked synchronously.

![DynamoDB Streams Lambda](/assets/aws-certified-developer-associate/dynamodb_streams_lambda.png "DynamoDB Streams Lambda")

## 19.16 Using DynamoDB Streams

Go to a table's details and find the *Exports and Streams* tab where you will find the following:

![DynamoDB Table Streams](/assets/aws-certified-developer-associate/dynamodb_table_streams.png "DynamoDB Table Streams")

In this case, we will **enable** the DynamoDB stream by cliking on *Enable* and selecting the `NEW_AND_OLD_IMAGES` option to see as much information as possible in stream records:

![DynamoDB Table Streams Enable](/assets/aws-certified-developer-associate/dynamodb_table_streams_enable.png "DynamoDB Table Streams Enable")

Enable the stream and this is what you will see:

![DynamoDB Table Streams Enabled](/assets/aws-certified-developer-associate/dynamodb_table_streams_enabled.png "DynamoDB Table Streams Enabled")

Now we need to **create a trigger for the stream (a Lambda function)**, so we click on *Create Trigger*:

![DynamoDB Table Streams Create Trigger](/assets/aws-certified-developer-associate/dynamodb_table_streams_create_trigger.png "DynamoDB Table Streams Create Trigger")

First, **create a Lambda function to use as trigger** using of the blueprints for DynamoDB (the chosen blueprint is `dynamodb-process-stream-python3` that just logs the stream records):

![DynamoDB Table Streams Configure Trigger](/assets/aws-certified-developer-associate/dynamodb_table_streams_configure_trigger.png "DynamoDB Table Streams Configure Trigger")

And **set the DynamoDB trigger in the function configuration** where you specify the DynamoDB table, the batch size, and the starting position:

![DynamoDB Table Streams Lambda Trigger](/assets/aws-certified-developer-associate/dynamodb_table_streams_lambda_trigger.png "DynamoDB Table Streams Lambda Trigger")

**Ensure the function has necessary permissions** to read from DynamoDB. Then, go back to the table's *Exports and Streams* tab and actually on *Create Trigger* to configure the trigger:

![DynamoDB Table Streams Configure Trigger 2](/assets/aws-certified-developer-associate/dynamodb_table_streams_configure_trigger_2.png "DynamoDB Table Streams Configure Trigger 2")

And it will appear in the list of triggers:

![DynamoDB Table Streams Trigger Created](/assets/aws-certified-developer-associate/dynamodb_table_streams_trigger_created.png "DynamoDB Table Streams Trigger Created")

And you can also see it as a trigger in the Lambda function:

![DynamoDB Table Streams Lambda Trigger 2](/assets/aws-certified-developer-associate/dynamodb_table_streams_lambda_trigger_2.png "DynamoDB Table Streams Lambda Trigger 2")

To **test this trigger**, just go to the DynamoDB table and create a new item. To see that the function is actually being triggered, go to the CloudWatch logs and see the logs of the Lambda function where you will see the stream records being logged. For example:

![DynamoDB Table Streams Lambda Logs](/assets/aws-certified-developer-associate/dynamodb_table_streams_lambda_logs.png "DynamoDB Table Streams Lambda Logs")

The trigger can be **disabled** at any time in the function configuration but also in the table's details.

## 19.17 DynamoDB Time To Live (TTL)

DynamoDB TTL is a feature that allows you to **define a time period after which the items in a table are deleted**.
- A deletion due to TTL has no extra cost, so it does not consume WCUs.

The **TTL attribute must be a `Number` data type with a Unix Epoch timestamp value**.

There are **two processes: one that checks items to flag those that are expired and another that deletes the expired items**.
- Expired items are deleted within 48 hours of expiration, so not immediately.
- Expired items (that have not been deleted yet) appear in reads/queries/scans and if you want them to be excluded, you need to filter them out.
- Expired items are deleted from both LSIs and GSIs.

![DynamoDB TTL](/assets/aws-certified-developer-associate/dynamodb_ttl.png "DynamoDB TTL")

A delete operation for each expired item is inserted into the DynamoDB stream (if enabled), so you can use it to recover the data if needed or any other use case.

A few use cases:
- Reduce stored data by keeping only current data.
- Adhere to data retention policies defined by regulations.

### 19.17.1 Setting Up TTL on Tables

Create a table called `DemoTTL` and insert a few items with an `Number` attribute called `expire_on` (can be named differently) that will be used as the TTL attribute. For instance:

![DynamoDB Item with TTL Attribute](/assets/aws-certified-developer-associate/dynamodb_item_with_ttl_attribute.png "DynamoDB Item with TTL Attribute")

Go to the tables's details where you can see the *Time to Live (TTL)* setting, which is currently disabled:

![DynamoDB Table TTL](/assets/aws-certified-developer-associate/dynamodb_table_ttl.png "DynamoDB Table TTL")

Then, go to the *Additional Settins* tab and scroll to the *Time to Live (TTL)* setting where you can enable it by clicking on *Enable*. This will prompt you to enter a **TTL attribute name** (in this case `expire_on`):

![DynamoDB Table TTL Enable](/assets/aws-certified-developer-associate/dynamodb_table_ttl_enable.png "DynamoDB Table TTL Enable")

After doing so, you can **run a preview** to see which items will be deleted after specifying a time period:

![DynamoDB Table TTL Preview](/assets/aws-certified-developer-associate/dynamodb_table_ttl_preview.png "DynamoDB Table TTL Preview")

Finally, click on *Enable TTL* to enable the TTL feature on the table. And the TTL will now be enabled:

![DynamoDB Table TTL Enabled](/assets/aws-certified-developer-associate/dynamodb_table_ttl_enabled.png "DynamoDB Table TTL Enabled")

## 19.18 Good to Know Concepts and Features

A few **CLI options** that are may come up in the exam:
- `--projection-expression`: specify one or more attributes to retrieve instead of all attributes.
    ```bash
    aws dynamodb scan \
        --table-name UserPosts \
        --projection-expression "user_id, content"
    ```

- `--filter-expression`: filter the results of a query or scan operation.
    ```bash
    aws dynamodb scan \
        --table-name UserPosts \
        --filter-expression "user_id = :u" \
        --expression-attribute-values '{":u":{"S":"john123"}}'
    ```

General **CLI pagination options**:
- `--page-size`: specify that CLI retrieves the full list of items but with a larger number of API calls instead of one API call (default is 1000 items) to avoid timeouts.
    - If you have 1000 items and you set the page size to 100, the CLI will make 10 API calls to retrieve all items.
    ```bash
    # If you have 3 items, it will make 3 API calls
    aws dynamodb scan \
        --table-name UserPosts \
        --page-size 1
    ```

- `--max-items`: maximum number of items to show in the CLI (returns `NextToken`).
    ```bash
    # If you have 3 items, it will make 1 API call
    # and return only 1 item and the 'NextToken'
    aws dynamodb scan \
        --table-name UserPosts \
        --max-items 1
    ```

- `--starting-token`: specify the last `NextToken` to retrieve the next set of items.
    - It works in conjunction with `--max-items` to retrieve the next set of items.
    - If you do not receive the `NextToken`, it means that you have retrieved all items.
    ```bash
    # If you have 3 items, it will make 1 API call
    # and return only 1 item and the 'NextToken'
    aws dynamodb scan \
        --table-name UserPosts \
        --max-items 1

    # Then, use the 'NextToken' to retrieve the next set of items
    aws dynamodb scan \
        --table-name UserPosts \
        --starting-token "NextToken"
    ```

## 19.19 Transactions in DynamoDB

Transactions are **coordinated, all-or-nothing operations (add/update/delete) to multiple items across one or more tables**.
- Provide ACID (atomicity, consistency, isolation, and durability)

![DynamoDB Transactions](/assets/aws-certified-developer-associate/dynamodb_transactions.png "DynamoDB Transactions")

Transaction **can be applied to both read and write operations**.
- Transactions consumes 2x WCUs and RCUs because DynamoDB performs 2 operations for every item (`prepare` and `commit`).

You have **3 read modes**:
- **Eventual consistency (default)**: returned data might be stale due to replication lag, but it is more scalable and offers better performance.
- **Strong consistency**: ensures the latest committed write is always read but with slightly higher latency.
- **Transactional**: ensures strongly consistent reads when reading from multiple tables in a transaction.

You have **2 write modes**:
- **Standard (default)**: writes data without transactional guarantees.
- **Transactional**: ensures all writes succeed or all fail together and is used when multiple dependent updates need to be applied atomically.

Two operations that are important to know:
- `TransactGetItems`: performs one or more `GetItem` operations.
- `TransactWriteItems`: performs one or more `PutItem`, `UpdateItem`, and `DeleteItem` operations.

A few use cases:
- Financial transactions.
- Managing orders.
- Multiplayer games.

### 19.19.1 Transactions Capacity Computations

This is **very important for the exam**. Refer to the section [19.5 Provisioned Capacity Mode](#195-provisioned-capacity-mode) for more information about how to calculate the capacity.

Examples (* 2 is because of the prepare and commit):
- We perform 3 transactional writes per second with item size 5 KB => we need `[3 * (5 KB / 1 KB)] * 2 = 30 WCUs`.
- We perform 5 transaction reads per second, with item size 5 KB => we need `[5 * (5 KB / 4 KB)] * 2 = 25 RCUs`.

## 19.20 DynamoDB used as Session State Cache

It is common to **use DynamoDB to store session states**. For example, to store user sessions in a web application after a user logs in.

### 19.20.1 Comparison with Other Services

- **vs ElastiCache**: ElastiCache is fully in-memory but DynamoDB is serverless, with both being key/value stores.
    - If the exam asks for a session store that is an in-memory key/value store, you need ElastiCache.
    - Instead, if it asks for an auto scaling key/value store, you need DynamoDB.
- **vs EFS**: EFS must be attached to EC2 instances as a network drive, so if you need to share the session state across multiple instances, you need to use EFS.
- **vs EBS and Instance Store**: EBS and Instance Store can only be used for local caching, not shared caching because they are attached to a single instance.
- **vs S3**: S3 is higher latency and not meant for small objects, so it is not a good option for session state.

The **best 3 for storing session state are DynamoDB, ElastiCache, and EFS**. DynamoDB is the best option for a serverless architecture, while ElastiCache is the best option for a fully in-memory solution.

## 19.21 DynamoDB Write Sharding

Imagine we have a voting application with two candidates, candidate A and candidate B. If we use a `Candidate_ID` property as partition key, we will get only two partitions, which will generate issues like the hot partition problem.

A **strategy** to address the above problem and **get a better distribution of items across partitions** consists of **adding a suffix to the partition key**. For example, we can use a `Candidate_ID` property with a random suffix (e.g., `Candidate_ID-1`, `Candidate_ID-2`, etc.).

![Write Sharding](/assets/aws-certified-developer-associate/dynamodb_write_sharding.png "Write Sharding")

Two **methods for choosing the suffix**:
- Sharding using random suffix: for example, use a random number as a suffix.
- Sharding using calculated suffix: for example, use the hash of a property as a suffix (e.g., `Candidate_ID-<hash of Candidate_ID>`).

## 19.22 DynamoDB Write Types

There are different types of writes in DynamoDB. The one you need depends on the use case and the requirements of your application:
- **Concurrent writes**: when multiple users are writing to the same item at the same time but a write may overwrite another write.
    ![Concurrent Writes](/assets/aws-certified-developer-associate/dynamodb_concurrent_writes.png "Concurrent Writes")
- **Conditional writes**: when you want to update an item only if a certain condition is met.
    ![Conditional Writes](/assets/aws-certified-developer-associate/dynamodb_conditional_writes.png "Conditional Writes")
- **Atomic writes**: when you want to update multiple items at the same time but you want to ensure that all updates are applied without one overwriting the other.
    ![Atomic Writes](/assets/aws-certified-developer-associate/dynamodb_atomic_writes.png "Atomic Writes")
- **Batch writes**: perform writes in batches.
    ![Batch Writes](/assets/aws-certified-developer-associate/dynamodb_batch_writes.png "Batch Writes")

## 19.23 DynamoDB Integration with S3

### 19.23.1 Storing Large Objects

To store large objects in DynamoDB, you can use S3 to store the object and then store the S3 object URL in DynamoDB. This way, you can store large objects in S3 and keep the metadata in DynamoDB.

The following image provides an example of using DynamoDB to store products' images stored in S3:

![Storing Large Objects in DynamoDB](/assets/aws-certified-developer-associate/dynamodb_storing_large_objects.png "Storing Large Objects in DynamoDB")

### 19.23.2 Indexing S3 Objects Metadata

You can use DynamoDB to index S3 objects metadata. For example, you can store the metadata of S3 objects in DynamoDB and then use DynamoDB to query the metadata.

![DynamoDB Indexing S3 Objects Metadata](/assets/aws-certified-developer-associate/dynamodb_indexing_s3_objects_metadata.png "DynamoDB Indexing S3 Objects Metadata")
