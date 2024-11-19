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

## 4.8 Gateway Load Balancer (GWLB)

Deploy, scale, and manage a fleet of 3rd party network virtual appliances in AWS. You want to **use GWLB when you need all the traffic in your network to go through a network of virtual appliances**, such as firewalls, intrusion detection and prevention systems, deep packet inspection systems, and payload manipulation.

All the process is transparent to client/application.

![GWLB](/assets/aws-certified-developer-associate/gwlb.png "GWLB")

This is possible because the GWLB operates at the **network layer (Layer 3)**, so it can route traffic based on IP protocol fields. The GWLB will act as:
- **Transparent Network Gateway** – single entry/exit point for all traffic in the network.
- **Load Balancer**: distributes traffic to your virtual appliances.

**If the exam questions you on using the GENEVE protocol on port 6081, it will be the GWLB**.

### 4.8.1 GWLB Target Groups

GWLB target groups can target:
- **EC2 instances**.
- **IP addresses**: must be private IPs that can also point to on-premises.

![GWLB Target Groups](/assets/aws-certified-developer-associate/gwlb_target_groups.png "GWLB Target Groups")

## 4.9 ELB Sticky Sessions (a.k.a. Session Affinity)

Sticky sessions means that the same client will always be redirected to the same instance behind a load balancer. This is useful to avoid users losing their session data.

It is possible to get sticky sessions with CLB, ALB, and NLB.

In ALB and CLB, it works by using a **cookie** that is inserted by the load balancer into the client's browser. The cookie contains the instance ID. So, when the client comes back, the cookie is read and the client is sent to the same instance. The cookie has an **expiration date**.

In the NLB, it works by using the **source IP** of the client to bind the client to an instance. So, the client will always be sent to the same instance as long as the instance is healthy.

Important to note that sticky sessions can cause **imbalaced load distribution** across instances.

### 4.9.1 ELB Sticky Sessions Cookies

There are two types of cookies for sticky sessions.

**Application-based Cookies** can be either:
- **Custom Cookie**, which is generated by the target (which also controls the expiration):
    - Can include any custom attributes required by the application.
    - Cookie name must be specified individually for each target group and cannot use *AWSALB*, *AWSALBAPP*, or *AWSALBTG* because they are reserved for use by the ELB.
- **Application Cookie**, which is generated by the load balancer with the name *AWSALBAPP*.

**Duration-based Cookies**, which are generated by the load balancer with the name *AWSALB* for ALB and *AWSELB* for CLB.
- Contrary to application-based cookies, the cookie's expiration is set by ELB.

### 4.9.2 ELB Sticky Sessions Configuration

You need to enable sticky sessions in the load balancer at the target group level.

![Sticky Sessions Configuration](/assets/aws-certified-developer-associate/sticky_sessions_configuration.png "Sticky Sessions Configuration")

This will allow you to set the stickiness type. For example, load balancer-generated cookie (you can inspect the cookie in the browser):

![Sticky Sessions Type](/assets/aws-certified-developer-associate/sticky_sessions_type.png "Sticky Sessions Type")

Or you can have application-based cookie, where you need to add a custom cookie name:

![Sticky Sessions Application Cookie](/assets/aws-certified-developer-associate/sticky_sessions_application_cookie.png "Sticky Sessions Application Cookie")

## 4.10 Cross-Zone Load Balancing

![Cross-Zone Load Balancing Enabled](/assets/aws-certified-developer-associate/cross_zone_load_balancing_enabled.png "Cross-Zone Load Balancing Enabled")

![Cross-Zone Load Balancing Disabled](/assets/aws-certified-developer-associate/cross_zone_load_balancing_disabled.png "Cross-Zone Load Balancing Disabled")

**ALB has cross-zone load balancing enabled by default**. However, there is **no charge for data transfer between AZs** (which usually is not the default case for AWS). You **cannot disable it from the ALB attributes, you need to do it at the target group level**.

![Cross-Zone Load Balancing ALB](/assets/aws-certified-developer-associate/sticky_sessions_configuration.png "Cross-Zone Load Balancing ALB")

![Cross-Zone Load Balancing ALB Options](/assets/aws-certified-developer-associate/cross_zone_load_balancing_alb.png "Cross-Zone Load Balancing ALB Options")

**NLB and GWLB have cross-zone load balancing disabled by default** and you **pay for data transfer** between AZs. Cross-zone load balancing can be enabled/disabled from the NLB/GWLB attributes.

