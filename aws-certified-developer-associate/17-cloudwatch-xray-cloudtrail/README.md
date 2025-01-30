# 17 Monitoring and Audit: CloudWatch, X-Ray, and CloudTrail

Users care that the application is working:
- Application latency: avoid it increasing over time.
- Application outages: customer experience should not be degraded.
- Users contacting the IT department or complaining is not a good outcome.
- Troubleshooting and remediation.

Why monitoring?
- Prevent issues before they happen.
- Optimize performance and cost.
- Identify trends, e.g., for scaling patterns.
- Learning and improvement of the system and current architecture/processes.

## 17.1 Monitoring in AWS

**CloudWatch**:
- **Metrics**: collect and track key metrics.
- **Logs**: collect, monitor, analyze and store log files.
- **Events**: send notifications when certain events happen in your AWS.
- **Alarms**: react in real-time to metrics/events.

**X-Ray**:
- Troubleshooting application performance and errors.
- Distributed tracing of microservices: for example, how they interact with each other.

**CloudTrail**:
- Internal monitoring of API calls being made.
- Audit changes to AWS resources by your users.

## 17.2 CloudWatch Metrics

CloudWatch provides metrics for every services in AWS.

**Metrics**:
- Are **variables to monitor**: for example, `CPUUtilization`, and `NetworkIn`.
- Belong to **namespaces**.
- Have **dimension**: attributes of metrics (e.g., instance id, environment, etc.).
- Can have up to 30 dimensions per metric.
- Have **timestamps**.

You can create CloudWatch dashboards of metrics.

By going into the CloudWatch console, you can **see and filter metrics for each service**:

![CloudWatch Metrics](/assets/aws-certified-developer-associate/cloudwatch_metrics.png "CloudWatch Metrics")

For example, the `CPUCreditBalance` metric for an EC2 instance (you can also download the data as a CSV):

![CloudWatch Metrics CPUCreditBalance](/assets/aws-certified-developer-associate/cloudwatch_metrics_cpucreditbalance.png "CloudWatch Metrics CPUCreditBalance")

### 17.2.1 EC2 Detailed Monitoring

By default, EC2 instance metrics have metrics every 5 minutes, but with detailed monitoring enabled (for a cost), you get data every 1 minute.
- **Use detailed monitoring if you want to scale faster for your ASG**.

AWS free tier allows us to have 10 detailed monitoring metrics.

**Note**: EC2 memory usage is by default not pushed, thus it must be pushed from inside the instance as a custom metric.

## 17.3 CloudWatch Custom Metrics

**For all metrics that are not provided by services, you can define and send your own custom metrics to CloudWatch**. For example, memory (RAM) usage, disk space, number of logged in users, etc.

To do so, use the `PutMetricData` API. And you can **use dimensions (attributes) to categorize and filter metrics**:
- `Instance.id`.
- `Environment.name`.
- ...

You can specify a **metric resolution** using the `StorageResolution` API parameter with two possible values:
- **Standard**: 60 seconds.
- **High Resolution**: 1/5/10/30 second(s) but you will incur higher costs.

**Important for the exam**: you can push metric data points up until two weeks in the past or two hours in the future (make sure to configure your EC2 instance time correctly).

### 17.3.1 Pushing Custom Metrics

To push a custom metric to CloudWatch, you can use the AWS CLI:

```bash
aws cloudwatch put-metric-data \
    --namespace MyNameSpace \
    --metric-name Buffers \
    --value 123456 \
    --unit Count \
    --dimensions InstanceID=1-23456789,InstanceType=m1.small \
    --timestamp "$(date -u +"%Y-%m-%dT%H:%M:%SZ")"
    --region us-east-1
```

Then, when you go in the CloudWatch console, you can see the **custom namespace** that was defined with `--namespace MyNameSpace`:

![CloudWatch Custom Metrics Namespace](/assets/aws-certified-developer-associate/cloudwatch_custom_namespace.png "CloudWatch Custom Metrics Namespace")

If you click on it, you can see the **dimensions** that were defined with `--dimensions InstanceID=1-23456789,InstanceType=m1.small`:

![CloudWatch Custom Metrics Dimensions](/assets/aws-certified-developer-associate/cloudwatch_custom_dimensions.png "CloudWatch Custom Metrics Dimensions")

Then, when you click on the dimensions, you can see the **metric** that was defined with `--metric-name Buffers`:

![CloudWatch Custom Metrics Metric](/assets/aws-certified-developer-associate/cloudwatch_custom_metric.png "CloudWatch Custom Metrics Metric")

If you click on the metric, you can see that the **graph of the metric** updates.

## 17.4 CloudWatch Logs

CloudWatch Logs is the place where you can **store logs from your application**. To do so, you need to define:
1. **Log groups**: they usually represent an application, you can give them an arbitrary name.
2. **Log streams**: they organize log streams within a log group, usually one log stream per instance within the application/log file/container.
3. **Log expiration policies**: never expire, or anywhere from 1 day to 10 years.

CloudWatch Logs can send logs to:
- S3 (exports).
- Kinesis Data Streams.
- Amazon Data Firehose.
- Lambda.
- OpenSearch.

**Logs are encrypted by default** and you **can also setup KMS-based encryption with your own keys**.

### 17.4.1 Sources to Send Logs to CloudWatch

- SDK.
- CloudWatch Logs Agent.
- CloudWatch Unified Agent: this is now sort of deprecated.
- Elastic Beanstalk: sends collection of logs from application.
- ECS: sends collection of logs from containers.
- Lambda: sends collection of logs from functions.
- VPC Flow Logs: sends VPC specific logs.
- API Gateway: sends all the requests made to the API gateway.
- CloudTrail: sends logs based on filters.
- Route53: logs DNS queries.
