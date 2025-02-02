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
- CloudWatch Unified Agent.
- Elastic Beanstalk: sends collection of logs from application.
- ECS: sends collection of logs from containers.
- Lambda: sends collection of logs from functions.
- VPC Flow Logs: sends VPC specific logs.
- API Gateway: sends all the requests made to the API gateway.
- CloudTrail: sends logs based on filters.
- Route53: logs DNS queries.

### 17.4.2 CloudWatch Logs Insights to Query Logs

It is an interactive, pay-as-you-go, and integrated **log analytics capability for CloudWatch Logs**. The visualizations it generates can also be exported or added to dashboards (CloudWatch Dashboards).

![CloudWatch Logs Insights](/assets/aws-certified-developer-associate/cloudwatch_logs_insights.png "CloudWatch Logs Insights")

It allows you to **search and analyze log data** stored in CloudWatch Logs. For example, you can find a specific IP inside a log, count occurrences of *ERROR* in your logs, etc.

![CloudWatch Logs Insights Query](/assets/aws-certified-developer-associate/cloudwatch_logs_insights_query.png "CloudWatch Logs Insights Query")

CloudWatch Logs Insights **provides a purpose-built query language**, which:
- Automatically discovers fields from AWS services and JSON log events.
- Fetches desired event fields, filters based on conditions, calculates aggregate statistics, sorts events, limits the number of events, and more.
- Can save queries and add them to CloudWatch Dashboards.
- Can query multiple log groups in different AWS accounts.

CloudWatch Logs Insights is a **query engine, not a real-time engine, so it will only query historical data when you run a query**.

### 17.4.3 S3 Exports for CloudWatch Logs

This feature allows you to **batch export log data from CloudWatch Logs to S3**.
- Log data can take up to 12 hours to become available for export.
- The API call to intiate this process is `CreateExportTask`.
- It is batched, so it is not near-real time nor real-time.
- For real-time, use *Logs Subscriptions* instead.

### 17.4.4 CloudWatch Logs Subscriptions

This feature allows you to **get real-time log events from CloudWatch Logs for processing and analysis**. You can:
- Send logs to Kinesis Data Streams, Amazon Data Firehose, or Lambda.
- Use **Subscription Filters** to filter which logs are delivered to your destination.

![CloudWatch Logs Subscriptions](/assets/aws-certified-developer-associate/cloudwatch_logs_subscriptions.png "CloudWatch Logs Subscriptions")

### 17.4.5 CloudWatch Logs Aggregation Multi-Account and Multi-Region Using Subscription Filters

You can **use subscription filters (with Kinesis Data Streams or Amazon Data Firehose) to aggregate logs from multiple accounts and regions into a single account**. For example:

![CloudWatch Logs Aggregation](/assets/aws-certified-developer-associate/cloudwatch_logs_aggregation.png "CloudWatch Logs Aggregation")

The way it works behind the scenes is that you have **a subscription filter in the source account that sends log in a subscription destination in the destination account**.

![CloudWatch Logs Aggregation Inner Workings](/assets/aws-certified-developer-associate/cloudwatch_logs_aggregation_inner_workings.png "CloudWatch Logs Aggregation Inner Workings")

For it to work, you need to:
- **Attach a destination access policy to the subscription destination**.
- Create an IAM role in the destination account that allows it to put records in Kinesis Data Streams or Firehose.

![CloudWatch Logs Aggregation Policy and Role](/assets/aws-certified-developer-associate/cloudwatch_logs_aggregation_policy_role.png "CloudWatch Logs Aggregation Policy and Role")

## 17.5 Using CloudWatch Logs

In the CloudWatch console, you can see the **log groups** that were defined. From the group name, you can see the service/application that is being logged. By clicking on a group, you can see the **log streams** within the group.

![CloudWatch Logs Log Groups](/assets/aws-certified-developer-associate/cloudwatch_log_groups.png "CloudWatch Logs Log Groups")

By clicking on a log stream, you can see the **log events** that were logged. You can also search for specific log events.

![CloudWatch Logs Log Streams](/assets/aws-certified-developer-associate/cloudwatch_log_events.png "CloudWatch Logs Log Streams")

From within the log group details, you can edit the **retention settings**:

