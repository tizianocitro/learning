# 5 Relational Database Service, Aurora, and ElastiCache

## 5.1 Relational Database Service (RDS)

It is a **managed DB service for DB that use SQL as a query language**.

It allows you to create databases in the cloud that are managed by AWS:
- Postgres.
- MySQL.
- MariaDB.
- Oracle.
- Microsoft SQL Server.
- IBM DB2.
- Aurora (AWS Proprietary database).

### 5.1.1 Advantages of Using RDS over Deploying your own DB on EC2

- Automated provisioning of the DB.
- Automatic underlying OS patching.
- Continuous backups and restore to specific timestamp (**Point in Time Restore**).
- Monitoring dashboards.
- Read replicas for improved read performance.
- Multi AZ setup for disaster recovery.
- Maintenance windows for upgrades.
- Scaling capability (vertical and horizontal).
- Storage backed by the EBS service.

However, **you cannot SSH into the instances behind the RDS DB**.

### 5.1.2 RDS Storage Auto Scaling

**This is a very important feature that can come up in the exam**.

This feature helps you increase storage on your RDS DB instance dynamically. **When RDS detects you are running out of free database storage, it will automatically scale your storage**. You can avoid manually scaling your database storage.
- **Useful for applications with unpredictable workloads**.
- Supports all RDS database engines.

![RDS Storage Auto Scaling](/assets/aws-certified-developer-associate/rds_storage_auto_scaling.png "RDS Storage Auto Scaling")

You have to set **Maximum Storage Threshold** (maximum limit for DB storage) as you do not want it to scale indefinitely.

RDS automatically modifies storage if:
- Free storage is less than 10% of allocated storage.
- Low-storage lasts at least 5 minutes.
- 6 hours have passed since last modification.

## 5.2 RDS Read Replicas (for Read Scalability)

Read replicas help you scale your reads.

![RDS Read Replicas](/assets/aws-certified-developer-associate/rds_read_replicas.png "RDS Read Replicas")

You can have **up to 15 read replicas** and they **can be within AZ, cross AZ, or cross region**.

**Replication is ASYNC, so reads are eventually consistent**. When reading from a read replica you might read old data because the replica did not have the time to replicate the latest data yet.

**Replicas can be promoted to their own DB**, so that they can also process writes. However, doing so will take them out of the replication process with the main DB.

If you want to use read replicas, your applications must update the connection string to leverage the read replicas.

**Read replicas are used for SELECT only kind of statements**. You cannot use INSERT, UPDATE, and DELETE.

### 5.2.1 Read Replicas Use Case

You have a production database that is taking on normal load and want to run a reporting application to run some analytics. Running this reporting application on the production database would slow down the production database as it would have to process the analytics queries. Instead, you can create a read replica and run the reporting workload there. In this way, the production database is not affected.

![Read Replicas Use Case](/assets/aws-certified-developer-associate/read_replicas_use_case.png "Read Replicas Use Case")

### 5.2.2 Read Replicas Network Cost

In AWS there is a network cost when data goes from one AZ to another but there are some exceptions, particularly when using managed services.

**For RDS Read Replicas within the same region, you do not pay that fee**. Otherwise, if cross region, you would have to pay the fee.

![Read Replicas Network Cost](/assets/aws-certified-developer-associate/read_replicas_network_cost.png "Read Replicas Network Cost")

## 5.3 RDS Multi AZ (for Disaster Recovery)

It is essential **for the exam to understand the difference between Read Replicas and Multi AZ**.

**Multi AZ allows you to have an exact copy of your production database in another AZ**. AWS handles the replication for you, so when your production database is written to, this write will be synchronously replicated to the standby database. So, it offers **SYNC replication**.

You also use **one DNS name for automatic application failover to the standby instance** in case of a failure without manual intervention (the application will automatically connect to the standdy DB). Having one DNS name for all the databases means that you do not have to worry about updating the connection string in your application.

So, this feature **increases availability and provides fault tolerance** thanks to failover in case of loss of AZ, loss of network, instance or storage failure. **The standby DB is not for scaling, it is only for failover**.

