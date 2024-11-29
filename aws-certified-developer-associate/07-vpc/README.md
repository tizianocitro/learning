# 7 Virtual Private Cloud (VPC)

**A Virtual Private Cloud (VPC) is a private network to deploy your resources** and is a **regional resource**.

A VPC has a CIDR range that defines the range of IP addresses that can be used in the VPC.

## 7.1 Subnets

**Subnets allow you to partition your network inside your VPC** and are **AZ resources**. You can have multiple subnets in a VPC.

Two types of subnets:
- **Public subnet**: a subnet that is accessible from the internet.
- **Private subnet**: a subnet that is not accessible from the internet.

To define access to the internet and between subnets, we use **route tables**.

The below image shows a VPC with 2 subnets (public and private):

![VPC with 2 subnets](/assets/aws-certified-developer-associate/vpc_subnets.png "VPC with 2 subnets")

How it works in AWS:

![VPC in AWS](/assets/aws-certified-developer-associate/vpc_aws.png "VPC in AWS")

## 7.2 Internet Gateway (IGW) and NAT Gateway

**Internet gateways helps our VPC instances connect with the internet**.

**Public subnets have a route to the internet gateway**, this is how they can access the internet.

**To allow private subnets to access the internet, we can use a NAT gateways**. You need to deploy a NAT gateway in a public subnet and add a route to the NAT gateway in the private subnet. Then, you need to create a route from the NAT gateway to the internet gateway, which allows the private subnet to access the internet.

**In alternative to NAT gateways, you can use NAT instances**. The main difference is that NAT instances are managed by you, while NAT gateways are managed by AWS.

![IGW and NAT Gateway](/assets/aws-certified-developer-associate/igw_nat_gateway.png "IGW and NAT Gateway")

## 7.3 Network Access Control Lists (NACLs) and Security Groups

**NACLs (Network ACLs)**:
- A firewall which **controls traffic from and to the subnet**.
- Can have **ALLOW and DENY rules**.
- Are **attached at the subnet level**.
- Rules **only include IP addresses**.
- **Are stateless**: if you allow inbound traffic, you must allow outbound traffic.
- The default NACL in the default VPC allows all inbound and outbound traffic.

**Security groups**:
- A firewall that **controls traffic to and from an Elastic Network Interface (ENI)/EC2 Instance**.
- Can have **only ALLOW rules**.
- Rules **include IP addresses and other security groups**.
- **Are stateful**: if you allow inbound traffic, outbound traffic (the return traffic) is automatically allowed.

As you can see from the image below, **before reaching EC2 instances/AWS resources or leaving the subnet, traffic must pass through the NACL**.

![NACLs and Security Groups](/assets/aws-certified-developer-associate/nacls_security_groups.png "NACLs and Security Groups")

NACLs vs security groups:

![NACLs vs Security Groups](/assets/aws-certified-developer-associate/nacls_vs_security_groups.png "NACLs vs Security Groups")

## 7.4 VPC Flow Logs

This allows you to **capture information about IP traffic going into your interfaces**:
- VPC Flow Logs.
- Subnet Flow Logs.
- Elastic Network Interface Flow Logs.

It helps to monitor and troubleshoot connectivity issues:
- Subnets to internet: why the subnet is not connecting to the internet.
- Subnets to subnets: why the subnet is not connecting to another subnet.
- Internet to subnets: why the internet cannot reach the subnet.

It captures network information from AWS managed interfaces too: ELB, ElastiCache, RDS, Aurora, etc.

VPC Flow logs data can go to S3, CloudWatch Logs, and Kinesis Data Firehose.

## 7.5 VPC Peering

VPC peering allows you to **privately connect two VPCs**. It makes them behave as if they were in the same network, which **allows you to route traffic between them using private IP addresses**. These **VPCs can be also in different regions**.

![VPC Peering](/assets/aws-certified-developer-associate/vpc_peering.png "VPC Peering")