![Cross-Zone Load Balancing NLB/GWLB](/assets/aws-certified-developer-associate/cross_zone_load_balancing_nlb_gwlb.png "Cross-Zone Load Balancing NLB/GWLB")

CLB has cross-zone load balancing disabled by default but you do not pay for data transfer between AZs even if you enable it.

## 4.11 ELB SSL/TLS Certificates

An SSL Certificate allows traffic between your clients and your load balancer to be encrypted in transit (in-flight encryption). Nowadays, TLS certificates are mainly used, but people still refer to them as SSL certificates.

Public SSL certificates are issued by Certificate Authorities (e.g., Comodo, GoDaddy, Digicert, Letsencrypt, ...) and have an expiration date that you set and must be renewed.

Load balancers are responsible for **SSL termination**, which means that the connection between the clients and the load balancer is encrypted, but the connection between the load balancer and the EC2 instances can also not be encrypted. For example:

![SSL/TLS Certificates](/assets/aws-certified-developer-associate/ssl_tls_certificates.png "SSL/TLS Certificates")

The load balancer uses an **X.509 certificate (SSL/TLS server certificate)**. Such certificates can be managed using AWS Certificate Manager (ACM) or you can create/upload your own certificates.

When you create an **HTTPS listener, you must specify a default certificate**.
- You can add an optional list of certificates to support multiple domains.
- Clients can use **Server Name Indication (SNI)** to specify the hostname they reach.
- Ability to specify a security policy to support older versions of SSL/TLS (also called legacy clients).

### 4.11.1 Server Name Indication (SNI)

SNI **solves the problem of loading multiple SSL certificates onto one web server** to serve multiple websites.

It is a newer protocol and **requires the client to indicate the hostname of the target server in the initial SSL handshake**. This allows the server to determine the correct certificate to present, or return the default one.

![SNI](/assets/aws-certified-developer-associate/sni.png "SNI")

**Note**:
- Only works for ALB and NLB (newer generation) and CloudFront.
- Does not work for CLB (older generation).

**For the exam, when you see multiple SSL certificates onto your load balancer, think ALB or NLB with SNI**.

### 4.11.2 SSL Certificates Configurations

**Application Load Balancer (v2)** and **Network Load Balancer (v2)**:
- Support multiple listeners with multiple SSL certificates.
- Use Server Name Indication to make it work.

**Classic Load Balancer (v1)**:
- Supports only one SSL certificate.
- Must use multiple CLBs for multiple hostname with multiple SSL certificates.

### 4.11.3 Add SSL/TLS Certificates to ALB/NLB

You add a new HTTPS listener to the load balancer and then you can add a certificate to the listener. The policy is used to define how you negotiate the certificate.

![Add SSL/TLS Certificate](/assets/aws-certified-developer-associate/add_ssl_tls_certificate.png "Add SSL/TLS Certificate")

You also need an action for HTTPS listeners:

![HTTPS Listener Action](/assets/aws-certified-developer-associate/https_listener_action.png "HTTPS Listener Action")

## 4.12 Connection Draining/Deregistration Delay

This is a feature that can come up during the exam. It is called **Connection Draining** in CLB and **Deregistration Delay** in ALB and NLB.

This feature ensures that **the load balancer will stop sending new requests to the instance that is draining/deregistering (going unhealthy) but will still complete in-flight requests to that instance**. 

**The load balancer will give time for the instance to complete the existing requests**. Any new request will not be sent to that instance.

![Connection Draining](/assets/aws-certified-developer-associate/connection_draining.png "Connection Draining")

The time given by the load balancer to complete the in-flight requests goes **from 1 to 3600 seconds (default is 300 seconds)**.
- It can be disabled by setting the time to 0.
- Set the time to a low value if your requests are short.

## 4.13 Auto Scaling Group (ASG)

The goal of an ASG is to:
- **Scale out** (add EC2 instances) to match an increased load.
- **Scale in** (remove EC2 instances) to match a decreased load.
- Ensure you have a minimum and a maximum number of EC2 instances running.
- **Automatically register new instances to a load balancer** (if any). When an ASG is linked to an ELB, the ASG will automatically register new instances in the ASG to the ELB.
- Re-create an EC2 instance in case a previous one is terminated, for example, if they fail the ELB health check and are deemed unhealthy.

**ASG are free, meaning you only pay for the underlying EC2 instances**.

The following is a configuration example of an ASG:

![ASG](/assets/aws-certified-developer-associate/asg.png "ASG")

As said, ASG works with ELB:

![ASG with ELB](/assets/aws-certified-developer-associate/asg_with_elb.png "ASG with ELB")

