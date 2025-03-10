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
        # The type of resource you want to create
        Type: AWS::EC2::Instance
        # Properties of the resource
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

## 15.6 Updating a Stack

To update a stack, you can use the *Update* button in the *Stacks* dashboard:

![Stacks Dashboard Update](/assets/aws-certified-developer-associate/cf_stacks_dashboard.png "Stacks Dashboard Update")

Which will allow you to:
- Replace the current template.
- Edit the current template.
- Keep the current template and only update the parameters, for example.

![Updating a Stack](/assets/aws-certified-developer-associate/cf_update_stack.png "Updating a Stack")

For instance, you can upload the following template that also has a parameter `SecurityGroupDescription` to replace the previous one:
```yaml
Parameters:
    SecurityGroupDescription:
        # | indicates a multiline string
        Description: |
            Security Group Description
        Type: String

Resources:
    MyInstance:
        Type: AWS::EC2::Instance
        Properties:
            AvailabilityZone: us-east-1a
            ImageId: ami-0a3c3a20c09d6f377
            InstanceType: t2.micro
            SecurityGroups:
                - !Ref SSHSecurityGroup
                - !Ref ServerSecurityGroup

    MyEIP:
        Type: AWS::EC2::EIP
        Properties:
            InstanceId: !Ref MyInstance
        DependsOn: MyInstance

    SSHSecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
            GroupDescription: |
                Enable SSH access via port 22
            SecurityGroupIngress:
                - CidrIp: 0.0.0.0/0
                  FromPort: 22
                  IpProtocol: tcp
                  ToPort: 22

    ServerSecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
            GroupDescription:
                !Ref SecurityGroupDescription
            SecurityGroupIngress:
                - IpProtocol: tcp
                  FromPort: 80
                  ToPort: 80
                  CidrIp: 0.0.0.0/0
                - IpProtocol: tcp
                  FromPort: 22
                  ToPort: 22
                  CidrIp: 192.168.1.1/32
```

When you move to the next step, you will be prompted to provide a value for the parameter `SecurityGroupDescription`:

![Providing Parameter Value](/assets/aws-certified-developer-associate/cf_parameter_value.png "Providing Parameter Value")

The rest of the options are the same as the creation of a stack. However, at the end of the review, you can see a **previow of the changes** that will be made as part of the stack update:

![Reviewing Changes](/assets/aws-certified-developer-associate/cf_review_changes.png "Reviewing Changes")

**Note**: existing resources might not need to be replaced because they can be updated in place. You can check whether a change requires a replacement (or other actions) by looking at the `Update requires` field in the documentation of the resource type. For example, the `ImageId` property of an EC2 instance requires replacement, while the `IamInstanceProfile` property does not.

![Update Requires](/assets/aws-certified-developer-associate/cf_update_requires.png "Update Requires")

The whole process of updating will take some time, and you can see the events generated in the *Events* tab of the stack.

Once completed, you can check that the `ServerSecurityGroup` security groups to verify that the description you passed as parameter is actually being used:

![Security Group Description](/assets/aws-certified-developer-associate/cf_security_group_description.png "Security Group Description")

## 15.7 Resources

Resources are the core of CloudFormation templates and only **mandatory section**. They represent the different AWS components that will be created and configured.
- Resources are declared and can reference each other.
- AWS figures out creation, updates and deletes of resources for us.

**Resource types identifiers** are of the form:

![Resource Type Identifier](/assets/aws-certified-developer-associate/cf_resource_type_identifier.png "Resource Type Identifier")

There are over *700 types of resources available* and all of them can be found at [https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html).

**Properties are used to configure the resources**. In the documentation of each resource, you can find the properties that can be used to configure the resource. For example, the following image shows the `ImageId` and `IamInstanceProfile` properties of an EC2 instance in the documentation:

![EC2 Resourse Documentation](/assets/aws-certified-developer-associate/cf_update_requires.png "EC2 Resourse Documentation")

Following is an example of an EC2 instance resource in YAML format with three properties (`AvailabilityZone`, `ImageId`, and `InstanceType`):
```yaml
Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            AvailabilityZone: us-east-1a
            ImageId: ami-0a3c3a20c09d6f377
            InstanceType: t2.micro
```

### 15.7.1 Resources-related FAQs

