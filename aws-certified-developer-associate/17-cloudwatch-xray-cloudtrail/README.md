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

## 17.2.2 CloudWatch Custom Metrics