![CloudWatch Logs Retention Settings](/assets/aws-certified-developer-associate/cloudwatch_retention_settings.png "CloudWatch Logs Retention Settings")

### 17.5.1 Creating a Log Group and Stream

To create a log group, click on *Create LogGgroup* and define the **log group name** and **retention settings**:

![CloudWatch Logs Log Group Creation](/assets/aws-certified-developer-associate/cloudwatch_log_group_creation.png "CloudWatch Logs Log Group Creation")

From the log group, you can create a log stream by clicking on *Create Log Stream* in the *Log Streams* tab:

![CloudWatch Logs Log Stream Creation](/assets/aws-certified-developer-associate/cloudwatch_log_stream_creation.png "CloudWatch Logs Log Stream Creation")

### 17.5.2 Creating a Metric Filter

In a log group, you can define **metric filters** to extract metric data from logs and create alarms for these metrics. To do so, go into the *Metric Filters* tab and click on *Create Metric Filter*.

First, define the **filter pattern**:

![CloudWatch Logs Metric Filters](/assets/aws-certified-developer-associate/cloudwatch_metric_filters.png "CloudWatch Logs Metric Filters")

You can also test the pattern to see if it matches any log events:

![CloudWatch Logs Metric Filters Test](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_test.png "CloudWatch Logs Metric Filters Test")

Then, you define the filter's **name** and the **metric details**, such as the namespace, metric name, value, and unit:

![CloudWatch Logs Metric Filters Details](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_details.png "CloudWatch Logs Metric Filters Details")

After creating the metric filter, you can see the metric in CloudWatch Metrics.

To **create an alarm for a metric filter**, go into the *Metric Filters* tab and click on the metric filter. Then, click on *Create Alarm*.

![CloudWatch Logs Metric Filters Alarm](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_alarm.png "CloudWatch Logs Metric Filters Alarm")

### 17.5.3 Creating a Subscription Filter

To create a **subscription filter**, go into the *Subscription Filters* tab and click on *Create* to see the options:

![CloudWatch Logs Subscription Filters Creation](/assets/aws-certified-developer-associate/cloudwatch_subscription_filters_creation.png "CloudWatch Logs Subscription Filters Creation")

You **can create up to 20 subscription filters per log group**.

### 17.5.4 Exporting Data to S3

To export data to S3, go into the *Actions* dropdown and click on *Export Data to Amazon S3*:

![CloudWatch Logs Export to S3](/assets/aws-certified-developer-associate/cloudwatch_export_to_s3.png "CloudWatch Logs Export to S3")

And configure the data export by choosing the **S3 bucket** and the **time range**:

![CloudWatch Logs Export to S3 Configuration](/assets/aws-certified-developer-associate/cloudwatch_export_to_s3_configuration.png "CloudWatch Logs Export to S3 Configuration")

### 17.5.5 Querying Data with CloudWatch Logs 

To query data with CloudWatch Logs, go into the *Logs Insights* tab and **run a query**:

![CloudWatch Logs Insights Query](/assets/aws-certified-developer-associate/cloudwatch_logs_insights_querying.png "CloudWatch Logs Insights Query")

Using the *Export Results* feature, you can export query results.

You can also **save queries** and look at sample queries:

![CloudWatch Logs Insights Save Query](/assets/aws-certified-developer-associate/cloudwatch_logs_insights_save_query.png "CloudWatch Logs Insights Save Query")

### 17.5.6 CloudWatch Logs Live Tail

Choose a log group and a stream in it. Then, on the stream details, click on *Start Tailing* to see the live tail console:

![CloudWatch Logs Live Tail](/assets/aws-certified-developer-associate/cloudwatch_live_tail.png "CloudWatch Logs Live Tail")

From this console, click on *Apply Filter* to filter the logs and the **live tail console will start waiting for events that match the filter**.

To test it, go into the log stream and use the *Actions* dropdown to click *Create Log Event* to **log an event**:

![CloudWatch Logs Live Tail Test](/assets/aws-certified-developer-associate/cloudwatch_live_tail_test.png "CloudWatch Logs Live Tail Test")

Then, go back to the live tail console and you will see the event that was logged:

![CloudWatch Logs Live Tail Event](/assets/aws-certified-developer-associate/cloudwatch_live_tail_event.png "CloudWatch Logs Live Tail Event")