1. Can I create a dynamic number of resources? Yes, you can by using *CloudFormation Macros and Transform*.
2. **Is every AWS Service supported**? Almost. Only a select few niches are not there yet but you can work around that using **CloudFormation Custom Resources**.

## 15.8 Parameters

Parameters are a way to **provide inputs to templates**. They should be **used if a resource configuration is likely to change** in the future because you won't have to re-upload a template to change its content.

![Parameters](/assets/aws-certified-developer-associate/cf_parameters.png "Parameters")

They are important if:
- You want to reuse your templates.
- Some inputs cannot be determined ahead of time.

Parameters are extremely powerful, controlled, and can prevent errors from happening in templates thanks to **types** (e.g., `Type: String`).

The following is an example of a parameter of type string in YAML format:
```yaml
Parameters:
    SecurityGroupDescription:
        Description: |
            Security Group Description
        Type: String
```

The `Fn::Ref` function can be leveraged to **reference parameters**.
- The shorthand for this in YAML is `!Ref`.
- This function can also reference other elements within the template.
- Parameters can be used anywhere in a template

Considering the parameter `SecurityGroupDescription` defined above, we can reference it in the `ServerSecurityGroup` resource as follows:
```yaml
ServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
        # Reference the parameter
        GroupDescription:
            !Ref SecurityGroupDescription
        SecurityGroupIngress:
            - IpProtocol: tcp
              FromPort: 80
              ToPort: 80
              CidrIp:
```

Also, as we where saying, `!Ref` can be used to **reference other resources within the template**. For example, the following `MyInstance` EC2 instance resource references the `ServerSecurityGroup` resource defined above:
```yaml
Resources:
    MyInstance:
        Type: AWS::EC2::Instance
        Properties:
            AvailabilityZone: us-east-1a
            ImageId: ami-0a3c3a20c09d6f377
            InstanceType: t2.micro
            SecurityGroups:
                - !Ref ServerSecurityGroup
```

### 15.8.1 Parameters Settings

Parameters can be controlled by all these settings:
1. **Type**:
    - String.
    - Number.
    - CommaDelimitedList.
    - List of Number.
    - AWS-Specific Parameter: to help catch invalid values by matching against existing values in the AWS account.
    - List of AWS-Specific Parameter.
    - SSM Parameter: to reference a parameter from SSM Parameter store.
2. **Description**: description of the parameter.
3. ConstraintDescription (string).
4. Min/MaxLength.
5. Min/MaxValue.
6. **Default**: default value if the user does not provide one.
7. AllowedValues (array).
8. AllowedPattern (regex).
9. NoEcho (boolean): to hide the value when someone is viewing the stack.
10. ...

### 15.8.2 Important Examples for the Exam

1. **AllowedValues**: in the following example, the parameter `InstanceType` can only be `t2.micro`, `t2.small`, or `t2.medium` (this will show a dropdown in the CloudFormation console) with a default value of `t2.micro`:
    ```yaml
    Parameters:
        InstanceType:
            Type: String
            Default: t2.micro
            AllowedValues:
                - t2.micro
                - t2.small
                - t2.medium
            Default: t2.micro

    Resources:
        MyEC2Instance:
            Type: AWS::EC2::Instance
            Properties:
                # Reference the parameter above
                InstanceType: !Ref InstanceType
                ImageId: ami-0a3c3a20c09d6f377
    ```

2. **NoEcho**: in the following example, the parameter `DBPassword` will not be shown when someone is viewing the stack:
    ```yaml
    Parameters:
        DBPassword:
            Type: String
            NoEcho: true

    Resources:
        MyDBInstance:
            Type: AWS::RDS::DBInstance
            Properties:
                MasterUsername: admin
                MasterUserPassword: !Ref DBPassword
                Engine: mysql
                DBInstanceClass: db.t2.micro
                AllocatedStorage: 20
    ```

## 15.9 Pseudo Parameters

AWS provides pseudo parameters in any CloudFormation template, which can be used at any time and are enabled by default. A few important pseudo parameters:

| Reference value | Example of returned value |
|------------------|------------------------|
| AWS::AccountId   | 123456789012           |
| AWS::Region      | us-east-1              |
| AWS::StackId     | arn:aws:cloudformation:us-east-1:123456789012:stack/MyStack/1a2345b6-0c78-4d9d-8e7e-4f0123456789 |
| AWS::StackName   | MyStack                |
| AWS::NotificationARNs | [ "arn:aws:sns:us-east-1:123456789012:MyTopic" ] |
| AWS::NoValue     | Does not return a value |

