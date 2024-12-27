# 13 Elastic Container Service, Elastic Container Registry, and Fargate

- **Elastic Container Service (ECS)**: Amazon's own container platform.
- **Elastic Container Registry (ECR)**: store container images on Amazon's image registry.
- **Fargate**: Amazon's own serverless container platform, which works with ECS and with EKS.
- **Elastic Kubernetes Service (EKS)**: Amazon's managed Kubernetes.

## 13.1 ECS EC2 Launch Type

You launch Docker containers on AWS by launching **ECS Tasks** on **ECS Clusters**.

An ECS cluster is a logical grouping things and with the EC2 launch type, those things are EC2 instances.

With the **EC2 Launch Type, you must provision and maintain the infrastructure** (the EC2 instances) that run your containers.
- **Each EC2 instance must run the ECS Agent that will register it in the ECS cluster**. The ECS Agent is a Docker container that runs on the EC2 instance.
- AWS takes care of starting/stopping containers when the EC2 instance is started/stopped.

The following image shows an ECS cluster with EC2 instances running the ECS Agent and Docker containers:

![ECS EC2 Launch Type](/assets/aws-certified-developer-associate/ecs_ec2_launch_type.png "ECS EC2 Launch Type")

## 13.2 ECS Fargate Launch Type

**The exams may ask a lot of questions about using Fargate**, because it is a serverless service.

You launch Docker containers on AWS but do not provision the infrastructure (no EC2 instances to manage) because **it is serverless**.

You **create task definitions and AWS runs ECS Tasks for you based on the CPU/RAM you need**.

**To scale**, you **just increase the number of tasks**. So, no need to add more EC2 instances.

![ECS Fargate Launch Type](/assets/aws-certified-developer-associate/ecs_fargate_launch_type.png "ECS Fargate Launch Type")

## 13.3 IAM Roles for ECS

As roles you use:
1. **EC2 Instance Profile for EC2 Launch Type only and used by the ECS agent**. The ECS agent:
    - Makes API calls to ECS.
    - Sends container logs to CloudWatch Logs.
    - Pulls Docker image from ECR.
    - References sensitive data in Secrets Manager or SSM Parameter Store.
2. **ECS Task Role for both EC2 and Fargate Launch Types**:
    - Allows each task to have a specific role.
    - You can use different roles for the different ECS Services.
    - The **task role is defined in the task definition**.

![IAM Roles for ECS](/assets/aws-certified-developer-associate/iam_roles_for_ecs.png "IAM Roles for ECS")

## 13.4 ECS Load Balancer Integrations