It is important that CIDR (IP address range) defined in the VPCs do not overlap.

**VPC peering connection is not transitive**, so it must be established for each VPC that need to communicate with one another. For example, if you connect VPC 1 to VPC 2 and VPC 2 to VPC 3, VPC 1 and VPC 3 are not connected, you still need to connect VPC 1 to VPC 1.

As a result, the more VPCs you want to connect, the more VPC peering connections you need to establish.

## 7.6 VPC Endpoints

**VPC Endpoints allow you to connect to AWS services using a private network instead of the public internet network**. All AWS services are reachable from the public internet, so when you connect to them, you are using the public internet. However, there are cases where the resources are in a private subnet (where they have no access to the public internet) and you want to connect to AWS services without going through the public internet.

In general, VPC Endpoints give you enhanced security and lower latency to access AWS services.

There are two types of VPC Endpoints:
- **VPC Endpoint Gateway**: only for S3 and DynamoDB.
- **VPC Endpoint Interface**: for the rest of the AWS services.

![VPC Endpoints](/assets/aws-certified-developer-associate/vpc_endpoints.png "VPC Endpoints")

## 7.7 Site to Site VPN and Direct Connect

**Site to Site VPN**:
- Connect an on-premises VPN to AWS.
- The connection is automatically encrypted.
- Goes over the public internet.

**Direct Connect**:
- Establish a physical connection between on-premises and AWS.
- The connection is private, secure and fast.
- Goes over a private network, so no public internet.
- Takes at least a month to establish because it requires work to establish a physical connection.

![Site to Site VPN and Direct Connect](/assets/aws-certified-developer-associate/site_to_site_vpn_direct_connect.png "Site to Site VPN and Direct Connect")

## 7.8 AWS 3 Tier Architecture

Our applications typically have an ELB that distributes traffic to EC2 instances. The ELB needs to be public, so it needs to be in a public subnet. To access the ELB, you need to do a DNS query using Route 53.

The ELB spreads traffic acroos EC2 instances that need to scale, so they will be in an auto-scaling group. Given the EC2 instances need to communicate only with the ELB, they can be in a private subnet. So, the EC2 instances will be spread across multiple AZs with one subnet per AZ.

Then, we will have another private subnet for the data/presistence layer (RDS, ElastiCache, etc). This is usually also called data subnet.

![AWS 3 Tier Architecture](/assets/aws-certified-developer-associate/aws_3_tier_architecture.png "AWS 3 Tier Architecture")

## 7.9 LAMP Stack on EC2

- **L**inux: OS for EC2 instances.
- **A**pache: web server that run on Linux on EC2.
- **M**ySQL: database on RDS.
- **P**HP: spplication logic running on EC2.

To these, you can add:
- Redis/Memcached via ElastiCache to include a caching layer.
- EBS drive to store local application data and software.

## 7.10 WordPress on AWS

A simple WordPress architecture on AWS:

![WordPress on AWS](/assets/aws-certified-developer-associate/wordpress_on_aws.png "WordPress on AWS")

But AWS provides a more complex and scalable architecture for WordPress:

![WordPress on AWS Scalable](/assets/aws-certified-developer-associate/wordpress_on_aws_scalable.png "WordPress on AWS Scalable")

## 7.11 VPC Summary for the Exam

- **Subnets**: tied to an AZ, network partition of the VPC.
- **Internet Gateway**: at the VPC level, provide Internet Access.
- **NAT Gateway/Instances**: give internet access to private subnets.
- **NACL**: stateless, subnet rules for inbound and outbound.
- **Security groups**: stateful, operate at the EC2 instance level or ENI.
- **VPC Peering**: connect two VPC with non overlapping IP ranges, non transitive.
- **VPC Endpoints**: provide private access to AWS Services within VPC.
- **VPC Flow Logs**: network traffic logs.
- **Site to Site VPN**: VPN over public internet between on-premises and AWS.
- **Direct Connect**: private connection between on-premises and AWS.