For instance, `AWS::AccountId` returns your AWS account ID, `AWS::Region` returns the region where the stack is being created, etc.


## 15.10 Mappings

Mappings are **fixed variables within templates**. They are very handy to differentiate between different environments (dev vs prod), regions, AMI types, etc.

All the values are hardcoded within the template. For example:
```yaml
Mappings:
    RegionMap:
        us-east-1:
            HVM64: ami-0ff8a91507f77f867
            HVMG2: ami-0a584ac55a7631c0c
        us-west-1:
            HVM64: ami-0bdb828fd58c52235
            HVMG2: ami-0e4176b225a5f442d
        eu-west-1:
            HVM64: ami-047bb4163c506cd98
            HVMG2: ami-0a9d27a9f4f5c0efc
```

Then, you can **reference the mapping** in the template using the `Fn::FindInMap` function to return a named value from a specific key.
- The shorthand for this in YAML is `!FindInMap`.
- The function takes three parameters: the **name of the mapping**, the **top-level key**, and the **second-level key** (e.g., `!FindInMap [ RegionMap, us-west-1, HVM64 ]`).
- You can use pseudo parameters in mappings as top-level or second-level keys.

For example, in a template you can use the `RegionMap` mapping like this:
```yaml
MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
        AvailabilityZone: us-east-1a
        ImageId:
            # Reference the mapping using the Aws::Region
            # pseudo parameter as the top-level key to
            # get the AMI ID for the current region
            !FindInMap [ RegionMap, !Ref "AWS::Region", HVM64 ]
        InstanceType: t2.micro
```

### 15.10.1 Mappings vs Parameters

Mappings are great when you know in advance all the values that can be taken and that they can be deduced from variables such as:
- Region.
- Availability Zone.
- AWS Account.
- Environment: dev vs prod.
- ...

Mappings allow safer control over the template.

However, if you have really user specific inputs, then you should use parameters.

## 15.11 Outputs

The `Outputs` section is optional and **declares optional output values that we can import into other stacks** if you export them first.

For example, outputs can be very useful if you define a network stack and output the variables such as VPC ID and Subnet IDs. Then, you can reference these outputs in other stacks, such as an application stack that needs the VPC ID.

![Outputs](/assets/aws-certified-developer-associate/cf_outputs.png "Outputs")

You can also view the outputs in the AWS console or by using the AWS CLI. They are the best way to perform some collaboration cross stack, as you let each expert handle their own part of the stack.

For example, we can create an `SSHSecurityGroup` resource as part of one template, and create an `ReusableSSHSecurityGroup` output that references that security group and exports it as `CompanySSHSecurityGroup`:
```yaml
Resources:
    SSHSecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
            GroupDescription: |
                Enable SSH access via port 22
            SecurityGroupIngress:
                - CidrIp: 0.0.0.0/0
                  FromPort: 22
                  IpProtocol: tcp
                  ToPort: 22

Outputs:
    ReusableSSHSecurityGroup:
        Description: |
            The SSH security group for the company
        Value: !Ref SSHSecurityGroup
        Export:
            Name: CompanySSHSecurityGroup
```

The **export name has to be unique within the region**.

Then, create a second template that leverages that security group by **importing the output** `CompanySSHSecurityGroup` via the `Fn::ImportValue` (shorthand `!ImportValue`) function:
```yaml
Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            AvailabilityZone: us-east-1a
            ImageId: ami-0a3c3a20c09d6f377
            InstanceType: t2.micro
            SecurityGroups:
                # Import the security group from
                # the other template
                - !ImportValue CompanySSHSecurityGroup
```

Given that **the imported value creates a dependency between the two stacks**, the stack that exports the value cannot be deleted before the stack that imports it.

## 15.12 Conditions

Conditions are used to **control the creation of resources or outputs based on a condition**. Each condition can reference another condition, parameter value or mapping.

Conditions can be whatever you want them to be, but common ones are:
- Environment: dev, test, prod.
- AWS Region.
- Any parameter value.

For example, you can create a condition that checks whether the environment is `prod` or `dev` to create a resource or not (the EBS volume in this case):

