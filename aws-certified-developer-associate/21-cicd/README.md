# 21 CICD: CodeCommit, CodePipeline, CodeBuild, and CodeDeploy

Manual steps (e.g., create AWS resources manually) make it very likely to do mistakes. We would like our code in a repository and have it deployed onto AWS:
- Automatically.
- Correctly.
- Making sure it is tested before being deployed.
- With possibility to go into different stages (e.g., dev, test, staging, prod)
- With manual approval where needed.

## 21.1 Technology Stack for CI/CD on AWS

For all of this, we need a **CI/CD pipeline**. AWS has several services to help us with this:
- `CodeCommit`: store code.
- `CodePipeline`: automate pipelines.
- `CodeBuild`: build and test code.
- `CodeDeploy`: deploy the code (no Elastic Beanstalk).
- `CodeStar`: manage software development activities in one place.
- `CodeArtifact`: store, publish, and share software packages.
- `CodeGuru`: automated code reviews using machine learning.

![Technology Stack](/assets/aws-certified-developer-associate/cicd_techstack.png "Technology Stack")

## 21.2 Continuous Integration (CI)

Developers push the code to a code repository often (e.g., GitHub, CodeCommit, Bitbucket). A testing/build server checks the code as soon as it is pushed (e.g., CodeBuild, Jenkins), builds and tests it. The developer gets feedback about the tests and checks that have passed/failed.

Advantages of CI:
- Find and fix bugs early.
- Deliver faster as the code is tested and deploy often.

![Continuous Integration](/assets/aws-certified-developer-associate/cicd_ci.png "Continuous Integration")

## 21.3 Continuous Delivery (CD)

Ensures that:
- Software can be released reliably whenever needed.
- Deployments happen often and are quick.

CD usually means automated deployment (e.g., CodeDeploy, Spinnaker).

![Continuous Delivery](/assets/aws-certified-developer-associate/cicd_cd.png "Continuous Delivery")

## 21.4 CodeCommit

Version control is the ability to understand the various changes that happened to the code over time, and possibly roll back. All these are enabled by using a version control system like Git.

CodeCommit is fully managed and highly available. It provides:
- Private Git repositories.
- No size limit on repositories (scale seamlessly). 
- Code availability only in AWS account, increasing security and compliance.
- Security by encryption, access control, etc.
- Integration with Jenkins, CodeBuild, and other CI tools.

### 21.4.1 CodeCommit Security

**Interactions** are done using Git.

**Authentication** via:
- SSH Keys: users can configure SSH keys in their IAM console.
- HTTPS: CLI credential helper or Git credentials for IAM users.

**Authorization** via IAM policies to manage user/role permissions to repositories.

**Encryption**:
- Repositories are automatically encrypted at rest using KMS.
- Encrypted in transit: you can only use HTTPS or SSH, which are both secure.

**Cross-account access**:
- Do not share SSH keys or your AWS credentials.
- Use an IAM role in your AWS account and use STS (`AssumeRole` API).

### 21.4.2 CodeCommit vs GitHub

![CodeCommit vs GitHub](/assets/aws-certified-developer-associate/cicd_codecommit_vs_github.png "CodeCommit vs GitHub")

### 21.4.3 CodeCommit is Deprecated

On July 25th 2024, AWS discontinued CodeCommit, so new customers cannot use the service and AWS recommends to migrate to an external Git solution.
- For example, use GitHub or GitLab instead.

## 21.5 CodePipeline

It is a visual workflow tool to orchestrate CICD pipelines.
- Sources: CodeCommit, ECR, S3, Bitbucket, GitHub, etc.
- Build phase: CodeBuild, Jenkins, CloudBees, TeamCity.
- Test phase: CodeBuild, AWS Device Farm, 3rd party tools, etc.
- Deploy phase: CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3, etc.
- Invoke phase: Lambda, Step Functions.

You can build stages, where each stage can have sequential actions and/or parallel actions.
- For example, a stage can involve `Build -> Test -> Deploy -> Load Testing -> ...`.
- Manual approval can be defined at any stage.

### 21.5.1 CodePipeline Artifacts

Each pipeline stage can create **artifacts**, which are stored in an S3 bucket and passed on to the next stage.

![CodePipeline Artifacts](/assets/aws-certified-developer-associate/cicd_codepipeline_artifacts.png "CodePipeline Artifacts")

### 21.5.2 Troubleshooting CodePipeline

For CodePipeline pipeline/action/stage execution state changes, you can use Amazon EventBridge. For example, you can create events for failed pipelines or cancelled stages.
- If CodePipeline fails a stage, the pipeline stops, and you can get information in the console.
- If CodePipeline cannot perform an action, make sure the IAM role attached has enough IAM permissions (IAM policy).

Additionally, CloudTrail can be used to audit AWS API calls.

## 21.6 Using CodePipeline to Deploy Changes to Elastic Beanstalk

Create an Elastic Beanstalk application and two environments `dev` and `prod`.

