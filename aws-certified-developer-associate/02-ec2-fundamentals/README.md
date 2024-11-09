# 2 EC2 Fundamentals

![EC2](/assets/aws-certified-developer-associate/ec2.png "EC2")

## 2.1 Budget Setup

It is possible to set budgets in the *Billing and Cost Management Home* in the AWS Console. To access billing data, you need to activate IAM access to **Billing Information**.

![Examples of Billing and Cost Management Home](/assets/aws-certified-developer-associate/budget_home.png "Billing and Cost Management Home")

Under the **Bills** option, you can see how much you have been charged for the usage of each service.
Using the **Free Tier** entry, you can see the free options and usages for AWS services. For example:

![Free Tier](/assets/aws-certified-developer-associate/free_tier.png "Free Tier")

The **forecast usage** column provides an indication on whether you will be able to remain in the limits offered by the AWS free tier. As you can see from the image, 400.000 seconds of AWS Lambda are free but the forecast usage is 826.667 seconds, so you won't remain in the limits of the free tier.

In *Budgets and Planning* you can create *Budgets* hat will alert you whenever you reach its threshold. AWS offers you some templates like the **Zero Spend Budget**, which will alert you as soom as you spend even a single penny. You can also create monthly budgets to say I do not want to spend more than 100$ in a month.

When you set a budget, you will receive an email when:
- The actual spend reaches 85% of the threshold.
- The actual spend reaches 100% of the threshold.
- The forecast spend is expected to reach 100% of the threshold.

You can also monitor the budgets from the budgets home.

![Monitor the budgets from the budgets home](/assets/aws-certified-developer-associate/budgets_home.png "Monitor the budgets from the budgets home")

## 2.2 EC2

**Elastic Compute Cloud (EC2)** is one of the IaaS AWS offerings.

EC2 mainly consists in the capability of:
- Renting Virtual Machines (VMs) (EC2).
- Storing data on virtual drives (EBS).
- Distributing load across machines (ELB).
- Scaling the services using an auto-scaling group (ASG).

### 2.2.1 EC2 Sizing & Configuration

You can change and configure a lot about your VMs::
- Operating System (OS): Linux, Windows or Mac OS.
- How much compute power and cores (CPU).
- How much random-access memory (RAM).
- How much storage space:
    - Network-attached via EBS or EFS.
    - Hardware via **EC2 Instance Store**.
- Network card:
    - Speed of the card.
    - Public IP address.
- Firewall rules: configured via security groups.
- Bootstrap script (configure at first launch) using **EC2 User Data**.

Here are a few examples of EC2 instance types with different configurations:

![Example of EC2 Instance Types](/assets/aws-certified-developer-associate/ec2_instance_types_example.png "Example of EC2 Instance Types")

**t2.micro** is part of the AWS free tier, which provides up to 750 hours per month.

### 2.2.2 EC2 User Data

It is possible to bootstrap your instances using an EC2 user data script, where **bootstrapping** means launching commands when a machine starts. The user data script is only run once at the instance first start and never again. Notice that the more you do in this script, the longer it takes to boot the VM.

EC2 user data is used to automate boot tasks such as:
- Installing updates.
- Installing software.
- Downloading common files from the internet.
- Anything else you can think of or need in your VM after it starts.

The EC2 user data script runs with the root user, so all **commands in the user data script will be executed with root priviledges**.

## 2.3 Launching an EC2 Instance

To launch an EC2 instance you can use the dedicated buttons in the dashboard:

![Launch EC2 instance](/assets/aws-certified-developer-associate/launch_ec2_instance.png "Launch EC2 instance")

Doing so will open a page in the console to configure the new VM. For instance, selecting the **Amazon Machine Image (AMI)** to start your machine. You can also create your own AMI to start custom machines with your own personalized configuration.

A few examples of other options in the following image:

![Options when creating an EC2 instance](/assets/aws-certified-developer-associate/ec2_config_options.png "Options when creating an EC2 instance")

It is also possible to review and compare different instance types when creating an EC2 instance.

![Review EC2 instance types](/assets/aws-certified-developer-associate/review_ec2_types.png "Review EC2 instance types")