![Conditions](/assets/aws-certified-developer-associate/cf_conditions.png "Conditions")

To define a condition, you can use the `Conditions` section in the template:
```yaml
Conditions:
    CreateProdResources:
        !Equals [ !Ref EnvType, prod ]
```

The **logical ID is how you name conditions** and is for you to choose it.

The **intrinsic function (logical)** can be any of the following:
- `Fn::And` (`!And`).
- `Fn::Equals` (`!Equals`).
- `Fn::If` (`!If`).
- `Fn::Not` (`!Not`).
- `Fn::Or` (`!Or`).

Then, you can **use the condition** in the resources section to create the EBS volume only if the environment is `prod`:
```yaml
Resources:
    MountPoint:
        Type: AWS::EC2::VolumeAttachment
        Condition: CreateProdResources
```

Conditions can be **applied** to resources, outputs, etc.

## 15.13 Intrinsic Functions

### 15.13.1 Must-Know Intrinsic Functions

These are very **important to know for the exam**:

| Function | Description |
|----------|-------------|
| `Fn::Ref`      | Used to reference resources, parameters, outputs, etc. |
| `Fn::GetAtt` | Used to get an attribute from a resource |
| `Fn::FindInMap` | Used to find a value in a mapping |
| `Fn::ImportValue` | Used to import values from another stack |
| `Fn::Base64` | Used to encode to Base64 |
| `Fn::And`, `Fn::Equals`, `Fn::If`, `Fn::Not`, `Fn::Or` | Condition functions used to conditionally create resources |

### 15.13.2 Other Intrinsic Functions

| Function | Description |
|----------|-------------|
| `Fn::Join` | Used to join values with a delimiter |
| `Fn::Sub` | Used to substitute variables |
| `Fn::ForEach` | Used to iterate over a list |
| `Fn::ToJsonString` | Used to convert a JSON object to a string |
| `Fn::Cidr` | Used to generate CIDR numbers |
| `Fn::GetAZs` | Used to get availability zones |
| `Fn::Select` | Used to select an element from a list |
| `Fn::Split` | Used to split a string |
| `Fn::Transform` | Used to transform a document |
| `Fn::Length` | Used to get the length of a string |

### 15.13.3 Fn::Ref

The` Fn::Ref` function can be **leveraged to reference**:
- **Parameters**: returns the value of the parameter.
- **Resources**: returns the *physical ID* of the underlying resource (e.g., EC2 ID).

The shorthand for this in YAML is `!Ref`.

For example, you can reference the `InstanceType` parameter in the `MyEC2Instance` resource:
```yaml
Parameters:
    InstanceType:
        Type: String
        Default: t2.micro
        AllowedValues:
            - t2.micro
            - t2.small
            - t2.medium
        Default: t2.micro

Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            # Reference the parameter above
            InstanceType: !Ref InstanceType
            ImageId: ami-0a3c3a20c09d6f377
```

### 15.13.4 Fn::GetAtt

**Attributes are attached to any resources you create**. To know the attributes of your resources, the best place to look at is the documentation.
- For example, the AZ of an EC2 instance.

For example, create an EBS volime in the same AZ of `MyEC2Instance`:
```yaml
Resources:
    Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            InstanceType: t2.micro
            ImageId: ami-0a3c3a20c09d6f377

    MyEBSVolume:
        Type: AWS::EC2::Volume
        Properties:
            Size: 100
            # Get the AZ of MyEC2Instance
            # - MyEC2Instance is the logical ID of the EC2 instance,
            # which is defined above in the template
            # - AvailabilityZone is the attribute of the EC2 instance
            AvailabilityZone:
                !GetAtt MyEC2Instance.AvailabilityZone
```

### 15.13.5 Fn::FindInMap

We use `Fn::FindInMap` to **return a named value from a specific key**.

The function takes **three parameters**:
- Name of the mapping.
- Top-level key.
- Second-level key.
- Example usage: `!FindInMap [ RegionMap, us-west-1, HVM64 ]`.

You **can use pseudo parameters in mappings as top-level or second-level keys**.

