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