Go to CodePipeline and create a new pipelineby clicking on the *Create Pipeline* button:

![CodePipeline Create](/assets/aws-certified-developer-associate/cicd_codepipeline_create.png "CodePipeline Create")

Select the **creation process**:

![CodePipeline Creation Process](/assets/aws-certified-developer-associate/cicd_codepipeline_create.png "CodePipeline Creation Process")

Assign it a **name** and select the **execution mode**:

![CodePipeline Name](/assets/aws-certified-developer-associate/cicd_codepipeline_name.png "CodePipeline Name")

And define the **service role**:

![CodePipeline Service Role](/assets/aws-certified-developer-associate/cicd_codepipeline_service_role.png "CodePipeline Service Role")

You can eventually setup **variables** for the pipeline:

![CodePipeline Variables](/assets/aws-certified-developer-associate/cicd_codepipeline_variables.png "CodePipeline Variables")

Next, select the **source provider**. In this case, we will use GitHub (Version 2):

![CodePipeline Source Provider](/assets/aws-certified-developer-associate/cicd_codepipeline_source_provider.png "CodePipeline Source Provider")

For this, you need to create a GitHub connection by clicking on the *Connect to GitHub* button:

![CodePipeline GitHub Connection](/assets/aws-certified-developer-associate/cicd_codepipeline_github_connection.png "CodePipeline GitHub Connection")

And enter it, alongside the **repository name** and **branch name**:

![CodePipeline GitHub Connection Name](/assets/aws-certified-developer-associate/cicd_codepipeline_github_connection_name.png "CodePipeline GitHub Connection Name")

Configure triggers for the pipeline. You can select to trigger the pipeline on every push (or pull request) to the repository, or only on specific branches or files:

![CodePipeline GitHub Triggers](/assets/aws-certified-developer-associate/cicd_codepipeline_github_triggers.png "CodePipeline GitHub Triggers")

![CodePipeline GitHub Triggers 2](/assets/aws-certified-developer-associate/cicd_codepipeline_github_triggers_2.png "CodePipeline GitHub Triggers 2")

You can then select the **build provider**, but we will skip this step for now (we will see more later):

![CodePipeline Build Provider](/assets/aws-certified-developer-associate/cicd_codepipeline_build_provider.png "CodePipeline Build Provider")

Next is the **deploy provider**:

![CodePipeline Deploy Provider](/assets/aws-certified-developer-associate/cicd_codepipeline_deploy_provider.png "CodePipeline Deploy Provider")

In this case, we will use Elastic Beanstalk to deploy to the `dev` environment, which requires the following information:

![CodePipeline Deploy Provider Beanstalk](/assets/aws-certified-developer-associate/cicd_codepipeline_deploy_provider_eb.png "CodePipeline Deploy Provider Beanstalk")

You can finally **create the pipeline**, and **it will be created and started**:

![CodePipeline Created](/assets/aws-certified-developer-associate/cicd_codepipeline_created.png "CodePipeline Created")

Until all stages are completed:

![CodePipeline Created 2](/assets/aws-certified-developer-associate/cicd_codepipeline_created_2.png "CodePipeline Created 2")

### 21.6.1 Adding Stages to CodePipeline

You can add stages to the pipeline by clicking on the *Edit* button and then clicking on the *Add Stage* button of the phase you want to add the new stage to:

![CodePipeline Add Stage](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage.png "CodePipeline Add Stage")

Give the new stage a **name**:

![CodePipeline Add Stage Name](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_name.png "CodePipeline Add Stage Name")

Configure the **action group** for the stage:

![CodePipeline Add Stage Action Group](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_name.png "CodePipeline Add Stage Action Group")

We will select Beanstalk as the action provider:

![CodePipeline Add Stage Action Group Beanstalk](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_action_group_eb.png "CodePipeline Add Stage Action Group Beanstalk")

And enter the specific information it requires:

![CodePipeline Add Stage Action Group Beanstalk Info](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_action_group_eb_info.png "CodePipeline Add Stage Action Group Beanstalk Info")

**Multiple action groups can be added to a stage**. This is useful for when you need a **manual approval** step like in this case because we are deploying to production:

![CodePipeline Add Stage Manual Approval](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_manual_approval.png "CodePipeline Add Stage Manual Approval")

So, it will appear that you need to approve the deployment before it is done:

![CodePipeline Add Stage Manual Approval 2](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_manual_approval_2.png "CodePipeline Add Stage Manual Approval 2")

Save the pipeline and it will be updated:

![CodePipeline Updated](/assets/aws-certified-developer-associate/cicd_codepipeline_updated.png "CodePipeline Updated")

Now, push a change to the GitHub repository and it will trigger the pipeline. This time, it will deployu to the `dev` environment and then **wait for a manual approval** to deploy to the `prod` environment:

![CodePipeline Waiting for Approval](/assets/aws-certified-developer-associate/cicd_codepipeline_waiting_for_approval.png "CodePipeline Waiting for Approval")