Key pairs are necessary if we want access via **SSH** to the created VM. A key pair can be created upon machine creation. For Mac OS, Linux, and Windows 10 and above use *.pem*, use *.ppk* for Windows 9 and older.

Regarding network settings, it is a good practice to set them via **security groups**.

For storage, you have the following configuration but more details will come.

![EC2 storage config](/assets/aws-certified-developer-associate/storage_ec2_options.png "EC2 storage config")

By clicking on *Advanced* it is possible to access more advanced customization options. A crucial one is **Delete on termination**, which means that the volume will be deleted when the VM is deleted, so all data in the volume will be lost.

![EC2 storage advanced config](/assets/aws-certified-developer-associate/storage_ec2_options_advanced.png "EC2 storage advanced config")

Continuing the process of creating the VM, it is possible to set the **user data script**. The example in the following image loads a script to start a `httpd` web server that displays a HTML page with an `<h1>` stating `Hello World from <hostname>`.

![EC2 user data](/assets/aws-certified-developer-associate/ec2_user_data.png "EC2 user data")

Finally, it is possible to review every setting in a summary section before launching the instance. Started instances appear as below in the *Instances* tab within the EC2 dashboard.

![Launced EC2 instance](/assets/aws-certified-developer-associate/launched_ec2_instance.png "Launced EC2 instance")

It will take some time to completely start an instance, as you can see from the *Pending* state. Once started the VM will have public/private IPv4 addresses that you can use to access it. Clicking on the instance will reveal all this information.

![EC2 instance details](/assets/aws-certified-developer-associate/ec2_instance_details.png "EC2 instance details")

An instance can then be stopped and when stopped you will be charged only for the storage it needs and not for the compute. When stopped, you can restart it or terminate it completely.
**Note**: restarting the VM will cause the public IP to change but the private IP will stay the same.

## 2.4 EC2 Instance Types

