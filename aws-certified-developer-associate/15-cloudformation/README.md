# 15 CloudFormation

CloudFormation is **a declarative way of defining AWS infrastructure resources** (most of them are supported) you need.

For example, within a **CloudFormation template**, you can specify that you need:
- A security group.
- Two EC2 instances using this security group.
- Two Elastic IPs for these EC2 instances.
- An S3 bucket.
- A load balancer (ELB) in front of these EC2 instances.

Then CloudFormation creates those for you, in the right order, with the exact configuration that you specify.

## 15.1 Benefits of CloudFormation

1. **Infrastructure as Code** (IaC):
    - No resources are manually created, which is excellent for control.
    - The code can be version controlled (e.g., using Git).
    - Changes to the infrastructure are reviewed through code.

2. **Cost**:
    - Each resources within the stack is tagged with an identifier, so you can easily see how much a stack costs you.
    - You can estimate the costs of your resources using the CloudFormation template.
    - Enables **savings strategies**: for example, in a development environment, you could automatically delete all resources in a template at 5 PM and recreate them at 8 AM, safely.

3. **Productivity**:
    - Ability to destroy and re-create an infrastructure on the cloud on the fly.
    - Automated generation of diagrams for your templates.
    - **Declarative programming**: no need to figure out ordering and orchestration.

4. **Separation of concern**: create many stacks for many applications, and many layers. For example, VPC stacks, network stacks, or application stacks.

5. **Do not re-invent the wheel**:
    - Leverage existing templates on the web.
    - Leverage the documentation.

## 15.2 How CloudFormation Works

Templates must be uploaded in S3 and then referenced in CloudFormation.

To update a template, we cannot edit previous ones. We have to upload a new version of the template to S3 and use it.

**AWS resources that need to be created are part of a stack**.
- Stacks are identified by a name.
- Deleting a stack deletes every single artifact that was created by CloudFormation.

![How CloudFormation Works](/assets/aws-certified-developer-associate/cf_how_it_works.png "How CloudFormation Works")

## 15.3 Deploying CloudFormation Templates

### 15.3.1 Manual Way

Manual deployment involves editing templates in the **Application Composer** or code editor and using the console to input parameters and more.

![Deploying CloudFormation Templates the Manual Way](/assets/aws-certified-developer-associate/cf_deploy_manual.png "Deploying CloudFormation Templates the Manual Way")

### 15.3.2 Automated Way

It involves editing templates in a YAML file and using the AWS CLI or continuous delivery tools to deploy the templates.

![Deploying CloudFormation Templates the Automated Way](/assets/aws-certified-developer-associate/cf_deploy_automated.png "Deploying CloudFormation Templates the Automated Way")

This is the recommended way when you want to fully automate your flow.

## 15.4 Building Blocks of Templates

Components of a template:
- **AWSTemplateFormatVersion**: identifies the capabilities of the template (e.g., `2010-09-09`).
- **Description**: comments about the template.
- **Resources (MANDATORY)**: defines the AWS resources to create.
- **Parameters**: the dynamic inputs for your template.
- **Mappings**: the static variables for your template.
- **Outputs**: references to what has been created.
- **Conditionals**: list of conditions to perform resource creation.

Helpers: **references** and **functions**.

All of these are discussed in following sections.

## 15.5 Creating a Stack

Go to the *CloudFormation*, then *Stacks*, and click on *Create Stack*.

![Creating a Stack](/assets/aws-certified-developer-associate/cf_create_stack.png "Creating a Stack")

Note that sample templates do not exist anymore. So, you can either use an existing template or build it from Application Composer.

### 15.5.1 Using Application Composer

If you **build it from Application Composer**, you can use the visual editor to create the template. You will have a dashboard like the following:

![Building a Stack from Application Composer](/assets/aws-certified-developer-associate/cf_build_stack.png "Building a Stack from Application Composer")

And can also see the template in the code editor (YAML or JSON):

![CloudFormation Template in Code Editor](/assets/aws-certified-developer-associate/cf_template_code_editor.png "CloudFormation Template in Code Editor")

### 15.5.2 Using Existing Template

If you **use an existing template**, you can:
- Provide the S3 URL of the template.
- Upload a template file.
- Sync a template file from Git.

The **template** can be like this where we create an EC2 instance:
```yaml
Resources:
    MyInstance:
        # This is the type of resource you want to create
        Type: AWS::EC2::Instance
        # Properties are the parameters for the resource
        Properties:
            AvailabilityZone: us-east-1a
            ImageId: ami-0a3c3a20c09d6f377
            InstanceType: t2.micro
```

And you can upload it using the **upload a template file** option:

![Uploading a Template](/assets/aws-certified-developer-associate/cf_upload_template.png "Uploading a Template")

As you can see, the template is uploaded to S3 when loaded.

Then, give the stack a **name** and eventually add **parameters**:

![Giving Stack a Name](/assets/aws-certified-developer-associate/cf_stack_name.png "Giving Stack a Name")

Next, you can optionally add **tags**, **permissions**, and **stack failure options**:

![Stack Failure Options](/assets/aws-certified-developer-associate/cf_stack_failure_options.png "Stack Failure Options")

As well as **advanced options**:

![Advanced Options](/assets/aws-certified-developer-associate/cf_advanced_options.png "Advanced Options")

Finally, you can review the stack and create it. See it in the *Stacks* dashboard with the resources being created and events generated as well (among other options):

![Stacks Dashboard](/assets/aws-certified-developer-associate/cf_stacks_dashboard.png "Stacks Dashboard")

**CloudFront tags the resources it creates**, for example, with the stack name and stack ID:

![CloudFormation Tags Resources](/assets/aws-certified-developer-associate/cf_tags_resources.png "CloudFormation Tags Resources")