For example, in a template you can use a `RegionMap` mapping like this:
```yaml
Mappings:
    RegionMap:
        us-east-1:
            HVM64: ami-0ff8a91507f77f867
            HVMG2: ami-0a584ac55a7631c0c
        us-west-1:
            HVM64: ami-0bdb828fd58c52235
            HVMG2: ami-0e4176b225a5f442d
        eu-west-1:
            HVM64: ami-047bb4163c506cd98
            HVMG2: ami-0a9d27a9f4f5c0efc

Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            AvailabilityZone: us-east-1a
            ImageId:
                # Reference the mapping using the Aws::Region
                # pseudo parameter as the top-level key to
                # get the AMI ID for the current region
                !FindInMap [ RegionMap, !Ref "AWS::Region", HVM64 ]
            InstanceType: t2.micro
```

### 15.13.6 Fn::ImportValue

Import values that are exported in other stacks. For this, we use the `Fn::ImportValue` function.

For example, we can create an `SSHSecurityGroup` resource as part of one template, and create an `ReusableSSHSecurityGroup` output that references that security group and exports it as `CompanySSHSecurityGroup`:
```yaml
Resources:
    SSHSecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
            GroupDescription: |
                Enable SSH access via port 22
            SecurityGroupIngress:
                - CidrIp: 0.0.0.0/0
                  FromPort: 22
                  IpProtocol: tcp
                  ToPort: 22

Outputs:
    ReusableSSHSecurityGroup:
        Description: |
            The SSH security group for the company
        Value: !Ref SSHSecurityGroup
        Export:
            Name: CompanySSHSecurityGroup
```

The **export name has to be unique within the region**.

Then, create a second template that leverages that security group by **importing the output** `CompanySSHSecurityGroup` via the `Fn::ImportValue` (shorthand `!ImportValue`) function:
```yaml
Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            AvailabilityZone: us-east-1a
            ImageId: ami-0a3c3a20c09d6f377
            InstanceType: t2.micro
            SecurityGroups:
                # Import the security group from
                # the other template
                - !ImportValue CompanySSHSecurityGroup
```

### 15.13.7 Fn::Base64

Convert trings to their Base64 representation, e.g., `!Base64: "Encode me as Base64"`.

A common use case is to encode user data for EC2 instances in the `UserData` property:
```yaml
WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
        # Other properties...
        UserData:
            !Base64: |
                #!/bin/bash
                yum update -y
                yum install -y httpd
```

### 15.13.8 Condition Functions

Condition functions are:
- `Fn::And` (`!And`).
- `Fn::Equals` (`!Equals`).
- `Fn::If` (`!If`).
- `Fn::Not` (`!Not`).
- `Fn::Or` (`!Or`).

And they can be used in the `Conditions` section of the template to conditionally create resources.

For example, you can create a condition that checks if the environment is `prod` using the `!Equals` condition function:
```yaml
Conditions:
    CreateProdResources:
        !Equals [ !Ref EnvType, prod ]
```

## 15.14 Rollbacks

1. **Stack creation fails**:
    - Default: everything rolls back ands get deleted. You can look at logs to understand what happened but not at the resources.
    - You can disable the rollback and troubleshoot what happened.
2. **Stack update fails**:
    - Default: everything rolls back to the previous known working state.
    - You can investigate using the logs.

When performing a rollback, there is a chance the **rollback can fail**. In this case, you can fix resources manually and continue with the rollback using the `ContinueUpdateRollback` API, CLI command, or console option.

### 15.14.1 Triggering a Rollback

To trigger a failure, you can just update an existing template containing a `MyEC2Instance` resource with a new template where the `MyEC2Instance` resource references a non-existing AMI ID, like this:
```yaml
MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
        AvailabilityZone: us-east-1a
        ImageId: ami-12345678
        InstanceType: t2.micro
```

When you do the upload on the console, you can configure **stack failure options**:

![Stack Failure Options for Rollback](/assets/aws-certified-developer-associate/cf_stack_failure_options_rollback.png "Stack Failure Options for Rollback")

Then, when you update the stack, it will fail and you can see the rollback in the *Events* tab of the stack:

![Rollback Events](/assets/aws-certified-developer-associate/cf_rollback_events.png "Rollback Events")

## 15.15 Service Roles

CloudFront uses service roles, which are IAM roles that allow CloudFormation to create/update/delete stack resources on your behalf.
- For service roles to work, a user must have the `iam:PassRole` permissions.

