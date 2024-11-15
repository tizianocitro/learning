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
- From: t2.nano with 0.5G of RAM and 1 vCPU (the smallest)
- To: u-12tb1.metal with 12.3 TB of RAM and 448 vCPUs (the biggest)

**Horizontal scaling**: changing the number of instances. Scale out when increasing the the number of instances, scale in when decreasing it.
- **Auto Scaling Group (ASG)**: increase/decrease the number of instances.
- **Load Balancer**: distribute work across multiple instances.

**High availability**: run instances for the same application across multiple AZs.
- Auto Scaling Group with multi-AZ enabled.
- Load Balancer with multi-AZ enabled.

## 4.4 Elastic Load Balancer (ELB)

Load Balances are **servers that forward traffic to multiple servers (e.g., EC2 instances) downstream** and provide one single point of access to all of them.

![ELB](/assets/aws-certified-developer-associate/elb.png "ELB")

**Why use an ELB?**
- Spread load across multiple downstream instances.
- Expose a single point of access (DNS) to your application.
- Seamlessly handle failures of downstream instances because ELB will stop sending traffic to unhealthy instances (monitoring their health).
- Do regular health checks to your instances.
- Provide SSL termination (to have HTTPS) for your websites.
- Enforce stickiness with cookies.
- High availability across zones.
- Separate public traffic from private traffic.

ELB is a managed load balancer that AWS takes care of.
- AWS guarantees that it will be working.
- AWS takes care of upgrades, maintenance, high availability.
- AWS provides only a few configuration knobs.

It cost less to setup your own load balancer on EC2 instances, but it will be less reliable, you will have to manage it yourself, and it will not be as scalable. So, it will require a lot more effort on your end.

It is integrated with many AWS services, some are: 
- EC2, ECS, EKS, Lambda.
- AWS Certificate Manager (ACM), CloudWatch. Route 53.

### 4.4.1 Health Checks

Health Checks are crucial for load balancers as they enable the load balancer to know if instances it forwards traffic to are available to reply to requests.

The health check is done on a port and a route (/health is common) and if the response is not 200 (OK), then the instance is unhealthy.

![Health Check example](/assets/aws-certified-developer-associate/elb_health_checks.png "Health Check example")

### 4.4.2 Load Balancer Security Groups

**Load balancers will have a security group attached to them that allows traffic on specific ports from anywhere** (e.g.,  port 80 for HTTP and port 443 for HTTPS).

Then, **each EC2 instance behind the load balancer will also have a security group attached to them that allows traffic only from the security group of the load balancer**.

![ELB Security Groups](/assets/aws-certified-developer-associate/elb_security_groups.png "ELB Security Groups")

## 4.5 Types of Load Balancers on AWS

AWS has **4 types** of load balancers:
1. **Classic Load Balancer (CLB)** (v1, old generation) from 2009.
    - Supports HTTP, HTTPS, TCP, and SSL but it is recommended to use the newer generation as this is getting deprecated.
    - Not part of the exam anymore.
2. **Application Load Balancer (ALB)** (v2, new generation) from 2016.
    - Supports HTTP, HTTPS, and WebSocket.
3. **Network Load Balancer (NLB)** (v2, new generation) from 2017. 
    - Supports TCP, TLS, and UDP.
4. **Gateway Load Balancer (GWLB)** from 2020.
    - Operates at layer 3 (Network layer) on IP Protocol.

Overall, it is recommended to use the newer generation load balancers as they provide more features.

Some load balancers can be setup as **internal** (private) or **external** (public) ELBs.

## 4.6 Application Load Balancer (ALB)

ALB is a **Layer 7 (HTTP) load balancer**. It supports:
- Load balancing to multiple HTTP applications across machines in target groups.
- Load balancing to multiple applications on the same machine (e.g., containers).
- HTTP/2 and WebSocket.
- Redirects, e.g., from HTTP to HTTPS.

It routes requests based on the content of the request to different **target groups**:
- **Routing based on path in URL**: example.com/users and example.com/posts can be routed to different target groups.
- **Routing based on hostname in URL**: primary.example.com and secondary.example.com can be routed to different target groups.
- **Routing based on query string and headers**: for example, a query string such as example.com/users?id=123&order=false.

![ALB Routing](/assets/aws-certified-developer-associate/alb_routing.png "ALB Routing")

