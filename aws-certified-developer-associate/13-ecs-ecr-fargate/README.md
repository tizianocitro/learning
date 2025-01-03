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
- **Task role (very important at the exam)**: allows the containers in the task to call AWS services. 
- **Task execution role**: allows the container agent to call AWS services.

![ECS Task Definition Infrastructure Requirements](/assets/aws-certified-developer-associate/ecs_task_definition_infrastructure_requirements.png "Create ECS Task Definition Infrastructure Requirements")

![ECS Task Definition Task Roles](/assets/aws-certified-developer-associate/ecs_task_definition_task_roles.png "Create ECS Task Definition Task Roles")

Then, you need to **configure the containers in the task (at least one has to be an essential container)**. You can add multiple containers to the task definition. **Some of the information you need to specify for each container** are indicated in the following images:

![ECS Task Definition Containers](/assets/aws-certified-developer-associate/ecs_task_definition_containers.png "Create ECS Task Definition Containers")

![ECS Task Definition Containers 2](/assets/aws-certified-developer-associate/ecs_task_definition_containers_2.png "Create ECS Task Definition Containers 2")

**If a container is configured as essential, the task will stop if the container stops**. If a container is not essential, the task will continue to run if the container stops.

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

## 13.12 ECS Task Definitions

Task definitions are **metadata in JSON form** (the console provides a UI to build them) to tell ECS how to run one or more Docker containers (up to 10 containers).

A **task definition defines up to 10 containers and contains crucial information**, such as:
- Image name.
- Port binding for container and host.
- Memory and CPU required.
- Environment variables.
- Networking information.
- IAM role.
- Logging configuration (e.g. CloudWatch).

### 13.12.1 Load Balancing and Port Mapping with EC2 Launch Type

