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

This is how you can **expose your ECS containers to the internet**. You can **expose your ECS tasks via HTTP/HTTPS endpoints** using:
- **Application Load Balancer**: works for most use cases.
- **Network Load Balancer**: recommended only for high throughput/high performance use cases, or to pair it with AWS Private Link.
- **Classic Load Balancer**: legacy, it is supported but not recommended (no advanced features and no Fargate).

![ECS Load Balancer Integrations](/assets/aws-certified-developer-associate/ecs_load_balancer_integrations.png "ECS Load Balancer Integrations")

## 13.5 ECS Data Volumes via EFS

They allow for **data persistence and data sharing between containers**.

With EFS, you can **mount EFS file systems onto ECS tasks**.
- It works for both EC2 and Fargate launch types.
- **Tasks running in any AZ will share the same data in the EFS file system**.

Using **Fargate with EFS provides a full serverless experience** because EFS and Fargate are serverless services.

![ECS Data Volumes via EFS](/assets/aws-certified-developer-associate/ecs_data_volumes_via_efs.png "ECS Data Volumes via EFS")

**Use case**: persistent multi-AZ shared storage for your containers.

Note: **S3 cannot be mounted as a file system on ECS tasks**.

## 13.6 Creating an ECS Cluster

To create an ECS cluster, go into the ECS console and find the *Clusters* section. Click on *Create Cluster* to create a new cluster.

Enter a **name** and optionally a **namespace** for the cluster:

![ECS Cluster Name](/assets/aws-certified-developer-associate/ecs_cluster_name.png "Create ECS Cluster Name")

Then select the **infrastructure** type (launch type) for the cluster. You can choose between EC2, Fargate, and **ECS Anywhere**:

![ECS Cluster Launch Type](/assets/aws-certified-developer-associate/ecs_cluster_launch_type.png "Create ECS Cluster Launch Type")

You can **choose both EC2 and Fargate launch types at the same time**:

![ECS Cluster EC2 and Fargate](/assets/aws-certified-developer-associate/ecs_cluster_ec2_and_fargate.png "Create ECS Cluster EC2 and Fargate")

To **use EC2 launch type**, you need to:
- Create a new ASG or use an existing one.
- Choose the OS.
- Choose the instance type.
- Optionally configure an SSH key pair.
- Set the desired capacity.
- Configure the root EBS volume.

![ECS Cluster EC2 Instances](/assets/aws-certified-developer-associate/ecs_cluster_ec2_instances.png "Create ECS Cluster EC2 Instances")

If you use the EC2 launch type, you need to **configure the networking (VPC, subnets, and security groups)**:

![ECS Cluster Networking](/assets/aws-certified-developer-associate/ecs_cluster_networking.png "Create ECS Cluster Networking")

For the **security group**, you can create a new one or use an existing one:

![ECS Cluster Security Group](/assets/aws-certified-developer-associate/ecs_cluster_security_group.png "Create ECS Cluster Security Group")

Before creating the cluster, you can also set **monitoring** and **tags**:

![ECS Cluster Monitoring and Tags](/assets/aws-certified-developer-associate/ecs_cluster_monitoring_and_tags.png "Create ECS Cluster Monitoring and Tags")

After clicking on *Create*, you can go into your ASG and see the instances being created (it depends on the capacity settings). When the cluster is ready, you can see it in the ECS console:

![ECS Cluster Ready](/assets/aws-certified-developer-associate/ecs_cluster_ready.png "ECS Cluster Ready")

If you click on the cluster, you can see its **properties**:

![ECS Cluster Properties](/assets/aws-certified-developer-associate/ecs_cluster_properties.png "ECS Cluster Properties")

In the *Infrastructure* tab, you can see **capacity providers**. In the image, there are 3 capacity providers:
- **FARGATE**: you can launch Fargate tasks in the cluster.
- **FARGATE_SPOT**: you can launch Fargate tasks in spot mode (like EC2 spot instances) in the cluster.
- **ASG**: you can launch EC2 instances in the cluster directly from the specified ASG. The EC2 instances in the ASG will also appear in the **container instances** section.

