# 5 AWS Fundamentals: RDS, Aurora, and ElastiCache

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

Next is **database authentication**:

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