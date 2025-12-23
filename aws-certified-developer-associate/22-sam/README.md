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