![RDS Multi AZ](/assets/aws-certified-developer-associate/rds_multi_az.png "RDS Multi AZ")
**The read replicas can be setup as Multi AZ for disaster recovery, and this is a common exam question**.

### 5.3.1 From Single AZ to Multi AZ

It is a **zero downtime operation** (no need to stop the DB). Just click on modify for the database and enable Multi AZ.

The following happens internally:
- A snapshot is taken.
- A new standby DB is restored from the snapshot in a new AZ.
- When the standby DB is active, synchronization is established between the two databases

![From Single AZ to Multi AZ](/assets/aws-certified-developer-associate/from_single_az_to_multi_az.png "From Single AZ to Multi AZ")

## 5.4 Creating RDS DB

Go to *Databases* under *RDS* in the AWS Management Console and click on *Create Database*.

Choose the **database engine** and the its version as you need.

![Choose Database Engine](/assets/aws-certified-developer-associate/choose_database_engine.png "Choose Database Engine")

Then, you have to select the **deployment option for availability and durability**. You can choose between:

![Deployment Options](/assets/aws-certified-developer-associate/deployment_options.png "Deployment Options")

Next are some settings (ID, credentials) about the databases:

![Database Instance Settings](/assets/aws-certified-developer-associate/database_instance_settings.png "Database Instance Settings")

And the **database instance class**:

![Database Instance Class](/assets/aws-certified-developer-associate/database_instance_class.png "Database Instance Class")

For the **storage type**, you want to use the PIOPS (Provisioned IOPS) io1 for production workloads: 

![Storage Type](/assets/aws-certified-developer-associate/storage_type.png "Storage Type")

You can also enable **storage autoscaling** for when you are running out of storage:

![Storage Autoscaling](/assets/aws-certified-developer-associate/storage_autoscaling.png "Storage Autoscaling")

**Network settings**:
- VPC, subnet, and AZ.
- Security group within the VPC.
- Database port (as additional configuration).
- Make the database publicly accessible via a public IP (not recommended for production).

Next is **database authentication** (*Oracle* does not support IAM authentication):

![Database Authentication](/assets/aws-certified-developer-associate/database_authentication.png "Database Authentication")

There is also the option for monitoring and additional configuration:

![Additional Configuration](/assets/aws-certified-developer-associate/additional_configuration.png "Additional Configuration")

Among which, **backup** is crucial:

![RDS Backup](/assets/aws-certified-developer-associate/rds_backup.png "RDS Backup")

As well as **logs export to CloudWatch Logs** and **maintenance** for automatic minor version updates:

![RDS Maintenance](/assets/aws-certified-developer-associate/rds_maintenance.png "RDS Maintenance")

The **deletion protection** is also important because it will prevent accidental deletion of the database. So, to delete the database, you will have to disable this feature first.

Finally, AWS will show you an estimate of the monthly costs:

![RDS Cost Estimate](/assets/aws-certified-developer-associate/rds_cost_estimate.png "RDS Cost Estimate")

Click *Create Database* and you will be able to see the created database in the RDS dashboard.

![Created RDS DB](/assets/aws-certified-developer-associate/created_rds_db.png "Created RDS DB")

Then, you can connect to the database using the endpoint, port and the credentials you set up. Also, you need to have a TCP inbound rule enabled in the DB's security group (and eventually make the DB publicly accessible, if needed). **Connectivity details** can be seen in the *Connectivity & Security* tab.

![Connectivity & Security](/assets/aws-certified-developer-associate/connectivity_security.png "Connectivity & Security")

While the **monitoring dashboard** can be seen in the *Monitoring* tab:

![RDS Monitoring Dashboard](/assets/aws-certified-developer-associate/rds_monitoring_dashboard.png "RDS Monitoring Dashboard")

From the *Actions* menu, you can:
- Restore to point in time.
- Take a snapshot.
- Migrate a snapshot (into a different region, for example).

![RDS Actions](/assets/aws-certified-developer-associate/create_rds_read_replica.png "RDS Actions")

## 5.5 Creating RDS Read Replica

Go to the RDS dashboard and select the database you want to create a read replica for. Then, click on *Actions* and *Create Read Replica*.

