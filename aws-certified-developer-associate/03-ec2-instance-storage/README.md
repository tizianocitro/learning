# 3 EC2 Instance Storage

## 3.1 EBS Volumes

An **Elastic Block Store (EBS) volume is a network drive that you can attach to your instances while they run**. A good analogy is to think of them as network USB sticks that you can attach to different computers to share files and other resources.

EBS volumes allow your instances to persist data, even after their termination. This allows you to start a new instance and get back all the data on the previously terminated instance by just attaching the same EBS volume.

They can only be **mounted to one instance at a time** and are **bound to a specific AZ**. Note that **an EC2 instance can have multiple EBS volumes attached to it**. Also, EBS volumes can be created and left unattached, so that we can just attach them on-demand.

The free tier offers 30 GB of free EBS storage of type General Purpose (SSD) or Magnetic per month.

EBS volumes are network drives (i.e. not physical drives):
- They **use the network to communicate with the instance**, which means there might be a bit of latency between the volume and the machine.
- They can be detached from an EC2 instance and attached to another one quickly.

EBS volumes are locked to an AZ:
- An EBS volume in *us-east-1a* cannot be attached to *us-east-1b*.
- To move a volume across AZs, you first need to create a **snapshot** from it and recreate it in another AZ using the snapshot.

EBS volumes need to have a **provisioned capacity** expressed in terms of size in GBs and IO Operations Per Second (IOPS):
- You get billed for all the provisioned capacity.
- You can increase the capacity of the drive over time.

![Examples of EBS Volumes](/assets/aws-certified-developer-associate/ebs_volumes.png "Examples of EBS Volumes")

You can access all EBS volumes from the *Volumes* entry in the EBS menu.

### 3.1.1 Delete on Termination Attribute

The *Delete on Termination* attribute **controls EBS volumes behaviour when an EC2 instance terminates**:
- By default, the root EBS volume is deleted (attribute enabled).
- By default, any other attached EBS volume is not deleted (attribute disabled).

This can be controlled by the AWS console/AWS CLI and a good use case for it is preserving the root volume when instance is terminated to save some data.

![EBS Volume Delete on Termination](/assets/aws-certified-developer-associate/ebs_volume_delete_on_termination.png "EBS Volume Delete on Termination")

## 3.2 Creating EBS Volumes

There are different settings when creating an EBS volume, such as the type of storage and AZ.

![EBS Volume Settings](/assets/aws-certified-developer-associate/ebs_volume_settings.png "EBS Volume Settings")

The AZ is particularly important because it needs to be in the same AZ where the VM we want to attach it to is running. The AZ where an instance is running is indicated under the *Networking* tab in the VM details. Meanwhile. the EBS volumes attached to a machine can be seen in the *Storage* tab.

From the list of volumes (shown in the image below), it is possible to select a volume and have a look at its details but it is also possible to use the **Actions** button to perform operations on the volume like attaching it to a machine, deleting it, and more.

![EBS Volumes Actions](/assets/aws-certified-developer-associate/ebs_volumes_actions.png "EBS Volumes Actions")

## 3.3 EBS Snapshots

Snapshots allow you to make a **backup (snapshot) of your EBS volume** at a point in time. It is not necessary to detach volumes to do a snapshot, but it is recommended.

The great thing is that **snapshots can be copied across AZs or regions**, making them useful for migrating the volume from an AZ/regions to another.

![EBS Snapshots](/assets/aws-certified-developer-associate/ebs_snapshots.png "EBS Snapshots")

You can access all EBS snapshots from the *Snapshots* entry in the EBS menu. All actions that you can perform on snapshots are available via a dedicated *Actions* button.

### 3.3.1 EBS Snapshot Archive

You can move a snapshot to an **archive tier** that is a lot cheaper (75% currently) but it will take within 24 to 72 hours to restore the archive should you need it.

![EBS Snapshots Archive](/assets/aws-certified-developer-associate/ebs_snapshots_archive.png "EBS Snapshots Archive")

### 3.3.2 Recycle Bin for EBS Snapshots

It is possible to **setup retention rules to retain deleted snapshots** so you can **recover them after an accidental deletion**.

The **retention period** can be from 1 day to 1 year.

![EBS Snapshots Bin](/assets/aws-certified-developer-associate/ebs_snapshots_bin.png "EBS Snapshots Bin")

### 3.3.3 Fast Snapshot Restore (FSR)

Fast Snapshot Restore forces the full initialization of a snapshot to have no latency on  first use. It is very expensive but useful when the snapshot is very big and you need to initialize an EBS volume from it quickly.

## 3.4 Creating EBS Snapshots