![ECS Cluster Capacity Providers](/assets/aws-certified-developer-associate/ecs_cluster_capacity_providers.png "ECS Cluster Capacity Providers")

What these providers mean is that, when you create an ECS task, it can be launched on Fargate, Fargate Spot, or on the EC2 instances in the ASG. The instances will also appear in the **container instances** section:

![ECS Cluster Container Instances](/assets/aws-certified-developer-associate/ecs_cluster_container_instances.png "ECS Cluster Container Instances")

## 13.7 Creating an ECS Task Definition

Go into the ECS console and find the *Task Definitions* section, and then click on *Create new Task Definition*.

Enter the **task definition family**:

![ECS Task Definition Family](/assets/aws-certified-developer-associate/ecs_task_definition_family.png "Create ECS Task Definition Family")

Next is to configure the **infrastructure requirements**, which define where the task can be launched. The requirements specify the:
- Possible launch type: EC2 or Fargate.
- OS architecture.
- Task size: CPU and memory needs.
- Task role: allows the containers in the task to call AWS services. 
- Task execution role: allows the container agent to call AWS services.

![ECS Task Definition Infrastructure Requirements](/assets/aws-certified-developer-associate/ecs_task_definition_infrastructure_requirements.png "Create ECS Task Definition Infrastructure Requirements")

![ECS Task Definition Task Roles](/assets/aws-certified-developer-associate/ecs_task_definition_task_roles.png "Create ECS Task Definition Task Roles")

Then, you need to **configure the containers in the task**. You can add multiple containers to the task definition. **Some of the information you need to specify for each container** are indicated in the following images:

![ECS Task Definition Containers](/assets/aws-certified-developer-associate/ecs_task_definition_containers.png "Create ECS Task Definition Containers")

![ECS Task Definition Containers 2](/assets/aws-certified-developer-associate/ecs_task_definition_containers_2.png "Create ECS Task Definition Containers 2")

You can also **configure the storage** for the task. You can choose between:

![ECS Task Definition Storage](/assets/aws-certified-developer-associate/ecs_task_definition_storage.png "Create ECS Task Definition Storage")

As with the cluster, you can also set **monitoring** and **tags** for the task definition. Finally, you can create the task definition.

Now **you can use this task definition to create an ECS service**.

## 13.8 Creating an ECS Service

An **ECS service is a long-running task that runs on ECS**. It is a way to ensure that a specified number of tasks are running at any given time. For example, a web application.

On the opposite, **an ECS task is a one-time task: it does its job and stops**. For example, a batch job.

To create an ECS service, go into the ECS console and find the cluster you want to create the service in, and then click on *Create* in the *Services* tab.

![ECS Service Create](/assets/aws-certified-developer-associate/ecs_service_create.png "Create ECS Service")

Set the **compute configuration** by selecting the **compute options**, the **launch type** (*FARGATE*, *EC2*, and *EXTERNAL*), and the **platform version** (e.g., *Latest*):

![ECS Service Compute Configuration](/assets/aws-certified-developer-associate/ecs_service_compute_configuration.png "Create ECS Service Compute Configuration")

Next is the **deployment configuration where you coose between a service and a task**:

![ECS Service Deployment Configuration](/assets/aws-certified-developer-associate/ecs_service_deployment_configuration.png "Create ECS Service Deployment Configuration")

In this case, we are creating a service, so we need to **select Service**. Then, you need to set the **family**, the **service type (replica or daemon)**, and a **name**. The **desired tasks** defines how many tasks/replicas you want to run.

![ECS Service Deployment Configuration 2](/assets/aws-certified-developer-associate/ecs_service_deployment_configuration_2.png "Create ECS Service Deployment Configuration 2")