You can **use service roles to give ability to users to create/update/delete the stack resources even if they do not have permissions to work with the direct resources** in the stack.
- For example, the user does not have permissions to create an S3 bucket, but they can create a CloudFormation stack that creates an S3 bucket.

    ![Service Roles](/assets/aws-certified-developer-associate/cf_service_roles.png "Service Roles")

**Use case**: you want to achieve the least privilege principle but you do not want to give the user all the required permissions to create the stack direct resources.

### 15.15.1 Using Service Roles

Go to IAM roles and create a new role for an AWS service. Then, **select CloudFormation as the service that will use this role**:

![Creating a Service Role](/assets/aws-certified-developer-associate/cf_create_service_role.png "Creating a Service Role")

Then, **attach the required policies** to the role. For example, you can attach the `AmazonS3FullAccess` policy to allow CloudFormation to work with S3 buckets (e.g., creation):

![Attaching Policies to Service Role](/assets/aws-certified-developer-associate/cf_attach_policies_service_role.png "Attaching Policies to Service Role")

Finally, give a **name** and **description** to the role and create it.

![Service Role Creation](/assets/aws-certified-developer-associate/cf_service_role_creation.png "Service Role Creation")

Then, when creating a stack, you can **select the service role** you created:

![Selecting Service Role](/assets/aws-certified-developer-associate/cf_select_service_role.png "Selecting Service Role")

## 15.16 Capabilities

When using some CloudFormation features, you need to acknowledge that you are aware of the capabilities that you are enabling.

1. **CAPABILITY_NAMED_IAM** and **CAPABILITY_IAM**:
    - Necessary to enable when your CloudFormation template is creating or updating IAM resource. For example, IAM user, role, group, policy, access keys, instance profile, etc.
    - Specify **CAPABILITY_NAMED_IAM if the resources are named**.

2. **CAPABILITY_AUTO_EXPAND**:
    - Necessary when your CloudFormation template includes **macros** or **nested stacks** (stacks within stacks) to perform dynamic transformations.
    - You are basically acknowledging that your template may change before deploying.

If you get `InsufficientCapabilitiesException` when launching a template, you need to add the necessary capabilities. It is a **security measure for CloudFormation to throw an exception if the capabilities have not been acknowledged** when deploying a template.

For example if you upload a template that creates IAM resources via the console, you will be prompted to **acknowledge the capabilities**:

![Acknowledging Capabilities](/assets/aws-certified-developer-associate/cf_acknowledge_capabilities.png "Acknowledging Capabilities")

## 15.17 Deletion Policy

`DeletionPolicy` is a **safety measure to preserve and backup resources** in a template that you can apply to resources to **control what happens when the template is deleted or when a resource is removed from a template**.
- **Default** is `Delete`: a resource is deleted when the stack is deleted.
- Other options: `Retain` and `Snapshot`.

### 15.17.1 Delete Option

With `Delete`, a resource is deleted when the stack is deleted or in any case of deletion.

The following is an example of a `DeletionPolicy: Delete` applied to an S3 bucket:
```yaml
Resources:
    MyBucket:
        Type: AWS::S3::Bucket
        DeletionPolicy: Delete
```

**Delete won't work on an S3 bucket if the bucket is not empty**: it is up to you to implement a custom resource that empties the bucket before deletion.

### 15.17.2 Retain Option

`Retain` is another option for `DeletionPolicy` that **prevents the resource from being deleted in case of deletes**.
- It works with any resource.

For exaple, the following is an example of a `DeletionPolicy: Retain` applied to a DynamoDB table:
```yaml
Resources:
    MyDynamoDBTable:
        Type: AWS::DynamoDB::Table
        Properties:
            TableName: MyTable
            AttributeDefinitions:
                - AttributeName: "Name"
                  AttributeType: "S"
            KeySchema:
                - AttributeName: "Name"
                  KeyType: "HASH"
            ProvisionedThroughput:
                ReadCapacityUnits: 5
                WriteCapacityUnits: 5
        DeletionPolicy: Retain
```

### 15.17.3 Snapshot Option

`Snapshot` allows you to create one final snapshot before deleting the resource. A few examples of supported resources are:
- EBS Volume.
- ElastiCache Cluster.
- ElastiCache ReplicationGroup.
- RDS DBInstance.
- RDS DBCluster.
- Redshift Cluster.
- Neptune DBCluster.
- DocumentDB DBCluster.
- ...