![Create RDS Read Replica](/assets/aws-certified-developer-associate/create_rds_read_replica.png "Create RDS Read Replica")

And configure the read replica (multiple configuration options):

![Configure RDS Read Replica](/assets/aws-certified-developer-associate/configure_rds_read_replica.png "Configure RDS Read Replica")

## 5.6 Amazon Aurora

Aurora is a proprietary technology from AWS (not open sourced).

**Postgres and MySQL are both supported as Aurora DB**, which means your drivers will work as if Aurora was a Postgres or MySQL database.

Aurora is AWS cloud optimized and claims:
- 5x performance improvement over MySQL on RDS.
- 3x the performance of Postgres on RDS.

**Aurora storage automatically grows** in increments of 10GB, up to 128 TB. So, **Aurora is auto expanding**.

Aurora can have up to 15 replicas and the replication process is faster than MySQL (sub 10 ms replica lag).

The failover in Aurora is instantaneous and way faster than RDS multi AZ.

Aurora costs more than RDS (20% more, currently) but is more efficient.

### 5.6.1 Aurora High Availability and Read Scaling

Aurora **stores 6 copies of your data across 3 AZ**:
- It needs only 4 copies out of 6 for writes.
- It needs only 3 copies out of 6 for reads.
- Self healing with peer-to-peer replication, for example, for data corruption.
- Storage is striped across hundreds of volumes.

**One Aurora instance takes writes (master)** but there is automated failover for master in less than 30 seconds should the master fail and a replica need to take its place.

Aurora supports **a master and up to 15 Aurora read replicas to serve reads** with support for **cross region replication**.

![Aurora High Availability and Read Scaling](/assets/aws-certified-developer-associate/aurora_high_availability_read_scaling.png "Aurora High Availability and Read Scaling")

### 5.6.2 Inner Working of an Aurora Cluster

In a cluster, you have a master instance and up to 15 read replicas that access an **auto expanding storage layer from 10GB to 128TB**. 

The **master** is the only one that can write to the storage layer but it can fail (e.g., crash). Aurora provides a **writer endpoint** that will always point to the writer.

**Read replicas can be auto scaled up to 15**, so that you always have the number of replicas that you actually need. As with the writer, read replicas have a **reader endpoint** that will always point to readers. It connects automatically to one of the read replicas based on a **load balancing algorithm that works at connection level**.

![Aurora Cluster](/assets/aws-certified-developer-associate/aurora_cluster.png "Aurora Cluster")

### 5.6.3 Aurora Features

- Automatic fail-over,
- Backup and Recovery.
- Isolation and security.
- Industry compliance.
- Push-button scaling.
- Automated patching with zero downtime.
- Advanced monitoring.
- Routine maintenance.
- **Backtrack**: restore data at any point of time without using backups.

## 5.7 Creating Amazon Aurora DB

Go to *Databases* under *RDS* in the AWS Management Console and click on *Create Database*.

Choose either **Aurora MySQL Compatible** or **Aurora Postgres Compatible** as the database engine:

![Choose Aurora Engine](/assets/aws-certified-developer-associate/choose_aurora_engine.png "Choose Aurora Engine")

And the **engine version**:

![Aurora Engine Version](/assets/aws-certified-developer-associate/aurora_engine_version.png "Aurora Engine Version")

Then, you have 2 options for **cluster storage configuration**:

![Aurora Cluster Storage Configuration](/assets/aws-certified-developer-associate/aurora_cluster_storage_configuration.png "Aurora Cluster Storage Configuration")

*I/O-Optimized* is for when you have a high load of reads and writes to your DB.

For **instance configuration**, you can choose the following options (among which **serverless**):

![Aurora Instance Configuration](/assets/aws-certified-developer-associate/aurora_instance_configuration.png "Aurora Instance Configuration")

If you select **serverless**, you will have to configure the **capacity settings**, which will define the number of **Aurora Capacity Units (ACUs)** the DB will use. The **minimum and maximum ACUs** will define the minimum and maximum capacity the DB can use.

![Aurora Serverless Capacity Settings](/assets/aws-certified-developer-associate/aurora_serverless_capacity_settings.png "Aurora Serverless Capacity Settings")

