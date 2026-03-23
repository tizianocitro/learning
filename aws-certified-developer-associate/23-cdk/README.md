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

### 23.1.1 CDK + SAM:
- You can use SAM CLI to locally test your CDK applications.
- You must first run `cdk synth` to generate the CloudFormation template.
- Then you can use `sam local invoke` to test Lambda functions defined in your CDK application.

![CDK + SAM](/assets/aws-certified-developer-associate/cdk_sam.png "CDK + SAM")