To create an EBS snapshot you need to use the *Create Snapshot* entry in an EBS volume's *Actions* menu.

![Create EBS Snapshots](/assets/aws-certified-developer-associate/create_ebs_snapshot.png "Create EBS Snapshots")

By doing so, you can configure the snapshot.

![Create Snapshot Settings](/assets/aws-certified-developer-associate/ebs_snapshot_settings.png "EBS Snapshot Settings")

Then you can use the created snapshot to create an EBS volume from it, copy the snapshot, and perform other actions. **When you create an EBS volume from a snapshot, you will see a reference to the snapshot in the EBS volume details**.

## 3.5 Amazon Machine Images (AMIs)

An AMI is **a customization of an EC2 instance where you add your own software, configuration, operating system, monitoring, and more**. They offer **faster boot and configuration time because all the software is pre-packaged** and does not need to be installed at boot time, for example, using EC2 user data scripts.

AMIs are **built for a specific region** but **can be copied across regions**.

You can launch EC2 instances from:
- **Public AMIs**: AWS provides and maintains them.
- **Your own AMIs**: you make and maintain them yourself.
- **AWS Marketplace AMIs**: AMIs someone else made and potentially sells on the marketplace.

## 3.6 Creating AMIs from EC2 instances

1. Start an EC2 instance and customize it (for example, adding a EC2 user data script).
2. Stop the instance for data integrity.
3. Build an AMI from the instance, this will also create EBS snapshots.
4. Launch instances using the built AMI.

![Custom AMI](/assets/aws-certified-developer-associate/custom_ami.png "Custom AMI")

By opening the context menu of the machine you want to create an AMI from, you will be able to select the option *Create Image* to create an AMI.

![Create AMI](/assets/aws-certified-developer-associate/create_ami.png "Create AMI")

This will allow you to configure your new AMI:

![AMI Info](/assets/aws-certified-developer-associate/ami_info.png "AMI Info")

When you confirm the creation of the AMI, it will be visible in the *AMIs* tab under the *Images* entry in the AWS Console.

![AMIs List](/assets/aws-certified-developer-associate/amis_list.png "AMIs List")

Then, using the *Launch instance from AMI* button, you can start an EC2 instance from the AMI.

![Launch EC2 from AMI](/assets/aws-certified-developer-associate/launch_ec2_from_ami.png "Launch EC2 from AMI")

Or you can select your own AMI when creating an instance, like in the following image.

![Launch EC2 from custom AMI](/assets/aws-certified-developer-associate/ec2_from_custom_ami.png "Launch EC2 from custom AMI")

## 3.7 (Local) EC2 Instance Store

EBS volumes are network drives with good but limited performance. If you need a high-performance hardware disk, use EC2 Instance Store. **EC2 Instance Store leverages the physical hard-drive connected to the physical server where the EC2 instance is running**.

It provide better I/O performance (very high IOPS) but **it is ephemeral storage**, meaning that the EC2 instances lose their storage if they’re stopped.

**It is a good fit for buffer, cache, scratch data or temporary content but not for long-term data** (this is better handled with EBS) as there is **risk of data loss if hardware fails**. Backups and replication are your responsibility.

An example of EC2 instance type that has instance store attached is the *i3* family.

**From an exam perspective, anytime you see very high-performance hardware attached volume for your EC2 instances, think of Local EC2 Instance Store**.

## 3.8 EBS Volume Types

EBS volumes come in **6 types**:
- **gp2/gp3 (SSD)**: General purpose SSD volume that balances price and performance for a wide variety of workloads.
- **io1/io2 Block Express (SSD)**: Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads.
- **st1 (HDD)**: Low cost HDD volume designed for frequently accessed, throughput-intensive workloads.
- **sc1 (HDD)**: Lowest cost HDD volume designed for less frequently accessed workloads.

**EBS volumes are characterized in size, throughput, and IOPS**.

Keep in mind that **only gp2/gp3 and io1/io2 Block Express can be used as boot volumes, where the root OS runs**.

**General Purpose SSD and Provisioned IOPS (PIOPS) SSD are the most important for the exam**.

### 3.8.1 EBS General Purpose SSD

It offers **cost effective storage and low-latency**. Usable for system boot volumes, virtual desktops, development and test environments.

**The size goes between 1 GiB and 16 TiB**.

**gp3** (newer generation of volumes):
- Offers a baseline of 3,000 IOPS and throughput of 125 MiB/s.
- It is possible to increase IOPS up to 16,000 and throughput up to 1000 MiB/s, independently. It means IOPS and throughput are not linked.

