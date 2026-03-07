# 22 Serverless Application Model (SAM)

SAM is a **framework for developing and deploying serverless applications**.

All the configuration is YAML code that is used to generate complex CloudFormation.

SAM:
- Supports anything from CloudFormation: outputs, mappings, parameters, resources.
- Can use CodeDeploy to deploy Lambda functions.
- Can help run Lambda functions, API Gateway, and DynamoDB locally.

## 22.1 Recipes

SAM is made of rceipes that are used to create the serverless application. The recipes are defined in a `template.yaml` file.
- Use the transform header to indicate that it is SAM template: `Transform: 'AWS::Serverless-2016-10-31'`.

Write applications using resources that are specific to SAM:
- `AWS::Serverless::Function` to define a Lambda function.
- `AWS::Serverless::Api` to define an API Gateway.
- `AWS::Serverless::SimpleTable` to define a DynamoDB table.

### 22.1.1 Useful Commands

- `sam init` to create a new SAM application.
- `sam build` to build the application.
    - The destination folder is `.aws-sam`.
    - Build artifacts will be stored in the `.aws-sam/build` directory.
- `sam validate` to validate the SAM template.
- `sam package` to package the application for deployment (optional because `sam deploy` can handle this automatically).
- `sam deploy` to deploy the application.
    - `sam deploy --guided` to walk through the deployment process interactively.
- `sam sync` to synchronize local changes with the deployed application.
    - `sam sync --watch` to watch for changes and automatically deploy them.
- `sam delete` to delete the deployed application.

### 22.1.2 Deployment Process

1. **Build**: Use `sam build` to compile the application and prepare it for deployment.
2. **Package**: Use `sam package` to package the application, uploading local artifacts to S3 and generating a CloudFormation template.
    - You can **proceed directly to deploy as this step is optional and not required anymore**.
3. **Deploy**: Use `sam deploy` to deploy the application, creating or updating the CloudFormation stack.

![SAM Deployment Process](/assets/aws-certified-developer-associate/sam_deployment_process.png "SAM Deployment Process")

### 22.1.3 SAM Accelerate

SAM Accelerate is a **set of features to reduce latency while deploying resources to AWS**.

Leverage them by running `sam sync` commands to:
- Synchronize your project declared in SAM templates to AWS.
- Synchronizes code changes to AWS without updating infrastructure, so using service APIs and bypassing CloudFormation.

**Options** for `sam sync`:
- `sam sync (no options)`: synchronizes code and infrastructure.
- `sam sync --code`: synchronizes code changes without updating infrastructure, it bypasses CloudFormation to provide updates in seconds.
- `sam sync --code --resource AWS::Serverless::Function`: synchronizes only all Lambda functions and their dependencies.
- `sam sync --code --resource-id HelloWorldLambdaFunction`: synchronizes only a specific resource by its ID, in this case, a Lambda function named `HelloWorldLambdaFunction`.
- `sam sync --watch`: monitors for file changes and automatically synchronize when changes are detected.
    - If changes include configuration, it uses `sam sync`.
    - If changes are code only, it uses `sam sync --code`.

## 22.2 Creating a SAM Project

1. Use the `sam init` command to create a new SAM project. You can use different options:
    - `--name`: specify the name of the project.
    - `--runtime`: specify the runtime (e.g., `python3.8`, `nodejs14.x`, etc.).
2. Create an S3 bucket to store the packaged application artifacts: `aws s3 mb s3://helloworld-sam-bucket`.
3. Package the application using the command below to upload the application code to S3 and generate a CloudFormation template saved in `generated-cf-template.yaml` (optional step, as `sam deploy` can handle this automatically):
    ```bash
    sam package \
        --s3-bucket helloworld-sam-bucket \
        --output-template-file generated-cf-template.yaml
    ```
4. Deploy the application using the command below:
    ```bash
    sam deploy \
        --template-file generated-cf-template.yaml \
        --stack-name helloworld-sam-stack \
        --capabilities CAPABILITY_IAM
    ```

You can see deployed SAM applications in the console under the *Applications* section in the Lambda console:

![SAM Applications in Lambda Console](/assets/aws-certified-developer-associate/sam_applications_lambda_console.png "SAM Applications in Lambda Console")

### 22.2.1 SAM Project Structure

The generated project will include:
- A `template.yaml` file with the SAM template.
- A directory structure for the application code: for example, `src/app.py` for Python applications.

### 22.2.2 Adding Lambda Functions to SAM Templates

The following is an example of a `template.yaml` file for a simple SAM application that **defines a Lambda function** that returns `"Hello, World!"`:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: A Hello World Lambda function.
Resources:
    HelloWorldFunction:
        Type: 'AWS::Serverless::Function'
        Properties:
            # It is file-name.function-name
            Handler: app.lambda_handler
            Runtime: python3.8
            # The CodeUri is the path to the source code directory
            CodeUri: src/
            Description: A function that returns "Hello, World!"
            MemorySize: 128
            Timeout: 3
```

The code of the function is in the `src/app.py` file, and can be as simple as:

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello, World!'
    }
```

### 22.2.3 Adding API Gateway to SAM Templates

