# 4 AWS Fundamentals: ELB and ASG

## 4.1 Scalability

Scalability means that an application/system can handle greater loads by adapting.

There are two kinds of scalability: vertical scalability and horizontal scalability (also called **elasticity**).

**Vertical scalability** is very common for non distributed systems, such as databases. For example, with vertical scaling, if your application runs on a t2.micro, vertically scaling that application means running it on a t2.large. 

On AWS, RDS and ElastiCache are services that can scale vertically by upgrading the underlining instance type. However, there is usually a limit to how much you can vertically scale due to hardware limits.

**Horizontal scalability** means increasing the number of instances/systems for your application.

Horizontal scaling implies distributed systems and is very common for web applications/modern applications. An example is increased by scaling the number of EC2 instances powering the backend of an application.

## 4.2 High Availability

High Availability usually goes hand in hand with horizontal scaling. High availability means running your application/system in at least 2 data centers/AZs.

The high availability can be:
- Passive (RDS Multi-AZ for example).
- Active (for horizontal scaling).

## 4.3 Scalability and High Availability for EC2

**Vertical scaling**: modify instance size. Scale up when increasing instance size. scale down when decreasing it.
- From: t2.nano - 0.5G of RAM, 1 vCPU (the smallest)
- To: u-12tb1.metal – 12.3 TB of RAM, 448 vCPUs (the biggest)

**Horizontal scaling**: changing the number of instances. Scale out when increasing the the number of instances, scale in when decreasing it.
- **Auto Scaling Group (ASG)**: increase/decrease the number of instances.
- **Load Balancer**: distribute work across multiple instances.

**High availability**: run instances for the same application across multiple AZs.
- Auto Scaling Group with multi-AZ enabled.
- Load Balancer with multi-AZ enabled.