In the **availability and durability** settings, we can add replicas to the cluster:

![Aurora Availability and Durability](/assets/aws-certified-developer-associate/aurora_availability_durability.png "Aurora Availability and Durability")

**Connectivity settings**, **additional configuration**, **authentication**, and **monitoring** are similar to RDS.

Aurora offers **local write forwarding** which allows you to write to a read replica.

![Aurora Local Write Forwarding](/assets/aws-certified-developer-associate/aurora_local_write_forwarding.png "Aurora Local Write Forwarding")

The created DB will be visible in the RDS dashboard. From here, you can see writer and readers.

![Created Aurora DB](/assets/aws-certified-developer-associate/created_aurora_db.png "Created Aurora DB")

To connect, you have two endpoints: the writer endpoint and the reader endpoint:

![Aurora Endpoints](/assets/aws-certified-developer-associate/aurora_endpoints.png "Aurora Endpoints")

However, each instance will also have its own endpoint.

From the *Actions* menu, you can:
- **Add a reader**.
- **Create a cross region read replica**.
- **Add replica auto scaling** via auto scaling policies.
- **Add a region**, which is only available if the version of Aurora has the global database feature enabled.
- Restore to point in time.

![Aurora Actions](/assets/aws-certified-developer-associate/aurora_actions.png "Aurora Actions")

An Aurora scaling policy can be created for the read replicas on the two following metrics:
- **Average CPU utilization of Aurora replicas**.
- **Average connections of Aurora replicas**.

Following an example of an **Aurora replica scaling policy**:

![Aurora Replica Scaling Policy](/assets/aws-certified-developer-associate/aurora_replica_scaling_policy.png "Aurora Replica Scaling Policy")

A scaling policy also needs capcity settings:

![Aurora Replica Scaling Policy Capacity Settings](/assets/aws-certified-developer-associate/aurora_replica_scaling_policy_capacity_settings.png "Aurora Replica Scaling Policy Capacity Settings")

## 5.8 Security in RDS and Aurora

- **At-rest encryption**, data is encrypted in the underlying storage:
    - **Database master and replicas encryption using AWS KMS**, this must be defined at launch time when you create the database.
    - If the master is not encrypted, the read replicas cannot be encrypted.
    - **To encrypt an un-encrypted database, go through a DB snapshot and restore as encrypted**.
- **In-flight encryption**: TLS-ready by default, use the AWS TLS root certificates client-side.
- **IAM Authentication**: you can use IAM roles to connect to your database instead of username and password.
- **Security Groups**: control network access to your RDS/Aurora DB.
- **No SSH available** except on *RDS Custom* servicer.
- **Audit Logs** (e.g., you want to know the queries made to DBs) can be enabled and sent to CloudWatch Logs for longer retention (otherwise they will be lost after a short period of time).

## 5.9 Amazon RDS Proxy

It is a fully managed, highly available database proxy for RDS. This **proxy allows applications to pool and share connections established with the database**.

It is serverless, has autoscaling, and is highly available as it in a multi AZ setting.

Important for the exam: **RDS Proxy improves database efficiency by reducing the stress on database resources (e.g., CPU, RAM) and minimize open connections (and timeouts)**. It also **reduces RDS and Aurora failover time by up 66% because it handles the failover itself instead of having the DB instances do it**.

It supports RDS, (MySQL, PostgreSQL, MariaDB, MS SQL Server) and Aurora (MySQL and PostgreSQL).

It does not require huge code changes for most applications as you just need to point your application to the proxy endpoint instead of the DB endpoint.

**RDS Proxy enforces IAM authentication and securely stores credentials in AWS Secrets Manager**. So, **if the exam asks about how to enforce IAM authentication to you DBs, RDS Proxy is the answer**.

**RDS Proxy is never publicly accessible as it must be accessed from VPC**, enhancing security.

A classical **use case for RDS Proxy** is when you have Lambda functions that need to connect to an RDS DB. Instead of having the Lambda functions open a connection to the DB, you can have them open a connection to the RDS Proxy. This is because Lambda functions can open and close connections very quickly, which can overwhelm the DB, and it may also happen that they open a lot of connections at the same time.

