# 14 Elastic Beanstalk

A lot of web applications have the same architecture (ALB + ASG), this is where Elastic Beanstalk comes in.

**Elastic Beanstalk is a managed servie for deploying an application on AWS**:
- Automatically handles capacity provisioning, load balancing, scaling, application health monitoring, instance configuration, etc.
- The application code is the only responsibility of the developer.

You still have full control over the configuration and it is free, you only pay for the underlying instances.
- It supports pretty much anything: Go, Java, .NET, PHP, Node.js, Python, Ruby, single-container Docker, multi-container Docker, preconfigured Docker, etc.

## 14.1 Components

- **Application**: a collection of Elastic Beanstalk components (environments, versions, configurations, etc.).
- **Application Version**: an iteration of your application code.
- **Environment**:
    - A collection of AWS resources running an application version (only one application version at a time but it can be updated).
    - **Tiers**: **Web Server Environment Tier** and **Worker Environment Tier**.
    - You can create multiple environments (dev, test, prod, and so on).

## 14.2 Web Server Environment Tier vs Worker Environment Tier

In the **web server environment tier**, the environment runs an ELB that redirects traffic to the EC2 instances running a web server within an ASG.

![Web Server Environment Tier](/assets/aws-certified-developer-associate/eb_web_server_environment_tier.png "Web Server Environment Tier")

In the **worker environment tier**, the environment runs a SQS queue that is polled by the EC2 instances (the workers) within an ASG. In this case, scaling is based on the number of messages in the queue.

![Worker Environment Tier](/assets/aws-certified-developer-associate/eb_worker_environment_tier.png "Worker Environment Tier")

You **can combine both tiers**, for example, you can have a web server environment tier that sends messages to a worker environment tier.

## 14.3 Deployment Modes

### 14.3.1 Single Instance Deployment Mode

It is great for development environments because it is the cheapest and fastest way to deploy.

It comprises:
- An EC2 instance running the application.
- A RDS master database instance.
- An Elastic IP address.

![Single Instance Deployment](/assets/aws-certified-developer-associate/eb_single_instance_deployment.png "Single Instance Deployment")

### 14.3.2 High Availability with Load Balancer Deployment Mode

It is great for production environments because it is highly available and scalable.

It comprises:
- An ASG spanning multiple AZs with multiple EC2 instances running the application.
- An ALB that distributes the traffic to the EC2 instances.
- A RDS master database instance, optionally with multi-AZ configuration.

![High Availability with Load Balancer Deployment](/assets/aws-certified-developer-associate/eb_high_availability_with_load_balancer_deployment.png "High Availability with Load Balancer Deployment")