An example of a `DeletionPolicy: Snapshot` applied to an RDS DBInstance:
```yaml
Resources:
    MyDBInstance:
        Type: AWS::RDS::DBInstance
        Properties:
            DBInstanceClass: db.t2.micro
            AllocatedStorage: 20
            Engine: mysql
            MasterUsername: admin
            MasterUserPassword: password
        DeletionPolicy: Snapshot
```

## 15.18 Stack Policies

During a stack update, all update actions are allowed on all resources by default. However, you can use a stack policy to control what actions can be performed during a stack update.

A **stack policy is a JSON document that defines the update actions that are allowed on specific resources during stack updates**.

The following policy allows updates on all resources except the `ProductionDatanbase`:
```json
{
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "Update:*",
            "Principal": "*",
            "Resource": "*"
        },
        {
            "Effect": "Deny",
            "Action": "Update:*",
            "Principal": "*",
            "Resource": "LogicalResourceId/ProductionDatabase"
        }
    ]
}
```

The goal is to **protect resources from unintentional updates**.
- When you set a stack policy, all resourced are protected, so all update actions are denied by default.
- You need to explicitly allow the update actions on specific resources.

## 15.19 Termination Protection

`TerminationProtection` allows you to prevent accidental deletes of CloudFormation stacks.

You can enabled it in the console or using the CLI. From the console, go to the stack details and click on *Edit Termination Protection* in the *Stack Actions* dropdown. This will open a modal where you can enable termination protection:

![Termination Protection](/assets/aws-certified-developer-associate/cf_termination_protection.png "Termination Protection")

After enabling termination protection, you will see a warning if you try to delete the stack:

![Termination Protection Warning](/assets/aws-certified-developer-associate/cf_termination_protection_warning.png "Termination Protection Warning")

## 15.20 Custom Resources

Custom resources are used to:
- Define resources not yet supported by CloudFormation.
- Define custom provisioning logic for resources can that be outside of CloudFormation: on-premises resources, 3rd party resources, etc.
- **Have custom scripts run during create/update/delete through Lambda functions**: for example, running a Lambda function to empty an S3 bucket before being deleted. This is a **common use case in the exam**.

### 15.20.1 How To Define Custom Resources

You can **define custom resources** in the `Resources` section of the template using:
- `AWS::CloudFormation::CustomResource`.
- `Custom::MyCustomResourceTypeName`: this is the recommended approach.

They are **backed by a Lambda function (most common) or an SNS topic**. For example, to define a custom resource that is backed by a Lambda function:
```yaml
Resources:
    MyCustomResourceUsingLambda:
        Type: Custom::MyLambdaResource
        Properties:
            # Service token has the form:
            # - arn:aws:lambda:<region>:<account-id>:function:<function-name>
            ServiceToken: |
                arn:aws:lambda:us-east-1:123456789012:function:MyLambdaFunction
            # Input to the Lambda function (optional)
            ExampleProperty: ExampleValue
```

In the example above:
- `ServiceToken` specifies where CloudFormation sends requests to, such as Lambda ARN or SNS ARN (required and must be in the same region)
- **Input data parameters** (optional).

### 15.20.2 Use Case: Deleting Content Before Deleting an S3 Bucket

You can't delete a non-empty S3 bucket. So, to delete a non-empty S3 bucket, you must first delete all the objects inside it.

You can use a custom resource to empty an S3 bucket before it gets deleted by CloudFormation. So, you can create a custom resource that when it gets deleted, it triggers a Lambda function that empties the bucket.

![Custom Resource to Empty S3 Bucket](/assets/aws-certified-developer-associate/cf_custom_resource_empty_s3_bucket.png "Custom Resource to Empty S3 Bucket")

## 15.21 StackSets

**Stack sets allow you to create, update, or delete stacks across multiple accounts and regions with a single operation or template**.

It uses target accounts to create, update, or delete stack instances in multiple regions from a stack set.

When you update a stack set, it updates all the stack instances across all the accounts and regions.

Usually, stack sets are applied into all accounts of an AWS Organization.

**Only an administrator account (or delegated administrator) can create stack sets**.

![StackSets](/assets/aws-certified-developer-associate/cf_stacksets.png "StackSets")