**gp2**:
- The small gp2 volumes can burst IOPS to 3,000.
- Size of the volume and IOPS are linked with the max IOPS is 16,000.
- It means that you get 3 IOPS per each GB you add, so at 5,334 GB we are at the max IOPS.

**The crucial part is that in gp3 IOPS and throughput can be scaled independently, while in gp2 they are linked together**.

### 3.8.2 EBS Provisioned IOPS (PIOPS) SSD

Good fit for critical business applications with sustained IOPS performance or **applications that need more than 16,000 IOPS**.

**For the exam, keep in mind that when you see databases workloads that are sensitive to storage performance and consistency, this is a type of volume that will work great**.

**io1 (between 4 GiB and 16 TiB)**:
- Max PIOPS are 64,000 for Nitro EC2 instances and 32,000 for other kind of instances.
- Can increase PIOPS independently from storage size.

**io2 Block Express (between 4 GiB and 64 TiB)**:
- They offer sub-millisecond latency.
- Max PIOPS are 256,000 with an IOPS:GiB ratio of 1,000:1.

**PIOPS supports EBS Multi-attach**.

**Important to remember that for over 32,000 IOPS, you need EC2 Nitro with io1 or io2**.

### 3.8.3 Hard Disk Drives (HDD)

The **cannot be used as boot volumes** and the **size goes between 125 GiB and 16 TiB**.

**Throughput Optimized HDD (st1)**:
- Great for big data, data warehouses, and log processing.
- Max throughput of 500 MiB/s and max IOPS 500.

**Cold HDD (sc1)**:
- Useful for data that is infrequently accessed.
- For scenarios where you need the lowest possible cost.
- Max throughput of 250 MiB/s and max IOPS 250.

## 3.9 EBS Multi-Attach (io1/io2 only)

**Attach the same EBS volume to multiple EC2 instances in the same AZ**. The volume is still tight to the AZ, so you cannot attach it to another AZ

**Multi-attach only works with up to 16 EC2 instances (this is important for the exam)** and you must use a file system that is cluster-aware (not XFS, EXT4, etc…).

Each instance has full read and write permissions to the high-performance volume.

Some use cases:
- Achieve higher application availability in clustered Linux applications (for example using Teradata).
- Applications must manage concurrent write operations.

![EBS Multi-Attach](/assets/aws-certified-developer-associate/ebs_multi-attach.png "EBS Multi-Attach")

## 3.10 Elastic File System (EFS)

It is a managed Network File System (NFS) that can be mounted on many EC2 instances, **also in a multi-AZ scenario**.

It uses the NFSv4 protocol, is highly available and scalable as it **scales automatically**. It follows a pay per use model, meaning that you **do not have to plan capacity in advance** but it is expensive as it costs 3x gp2, for example.

Data are encrypted at rest using KMS and EFS is **compatible with Linux-based AMIs only** (not Windows) because it offers a POSIX file system (Linux) that has a standard file API.

![EFS](/assets/aws-certified-developer-associate/efs.png "EFS")

### 3.10.1 EFS Perfomance

**EFS Scale**:
- 1000s of concurrent NFS clients, 10 GB+/s throughput.
- Growth to Petabyte-scale network file system, automatically.

**Performance Mode** (it is set at EFS creation time):
- **General Purpose (default and recommended option)**: latency-sensitive use cases (web server, CMS, ...).
- **Max I/O**: higher latency and throughput and highly parallel (big data, or media processing).

**Throughput Mode**:
- **Bursting**: 1 TB means you have 50MiB/s + burst of up to 100MiB/s. Basically, it grows in throughput as it has more storage.
- **Provisioned**: set your throughput regardless of storage size, e.g.,: 1 GiB/s for 1 TB of storage. It decorellates throughput from storage.
- **Elastic (simpler and now recommended option)**: automatically scales throughput up or down based on your workloads:
    - Up to 3GiB/s for reads and 1GiB/s for writes based on your workload.
    - Used for unpredictable workdloads only.

### 3.10.2 EFS Storage Class

**Storage Tiers**, which provide a lifecycle management feature to move file after N days between the different types of storage tiers using **lifecycle policies**:
- **Standard**: for frequently accessed files.
- **Infrequent access (EFS-IA)**: gives you a cost to retrieve files but has a lower price to store these files.
- **Archive**: for rarely accessed data (few times each year), a lot cheaper to store data on this tier (e.g., 50% less expensive).

In the following example, a file that has not been accessed for 60 days is moved from the EFS Standard tier to the EFS IA tier:

![EFS Storage Tiers](/assets/aws-certified-developer-associate/efs_storage_tiers.png "EFS Storage Tiers")