So, review the changes in the `dev` environment and click on the *Review* button to approve the deployment:

![CodePipeline Review Changes](/assets/aws-certified-developer-associate/cicd_codepipeline_review_changes.png "CodePipeline Review Changes")

And submit the approval so that the deployment to the `prod` environment can continue.

### 21.6.2 Accessing CodePipeline History

You can access the history of the pipeline by accessing the *History* section:

![CodePipeline History](/assets/aws-certified-developer-associate/cicd_codepipeline_history.png "CodePipeline History")

## 21.7 CodeBuild

It is a fully managed CI service that leverages Docker under the hood for reproducible builds.
- Use prepackaged Docker images or create your own custom ones.
- **Build projects can be defined either in CodePipeline or CodeBuild**.

It comprises:
- Source: CodeCommit, S3, Bitbucket, GitHub, etc.
- **Build instructions**: privided using the `buildspec.yml` file (best practice) or manually in the console.
- Output: logs can be stored in S3 and CloudWatch Logs.

Use:
- CloudWatch Metrics to monitor build statistics.
- EventBridge to detect failed builds and trigger notifications.
- CloudWatch Alarms to notify if you need thresholds for failures.

Supported environments with prepackaged Docker images are: Java, Ruby, Python, Go, Node.js, Android, .NET Core, PHP. However, you can extend any environment you like using custom Docker images.

### 21.7.1 How CodeBuild Works

The `buildspec.yml` file is at the root of the source code repository. It defines the build commands and settings used by CodeBuild to run the build. Reusable pieces and results are stored in S3 with the option of caching the build output to speed up future builds. Logs are stored in S3/CloudWatch Logs and can be viewed in the console.

![How CodeBuild Works](/assets/aws-certified-developer-associate/cicd_codebuild_how_it_works.png "How CodeBuild Works")

### 21.7.2 CodeBuild Buildspec File

- `buildspec.yml` file must be at the root of your code and defines the build commands and settings used by CodeBuild to run the build.

- `env` defines environment variables:
    - `variables`: plaintext variables.
    - `parameter-store`: variables stored in SSM Parameter Store.
    - `secrets-manager`: variables stored in AWS Secrets Manager.

- `phases` specifies the commands to run:
    - `install`: install dependencies you need for the build.
    - `pre_build`: final commands to execute before the build.
    - `build`: define the actual build commands.
    - `post_build`: finishing touches (e.g., create zip output).

- `artifacts` defines the final output that should be uploaded to S3 (encrypted with KMS).

- `cache` defines the files to cache (usually dependencies) to S3 to speedup future builds.

The following is an example of a `buildspec.yml` file:

```yaml
version: 0.2

env:
  variables:
    ENV: dev
  parameter-store:
    DB_USERNAME: myapp/db/username
  secrets-manager:
    DB_PASSWORD: /myapp/db/password

phases:
  install:
    runtime-versions:
      nodejs: latest
    commands:
      - echo "Installing dependencies..."
      - npm install
  pre_build:
    commands:
      - echo "Running pre-build commands..."
      - echo "Running tests..."
      - npm test
  build:
    commands:
      - echo "Building the application..."
      - npm run build
  post_build:
    commands:
      - echo "Build completed successfully."

artifacts:
    files:
        - '**/*'
    base-directory: build

cache:
  paths:
    - node_modules/**/*
```

## 21.8 Using CodeBuild to Build Code

Go to CodeBuild and create a new build project by clicking on the *Create project* button:

![CodeBuild Create Project](/assets/aws-certified-developer-associate/cicd_codebuild_create_project.png "CodeBuild Create Project")

Give the project a **name** (in this case, `MyFirstBuild`):

![CodeBuild Create Project Name](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_name.png "CodeBuild Create Project Name")

Select the **source provider**. In this case, we will use GitHub:

![CodeBuild Create Project Source Provider](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_source_provider.png "CodeBuild Create Project Source Provider")

Then, you can set the **primary source webhook events to trigger the build**. You can select to trigger the build  on different events such as:
- Push to a branch.
- Pull request created.
- Pull request updated.
- Pull request merged.
- Pull request closed.
- Pull request reopened.

![CodeBuild Create Project Webhook](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_webhook.png "CodeBuild Create Project Webhook")

Add **filter groups** to filter the events that will trigger the build. You can select to filter on:

![CodeBuild Create Project Filter Groups](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_filter_groups.png "CodeBuild Create Project Filter Groups")

Next, define the **environment**. You can select the **operating system** (Amazon Linux or Ubuntu), **runtime**, and **image**. You can also use custom images.

![CodeBuild Create Project Environment](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_environment.png "CodeBuild Create Project Environment")

The **environment's additional settings** allow you to configure options such as timeout, compute size, and **VPC (for resources in private subnets, such as databases)**:

![CodeBuild Create Project Environment Additional Settings](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_environment_additional_settings.png "CodeBuild Create Project Environment Additional Settings")