### 4.13.1 ASG Configuration

When you create an ASG, you need to configure:
- A **Launch Template** (older Launch Configurations are deprecated) with:
    - AMI and instance type.
    - EC2 user data.
    - EBS volumes.
    - Security groups.
    - SSH key pair.
    - IAM roles for the EC2 instances.
    - Network and subnets information.
    - Load balancer information.
    ![ASG Launch Template](/assets/aws-certified-developer-associate/asg_launch_template.png "ASG Launch Template")
- Minimum size.
- Maximum size.
- Initial capacity.
- Scaling policies (e.g., via CloudWatch alarms).

### 4.13.2 Scaling Policies with CloudWatch Alarms

It is possible to auto scale an ASG based on **CloudWatch alarms**.

An **alarm monitors a metric** (such as Average CPU or a custom metric) and metrics such as Average CPU are computed for the overall ASG instances.

Based on the alarm, you can create **scaling policies**. You can:
- Create **scale-out policies** (increase the number of instances).
- Create **scale-in policies** (decrease the number of instances).

## 4.14 Creating an ASG

To create an ASG, go to the EC2 dashboard and click on *Auto Scaling Groups* on the left side menu. Then, click on *Create Auto Scaling Group*.

Start by creating (or selecting) the launch template via the *Create a Launch Template* button:

![Create ASG Launch Template](/assets/aws-certified-developer-associate/create_asg_launch_template.png "Create ASG Launch Template")

Creating a launch requires to specify some information. Following the AMI:

![ASG Launch Template AMI](/assets/aws-certified-developer-associate/asg_launch_template_ami.png "ASG Launch Template AMI")

Then, the instance type, key pair, network settings (subnets), storage, and advanced details (e.g., user data).

After the launch template is created, you can select it in the ASG creation wizard and move to setting the network and **subnets (the AZs where the instances will be launched)**.

Next step is **advanced options**, where you can configure load balancing, health checks, and other additional settings like monitoring on CloudWatch.

![ASG Advanced Options](/assets/aws-certified-developer-associate/asg_advanced_options.png "ASG Advanced Options")

Then, configure the **group size settings**: minimum, desired, and maximum capacity.

![ASG Group Size](/assets/aws-certified-developer-associate/asg_group_size.png "ASG Group Size")

Finally, configure the **scaling policies** (optional) and create the ASG to see it in the list of ASGs in the EC2 dashboard.

![ASG Created](/assets/aws-certified-developer-associate/asg_created.png "ASG Created")

All the activities of the ASG are recorded in the *Activity History* tab of the ASG.

![ASG Activity History](/assets/aws-certified-developer-associate/asg_activity_history.png "ASG Activity History")

Then, in the *Instance management* tab, you can see the instances that are part of the ASG. The same instances will also appear as registered targets in the ELB (if you connect one to the ASG).

## 4.15 Scaling Policies

There are 3 types of scaling policies: **dynamic scaling**, **scheduled scaling**, and **predictive scaling**.

### 4.15.1 Dynamic Scaling

Two types of dynamic scaling: target tracking scaling and simple/step scaling.

**Target tracking scaling**: the most simple and easy to set up. You **select a metric and set a target value, and AWS will try to keep the metric close to the target value**.
- For example, I want the average ASG CPU to stay at around 40%.

**Simple/Step scaling**: the idea is that you can increase or decrease the ASG size **based on a CloudWatch alarm**. For example:
- When a CloudWatch alarm is triggered (example CPU > 70%), then add 2 units.
- When a CloudWatch alarm is triggered (example CPU < 30%), then remove 1 unit.

### 4.15.2 Scheduled Scaling

Anticipate a scaling based on known usage patterns. For example, increase the min capacity to 10 at 5 pm on Fridays because you know you will get more traffic then.

### 4.15.3 Predictive Scaling

AWS uses machine learning to understand your traffic patterns and make sure that you have the right number of instances.

AWS continuously forecasts load and schedule scaling ahead:

![Predictive Scaling](/assets/aws-certified-developer-associate/predictive_scaling.png "Predictive Scaling")

### 4.15.4 Metrics to Scale On

- **CPUUtilization**: average CPU utilization across your instances.
- **RequestCountPerTarget**: to make sure the number of requests per EC2 instances is stable.
- **Average Network In/Out**: if the application is network bound (e.g., many uploads and downloads).
- **Custom metric**: that you push using CloudWatch.

### 4.15.5 Scaling Cooldowns