![RDS Proxy](/assets/aws-certified-developer-associate/rds_proxy.png "RDS Proxy")

## 5.10 Amazon ElastiCache

The same way RDS is to get managed relational databases, **ElastiCache is to get managed Redis or Memcached**.

Caches are in-memory databases with really high performance and low latency.

Caching helps **reduce load off of databases for read intensive workloads** and **make your application stateless** by putting the state in ElastiCache.

AWS takes care of OS maintenance/patching, optimizations, setup, configuration, monitoring, failure recovery and backups.

Using ElastiCache involves heavy application code changes.

### 5.10.1 ElastiCache DB Cache

Application queries ElastiCache, if not available, get from RDS and store in ElastiCache. This helps relieve load in RDS.

![ElastiCache DB Cache](/assets/aws-certified-developer-associate/elasticache_db_cache.png "ElastiCache DB Cache")

Cache must have an **invalidation strategy** to make sure only the most current data is used in there.

### 5.10.2 ElastiCache User Session Store

User logs into any of the application and the application writes the session data into ElastiCache. When the user hits another instance of our application, the instance retrieves the data from ElastiCache and the user is already logged in.

![ElastiCache User Session Store](/assets/aws-certified-developer-associate/elasticache_user_session_store.png "ElastiCache User Session Store")

### 5.10.3 Redis vs Memcached

**Redis**:
- Multi-AZ with auto-failover.
- Read replicas to scale reads and have high availability.
- Data durability using AOF persistence.
- Backup and restore features.
- Supports sets and sorted sets.

![Redis](/assets/aws-certified-developer-associate/redis.png "Redis")

**Memcached**:
- Multi-node for partitioning of data (sharding).
- No high availability (replication).
- Non persistent.
- Backup and restore only on the serverless version.
- Multi-threaded architecture.

![Memcached](/assets/aws-certified-developer-associate/memcached.png "Memcached")

## 5.11 Creating Amazon ElastiCache

Go to *ElastiCache* in the AWS Management Console and click on *Create* to choose between Redis and Memcached.

Configure deployment options and creation method:

![ElastiCache Configuration](/assets/aws-certified-developer-associate/elasticache_configuration.png "ElastiCache Configuration")

By choosing **cluster mode**, you can configure everything. **Disabling cluster mode will result in a single shard with up to 5 read replicas**, while **enabling it will result in multiple shards** for increased scalability and availability.

![ElastiCache Cluster Mode](/assets/aws-certified-developer-associate/elasticache_cluster_mode.png "ElastiCache Cluster Mode")

After, you need to insert cluster information like cluster name and description.

You can run ElastiCache in the cloud or on-premises using **AWS Outposts**: 

![ElastiCache Location](/assets/aws-certified-developer-associate/elasticache_location.png "ElastiCache Location")

And, as you can see from the image above, you can chose to use the **multi AZ** feature (which will have auto-failover by default) or disable it and chose if you want auto-failover.

Then, **cluster settings**, such as the node type and number of replicas:

![ElastiCache Cluster Settings](/assets/aws-certified-developer-associate/elasticache_cluster_settings.png "ElastiCache Cluster Settings")

You also need to configure the **subnet group** to define the subnets in the VPC where you can run the cache, as well as the AZ placements.

In the **advanced settings**, you can enable **encryption at rest** and **encryption in transit** (which also gives you access control).

![ElastiCache Advanced Settings](/assets/aws-certified-developer-associate/elasticache_advanced_settings.png "ElastiCache Advanced Settings")

Next is **automatic backups** and **maintenance windows** (both like RDS). Finally, you can create the cache and access it from the ElastiCache dashboard.

![ElastiCache Details](/assets/aws-certified-developer-associate/elasticache_details.png "ElastiCache Details")

## 5.12 Caching Implementation Considerations

- You have to consider that data may be out of date, caches provide eventual consistentcy.
- Is caching effective for your data?
    - **Pattern**: data changing slowly, few keys are frequently needed.
    - **Anti patterns**: data changing rapidly, all large key space frequently needed.
- Is data structured well for caching? E.g., key value caching, or caching of aggregations results.