![CodeBuild Create Project Environment Additional Settings 2](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_environment_additional_settings_2.png "CodeBuild Create Project Environment Additional Settings 2")

Then, you need to provide the **buildspec that defines what to actually do during the build process**. You can either specify the commands or use a `buildspec.yml` file present in the source code repository.

![CodeBuild Create Project Buildspec](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_buildspec.png "CodeBuild Create Project Buildspec")

Configure whether to store the **artifacts** in S3:

![CodeBuild Create Project Artifacts](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_artifacts.png "CodeBuild Create Project Artifacts")

And configure the **logs**. You can store the logs in S3 and/or CloudWatch Logs:

![CodeBuild Create Project Logs](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_logs.png "CodeBuild Create Project Logs")

Create the project.

### 21.8.1 Starting a Build

You can **start a build** by clicking on the *Start build* button:

![CodeBuild Create Project Created](/assets/aws-certified-developer-associate/cicd_codebuild_create_project_created.png "CodeBuild Create Project Created")

You can see logs as the build is running:

![CodeBuild Project Build Logs](/assets/aws-certified-developer-associate/cicd_codebuild_project_build_logs.png "CodeBuild Project Build Logs")

Once it is done, you can see the logs in CloudWatch Logs as well:

![CodeBuild Project Build Logs CloudWatch](/assets/aws-certified-developer-associate/cicd_codebuild_project_build_logs_cloudwatch.png "CodeBuild Project Build Logs CloudWatch")

And if it fails, you can see the logs to understand what went wrong and **retry the build**:

![CodeBuild Project Retry Build](/assets/aws-certified-developer-associate/cicd_codebuild_project_retry_build.png "CodeBuild Project Retry Build")

### 21.8.2 Build History and Phase Details

An **history of the builds** is available in the *Build history* section:

![CodeBuild Project Build History](/assets/aws-certified-developer-associate/cicd_codebuild_project_build_history.png "CodeBuild Project Build History")

Meanwhile, you can see the **details of each phase** of the build in the *Phase details* section:

![CodeBuild Project Build Phase Details](/assets/aws-certified-developer-associate/cicd_codebuild_project_build_phase_details.png "CodeBuild Project Build Phase Details")

## 21.9 Integrating CodeBuild with CodePipeline to Build Code

To integrate CodeBuild with CodePipeline, you need to **remove webhooks** from the CodeBuild project. This is because **CodePipeline will trigger the build when it reaches the build stage**.

Then, go to CodePipeline and edit the pipeline you want to integrate with CodeBuild. Click on the *Edit* button and add a new stage to the pipeline:

![CodePipeline Add Stage CodeBuild](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_codebuild.png "CodePipeline Add Stage CodeBuild")

And configure CodeBuild as the **action provider**:

![CodePipeline Add Stage CodeBuild Action Provider](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_codebuild_action_provider.png "CodePipeline Add Stage CodeBuild Action Provider")