Among other options, you then have to configure the **networking** options (e.g., VPC, subnets, security groups):

![ECS Service Networking](/assets/aws-certified-developer-associate/ecs_service_networking.png "Create ECS Service Networking")

We can create a **security group** for the service that allows inbound traffic on port 80 because we are running a task definition with a NGINX container. We also enable a public IP:

![ECS Service Security Group](/assets/aws-certified-developer-associate/ecs_service_security_group.png "Create ECS Service Security Group")

Next, we configure the load balancer by creating a new ALB:

![ECS Service Load Balancer](/assets/aws-certified-developer-associate/ecs_service_load_balancer.png "Create ECS Service Load Balancer")

![ECS Service Load Balancer 2](/assets/aws-certified-developer-associate/ecs_service_load_balancer_2.png "Create ECS Service Load Balancer 2")

Before creation, you can also configure **service auto scaling**, **task placement**, and **tags**:

![ECS Service Other Options](/assets/aws-certified-developer-associate/ecs_service_other_options.png "Create ECS Service Other Options")

After creating the service, it can take a few minutes to deploy successfully. You can see the service in the console:

![ECS Service Created](/assets/aws-certified-developer-associate/ecs_service_created.png "Create ECS Service Created")

And by clicking on it, you can see its **details** (from the tabs here, you can also access the task's containers logs):

![ECS Service Details](/assets/aws-certified-developer-associate/ecs_service_details.png "Create ECS Service Details")

Now that it is ready, you can use the **DNS name of the ALB** to access the NGINX container. If you have configured more tasks in the service, you will see that the ALB will distribute the traffic betwseen them. As a result, you will receive different responses from the different NGINX containers.

## 13.9 ECS Service Auto Scaling

Automatically increase/decrease the desired number of ECS tasks.

**Amazon ECS Auto Scaling** uses **AWS Application Auto Scaling**, which can scale based on:
- **ECS service average CPU utilization**.
- **ECS service average memory utilization**: scale on RAM.
- **ALB request count per target**: metric coming from the ALB.

Three types of auto scaling:
1. **Target tracking**: scale based on target value for a specific CloudWatch metric.
2. **Step scaling**: scale based on a specified CloudWatch Alarm.
3. **Scheduled scaling**: scale based on a specified date/time (predictable changes).

ECS service auto scaling (task level) is not the same as EC2 auto scaling (EC2 instance level).
- This is why Fargate auto scaling is much easier to setup because of serverless.

### 13.9.1 EC2 Launch Type: Auto Scaling EC2 Instances

You can accommodate the ECS Service scaling by adding underlying EC2 instances.

Two options:
1. **Auto Scaling Group Scaling**:
    - Scale your ASG based on CPU utilization.
    - Add EC2 instances over time.
2. **ECS Cluster Capacity Provider**:
    - Used to automatically provision and scale the infrastructure for ECS tasks.
    - The capacity provider is paired with an ASG.
    - Adds EC2 instances when you are missing capacity (CPU, RAM, ...).

### 13.9.2 Example of ECS Auto Scaling based on CPU Utilization

In the image below, we have an ECS service with a target tracking scaling based on the average CPU utilization of the ECS service. When the CPU utilization is above a defined threshold, CloudWatch Metric will trigger a CloudWatch alarm to scale the ECS service by adding a new task in the service.

![ECS Auto Scaling CPU Utilization](/assets/aws-certified-developer-associate/ecs_auto_scaling_cpu_utilization.png "ECS Auto Scaling CPU Utilization")

## 13.10 ECS Rolling Updates

When updating an ECS service from v1 to v2, you can control how many tasks can be started and stopped, and in which order.

When updating the service, you can set the **minimum healthy percent (default 100%)** and the **maximum percent (default 200%)** of tasks that are allowed to be in the *RUNNING* state during a deployment. The image below shows the rolling update process:

![ECS Rolling Updates](/assets/aws-certified-developer-associate/ecs_rolling_updates.png "ECS Rolling Updates")

Now let's see two scenarios: **Min 50% - Max 100%** and **Min 100% - Max 150%**.

### 13.10.1 Scenario 1: Min 50% - Max 100%

Considering a starting number of 4 tasks, the following steps will occur:
- **T1**: 4 v1 tasks are running.
- **T2**: 2 v1 tasks are stopped (because we can terminate 50% of the tasks before starting new ones because the minimum is 50%).
- **T3**: 2 new v2 tasks are started.
- **T4**: 2 v1 and 2 v2 tasks are running.
- **T5**: 2 v1 tasks are stopped.
- **T6**: 2 new v2 tasks are started.
- **T7**: 4 v2 tasks are running.

![ECS Rolling Updates Scenario 1](/assets/aws-certified-developer-associate/ecs_rolling_updates_scenario_1.png "ECS Rolling Updates Scenario 1")

### 13.10.2 Scenario 2: Min 100% - Max 150%

Considering a starting number of 4 tasks, the following steps will occur:
- **T1**: 4 v1 tasks are running.
- **T2**: 2 v2 tasks are started (because tasks cannot be terminated as the minimum is 100%).
- **T3**: 4 v1 and 2 v2 tasks are running.
- **T4**: 2 v1 tasks are stopped.
- **T5**: 2 v1 and 2 v2 tasks are running.
- **T6**: 2 v2 tasks are started.
- **T7**: 2 v1 and 4 v2 tasks are running.
- **T8**: 2 v1 tasks are stopped.
- **T9**: 4 v2 tasks are running.

![ECS Rolling Updates Scenario 2](/assets/aws-certified-developer-associate/ecs_rolling_updates_scenario_2.png "ECS Rolling Updates Scenario 2")

## 13.11 ECS Solutions Architectures

Here are some useful-to-know ECS solutions architectures.

### 13.11.1 ECS Tasks Invoked by EventBridge

In this architecture, an EventBridge rule triggers an ECS task on Fargate to process the event.

The ECS task can be used to process events from various sources (e.g., S3, DynamoDB, etc.). In this case, the task is processing an S3 event and then stores the result in DynamoDB.

To interact with S3 and DynamoDB, the ECS task needs the appropriate permissions. These permissions are granted by the ECS Task Role.

![ECS Tasks Invoked by EventBridge](/assets/aws-certified-developer-associate/ecs_tasks_invoked_by_eventbridge.png "ECS Tasks Invoked by EventBridge")

### 13.11.2 ECS Tasks Invoked by EventBridge Schedule

In this architecture, an EventBridge rule triggers an ECS task on Fargate on a schedule, every 1 hour in this case.

![ECS Tasks Invoked by EventBridge Schedule](/assets/aws-certified-developer-associate/ecs_tasks_invoked_by_eventbridge_schedule.png "ECS Tasks Invoked by EventBridge Schedule")

### 13.11.3 ECS Tasks Polling Messages from SQS Queue

In this architecture, an ECS service has 3 tasks that poll messages from an SQS queue and processe them.

ECS Service Auto Scaling is enabled and used to scale the number of tasks based on the number of messages in the SQS queue. The more messages in the queue, the more tasks in the service.

![ECS Tasks Polling Messages from SQS Queue](/assets/aws-certified-developer-associate/ecs_tasks_polling_messages_from_sqs_queue.png "ECS Tasks Polling Messages from SQS Queue")

### 13.11.4 Intercept Stopped Tasks using EventBridge

In this architecture, an EventBridge rule intercepts stopped tasks and sends a notification to an SNS topic.

The notification can be used to send an email to an administrator, for example.

![Intercept Stopped Tasks using EventBridge](/assets/aws-certified-developer-associate/intercept_stopped_tasks_using_eventbridge.png "Intercept Stopped Tasks using EventBridge")