ALB is a **great fit for microservices and container-based application** (Docker/Amazon ECS) because it has a port mapping feature to redirect to a dynamic port in ECS, allowing us to **have only one ALB for multiple containers/applications**. In comparison, we used to need multiple Classic Load Balancer per application.

### 4.6.1 ALB Target Groups

Target groups are **used to route requests to one or more registered targets**. They are used to route requests to:
- EC2 instances, which can also be managed by an Auto Scaling Group on HTTP.
- ECS tasks, managed by ECS itself on HTTP.
- Lambda functions where HTTP requests are translated into JSON events.
- IP addresses, which must be private IP addresses.

An ALB can **route to multiple target groups** but **health checks are done at the target group level**.

### 4.6.2 ALB Target Groups By Query String

In the following example, the ALB routes requests to two target groups based on the query string. The first target group is for users accessing the application from mobile devices, and the second target group is for users accessing the application from desktop devices.

![ALB Target Groups](/assets/aws-certified-developer-associate/alb_target_groups.png "ALB Target Groups")

### 4.6.3 Client to ALB to EC2 Communication

ALB will provide a fixed hostname like XXX.region.elb.amazonaws.com.

ALB performs **connection termination**, so the application servers do not see the client's IP directly as the true IP of the client is inserted in the header *X-Forwarded-For*, while you can get the port from *X-Forwarded-Port* and protocol from *X-Forwarded-Proto*.

![ALB Client to EC2 Communication](/assets/aws-certified-developer-associate/alb_client_to_ec2_communication.png "ALB Client to EC2 Communication")

### 4.6.4 Launching an ALB

First thing is to launch EC2 instances to route traffic to.

To create load balancers, go to the EC2 dashboard and click on *Load Balancers* on the left side menu. Then, click on *Create Load Balancer* and select *Application Load Balancer*.

![Create ALB](/assets/aws-certified-developer-associate/create_alb.png "Create ALB")

To select the load balancer type there is a dedicated wizard:

![Select ALB Type](/assets/aws-certified-developer-associate/select_alb_type.png "Select ALB Type")

The main **difference between ALB and NLB is the protocol they operate on**. The ALB operates on HTTP and HTTPS, while the NLB operates on TCP, TLS, and UDP, and is used for extreme performance (high number of requests per second with low latency).

The Gateway Load Balancer instead serves different purposes, such as security, intrusion detection, firewall, etc. So, its objective is to analyze the network traffic.

Then, configure the ALB (some properties are presented below):

![Configure ALB](/assets/aws-certified-developer-associate/configure_alb.png "Configure ALB")

And it is important to properly set the network mapping, indicating the AZs where the ALB will be available:

![ALB Network Mapping](/assets/aws-certified-developer-associate/alb_network_mapping.png "ALB Network Mapping")

Then, create and assign a security group to the ALB:

![ALB Security Group](/assets/aws-certified-developer-associate/alb_security_group.png "ALB Security Group")

Next step is to **create and assign a target group** (containing the EC2 instances we launched beforehand) to the ALB. So, select the *Instances* target type to route to your EC2 instances:

![ALB Target Group Configuration](/assets/aws-certified-developer-associate/alb_target_group_configuration.png "ALB Target Group Configuration")

And set the target group's protocol you need, for example HTTP, and the port is the port the target is listening on.

![ALB Target Group Protocol](/assets/aws-certified-developer-associate/alb_target_group_protocol.png "ALB Target Group Protocol")

Then, configure the health checks in the target group:

![ALB Health Checks](/assets/aws-certified-developer-associate/alb_health_checks.png "ALB Health Checks")

Finally, register the targets (EC2 instances) to the target group by paying attention to the port where the instances will have to listen:

![ALB Register Targets](/assets/aws-certified-developer-associate/alb_register_targets.png "ALB Register Targets")

Done that, you can create the target group to assign it to the ALB:

![Assign Target Group to ALB](/assets/aws-certified-developer-associate/alb_assign_target_group.png "Assign Target Group to ALB")

With the target group created and assigned, you can create the ALB. To access the created ALB, use the *Load Balancers* section in the EC2 dashboard.

![ALB Created](/assets/aws-certified-developer-associate/alb_created.png "ALB Created")

The created ALB will have a DNS name that you can use to access the applications running on the EC2 instances in the target group with the benefit of load balancing. So, if the EC2 instances are healthy, the ALB will route the traffic to them.