To add an API Gateway to the SAM template defined in section [22.2.2 SAM Template with Lambda Functions](#2222-sam-template-with-lambda-functions), you can add the `Events` property to the `HelloWorldFunction` resource to **allow the Lambda function to be triggered by an API Gateway event**. By doing so, SAM automatically creates an API Gateway for the function.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: A Hello World Lambda function with API Gateway.
Resources:
    HelloWorldFunction:
        Type: 'AWS::Serverless::Function'
        Properties:
            Handler: app.lambda_handler
            Runtime: python3.8
            CodeUri: src/
            Description: A function that returns "Hello, World!"
            MemorySize: 128
            Timeout: 3
            # Defines the event source for API Gateway
            Events:
                HelloWorldApi:
                    Type: Api
                    Properties:
                        Path: /hello
                        Method: GET
```

The `src/app.py` file can be updated to provide a response in the expected format for API Gateway:

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello, World!'
        'headers': {
            'Content-Type': 'application/json'
        }
    }
```

You can see the API Gayeway's stage, deployment, and REST API being created by CloudFormation in the console after running the `sam deploy` command.

### 22.2.4 Adding DynamoDB to SAM Templates

Write a function that interacts with DynamoDB like the following example:

```python
import boto3
import json
import os

region_name = os.environ['REGION_NAME']
table_name = os.environ['TABLE_NAME']

dynamodb_client = boto3.client(
    'dynamodb',
    region_name=region_name
)

def respond(err, res=None):
    return {
        'statusCode': '400' if err else '200',
        'body': err.message if err else json.dumps(res),
        'headers': {
            'Content-Type': 'application/json',
        },
    }


def lambda_handler(event, context):
    print("Received event: " + json.dumps(event, indent=2))
    scan_result = dynamodb_client.scan(TableName=table_name)
    return respond(None, res=scan_result)
```

The functions is:
- Reading two environment variables: `REGION_NAME` and `TABLE_NAME`, which need to be defined in the SAM template using the `Environment` property.
- Scanning a DynamoDB table and returning the results, so we need to **define the table in the SAM template**. For this, we also need to **add a policy to allow the Lambda function to access the DynamoDB table**.

The following example adds a DynamoDB table and sets the environment variables for the Lambda function to the SAM template in section [22.2.3 Adding API Gateway to SAM Templates](#2223-adding-api-gateway-to-sam-templates):

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: A Hello World Lambda function with API Gateway.
Resources:
    HelloWorldFunction:
        Type: 'AWS::Serverless::Function'
        Properties:
            Handler: app.lambda_handler
            Runtime: python3.8
            CodeUri: src/
            Description: A function that returns "Hello, World!"
            MemorySize: 128
            Timeout: 3
            # Defines the environment variables
            Environment:
                Variables:
                    TABLE_NAME: !Ref HelloWorldTable
                    REGION_NAME: !Ref AWS::Region
            Events:
                HelloWorldApi:
                    Type: Api
                    Properties:
                        Path: /hello
                        Method: GET
            Policies:
            - DynamoDBCrudPolicy:
                TableName: !Ref HelloWorldTable  
    # Defines the DynamoDB table
    HelloWorldTable:
        Type: AWS::Serverless::SimpleTable
        Properties:
            PrimaryKey:
                Name: greeting
                Type: String
            ProvisionedThroughput:
                ReadCapacityUnits: 2
                WriteCapacityUnits: 2
```

## 22.3 SAM Policy Templates

List of templates to apply permissions to Lambda functions. They will result in IAM policies that are attached to the Lambda functions' execution role.

A few examples:
- `S3ReadPolicy`: Gives read only permissions to objects in S3.
- `SQSPollerPolicy`: Allows to poll an SQS queue.
- `DynamoDBCrudPolicy`: Allows to perform CRUD operations on a DynamoDB table.

To use them, you can add them to the `Policies` property of the Lambda function in the SAM template:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: A Lambda function that connects to DynamoDB, S3, and SQS.
Resources:
    HelloWorldFunction:
        Type: 'AWS::Serverless::Function'
        Properties:
            Handler: app.lambda_handler
            Runtime: python3.8
            CodeUri: src/
            Description: A function that returns "Hello, World!"
            MemorySize: 128
            Timeout: 3
            Policies:
            - DynamoDBCrudPolicy:
                TableName: !Ref HelloWorldTable
            - S3ReadPolicy:
                BucketName: !Ref HelloWorldBucket
            - SQSPollerPolicy:
                QueueName: !Ref HelloWorldQueue
# Define other resources here...
```

## 22.4 SAM and CodeDeploy

SAM framework natively uses CodeDeploy to update Lambda functions.

You can use:
- **Traffic shifting** to gradually shift traffic from the old version of a Lambda function to a new version.
- **Pre and post traffic hooks to validate deployment before the traffic shift starts and after it ends**.
- Easy and automated rollbacks using CloudWatch Alarms.

![SAM and CodeDeploy](/assets/aws-certified-developer-associate/sam_codedeploy.png "SAM and CodeDeploy")

### 22.4.1 SAM Template Options for CodeDeploy

The options discussed below can be added to the `Properties` of the `AWS::Serverless::Function` resource in the SAM template like `Handler`, `Runtime`, `CodeUri`, etc.

**AutoPublishAlias**:
1. Detects when new code is being deployed.
2. Creates and publishes an updated version of the function with the latest code.
3. Points the alias to the updated version.

```yaml
AutoPublishAlias: live
```

**DeploymentPreference**: `Canary`, `Linear`, `AllAtOnce`.
- **Alarms**: alarms that can trigger a rollback.
- **Hooks**: pre and post traffic shifting Lambda functions to validate the deployment.

```yaml
DeploymentPreference:
    Type: Canary10Percent10Minutes
    Alarms:
        - !Ref ErrorMetricGreaterThanZeroAlarm
    Hooks:
        PreTraffic: !Ref PreTrafficHook
        PostTraffic: !Ref PostTrafficHook
```

With these options, when you deploy the SAM application, it will automatically create a new version of the function for the alias, and apply the canary deployment strategy with the specified alarms and hooks. To see it in action, check the CodeDeploy console after deploying the SAM application:

![SAM Deployment with CodeDeploy](/assets/aws-certified-developer-associate/sam_codedeploy_deployment.png "SAM Deployment with CodeDeploy")
