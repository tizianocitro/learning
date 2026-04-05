# 23 Cloud Development Kit (CDK)

CDK allows you to define Cloud infrastructure using programming languages: JavaScript/TypeScript, Python, Java, and .NET.

CDK is organized into stacks, which are collections of high-level components called constructs. For example:

```typescript
class ECSStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const vpc = new ec2.Vpc(this, 'MyVpc', {
      maxAzs: 2 // Default is all AZs in the region
    });

    const ecsCluster = new ecs.Cluster(this, 'EcsCluster', {
      vpc: vpc
    });

    // Create a public load-balanced Fargate service
    new ecsPatterns.ApplicationLoadBalancedFargateService(this, 'MyFargateService', {
      cluster: ecsCluster,
      memoryLimitMiB: 512,
      cpu: 256,
      taskImageOptions: {
        image: ecs.ContainerImage.fromRegistry('amazon/amazon-ecs-sample')
      }
      publicLoadBalancer: true
    });
  }
}
```

The code is compiled into a CloudFormation template (JSON/YAML).

![CDK Diagram](/assets/aws-certified-developer-associate/cdk_diagram.png "CDK Diagram")

You can therefore deploy infrastructure and application code together.
- It is great for Lambda functions and Docker containers in ECS/EKS.

## 23.1 CDK vs SAM

**SAM**:
- Serverless focused.
- Allows you to write templates declaratively in JSON or YAML.
- Great for quickly getting started with Lambda.
- Leverages CloudFormation.

**CDK**:
- All AWS services.
- Allows you to write Cloud infrastructure in a programming language.
- Leverages CloudFormation.

### 23.1.1 CDK + SAM

- You can use SAM CLI to locally test your CDK applications.
- You must first run `cdk synth` to generate the CloudFormation template.
- Then you can use `sam local invoke` to test Lambda functions defined in your CDK application.

![CDK + SAM](/assets/aws-certified-developer-associate/cdk_sam.png "CDK + SAM")

## 23.2 Creating a CDK Application

We will create a CDK application that allows users to upload images to an S3 bucket. On upload, the image will be processed by a Lambda function that uses Rekognition to analyze it and stores the results in DynamoDB.

![CDK Application](/assets/aws-certified-developer-associate/cdk_application.png "CDK Application")

First, **install the CDK CLI**:

```bash
sudo npm install -g aws-cdk-lib
```

Next, **create a new CDK application**:

```bash
mkdir cdk-app
cd cdk-app
cdk init app --language=javascript
```

And **verify that the CDK application was created** successfully:

```bash
cdk ls
```

This should have create a file `lib/cdk-app-stack.js` with a basic stack definition. Edit this file to **define your CDK resources**:

```javascript
const cdk = require('aws-cdk-lib');

const s3 = require("aws-cdk-lib/aws-s3");
const iam = require("aws-cdk-lib/aws-iam");
const lambda = require("aws-cdk-lib/aws-lambda");
const lambdaEventSource = require("aws-cdk-lib/aws-lambda-event-sources");
const dynamodb = require("aws-cdk-lib/aws-dynamodb");

class CdkAppStack extends cdk.Stack {
    constructor(scope, id, props) {
        super(scope, id, props);

        // Create an S3 bucket for image uploads
        const bucket = new s3.Bucket(this, imageBucket, {
            removalPolicy: cdk.RemovalPolicy.DESTROY,
        });
        new cdk.CfnOutput(this, "Bucket", { value: bucket.bucketName });

        // Create role for Lambda function
        const role = new iam.Role(this, "cdk-rekn-lambdarole", {
            assumedBy: new iam.ServicePrincipal("lambda.amazonaws.com"),
        });

        // Attach policies to the role
        role.addToPolicy(
            new iam.PolicyStatement({
                effect: iam.Effect.ALLOW,
                actions: [
                    "rekognition:*",
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents",
                ],
                resources: ["*"],
            })
        );

        // Create DynamoDB table to store analysis results
        const table = new dynamodb.Table(this, "cdk-rekn-imagetable", {
            partitionKey: { name: "Image", type: dynamodb.AttributeType.STRING },
            removalPolicy: cdk.RemovalPolicy.DESTROY,
        });
        new cdk.CfnOutput(this, "Table", { value: table.tableName });

        // Create Lambda function to process image uploads
        const lambdaFn = new lambda.Function(this, "cdk-rekn-function", {
            code: lambda.AssetCode.fromAsset("lambda"),
            runtime: lambda.Runtime.PYTHON_3_9,
            handler: "index.handler",
            role: role,
            environment: {
                TABLE: table.tableName,
                BUCKET: bucket.bucketName,
            },
        });

        // Add event source to trigger Lambda on S3 object creation
        lambdaFn.addEventSource(
            new lambdaEventSource.S3EventSource(bucket, {
                events: [s3.EventType.OBJECT_CREATED],
            })
        );

        // Grant Lambda permissions to read from S3 and write to DynamoDB
        bucket.grantReadWrite(lambdaFn);
        table.grantFullAccess(lambdaFn);
    }
}
```

For the above to work, you will also need to create a Lambda function in the `lambda` directory with the following code in `index.py`:

```python
from __future__ import print_function
import boto3
import json
import os
from boto3.dynamodb.conditions import Key, Attr

minConfidence = 60

def handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']

    analyzeImage(bucket, key)

def analyzeImage(bucket, key):
    print("Detected the following image in S3")
    print("Bucket: " + bucket + " key name: " + key)

    client = boto3.client("rekognition")

    response = client.detect_labels(
        Image={"S3Object": {"Bucket": bucket, "Name": key}},
        MaxLabels=10, MinConfidence=minConfidence
    )

    dynamodb = boto3.resource("dynamodb")
    imageLabelsTable = os.environ["TABLE"]
    table = dynamodb.Table(imageLabelsTable)

    table.put_item(Item={"Image": key})

    objectsDetected = []
    for label in response["Labels"]:
        newItem = label["Name"]
        objectsDetected.append(newItem)
        objectNum = len(objectsDetected)
        itemAtt = f"object{objectNum}"
        response = table.update_item(
            Key={"Image": key},
            UpdateExpression=f"set {itemAtt} = :r",
            ExpressionAttributeValues={":r": f"{newItem}"},
            ReturnValues="UPDATED_NEW"
        )
```

Then, **bootstrap your CDK environment** to prepare it for deployment. This will create a `CDKToolkit` stack in your AWS account that contains resources required by CDK.

```bash
cdk bootstrap
```

And **generate the CloudFormation template**:

```bash
cdk synth
```

Finally, **deploy your CDK application**:

```bash
cdk deploy
```

You can check that the stack is being created by visiting the console and navigating to CloudFormation. You should see the `CkdAppStack` stack with the resources defined in your CDK application:

![CDK Stack in CloudFormation](/assets/aws-certified-developer-associate/cdk_stack.png "CDK Stack in CloudFormation")

If you want to **destroy the stack** later, you can run:

```bash
cdk destroy
```

## 23.3 Constructs

A **construct is a component that encapsulates everything CDK needs to create the final CloudFormation stack**.

A construct can represent a single AWS resource (e.g., S3 bucket) or multiple related resources (e.g., worker queue with compute).

Two components:
- **Construct Library**: a collection of constructs included in AWS CDK which contains constructs for every AWS resource.
    - It contains 3 different levels of constructs available: L1, L2, and L3.
- **Construct Hub**: a collection of additional constructs from AWS, third parties, and open-source CDK community.

### 23.3.1 L1 Constructs

Also called *CFN Resources* because they represent all resources directly available in CloudFormation.
- They are recognizable because construct names start with `Cfn`: for example, `CfnBucket`.
- They require you to explicitly configure all resource properties.

The following is an example of using the L1 construct `CfnBucket` to create a S3 bucket:

```javascript
const s3 = require ('aws-cdk-lib/aws-s3');

const bucket = new s3.CfnBucket(this, 'MyBucket', {
    bucketName: 'MyBucket'
});
```

These constructs are periodically generated from CloudFormation resource specification.

They are great if you want to migrate your CloudFormation templates to CDK in a one-by-one fashion.

### 23.3.2 L2 Constructs

L2 constructs represent AWS resources but with a higher level referred to as intent-based API.

They provide similar functionalities as L1 constructs but with convenient defaults and boilerplate (e.g., a method to get the bucket URL).
- You do not need to know all the details about the resource properties.
- Provide methods that make it simpler to work with the resource: for example, `bucket.addLifeCycleRule()`.

The following is an example of using the L2 construct `Bucket` to create a S3 bucket:

```javascript
const s3 = require ('aws-cdk-lib/aws-s3');

const bucket = new s3.Bucket(this, 'MyBucket', {
    versioned: true,
    encryption: s3.BucketEncryption.KMS
});

// Returns the HTTPS URL of an S3 Object
const objectUrl = bucket.urlForObject(
    'MyBucket/MyObject'
);
```

### 23.3.3 L3 Constructs

Also called *Patterns*, they represent multiple related resources to helps completing common tasks in AWS. For example:
- `aws-apigateway.LambdaRestApi`: it represents an API Gateway backed by a Lambda function.
- `aws-ecs-patterns.ApplicationLoadBalancerFargateService`: it represents an architecture that includes a Fargate cluster with an ALB.

The following is an example of `aws-apigateway.LambdaRestApi`:

```javascript
const api = new apigateway.LambdaRestApi(this, 'myapi', {
    handler: backend,
    proxy: false
});

const items = api. root.addResource('items');
items.addMethod ('GET'); // GET /items
items.addMethod ('POST'); // POST /items

const item = items.addResource ('{item}');
item.addMethod ('GET'); // GET /items/{item}
item.addMethod ('DELETE', new apigateway.HttpIntegration(
    'http://amazon.com'
));
```

## 23.4 Commands to Know

| Command | Description |
| ------- | ----------- |
| `npm install -g aws-cdk-lib` | Installs the CDK CLI and libraries. |
| `cdk init app` | Creates a new CDK project from a specified template. |
| `cdk synth` | Synthesizes and prints the CloudFormation template. |
| `cdk bootstrap` | Deploys the `CDKToolkit` stack to prepare the environment so that CDK can work properly. |
| `cdk deploy` | Deploys stacks. |
| `cdk diff`  | Shows the differences betweem the local CDK and the deployed stack. |
| `cdk destroy` | Destroys stacks. |