You can see the status of the instances in the target group in the *Targets* tab of the target group:

![ALB Target Status](/assets/aws-certified-developer-associate/alb_target_status.png "ALB Target Status")

And **if one instance is stopped or fails, its status will be marked as unused** and the ALB will not route traffic to it until it becomes healthy again.

![ALB Target Unhealthy](/assets/aws-certified-developer-associate/alb_target_unhealthy.png "ALB Target Unhealthy")

### 4.6.5 Network Security in ALB

With the current setup, you can access instances directly using their public IP. However, it is recommended to **restrict access to the instances only through the ALB**. To do so, you can change the security group of the instances to **allow traffic only from the security group of the ALB**.

### 4.6.6 ALB Rules

You can add other rules on the ALB listeners to customize the routing of the traffic.

![ALB Rules](/assets/aws-certified-developer-associate/alb_rules.png "ALB Rules")

Each rule has **one or more conditions** indicating what do you want to filter with the rule. For example, you can filter by the path, the host, the HTTP header, the query string, etc.

![ALB Rule Condition](/assets/aws-certified-developer-associate/alb_rule_condition.png "ALB Rule Condition")

For example, you can filter paths that contain */error*.

![ALB Rule Path Filter](/assets/aws-certified-developer-associate/alb_rule_path_filter.png "ALB Rule Path Filter")

And this is how the rule will look like at this point:

![ALB Rule Example](/assets/aws-certified-developer-associate/alb_rule_example.png "ALB Rule Example")

Next step is to add an **action** to the rule. The action can be to forward the request to a specific target group, to redirect the request to a different URL (for example, redirect HTTP to HTTPS), or to return a fixed response.

![ALB Rule Action](/assets/aws-certified-developer-associate/alb_rule_action.png "ALB Rule Action")

For the forward to a target group action, you can select the target group(s) you want to forward the request to, as well as the weight of each target group. For example, if you have two target groups with weights 1 and 2, the traffic will be distributed as 33.33% and 66.67%, respectively.

Finally, you need to set the **priority** of the rule, so that if a request matches multiple rules, the rule with the highest priority will be applied.

## 4.7 Network Load Balancer (NLB)

NLB operates at **Layer 4** allowing you to:
- Forward TCP and UDP traffic to your instances (**when you read TCP/UDP in your exam, think NLB**).
- Handle millions of request per seconds.
- Benefit from ultra-low latency.

NLB has **one static IP per AZ** but you can assign Elastic IPs to AZs. Elastic IPs are useful for when you need to expose your application via a set of static IP addresses.

**If the exams mentions that your application should be accessed via a limited set of static IPs, think about NLB as an option**.

![NLB](/assets/aws-certified-developer-associate/nlb.png "NLB")

As you can see from the image above, you can use TCP in front of the load balancer but still use HTTP/HTTPS behind the load balancer.

### 4.7.1 NLB Target Groups

NLB target groups can target:
- **EC2 instances**.
- **IP addresses**: must be private IPs that can also point to on-premises.
- **Application Load Balancers**: you can use a NLB in front of an ALB so that you can get fixed/static IP addresses thanks to the NLB and can use ALB for custom rules on HTTP traffic.

![NLB Target Groups](/assets/aws-certified-developer-associate/nlb_target_groups.png "NLB Target Groups")

**Important for the exam: health checks support the TCP, HTTP and HTTPS Protocols**.

### 4.7.2 Launching a NLB

Same process as the ALB but in the wizard, select *Network Load Balancer*.

Most configuration are the same as the ALB but you are going to get a **fixed IP address for each AZ where the NLB will be available**.

Obviosuly, you can choose only amon TCP, TLS, and UDP protocols:

![NLB Protocols](/assets/aws-certified-developer-associate/nlb_protocols.png "NLB Protocols")

Here are also some example of **advanced configuration options for health checks**:

![NLB Health Checks](/assets/aws-certified-developer-associate/nlb_health_checks.png "NLB Health Checks")

And this is an example of NLB (as you can see, it also has a DNS name):

![NLB Created](/assets/aws-certified-developer-associate/nlb_created.png "NLB Created")

Remmber to add the NLB security group to the instances' security group to allow traffic from the NLB to reach the instances.

![NLB Security Group](/assets/aws-certified-developer-associate/nlb_security_group.png "NLB Security Group")