Also, you have to consider the **caching design pattern/strategy** that works best for your application.

### 5.12.1 Cache Aside (Lazy Loading, Lazy Population)

Cache aside is also called lazy loading or lazy population.

In this pattern, the application code is responsible for loading data into the cache. The application code first checks the cache to see if the data is there. If it is not, the application code loads the data from the database and then puts it into the cache, so that the next time the data is needed, it is already in the cache.

![Cache Aside](/assets/aws-certified-developer-associate/cache_aside.png "Cache Aside")

**Pros**:
- Only requested data is cached (the cache is not filled up with unused data).
- Node failures are not fatal (just increased latency to **warm the cache**).

**Cons**
- **Cache miss penalty that results in 3 round trips**, noticeable delay for that request.
- **Stale data**: data can be updated in the database and outdated in the cache.

An example of cache aside in Python:

```python
# get user from cache or DB
def get_user(user_id):
    # check if user is in cache
    user = cache.get(user_id)

    # if the user is not in the cache
    if not user:
        # get it from the DB and add it to the cache for next time
        user = db.query("SELECT * FROM users WHERE user_id = ?", user_id)
        cache.set(user_id, user)

    return user

# app code to get the user with id 29
user = get_user(29)
```

### 5.12.2 Write Through

In this pattern, the application code writes data to the DB and the cache at the same time. This pattern is **used when data is written more often than it is read**. So, when you read data, they are always in the cache.

![Write Through](/assets/aws-certified-developer-associate/write_through.png "Write Through")

**Pros**:
- Data in cache is never stale, reads are quick.
- In this pattern, you have a **write penalty** (each write requires 2 calls: cache and DB) and not a read penalty. However, users expect a write to be slower than a read, so this may impact the user experience less than slow reads.

**Cons**:
- **Missing data** because the cache will never have the data it needs until data is added/updated in the DB. Mitigation is to implement **cache aside alongside write through**.
- **Cache churn**: a lot of the data will never be read.

An example of write through in Python:

```python
# save user to DB and cache
def save_user(user_id, user):
    # save user to DB
    db.query("INSERT INTO users (user_id, user) VALUES (?, ?)", user_id, user)

    # save user to cache as well
    cache.set(user_id, user)

    return user

# app code to save a user with id 29
user = save_user(29, "John Doe")
```

You can combine write through with cache aside to mitigate the missing data issue, and it is very simple because, for cache aside, we had a `get_user()` function and here we have a `save_user()` function. So, just use this two for reads and writes respectively and you have a combined cache aside and write through pattern.

### 5.12.3 Cache Eviction and Time to Live

**Cache eviction** can occur in 3 ways:
- You **delete the item explicitly** in the cache.
- Item is evicted because the memory is full and it’s not recently used (Least Recently Used, **LRU**).
- You set an item time-to-live (or **TTL**), for example, 1 hour or 5 minutes. TTL can range from few seconds to hours or days depending on your application.

**If too many evictions happen due to memory because your cache is always at full capacity, you should scale up or out**.


### 5.12.4 Caching Summary

- **Cache aside** is easy to implement and **works for many situations** as a foundation, especially on the read side. So, **it can be your first way to go**.
- **Write through is usually combined with cache aside** as targeted for the queries or workloads that benefit from this optimization. So, **write through is usually an optimization on top of cache aside and not a first choice**.
- Setting a **TTL is usually not a bad idea, except when you are using write through**. Set it to a sensible value for your application.
- Only cache the data that makes sense (e.g., user profiles, blogs, ...) and not the data that is rarely accessed (e.g., logs) or sensitive data (e.g., credit card information).

## 5.13 Amazon MemoryDB for Redis

It is a Redis-compatible, durable, in-memory database service that delivers ultra-fast performance. with over 160 millions requests per second. It is a **durable in-memory database that has a Redis-like API**.

It is in-memory data but it is a durable data storage with multi AZ transactional log and scales seamlessly from tens of GBs to hundreds of TBs of storage.

Common use cases are web and mobile applications, online gaming, media streaming, etc.

![MemoryDB for Redis](/assets/aws-certified-developer-associate/memorydb_for_redis.png "MemoryDB for Redis")