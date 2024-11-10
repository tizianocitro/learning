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

### 3.1.1 Delete on Termination Attribute

The *Delete on Termination* attribute **controls EBS volumes behaviour when an EC2 instance terminates**:
- By default, the root EBS volume is deleted (attribute enabled).
- By default, any other attached EBS volume is not deleted (attribute disabled).

This can be controlled by the AWS console/AWS CLI and a good use case for it is preserving the root volume when instance is terminated to save some data.

![EBS Volume Delete on Termination](/assets/aws-certified-developer-associate/ebs_volume_delete_on_termination.png "EBS Volume Delete on Termination")