And make it use the `MyFirstBuild` project we created earlier in section [21.8 Using CodeBuild to Build Code](#218-using-codebuild-to-build-code):

![CodePipeline Add Stage CodeBuild Action Provider Project](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_codebuild_action_provider_project.png "CodePipeline Add Stage CodeBuild Action Provider Project")

And this is how it looks like:

![CodePipeline Stage Added](/assets/aws-certified-developer-associate/cicd_codepipeline_stage_added.png "CodePipeline Stage Added")

Now push a new change to the GitHub repository and it will trigger the pipeline. This time, it will start the build in CodeBuild after the source stage and before the deploy stage.

## 21.10 CodeDeploy

It is a deployment service that **automates application deployment**. It allows you to deploy new applications versions to:
- EC2 instances.
- On-premises servers.
- Lambda functions.
- ECS services.

It provides **automated rollback capability** in case of failed deployments or trigger of CloudWatch alarms.

It also offers **gradual deployment control**, for example, you can deploy to 10% of the instances first, then 20%, and so on.
- It also supports blue/green deployments.

A file named `appspec.yml` defines how the deployment happens.

### 21.10.1 Deploy to EC2 Instances/On-Premises Servers

Allows you **to deploy to EC2 Instances and on-premises servers by performing in-place deployments or blue/green deployments**.

To work, you must run the CodeDeploy Agent on the target instances because it is responsible for the deployment process.

You can define **deployment strategy**:
- `AllAtOnce`: most downtime because it deploys to all instances at once.
- `HalfAtATime`: deploys to half of the instances at a time, reducing capacity by 50%.
- `OneAtATime`: slowest option but has lowest impact on availability because only one instance is down at a time.
- `Custom`: define your own percentage of instances to deploy at a time.

### 21.10.2 CodeDeploy Agent

The CodeDeploy Agent must be running on the instances as a prerequisites.
- It can be installed manually.
- Or it can be installed and updated automatically if you are using Systems Manager.

The **instances must have sufficient permissions to access S3 to get deployment bundles** because the CodeDeploy Agent will download the deployment package from S3.

![CodeDeploy Agent](/assets/aws-certified-developer-associate/cicd_codedeploy_agent.png "CodeDeploy Agent")

### 21.10.3 In-place vs Blue/Green Deployments to EC2 Instances/On-Premises Servers

**In-place deployments** update the existing instances. For example, with `HalfAtATime`, it will update half of the instances at a time, then the other half. So, if you have 4 instances, it will update 2 of them, then the other 2.

![In-place Deployment](/assets/aws-certified-developer-associate/cicd_codedeploy_in_place_deployment.png "In-place Deployment")

**Blue/green deployments** create a new set of instances (the green ones) and deploy the new version there. Once the deployment is successful, it switches traffic to the new instances.

![Blue/Green Deployment](/assets/aws-certified-developer-associate/cicd_codedeploy_blue_green_deployment.png "Blue/Green Deployment")

### 21.10.4 Deploy to Lambda Functions

CodeDeploy can help you **automate traffic shift from Lambda function versions or aliases**.
- This feature is integrated within the SAM framework.

The way it works is that you can define a **traffic shifting policy** to gradually shift traffic from the old version to the new version of the Lambda function:

![CodeDeploy Lambda Traffic Shifting](/assets/aws-certified-developer-associate/cicd_codedeploy_lambda_traffic_shifting.png "CodeDeploy Lambda Traffic Shifting")

There are different traffic shifting policies you can use:
- **Linear**: grow traffic every N minutes until 100%.
  - `LambdaLinear10PercentEvery3Minutes`: increase traffic by 10% every 3 minutes.
  - `LambdaLinear10PercentEvery10Minutes`: increase traffic by 10% every 10 minutes.
- **Canary**: try X percent then 100% after N minutes.
  - `LambdaCanary10Percent5Minutes`: try 10% of the traffic for 5 minutes, then shift the rest.
  - `LambdaCanary10Percent30Minutes`: try 10% of the traffic for 30 minutes, then shift the rest.
- **AllAtOnce**: immediate traffic shift to the new version.

### 21.10.5 Deploy to ECS Services

CodeDeploy can help you automate the deployment of a new ECS task definition.
- It supports only blue/green deployments.

![CodeDeploy ECS Deployment](/assets/aws-certified-developer-associate/cicd_codedeploy_ecs_deployment.png "CodeDeploy ECS Deployment")

You need to define a **traffic shifting policy** to gradually shift traffic from the old task definition to the new one:
- **Linear**: grow traffic every N minutes until 100%.
  - `ECSLinear10PercentEvery3Minutes`: increase traffic by 10% every 3 minutes.
  - `ECSLinear10PercentEvery10Minutes`: increase traffic by 10% every 10 minutes.
- **Canary**: try X percent then 100% after N minutes.
  - `ECSCanary10Percent5Minutes`: try 10% of the traffic for 5 minutes, then shift the rest.
  - `ECSCanary10Percent30Minutes`: try 10% of the traffic for 30 minutes, then shift the rest.
- **AllAtOnce**: immediate traffic shift to the new task definition.

## 21.11 Using CodeDeploy to Deploy to EC2 Instances

### 21.11.1 Prerequisites

Before starting, **create a service role for CodeDeploy** to allow it to access the resources it needs. This role should have the `AWSCodeDeployRole` policy attached to it.

![CodeDeploy Service Role](/assets/aws-certified-developer-associate/cicd_codedeploy_service_role.png "CodeDeploy Service Role")

Then, **create a role for the EC2 instance we will deploy to**. This role should allow the instance to read from S3.

### 21.11.2 Creating CodeDeploy Applications

Go to CodeDeploy and **create a new application** by clicking on the *Create application* button:

![CodeDeploy Create Application](/assets/aws-certified-developer-associate/cicd_codedeploy_create_application.png "CodeDeploy Create Application")

Give the application a **name** and select the **compute platform**. In this case, we will use `EC2/On-premises`:

![CodeDeploy Create Application Name](/assets/aws-certified-developer-associate/cicd_codedeploy_create_application_name.png "CodeDeploy Create Application Name")

And create the application:

![CodeDeploy Application Created](/assets/aws-certified-developer-associate/cicd_codedeploy_application_created.png "CodeDeploy Application Created")

### 21.11.3 Creating Deployment Groups in Applications

Before creating a deployment group, you need to **create an EC2 instance** and **install the CodeDeploy Agent on it**.
- Assign it the role you created earlier that allows it to read from S3.
- Tag the instance with a key-value pair that you will use to identify it in the deployment group.
- For example, `Key: Environment, Value: Development` and `Key: Environment, Value: Production`.

To install the CodeDeploy Agent, you can use the following commands:

```bash
sudo yum update -y
sudo yum install -y ruby wget

# Download and install the CodeDeploy Agent
wget https://aws-codedeploy-eu-west-3.s3.eu-west-3.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto

# Check if the CodeDeploy Agent is running
sudo service codedeploy-agent status
```

Once the CodeDeploy Agent is installed and running, you can create a deployment group. Go to the CodeDeploy application and click on the *Create Deployment Group* button:

![CodeDeploy Create Deployment Group](/assets/aws-certified-developer-associate/cicd_codedeploy_application_created.png "CodeDeploy Create Deployment Group")

Give the deployment group a **name**:

![CodeDeploy Create Deployment Group Name](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_group_name.png "CodeDeploy Create Deployment Group Name")

Assign the **service role** you created earlier and select the **deployment type**. In this case, we will use `In-place`:

![CodeDeploy Create Deployment Group Service Role](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_group_service_role.png "CodeDeploy Create Deployment Group Service Role")

Then, select the **environment configuration**. In this case, we will use `Amazon EC2 instances`:

![CodeDeploy Create Deployment Group Environment Configuration](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_group_environment_configuration.png "CodeDeploy Create Deployment Group Environment Configuration")

Once you select the environment configuration, you need to specify how to **identify the instances via tags**.
- We will use the tags we created earlier to identify the instances. For example, `Key: Environment, Value: Development`.

![CodeDeploy Create Deployment Group Environment Configuration Tags](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_group_environment_configuration_tags.png "CodeDeploy Create Deployment Group Environment Configuration Tags")

You can optionally enable installation and update of the CodeDeploy Agent on the instances via Systems Manager.
- You need to define a cron expression to schedule the updates.

![CodeDeploy Create Deployment Group CodeDeploy Agent](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_group_codedeploy_agent.png "CodeDeploy Create Deployment Group CodeDeploy Agent")

Next are the **deployment settings**. You can select the deployment strategy among the ones presented earlier in section [21.10.1 Deploy to EC2 Instances/On-Premises Servers](#21101-deploy-to-ec2-instanceson-premises-servers):

![CodeDeploy Create Deployment Group Deployment Settings](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_group_deployment_settings.png "CodeDeploy Create Deployment Group Deployment Settings")

Optionally, you can **enable load balancer** support if you are using a load balancer in front of your instances. This will allow CodeDeploy to register the instances with the load balancer during the deployment.

![CodeDeploy Create Deployment Group Load Balancer Support](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_group_load_balancer_support.png "CodeDeploy Create Deployment Group Load Balancer Support")

Finally, create the deployment group:

![CodeDeploy Create Deployment Group Created](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_group_created.png "CodeDeploy Create Deployment Group Created")

### 21.11.4 Creating Deployments in Deployment Groups

To create a deployment, go to the deployment group you just created and click on the *Create Deployment* button:

![CodeDeploy Create Deployment](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_group_created.png "CodeDeploy Create Deployment")

It will be associated to the deployment group and you can **specify where to get the application revisions from**. In this case, we will use S3.
1. Create an S3 bucket and specify the full path to the application revision in the bucket (e.g., `s3://my-bucket/my-app-revision.zip`).
2. Upload the application revision (e.g., a zip file) to the S3 bucket.
3. The revision must contain an `appspec.yml` file that defines how the deployment should happen.

![CodeDeploy Create Deployment S3](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_s3.png "CodeDeploy Create Deployment S3")

The following is an example of an `appspec.yml` file:

```yaml
version: 0.0
os: linux

files:
  - source: /index.html
    destination: /var/www/html/

hooks:
  BeforeInstall:
    - location: scripts/install_dependencies
      timeout: 300
      runas: root
    - location: scripts/start_server
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server
      timeout: 300
      runas: root
```

You can set an optional **deployment description**, as well as other options:

![CodeDeploy Create Deployment Other Options](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_options.png "CodeDeploy Create Deployment Other Options")

Finally, create the deployment:

![CodeDeploy Create Deployment Created](/assets/aws-certified-developer-associate/cicd_codedeploy_create_deployment_created.png "CodeDeploy Create Deployment Created")

And check its progress in the *Deployment Lyfecycle Events* section:

![CodeDeploy Deployment Lifecycle Events](/assets/aws-certified-developer-associate/cicd_codedeploy_deployment_lifecycle_events.png "CodeDeploy Deployment Lifecycle Events")

By clicking on the *View events* button, you can see the details of each event:

![CodeDeploy Deployment Lifecycle Events Details](/assets/aws-certified-developer-associate/cicd_codedeploy_deployment_lifecycle_events_details.png "CodeDeploy Deployment Lifecycle Events Details")

If the deployment:
- Fails: you can see the error message and the logs to understand what went wrong, and also **retry the deployment**.
- Is successful: you can access the application by going to the public IP address of the EC2 instance (or the load balancer if you are using one).

Under deployments, you can see the **history of the deployments**:

![CodeDeploy Deployment History](/assets/aws-certified-developer-associate/cicd_codedeploy_deployment_history.png "CodeDeploy Deployment History")

## 21.12 CodeDeploy Deployments to ASG

It is possible to perform in-place and blue/green deployments.

**In-place deployment**:
- Updates existing EC2 instances.
- Newly created EC2 instances by an ASG will also get automated deployments.

**Blue/green deployment**:
- A new ASG is created by copying the settings of the old ASG.
- You need to choose how long to keep the old EC2 instances in the old ASG.
- Must be using an ELB to route traffic to ASGs.

![CodeDeploy ASG Deployment](/assets/aws-certified-developer-associate/cicd_codedeploy_asg_deployment.png "CodeDeploy ASG Deployment")

## 21.13 CodeDeploy Redeploy and Rollbacks

Deployments can be rolled back:
- Automatically: rollback when a deployment fails or rollback when a CloudWatch alarm thresholds are met.
- Manually.

It is possible to disable rollbacks, so CodeDeploy does not perform rollbacks for the deployment.

If a roll back happens, CodeDeploy redeploys the last known good revision as a new deployment (not a restored version).
- It is **important for the exam to understand that CodeDeploy does not restore the previous version, it redeploys the last known good revision**.

## 21.14 CodeArtifact

Software packages depend on each other to be built (i.e., code dependencies), and new ones are created.

**Storing and retrieving these dependencies is called artifact management**.

Traditionally you need to setup your own artifact management system but **CodeArtifact is a secure, scalable, and cost-effective artifact management for software development** that you can use to store and retrieve software packages.
- It works with common dependency management tools such as Maven, Gradle, npm, yarn, twine, pip, and NuGet.
- Developers and CodeBuild can retrieve dependencies from CodeArtifact.

CodeArtifact caches dependencies from public repositories such as Maven Central, npm registry, PyPI, NuGet Gallery, etc. In this way, you can **use CodeArtifact as a proxy to these public repositories in your VPC**. Even if the public repository is down, you can still retrieve the dependencies from CodeArtifact.

![CodeArtifact](/assets/aws-certified-developer-associate/cicd_codeartifact.png "CodeArtifact")

### 21.14.1 Integration with EventBridge

**Events are created when a package version is created, modified, or deleted**.

You can **use these events to trigger actions** in your application, such as sending notifications or starting a pipeline in CodePipeline.

![CodeArtifact EventBridge](/assets/aws-certified-developer-associate/cicd_codeartifact_eventbridge.png "CodeArtifact EventBridge")

### 21.14.2 CodeArtifact Authorization

Any artifact repository can be accessed by any user or service in the account. However, you need to **create a resource to allow other accounts to access your artifact repository**.
- A given principal can either read all the packages in a repository or none of them.

The following is an example of a resource policy that allows another account to read all the packages in the repository:

![CodeArtifact Resource Policy](/assets/aws-certified-developer-associate/cicd_codeartifact_resource_policy.png "CodeArtifact Resource Policy")

## 21.15 Creating CodeArtifact Repositories

Go to CodeArtifact and create a new repository by clicking on the *Create repository* button:

![CodeArtifact Create Repository](/assets/aws-certified-developer-associate/cicd_codeartifact_create_repository.png "CodeArtifact Create Repository")

Give the repository a **name** and optionally a **description**:

![CodeArtifact Create Repository Name](/assets/aws-certified-developer-associate/cicd_codeartifact_create_repository_name.png "CodeArtifact Create Repository Name")

And select the **upstream repositories**. You can select public repositories such as Maven Central, npm registry, PyPI, NuGet Gallery, etc. to cache dependencies from them:

![CodeArtifact Create Repository Upstream Repositories](/assets/aws-certified-developer-associate/cicd_codeartifact_create_repository_upstream_repositories.png "CodeArtifact Create Repository Upstream Repositories")

Next is to set a **domain** for the repository, meaning where the repository will be stored. You can either use your AWS account or a new one and then configure the domain settings:

![CodeArtifact Create Repository Domain](/assets/aws-certified-developer-associate/cicd_codeartifact_create_repository_domain.png "CodeArtifact Create Repository Domain")

In the domain, you need also to set the **encryption key** to use for the repository. You can use the default AWS managed key or a custom KMS key:

![CodeArtifact Create Repository Domain Encryption Key](/assets/aws-certified-developer-associate/cicd_codeartifact_create_repository_domain_encryption_key.png "CodeArtifact Create Repository Domain Encryption Key")

Then, you can review and create the repository, also seeing a flow of how it works:

![CodeArtifact Create Repository Created](/assets/aws-certified-developer-associate/cicd_codeartifact_create_repository_created.png "CodeArtifact Create Repository Created")

Once the repository is created, you can see it in the list of repositories along with the upstream repositories (connected to the public) it is using:

![CodeArtifact Repository List](/assets/aws-certified-developer-associate/cicd_codeartifact_repository_list.png "CodeArtifact Repository List")

From the list of repositories, you can click on a repository to access its details:

![CodeArtifact Repository Details](/assets/aws-certified-developer-associate/cicd_codeartifact_repository_details.png "CodeArtifact Repository Details")

You can also edit it from here.

### 21.15.1 Adding Packages to Repositories

We will be using `pip` to add packages to the repository. 

First, you need to **configure the repository as a pip source**.

1. Run the following command to get a CodeArtifact authorization token:
  ```bash
  export CODEARTIFACT_AUTH_TOKEN=$(
    aws codeartifact get-authorization-token \
      --domain my-domain \
      --domain-owner 123456789012 \
      --region eu-central-1 \
      --query authorizationToken \
      --output text)
  ```

2. Configure `pip` to use the CodeArtifact repository as a source:
  ```bash
  pip config set global.index-url \
    https://aws:$CODEARTIFACT_AUTH_TOKEN@
    my-domain-123456789012
    .d.codeartifact.eu-central-1.amazonaws.com/pypi/my-repository/simple/
  ```

Then, you can now install packages using `pip`, for example:

```bash
pip install response
```

And, by going to the CodeArtifact repository, you can **see the package added** (along with its dependencies):

![CodeArtifact Package Added](/assets/aws-certified-developer-associate/cicd_codeartifact_package_added.png "CodeArtifact Package Added")

Clicking on a package, you can **see the package details**, including the versions available (you can have multiple versions of the same package):

![CodeArtifact Package Details](/assets/aws-certified-developer-associate/cicd_codeartifact_package_details.png "CodeArtifact Package Details")

### 21.15.2 Applying Repository Policies

Repositories can be accessed by any user or service in the account. However, you can **apply a repository policy to allow other accounts to access the repository**.

Go to the CodeArtifact repository and click on the *Apply Repository Policy* button to define a policy:

![CodeArtifact Apply Repository Policy](/assets/aws-certified-developer-associate/cicd_codeartifact_apply_repository_policy.png "CodeArtifact Apply Repository Policy")

## 21.16 Managing CodeArtifact Domains

Go to CodeArtifact and click on the *Domains* tab to manage domains:

![CodeArtifact Domains](/assets/aws-certified-developer-associate/cicd_codeartifact_domains.png "CodeArtifact Domains")

If you click on a domain, you can see its details, including the repositories it contains:

![CodeArtifact Domain Details](/assets/aws-certified-developer-associate/cicd_codeartifact_domain_details.png "CodeArtifact Domain Details")

And you can also **apply a domain policy to allow other accounts to access the repositories in the domain**:

![CodeArtifact Domain Policy](/assets/aws-certified-developer-associate/cicd_codeartifact_domain_policy.png "CodeArtifact Domain Policy")

## 21.17 CodeGuru

An **ML-powered service for automated code reviews and application performance recommendations**. It provides two functionalities:
- CodeGuru Reviewer: automated code reviews for static code analysis for development.
- CodeGuru Profiler: visibility/recommendations about application performance during runtime for production.

![CodeGuru](/assets/aws-certified-developer-associate/cicd_codeguru.png "CodeGuru")

### 21.17.1 CodeGuru Reviewer

It **looks at commits to identify critical issues, security vulerabilities, and hard-to-find bugs**. It:
- Can provide recommendations about common code best practices, resource leaks, security detection, input validation, etc.
- Was trained on millions of code reviews on 1000s of open-source and Amazon repositories.
- Supports Java and Python.
- Can be integrated with CodeCommit, GitHub, and Bitbucket.

![CodeGuru Reviewer](/assets/aws-certified-developer-associate/cicd_codeguru_reviewer.png "CodeGuru Reviewer")

### 21.17.2 CodeGuru Profiler

It **helps understand the runtime behavior of your application**. For example, by identifying if the application is using too much CPU on a logging rotuine, and more.
- It support applications running on AWS or on-premise with minimal overhead on applications.

A few features:
- Identify and remove code inefficiencies.
- Improve application performance: for example, reduce CPU utilization.
- Decrease compute costs.
- Provides heap summary: see which objects are using up memory.
- Anomaly detection.

### 21.17.3 CodeGuru Profiler Agent Configuration

CodeGuru Profiler works thanks to an agent. The agent is a Java agent that you can add to your application. It can be configured to fine-tune the profiling process:
- `MaxStackDepth`: the maximum depth of the stacks in the code that is represented in the profile.
  - If it finds method A, which calls method B, which calls method C, which calls method D, then the depth is 4.
  - With `MaxStackDepth = 2` the profiler evaluates only A and B.
- `MemoryUsageLimitPercent`: the memory percentage used by the profiler.
- `MinimumTimeForReportingInMilliseconds`: the minimum time between sending reports in milliseconds.
- `ReportingIntervalInMilliseconds`: the reporting interval used to report profiles in milliseconds.
- `SamplingIntervalInMilliseconds`: the sampling interval that is used to profile samples in milliseconds.
  - Reduce it to have a higher sampling rate, which means more data is collected, but it can increase the overhead on the application.