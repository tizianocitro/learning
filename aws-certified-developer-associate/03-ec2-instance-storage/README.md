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