When using EC2 instances, each machine has its own IP and executes multiple tasks that are under the same IP (the instance's one) but on different host ports that will be mapped to their container port.

**ECS will automatically assign a host port using Dynamic Host Port Mapping**, if you do not define the host port but only the container port in the task definition.

**Set `host port = 0 (or empty)` to enable random host port, which allows multiple containers of the same type to launch on the same EC2 container instance**. If you specify a host port, only one container of that type can be launched on the EC2 instance because when launching other containers of the same type, the host port will be already in use.

When adding an ALB in front of the Dynamic Host Port Mapping, the **ALB finds the right port on your EC2 instances**. However, you have to allow on the EC2 instance’s security group **any port from the ALB's security group**.

![Load Balancing and Port Mapping with EC2 Launch Type](/assets/aws-certified-developer-associate/load_balancing_and_port_mapping_with_ec2_launch_type.png "Load Balancing and Port Mapping with EC2 Launch Type")

### 13.12.2 Load Balancing and Port Mapping with Fargate Launch Type

With Fargate, each task has a unique private IP because there is no host (it is serverless). So, **you only need to define the container port (host port is not applicable)**.

In the image below, the ALB connects to all tasks in the ECS cluster on the same port but on their different IPs that they got via their Elastic Network Interface (ENI):
- ECS ENI security group: configured to allow port 80 from the ALB.
- ALB security group: configured to allow port 80/443 from web.

![Load Balancing and Port Mapping with Fargate Launch Type](/assets/aws-certified-developer-associate/load_balancing_and_port_mapping_with_fargate_launch_type.png "Load Balancing and Port Mapping with Fargate Launch Type")

### 13.12.3 One IAM Role per Task Definition

**The exam tests you on this**.

**IAM roles are assigned per task definition**. Each ECS task that is running in an ECS service gets all the permissions from the role assigned to the task definition that it is created from.

![One IAM Role per Task Definition](/assets/aws-certified-developer-associate/one_iam_role_per_task_definition.png "One IAM Role per Task Definition")

### 13.12.4 Environment Variables in Task Definitions

**Environment variables**:
- Hardcoded: e.g., fixe non-sensitive URLs.
- SSM Parameter Store: sensitive variables (e.g., API keys, shared configs).
- Secrets Manager: sensitive variables (e.g., DB passwords).

**Environment files (bulk)**: bulk load your environment variables from a file stored in S3.

![Environment Variables](/assets/aws-certified-developer-associate/ecs_environment_variables.png "Environment Variables")

### 13.12.5 Data Volumes (Bind Mounts) to Share Data in Task Definitions

**Bind mounts enable data sharing between multiple containers in the same task definition**. For example, is useful to share files for logging/metrics evaluation with *sidecar* containers, like in the following image:

![Data Volumes (Bind Mounts)](/assets/aws-certified-developer-associate/ecs_data_volumes_bind_mounts.png "Data Volumes (Bind Mounts)")

Bind mounts work for both EC2 and Fargate tasks:
- **EC2 tasks**: volumes use EC2 instance storage, so data are tied to the lifecycle of the EC2 instance.
- **Fargate tasks**: volumes use ephemeral storage, so data are tied to the container(s) using them.
    - Fargate offers from 20 GiB to 200 GiB (default 20 GiB) of ephemeral storage.

Use cases:
- Share ephemeral data between multiple containers (**this is really important for the exam**).
- **Sidecar container pattern**, where the sidecar container used to send metrics/logs to other destinations (separation of conerns).

## 13.13 More on ECS Task Definition Creation

To create a task definition, go into the ECS console and find the *Task Definitions* section, and then click on *Create new Task Definition* to choose among two options:

![ECS Task Definitions Creation Options](/assets/aws-certified-developer-associate/ecs_task_definitions_creation_options.png "Create ECS Task Creation Options")

Most of the **task definition configuration settings are discussed in** [13.7 Creating an ECS Task Definition](#137-creating-an-ecs-task-definition). Do, refer to that section for more details.

A crucial thing to remember is that at least one of the containers in the task definition must be marked as **essential**. If the essential container stops, the task stops. It is up to you to define which are the essential containers for your application. If a non-essential container stops, the task continues to run.

You can also use **private registries** to store your images:

![ECS Task Definitions Private Registry](/assets/aws-certified-developer-associate/ecs_task_definitions_private_registry.png "ECS Task Definitions Private Registry")

Following is how you can add **environment variables** to a container in the task definition:

![ECS Task Definitions Environment Variables](/assets/aws-certified-developer-associate/ecs_task_definitions_environment_variables.png "ECS Task Definitions Environment Variables")

the *ARN* value for the secret is the **ARN of the secret** in Secrets Manager, but the same is valid for SSM Parameter Store.

For logging, you can configure **log collection** from the container to CloudWatch or other destinations (with differnt configurations for each destination):

![ECS Task Definitions Logging](/assets/aws-certified-developer-associate/ecs_task_definitions_logging.png "ECS Task Definitions Logging")

Then, you can configure **health checks** and **start and stop timeouts** for the container.
- **Start timeout**: how long to wait for the container to start before it is considered unhealthy.
- **Stop timeout**: how long to wait for the container to shut down gracefully before forcefully terminating it.

![ECS Task Definitions Timeouts](/assets/aws-certified-developer-associate/ecs_task_definitions_timeouts.png "ECS Task Definitions Timeouts")

Finally, you can configure **monitoring at task definition level**.
- **Trace collection**: AWS adds a sidecar container for OpenTelemetry to route traces to **AWS X-Ray**.
- **Metrics collection**: AWS adds a sidecar container to route custom metrics to CloudWatch or Amazon Managed Service for Prometheus.

![ECS Task Definitions Monitoring](/assets/aws-certified-developer-associate/ecs_task_definitions_monitoring.png "ECS Task Definitions Monitoring")

Before creating, you can also set **tags** for the task definition.

## 13.14 ECS Tasks Placement

When an ECS task is started with EC2 launch type, ECS must determine where to place it, considering the constraints of CPU and memory (RAM).

Similarly, when a service scales in, ECS needs to determine which task to terminate.

To assist with this, you can define **task placement strategies** and **task placement constraints**.

All of this is **only for ECS tasks with EC2 launch type** (Fargate not supported).

### 13.14.1 Task Placement Process

**Task placement strategies are a best effort**. When ECS places a task, it uses the **following process to select the appropriate EC2 container instance**:
1. Identify which instances satisfy the CPU, memory, and port requirements.
2. Identify the instances that satisfy the task placement constraints.
3. Identify the instances that best satisfy the task placement strategies.
4. Select the instances.

## 13.15 ECS Task Placement Strategies

Task placement strategies are **used to determine where to place tasks within a cluster**. You can define multiple strategies for a task definition.

There are different strategies: **binpack**, **random**, and **spread**. You can mix them together in the same task definition.

The **exams tests you on your understanding of the different strategies**.

### 13.15.1 Binpack

**Tasks are placed on the least available amount of CPU and memory on instances**. This minimizes the number of EC2 instances in use for cost savings.

For example, you start with one EC2 instance, ECS will place as many tasks as possible on that instance until it can no longer fit any more tasks. Then, it will start a new EC2 instance and do the same there.

![ECS Binpack](/assets/aws-certified-developer-associate/ecs_binpack.png "ECS Binpack")

The JSON to define the binpack strategy on memory is as follows:
```json
"placementStrategy": [
    {
        "type": "binpack",
        "field": "memory"
    }
]
```

This is the strategy that **offers the most cost savings**.

### 13.15.2 Random

**Tasks are placed randomly on instances**. This is the default strategy.

The JSON to define the random strategy is as follows:
```json
"placementStrategy": [
    {
        "type": "random"
    }
]
```

### 13.15.3 Spread

**Tasks are placed evenly based on the specified value** (e.g., instanceId, attribute:ecs.availability-zone, ...)

![ECS Spread](/assets/aws-certified-developer-associate/ecs_spread.png "ECS Spread")

The JSON to define the spread strategy base on the availability zone is as follows:
```json
"placementStrategy": [
    {
        "type": "spread",
        "field": "attribute:ecs.availability-zone"
    }
]
```

### 13.15.4 Combining Strategies

You can combine strategies in the same task definition. For example, you can use binpack for memory and spread for availability zone:

```json
"placementStrategy": [
    {
        "type": "binpack",
        "field": "memory"
    },
    {
        "type": "spread",
        "field": "attribute:ecs.availability-zone"
    }
]
```

Or you can have multiple strategies of the same type. For example, you can have two spread strategies on different values:

```json
"placementStrategy": [
    {
        "type": "spread",
        "field": "attribute:ecs.availability-zone"
    },
    {
        "type": "spread",
        "field": "instanceId"
    }
]
```

## 13.16 ECS Task Placement Constraints

Task placement constraints are **used to place tasks based on specific rules**. You can define multiple constraints for a task definition.

Two types of constraints:
1. **distinctInstance**: place each task on a different container instance. The JSON is as follows:
    ```json
    "placementConstraints": [
        {
            "type": "distinctInstance"
        }
    ]
    ```
2. **memberOf**: place tasks on instances that satisfy an expression defined using the *Cluster Query Language*. The JSON that defines an expression to place tasks on instances with a specific instance type (t2 family) is as follows:
    ```json
    "placementConstraints": [
        {
            "type": "memberOf",
            "expression": "attribute:ecs.instance-type =~ t2.*"
        }
    ]
    ```

## 13.17 ECR

ECR allows you to store and manage Docker container images.

You can store images in a:
- **Private repository**: accessible only from your AWS account(s).
- **Public repository**: publish images to Amazon ECR Public Gallery at https://gallery.ecr.aws.

It is fully integrated with ECS and backed by S3. Access is controlled through IAM roles.

A few features it supports are:
- Image vulnerability scanning.
- Image versioning.
- Image tags.
- Image lifecycle.

![ECR with ECS on EC2](/assets/aws-certified-developer-associate/ecr_with_ecs_on_ec2.png "ECR with ECS on EC2")

### 13.17.1 Using ECR from the CLI

You need to login to ECR before you can push or pull images. To do so, you can use the following command:
```bash
aws ecr get-login-password --region <region> | \
docker login --username AWS \
--password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

The `aws ecr get-login-password --region <region>` command gets a password to use for the login that is then piped to the `docker login` command.

The `<>` are placeholders for the actual values. The above command could be something like:
```bash
aws ecr get-login-password --region us-east-1 | \
docker login --username AWS \
--password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

Then, you just run Docker commands to push and pull images. For example:
```bash
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/demo:latest

docker pull <aws_account_id>.dkr.ecr.<region>.amazonaws.com/demo:latest
```

In case an EC2 instance (or you) can't pull a Docker image, check IAM permissions.

### 13.17.2 Creating an ECR Repository and Pushing an Image to It

To create an ECR repository, go into the ECR console, and then under the *Repositories* section, click on *Create Repository*.

Enter a **name** for the repository and specify the **visibility (public or private)**:

![ECR Create Repository](/assets/aws-certified-developer-associate/ecr_create_repository.png "Create ECR Repository")

There is also an option for **tag immutability** (avoid to push the same tag twice), which prevents images from being overwritten. You can also use **KMS to encrypt the images**.

After creating the repository, you can push images to it. To do so, you need to login to ECR (as shown in [13.17.1 Using ECR from the CLI](#13171-using-ecr-from-the-cli)) and then push the image:
```bash
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/demo:latest
```

Notice that **the name of the repository is part of the image name**. What changes is the tag.

After pushing the image, you can see it in the ECR console within the repository:

![ECR Repository Image Pushed](/assets/aws-certified-developer-associate/ecr_repository_image_pushed.png "ECR Repository Image Pushed")

Finally, you can pull the image from the repository:
```bash
docker pull <aws_account_id>.dkr.ecr.<region>.amazonaws.com/demo:latest
```

## 13.18 AWS Copilot

**Copilot is a CLI tool** to build, release, and operate production-ready containerized apps.
- It is installed separately from the AWS CLI.

It **simplifies the process of running containerized applications** on App Runner, ECS, and Fargate **by allowing you focus on building the actual applications rather than setting up infrastructure**. It:
- Uses CloudFormation to provision all the required infrastructure for containerized applications: ECS, VPC, ELB, ECR, etc.
- Offers automated deployments with one command using CodePipeline.
- Enables deployment on multiple environments.
- Provides features for troubleshooting, health status, logs, etc.

![AWS Copilot](/assets/aws-certified-developer-associate/aws_copilot.png "AWS Copilot")

## 13.19 EKS

It is a service to **launch managed Kubernetes clusters on AWS**.
- Kubernetes is an open-source system for automatic deployment, scaling and management of containerized (usually Docker) application.
- EKS is an alternative to ECS with a similar goal but different API. 

Kubernetes is not proprietary to AWS, and, in a certain way, it provides standardization. **Kubernetes is cloud-agnostic**, so it can be used in any cloud (e.g., Azure, GCP, etc.).

EKS supports:
- EC2 launch mode if you want to deploy worker nodes.
- Fargate launch mode to deploy serverless containers.
- Multiple regions by deploying one EKS cluster per region.
- Logs and metrics collection with CloudWatch Container Insights.

**Use case**: your company is already using Kubernetes on-premises or in another cloud, and wants to migrate to AWS using Kubernetes. Or they simply want to use the Kubernetes API.

### 13.19.1 EKS Cluster with EC2 Worker Nodes

In the image below, you can see a multi-AZ EKS cluster with EC2 worker nodes.

![EKS Cluster with EC2 Worker Nodes](/assets/aws-certified-developer-associate/eks_cluster_with_ec2_worker_nodes.png "EKS Cluster with EC2 Worker Nodes")

### 13.19.2 EKS Node Types

1. **Managed node groups**:
    - Creates and manages nodes (EC2 instances) for you.
    - Nodes are part of an ASG managed by EKS.
    - Supports on-demand or spot instances.

2. **Self-managed nodes**:
    - Nodes are created and registered by you to the EKS cluster and managed by an ASG.
    - You can use the prebuilt AMI for EKS (*Amazon EKS Optimized AMI*) to simplify the process of setting up new nodes.
    - Supports on-demand or spot instances.

3. **AWS Fargate**: no maintenance required because there are no nodes to manage.

### 13.19.3 EKS Data Volumes

To be able to attach volumes to EKS pods, you need to specify a **StorageClass** manifest on your EKS cluster. It leverages a **Container Storage Interface (CSI)** compliant driver.

There is support for:
- EBS.
- EFS: only type of volume that works with Fargate.
- FSx for Lustre.
- FSx for NetApp ONTAP.

## 13.20 ECS Config File

Suppose you have an ECS cluster and you want to enable IAM roles for your ECS tasks so that they can make API requests to AWS services.

There is an ECS configuration option that you should enable in the `/etc/ecs/ecs.config` configuration file. The option is `ECS_ENABLE_TASK_IAM_ROLE`.