You can use different types of EC2 instances that are optimized for different use cases [https://aws.amazon.com/ec2/instance-types/](https://aws.amazon.com/ec2/instance-types/).

**Naming convention**: m5.2xlarge
- m: instance class.
- 5: generation (AWS improves the hardware over time).
- 2xlarge: size within the instance class, the greater the size the more the memory and cpu on the instance.

There are different families within the EC2 instance types, e.g., general purpose, compute optmized, memory optimized, and more.

Here are a few examples of EC2 instance types:

![Example of EC2 Instance Types](/assets/aws-certified-developer-associate/ec2_instance_types_example.png "Example of EC2 Instance Types")

There is a useful website accessible at [https://instances.vantage.sh](https://instances.vantage.sh) to compare different EC2 instance types.

### 2.4.1 Instance Type - General Purpose

Great for a diversity of workloads such as web servers or code repositories. They offer a **good balance between compute, memory, and networking**.

For example, the t2.micro is a general purpose EC2 instance.

### 2.4.2 Instance Type - Compute Optimized

Great **for compute-intensive tasks that require high performance processors**, such as:
- Batch processing workloads.
- Media transcoding.
- High performance web servers.
- High performance computing (HPC).
- Scientific modeling and machine learning.
- Dedicated gaming servers.

### 2.4.3 Instance Type - Memory Optimized

Fast performance **for workloads that process large data sets in memory (RAM)**. Some use cases are:
- High performance, relational/non-relational databases.
- Distributed web scale cache stores.
- In-memory databases optimized for BI (business intelligence).
- Applications performing real-time processing of big unstructured data.

### 2.4.4 Instance Type - Storage Optimized

Great **for storage-intensive tasks that require high, sequential read and write access to large data sets on local storage**. Some appliations are:
- High frequency online transaction processing (OLTP) systems
- Relational & NoSQL databases
- Cache for in-memory databases (for example, Redis)
- Data warehousing applications
- Distributed file systems

## 2.5 Security Groups

Security groups are the fundamental of network security in AWS as they define rules that control how traffic is allowed into (inbound) or out (outbound) of EC2 instances.

![Example of security group](/assets/aws-certified-developer-associate/security_groups.png "Example of security group")

Security groups only contain **allow** rules as what is not defined is denied by default. Security groups rules can reference by IP or by security group.

Security groups act as a **firewall** on EC2 instances and regulate:
- Access to ports.
- Authorized IP ranges, both IPv4 and IPv6.
- Control of inbound network (from other to the instance).
- Control of outbound network (from the instance to other).

![Security groups table](/assets/aws-certified-developer-associate/security_groups_table.png "Security groups table")

**Note**:
- Security groups can be attached to multiple instances.
- An EC2 instance can have multiple security groups attached to it.
- Security groups are associated to a region or to a VPC within the region.
- Security groups do live outside the EC2 instance, meaning that if traffic is blocked the EC2 instance won't see it as the traffic will never reach the machine.
- It is better to maintain one separate security group for SSH access.
- **If an application is not accessible (timeout), then it is a security group issue**.
- If the application gives a connection refused error, then it is an application error or it is not launched but the security group is working fine.
- **All inbound traffic is blocked by default but outbound traffic is authorized by default**.

### 2.5.1 Security Groups Diagram

The diagram below shows the workings of security groups when applied to EC2 instances.

![Security groups diagram](/assets/aws-certified-developer-associate/security_groups_diagram.png "Security groups diagram")

The allowed request will reach the machine.
The blocked request will result in a timeout.

### 2.5.2 Referencing other Security Groups

We can **allow inbound traffic from/outboud traffic to other security groups**.

In the case of inbound traffic, we will allow any VM with the other security group to send traffic to our VM. On the opposite, the security group referenced in the outbound traffic will allow our VM to send traffic to the instance with the other security group.

The diagram below shows an example of references to other security groups for allowing inbound traffic:

![Referencing other Security Groups diagram](/assets/aws-certified-developer-associate/referencing_other_security_groups.png "Referencing other Security Groups diagram")

## 2.6 Ports to Know

- **22** = SSH (Secure Shell) - log into a Linux instance.
- **21** = FTP (File Transfer Protocol) – upload files into a file share.
- **22** = SFTP (Secure File Transfer Protocol) – upload files using SSH.
- **80** = HTTP – access unsecured websites.
- **443** = HTTPS – access secured websites.
- **3389** = RDP (Remote Desktop Protocol) – log into a Windows instance.

## 2.7 Creating Security Groups

You can create security group from the dedicated dashboard in the console.

![Create Security Groups](/assets/aws-certified-developer-associate/create_security_group.png "Create Security Groups")

By accessing the appropriate tab, it is possible to change the desired rules. The image above shows the *Inbound Rules* tab, which can be edited using the dashboard in the image below.

![Edit Security Groups rules](/assets/aws-certified-developer-associate/edit_security_group_rules.png "Edit Security Groups rules")

## 2.8 EC2 Instance Connect and SSH

It **provides access to EC2 instances directly from within the browser**. A useful thing is that EC2 Instance Connect will automatically manage temporary SSH keys (uploaded onto EC2 by AWS) to allow access to the VM. However, it is not available for all instances (works only out-of-the-box with Amazon Linux 2), so for some machines it is necessary to use SSH.

To connect to a machine using EC2 Instance Connect or SSH, it is essential that the machine has a security group allowing inbound SSH traffic to a desired port (default is 22).

**Note**: You should never enter your IAM access key ID and secret in an EC2 instance because everyone with access to that machine will be able to issue AWS commands on your behalf. Instead you should add an IAM Role to the EC2 instance from its *Security* options to provide the EC2 instance with AWS credentials to issue commands.

![Add IAM Role to EC2 instance](/assets/aws-certified-developer-associate/add_iam_role_to_ec2_instance.png "Add IAM Role to EC2 instance")

## 2.9 EC2 Instance Purchasing Options

- **On-Demand Instances**: meant for short workloads and offer predictable pricing with pay by second model.
- **Reserved (1 or 3 years term)**:
    - **Reserved Instances**: for long workloads.
    - **Convertible Reserved Instances**: for long workloads with flexible instance types, so for when you want to change the instance type of your VMs over time.
- **Savings Plans (1 or 3 years term)**: commitment to an amount of usage in terms of dollars, and are meant for long workload.
- **Spot Instances**: for short workloads, cheap but you can lose instances at any time, making them less reliable.
- **Dedicated Hosts**: book an entire physical server with control over instance placement.
- **Dedicated Instances**: the hardware is dedicated to you, so no other customers will share your hardware.
- **Capacity Reservations**: reserve capacity in a specific AZ for any duration.

### 2.9.1 EC2 On-Demand Instances

You pay for what you use:
- Linux or Windows: billing per second, after the first minute.
- All other operating systems: billing per hour.

It has the **highest cost among purchasing options but no upfront payment**. It is not meant for long-term commitment as it is **recommended for short-term and un-interrupted workloads**, where
you can't predict how the application will behave.

### 2.9.2 EC2 Reserved Instances

They offer up to X% (72% currently) discount compared to on-demand instances.

You reserve a specific instance attribute, such as instance type, region, tenancy, and OS.

The **reservation period** can be:
- 1 year (+ discount).
- 3 years (+++ discount).

There are different **payment options**:
- No Upfront (+ discount).
- Partial Upfront (++ discount).
- All Upfront (+++ discount).

Reserved Instance have a **scope**, meaning you can reserve capacity at regional or zonal (reserve capacity in an AZ) level.

They are **recommended for steady-state usage applications (think database)** and you can buy and sell reserved machines in the **Reserved Instance Marketplace**.

An offering of reserved instances are **Convertible Reserved Instance**, which offer you the flexibility of changing the instance type, instance family, OS, scope and tenancy. However, this flexibility comes at the cost of less discount (66% currently), they cost more than normal reserved instances.

### 2.9.3 EC2 Savings Plans

They offer a discount based on long-term usage but you do not commit to a certain instance attribute. You **commit to a certain type of usage, for example $10/hour for 1 or 3 years**. **Usage beyond the savings plan is billed at the on-demand price**.

You are **locked to a specific instance family and AWS region (e.g., m5 in us-east-1)** but you are flexible across:
- Instance Size (e.g., m5.xlarge, m5.2xlarge).
- OS (e.g., Linux, Windows).
- Tenancy (Host, Dedicated, Default).

### 2.9.4 EC2 Spot Instances

They offer discount of up to 90% compared to on-demand instances.

The are instances that **you can lose at any point of time if your max price** (the price you are paying for them) **is less than the current spot price**.

These are the **most cost-efficient instances in AWS and are useful for workloads that are resilient to failure**, such as batch jobs, data analysis, image processing, any distributed workloads, workloads with a flexible start and end time.

Mind that **they are not suitable for critical jobs or databases** and **the exam tests you on this**.

### 2.9.5 EC2 Dedicated Hosts

They **provide a physical server** with EC2 instance capacity fully dedicated to your use and allow you to address compliance requirements and use your existing server-bound software licenses (per-socket, per-core, per-VM software licenses).

**Purchasing options**:
- On-demand: pay per second for active Dedicated Host.
- Reserved: 1 or 3 years (No Upfront, Partial Upfront, All Upfront).

It is **the most expensive option** but it is **useful for software that have complicated licensing model (BYOL - Bring Your Own License) or for companies that have strong regulatory or compliance needs**.

### 2.9.6 EC2 Dedicated Instances

Instances run on hardware that’s dedicated to you, so **the hardware is not shared with other customers**. Not that you may share hardware with other instances in the same account.

![EC2 Dedicated Hosts VS Dedicated Instances](/assets/aws-certified-developer-associate/dedicated_hosts_vs_instances.png "EC2 Dedicated Hosts VS Dedicated Instances")

### 2.9.7 EC2 Capacity Reservations

Reserve on-demand instances capacity in a specific AZ for any duration. You **always have access to EC2 capacity in the AZ when you need it but you are charged for it at the on-demand price even if you are not running any instance**.

There is no time commitment (create/cancel anytime) and no billing discounts. You can combine it with Reserved Instances and Savings Plans to benefit from billing discounts.

**They are suitable for short-term, uninterrupted workloads that needs to be in a specific AZ**.

### 2.9.8 Example of Price Comparison

The following is a price comparison for an *m4.large* in *us-east-1* region:

![Example of Price Comparison](/assets/aws-certified-developer-associate/ec2_price_comparison.png "Example of Price Comparison")