After a scaling activity happens, you are in the **cooldown period**. By default it is **300 seconds, which is 5 minutes**.

**During the cooldown period, the ASG will not launch or terminate additional instances**. This allows for metrics to stabilize and see the effect of the scaling activity on the instances, and get the new metrics.

![Scaling Cooldowns](/assets/aws-certified-developer-associate/scaling_cooldowns.png "Scaling Cooldowns")

**Advice**: Use a ready-to-use AMIs to reduce configuration time to serve requests fasters and reduce the cooldown period, so that scaling can be more dynamic and react faster.

## 4.16 Creating a Scaling Policy on an ASG

To create a scaling policy, go to the EC2 dashboard and click on *Auto Scaling Groups* on the left side menu. Then, select the ASG you want to create a scaling policy for and click on the *Automatic Scaling* tab.

![Create Scaling Policy](/assets/aws-certified-developer-associate/create_scaling_policy.png "Create Scaling Policy")

After a policy is created, you can see it in the list of scaling policies in the same tab:

![Scaling Policy Created](/assets/aws-certified-developer-associate/scaling_policy_created.png "Scaling Policy Created")

### 4.16.1 Dynamic Scaling Policy

To create a dynamic scaling policy, click on the *Create Dynamic Scaling Policy* button and then select the **Policy Type**: target tracking scaling, simple scaling, and step scaling.

Following is an example of a **simple scaling policy**. The action can be add, remove, or set the number of instances or percentage of instances.

![Simple Scaling Policy](/assets/aws-certified-developer-associate/simple_scaling_policy.png "Simple Scaling Policy")

In a simple policy, you can add only one step to increase or decrease the number of instances. On the other hand, in a **step policy**, you can add multiple steps to increase or decrease the number of instances based on the CloudWatch alarm.

![Step Scaling Policy](/assets/aws-certified-developer-associate/step_scaling_policy.png "Step Scaling Policy")

Regarding the **target tracking policy**, the CloudWatch alarm is created automatically based on the target value you set.

![Target Tracking Scaling Policy](/assets/aws-certified-developer-associate/target_tracking_scaling_policy.png "Target Tracking Scaling Policy")

### 4.16.2 Scheduled Scaling Policy

To create a scheduled scaling policy, click on the *Scheduled Actions* tab and then click on *Create Scheduled Action*. Then, insert the desired capacity, min capacity, and max capacity, the **recurrence** (e.g., once, every week, every day, every 30 min, also using *Cron*) and the **start time (i.e., the time for the first scheduled action to run)**.

![Scheduled Scaling Policy](/assets/aws-certified-developer-associate/scheduled_scaling_policy.png "Scheduled Scaling Policy")

### 4.16.3 Predictive Policy

To create a predictive policy, click on the *Predictive Scaling Policies* tab and then click on *Create Predictive Scaling Policy*. Then, enable the **Scale Based on Forecast** option, and set the metrics and target utilization to forecast.

![Predictive Scaling Policy](/assets/aws-certified-developer-associate/predictive_scaling_policy.png "Predictive Scaling Policy")

### 4.16.4 Monitor Scaling Metrics

You can monitor the scaling metrics in the *Monitoring* tab of the ASG:

![Monitor Scaling Metrics](/assets/aws-certified-developer-associate/monitor_scaling_metrics.png "Monitor Scaling Metrics")

You can tools that have been developed on purpose to stress test your machines/application and see how it scales and the ASG reacts.

The **CloudWatch alarms** created by the ASG can be seen in the *Alarms* tab of the CloudWatch dashboard. The first alarm triggers a scale-in policy to reduce the number of instances when the CPUUtilization is below 28% for 15 datapoints within 15 minutes. The second one triggers a scale-out policy to increase the number of machines when the CPUUtilization is above 40% for 3 datapoints within 3 minutes.

![Scaling CloudWatch Alarms](/assets/aws-certified-developer-associate/scaling_cloudwatch_alarms.png "Scaling CloudWatch Alarms")

## 4.17 ASG Instance Refresh

If your **need to update the launch template and then re-create all EC2 instances with this new template**. Instead of terminating all instances and waiting for them to come back, you can use the ASG native feature of instance refresh.

You need to set a **minimum healthy percentage** that tells to the ASG how many instances should be healthy at any point in time. The ASG will then start replacing instances one by one until all instances with the old template are replaced by instances with the new template.

![ASG Instance Refresh](/assets/aws-certified-developer-associate/asg_instance_refresh.png "ASG Instance Refresh")

You also have to specify a warm-up time to define how long until the instance is ready to use.