**Availability and Durability**:
- **Standard/Regional**: great for multi-AZ settings, for example, to resist to disasters in your production environment.
- **One Zone**: one AZ settings, which are great for development environments. It has backup enabled by default and is compatible with IA using the EFS One Zone-IA option.

## 3.11 Using EFS

### 3.11.1 Creating a File System

When creating an EFS we need to specify a VPC.

![EFS Creation](/assets/aws-certified-developer-associate/efs_creation.png "EFS Creation")

But if you click on the *Customize* button, you can enter a lot of other options for the **File System Settings**, such as file system type and encryption settings.

![EFS Options](/assets/aws-certified-developer-associate/efs_options.png "EFS Options")

And very important to save costs is the *Lifecycle Management* section:

![EFS Lifecycle Management](/assets/aws-certified-developer-associate/efs_lifecyle_management.png "EFS Lifecycle Management")

It is also possible to set the aforementioned *Performance Settings*. Enhanced is just a way to group the *Elastic* and *Provisioned* settings.

![EFS Performance Settings](/assets/aws-certified-developer-associate/efs_performance_settings.png "EFS Performance Settings")

For the *Provisioned* setting, you have to specify more options for the throughput and also have an indication of the throughput bill:

![EFS Provisioned Setting](/assets/aws-certified-developer-associate/efs_provisioned_setting.png "EFS Provisioned Setting")

Finaly, by clicking on the *Additional Settings*, you can set the *Performance Mode*.

![EFS Performance Mode](/assets/aws-certified-developer-associate/efs_performance_modes.png "EFS Performance Mode")

The **recommended configution by AWS is EFS with Elastic throughput mode and General Purpose performance mode**.

Then, you need to configure the **Network Access** and you have to select a VPC and the **mount targets** with a security group for the file system for each target. You can create 1 mount target per AZ.

![EFS Mount Targets example](/assets/aws-certified-developer-associate/efs_mount_targets.png "EFS Mount Targets example")

Next, you can configure an optional **File System Policy**, also using a JSON editor.

After the file system has been created, you can inspect it to access useful information, e.g., the current size.

![EFS Details example](/assets/aws-certified-developer-associate/efs_details.png "EFS Details example")

### 3.11.2 Mounting a File System into EC2 Instances

From the EC2 creation dashboard, you can add file systems to the EC2 instance you are creating but you need to **select a subnet first** (as you can see in the image below).

![EFS mount into EC2](/assets/aws-certified-developer-associate/efs_mount_ec2.png "EFS mount into EC2")

Select a subnet by updating the *Network Settings* of your EC2 instance.

![Select subnet for EC2](/assets/aws-certified-developer-associate/ec2_select_subnet.png "Select subnet for EC2")

After you do so, you can add file system to the EC2 instance by clicking on *Add Shared File System*.

![Enabled EFS for EC2 instance](/assets/aws-certified-developer-associate/efs_enabled_for_ec2.png "Enabled EFS for EC2 instance")

Which will result in the file system being attached to the instance on a **mount point** that can be customized.

A **security group will also be automatically created and added to the EFS mount target of the AZ where you are starting the instance**. The created security group allows inbound traffic over port 2049 and NFS protocol from the security group of the machine. So, the instance can access the file system.

![EFS attached to EC2](/assets/aws-certified-developer-associate/efs_attached_ec2.png "EFS attached to EC2")

To check that the file system is attached correctly, just SSH into the VM and run `ls <mount-point>`. Considering the same mount point of the image above, the command is `ls /mnt/efs/fs1`.

Because the file system is shared, creating a file from the instance into it, will make that file accessible by any other instance that has this same file system attached.

## 3.12 EBS vs EFS

### 3.12.1 EBS

EBS volumes:
- Are attached to one instance (except if using the multi-attach of io1/io2).
- Are locked at the AZ level.
- In gp2, IO increases if the disk size increases.
- In gp3 and io1, you can increase IO independently.

To migrate an EBS volume across AZ, you need to take a snapshot and restore the snapshot to another AZ. However, **EBS backups use IO and you should not run them while your application is serving traffic because they can impact performance**.

![EBS migration](/assets/aws-certified-developer-associate/ebs_migration.png "EBS migration")


Root EBS volumes of instances get terminated by default if the EC2 instance gets terminated but you can disable that.

### 3.12.2 EFS

With EFS, it is possible to mount 100s of instances across AZs but only Linux instances (POSIX). For example, use EFS to share website files (WordPress).

EFS has a higher price point than EBS but you can leverage storage tiers for cost savings.

![EFS example](/assets/aws-certified-developer-associate/efs_example.png "EFS example")
