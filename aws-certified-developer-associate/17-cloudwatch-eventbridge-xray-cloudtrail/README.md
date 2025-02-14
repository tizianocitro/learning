# 17 Monitoring and Audit: CloudWatch, EventBridge, X-Ray, and CloudTrail

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

By default, EC2 instance metrics have metrics every 5 minutes, but **with detailed monitoring enabled (for a cost), you get data every 1 minute**.
- Use detailed monitoring if you want to scale faster for your ASG.

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
- **High-resolution**: 1/5/10/30 second(s) but you will incur higher costs. The **minimum resulution for high-resolution custom metrics is then 1 second**.

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

**By default, logs in CloudWatch Logs never expire**. Using log retention policies, you can define how long you want to keep the logs, thus customizing this setting.

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

## 17.6 CloudWatch Logs For EC2

By default, no logs from your EC2 machine will go to CloudWatch. You need to **run a CloudWatch logs agent on EC2 to push the log files you want to CloudWatch**.
- Make sure IAM permissions are correct.
- The CloudWatch logs agent can be setup on-premise resources too.

![CloudWatch Logs Agent](/assets/aws-certified-developer-associate/cloudwatch_logs_agent.png "CloudWatch Logs Agent")

### 17.6.1 CloudWatch Logs Agent vs CloudWatch Unified Agent

They are both for virtual servers (e.g., EC2 instances, on-premise servers, etc.).

**CloudWatch Logs Agent**:
- Old version of the agent.
- Can only send to CloudWatch Logs.

**CloudWatch Unified Agent** (unified because it **can do both logs and metrics**):
- Collects additional system-level metrics, such as RAM, processes, etc.
- Collects logs to send to CloudWatch Logs.
- Centralized configuration using SSM Parameter Store (makes it easier to manage).

### 17.6.2 CloudWatch Unified Agent

It can do both logs and metrics. It collects metrics directly on your Linux servers/EC2 instances.

It can **collect a lot more metrics and at a higher granularity level than the old logs agent**:
- CPU: active, guest, idle, system, user, steal.
- Disk metrics: free, used, total.
- Disk IO: writes, reads, bytes, iops.
- RAM: free, inactive, used, total, cached.
- Netstat: number of TCP and UDP connections, net packets, bytes.
- Processes: total, dead, bloqued, idle, running, sleep.
- Swap Space: free, used, used percentage.

**Out-of-the-box metrics for EC2 are for disk, CPU, and network but at a high level**. So, **if you are asked for more detailed and granular metrics, think of CloudWatch Unified Agent**.

## 17.7 CloudWatch Logs Metric Filters

CloudWatch Logs can use filter expressions to **search for data from log events and make metrics out of it**.
- For example, find a specific IP inside of a log or count occurrences of *ERROR* in your logs.
- You can specify up to 3 simensions for the metric filter, optionally.

**Metric filters can be used to trigger alarms**. For example, you can have a filter that counts the number of *ERROR* messages in your logs and triggers an alarm if the count is greater than 10, which then sends an SNS notification:

![CloudWatch Logs Metric Filters Alarm](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_alarm_sns.png "CloudWatch Logs Metric Filters Alarms")

Filters do not retroactively filter data. **Filters only publish the metric data points for events that are ingested after the filter was created**.

## 17.8 Using CloudWatch Logs Metric Filters

In a log stream, you can define **metric filters** to extract metric data from logs and create alarms for these metrics. To do so, go to the *Log Events* in the stream and click on *Create Metric Filter*.
- As presented in [17.5.2 Creating a Metric Filter](#1752-creating-a-metric-filter), filters can be created also from the *Metric Filters* tab in the log group.

![CloudWatch Logs Metric Filters Creation in Stream](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_creation_stream.png "CloudWatch Logs Metric Filters Creation in Stream")

As you can see, in the image above, we were filtering for `404` errors in the logs. So, we want to create a metric filter for this. First, we define the **filter pattern**:

![CloudWatch Logs Metric Filters Pattern](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_pattern.png "CloudWatch Logs Metric Filters Pattern")

And test the pattern with data from the log stream:

![CloudWatch Logs Metric Filters Test](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_test_with_logs.png "CloudWatch Logs Metric Filters Test")

Then, enter the filter's **name**:

![CloudWatch Logs Metric Filters Name](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_name.png "CloudWatch Logs Metric Filters Name")

And **metric details**:

![CloudWatch Logs Metric Filters Details](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_metric_details.png "CloudWatch Logs Metric Filters Details")

Finally, create the metric filter, and it will appear in the *Metric Filters* tab:

![CloudWatch Logs Metric Filters Tab](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_tab.png "CloudWatch Logs Metric Filters Tab")

### 17.8.1 Testing the Metric Filter

To make the metric filter work, we need ot generate some events in the log stream. After doing so, a custom namespace called `MetricFilters` (as defined in metric details) will appear in CloudWatch Metrics:

![CloudWatch Logs Metric Filters Namespace](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_namespace.png "CloudWatch Logs Metric Filters Namespace")

By clicking on it, you can see the metrics' dimensions and by clicking on the dimensions, you can see the metrics (this will track only values from this point on):

![CloudWatch Logs Metric Filters Metrics](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_metrics.png "CloudWatch Logs Metric Filters Metrics")

### 17.8.2 Creating an Alarm for the Metric Filter

To create an alarm for a metric filter, go into the *Metric Filters* tab and click on the metric filter. Then, click on *Create Alarm*:

![CloudWatch Logs Metric Filters Alarm](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_alarm_creation.png "CloudWatch Logs Metric Filters Alarm")

This will redirect you to the CloudWatch Alarms creation page, where you can **define the alarm details** in terms of **metric**:

![CloudWatch Logs Metric Filters Alarm Details](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_alarm_details.png "CloudWatch Logs Metric Filters Alarm Details")

And **conditions**. In this case, the condition is that when the metric is greater than 50, the alarm should trigger:

![CloudWatch Logs Metric Filters Alarm Conditions](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_alarm_conditions.png "CloudWatch Logs Metric Filters Alarm Conditions")

Then, you need to configure the **actions (in this case an SNS action)**, such as sending a notification to an SNS topic:

![CloudWatch Logs Metric Filters Alarm Actions](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_alarm_actions.png "CloudWatch Logs Metric Filters Alarm Actions")

Before creation, give the alarm a **name** and **description**:

![CloudWatch Logs Metric Filters Alarm Name Description](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_alarm_name_description.png "CloudWatch Logs Metric Filters Alarm Name Description")

After creating the alarm, you can see it in CloudWatch Alarms:

![CloudWatch Logs Metric Filters Alarm Created](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_alarm_created.png "CloudWatch Logs Metric Filters Alarm Created")

And it will also be **linked to the metric filter** and appear in the *Metric Filters* tab:

![CloudWatch Logs Metric Filters Alarm Linked](/assets/aws-certified-developer-associate/cloudwatch_metric_filters_alarm_linked.png "CloudWatch Logs Metric Filters Alarm Linked")

## 17.9 CloudWatch Alarms

Alarms are used to **trigger notifications for any metric** with various options, such as sampling, percentage, max, and min.

An alarm has the following **states**:
- `OK`: the alarm is not triggered, so the threshold is not breached.
- `ALARM`: the alarm is triggered, so the threshold is breached.
- `INSUFFICIENT_DATA`: the alarm does not have enough data to decide if it is in `OK` or `ALARM` state.

There is **period to specify how many data points to evaluate before triggering the alarm**:
    - Length of time in seconds to evaluate the metric.
    - Can apply to high resolution custom metrics: 10 seconds, 30 seconds or multiples of 60 seconds. So, **an alarm set on a high-resolution custom metric can be triggered as often as 10 seconds**.

### 17.9.1 Alarm Targets

1. **EC2**: stop, terminate, reboot, or recover an EC2 instance.
2. **Auto Scaling**: increase or decrease the desired capacity of an ASG.
3. **SNS**: send notification to an SNS topic, from which you can do pretty much anything (e.g., send an email, trigger a Lambda function, etc.).

### 17.9.2 Composite Alarms

Alarms are on a single metric but you can use **composite alarms to monitor the states of multiple other alarms** using `AND` and `OR` conditions.

They are helpful to reduce alarm noise by creating complex composite alarms. For example, instead of being alerted when either the CPU is high or the disk is full, you can create a composite alarm that triggers only when both conditions are met.

![CloudWatch Composite Alarms](/assets/aws-certified-developer-associate/cloudwatch_composite_alarms.png "CloudWatch Composite Alarms")

### 17.9.3 EC2 Instance Recovery

A **status check** alarm checks:
- **Instance status**: checks the EC2 VM.
- **System status**: checks the underlying hardware.
- **Attached EBS status**: checks attached EBS volumes.

When this alarm is breached, you can **recover the instance**, for example, by migrating it. When you perform a recovery, you get the same:
- Private, public, and elastic IP.
- Metadata.
- Placement group.

![CloudWatch EC2 Instance Recovery](/assets/aws-certified-developer-associate/cloudwatch_ec2_instance_recovery.png "CloudWatch EC2 Instance Recovery")

### 17.9.4 Good To Know

1. Alarms can be created based on CloudWatch Logs metrics filters.
2. To test alarms and notifications, set the alarm state to `Alarm` using CLI:

```bash
aws cloudwatch set-alarm-state \
    --alarm-name MyAlarm \
    --state-value ALARM \
    --state-reason "testing purposes"
```

## 17.10 Creating a CloudWatch Alarm

We will **create an alarm that terminates an EC2 instance when the CPU utilization goes above 95% for 15 consecutive minutes**.

Create an EC2 instance and go into the CloudWatch console. Click on *Alarms* and then *Create Alarm*.

**Select a metric** (in this case, `CPUUtilization`):

![CloudWatch Alarm Select Metric](/assets/aws-certified-developer-associate/cloudwatch_alarm_select_metric.png "CloudWatch Alarm Select Metric")

And you will get something like in the below image. We have set the **evaluation period** to 5 minutes, which means that a new data point is evaluated every 5 minutes:

![CloudWatch Alarm Select Metric Selected](/assets/aws-certified-developer-associate/cloudwatch_alarm_select_metric_selected.png "CloudWatch Alarm Select Metric Selected")

Then, set the **conditions** for the alarm:

![CloudWatch Alarm Conditions](/assets/aws-certified-developer-associate/cloudwatch_alarm_conditions.png "CloudWatch Alarm Conditions")

In the additional configuration, you can **set the number of data points within the evaluation period that must be breaching the threshold to trigger the alarm**. In this case, we are indicating 3 out of 3 data points, meaning that the CPU utilization must be above 95% for 15 consecutive minutes to cause the alarm to trigger:

![CloudWatch Alarm Additional Configuration](/assets/aws-certified-developer-associate/cloudwatch_alarm_additional_configuration.png "CloudWatch Alarm Additional Configuration")

Next, we **configure the action to take when the alarm is triggered**. In this case, we want to stop the EC2 instance, so we configure an **EC2 action to terminate the instance**:

![CloudWatch Alarm EC2 Action Configuration](/assets/aws-certified-developer-associate/cloudwatch_alarm_ec2_action.png "CloudWatch Alarm EC2 Action Configuration")

Finally, give a **name** (e.g., `TerminateEC2OnHighCPU`), and optionally description, to the alarm and create it. Right after creation, the alarm will be in the `INSUFFICIENT_DATA` state:

![CloudWatch Alarm Insufficient Data State](/assets/aws-certified-developer-associate/cloudwatch_alarm_insufficient_data_state.png "CloudWatch Alarm Insufficient Data State")

As soon as the alarm has enough data to evaluate the metric, it will go into the `OK` or `ALARM` state. However, to **test the alarm and trigger it** (set to `ALARM` state), you can use the CLI command presented in [17.9.4 Good To Know](#1794-good-to-know):

```bash
aws cloudwatch set-alarm-state \
    --alarm-name TerminateEC2OnHighCPU \
    --state-value ALARM \
    --state-reason "testing purposes"
```

The state will then change to `ALARM` and the **action will be taken** (in this case, the EC2 instance will be terminated):

![CloudWatch Alarm Alarm State](/assets/aws-certified-developer-associate/cloudwatch_alarm_alarm_state.png "CloudWatch Alarm Alarm State")

## 17.11 CloudWatch Synthetics Canary

It is a **scripted monitoring tool** that allows you to monitor your endpoints and APIs using configurable scripts.

You can have a configurable script that monitors your APIs, URLs, websites, and more. This **allows you to reproduce what your customers do programmatically to find issues** before they are impacting your customers.

**CloudWatch Synthetics Canary checks the availability and latency of your endpoints and can store load time data and screenshots of the UI**.
- It can run once or on a schedule.
- It has integration with CloudWatch Alarms.
- It offers programmatic access to a headless Google Chrome browser.
- Scripts can be written in Node.js or Python.

![CloudWatch Synthetics Canary](/assets/aws-certified-developer-associate/cloudwatch_synthetics_canary.png "CloudWatch Synthetics Canary")

### 17.11.1 CloudWatch Synthetics Canary Blueprints

Some blueprints are available to help you get started with CloudWatch Synthetics Canary:
- Heartbeat Monitor: load URL, store screenshot, and an HTTP archive file.
- API Canary: test basic read and write functions of REST APIs.
- Broken Link Checker: check all links inside the URL that you are testing.
- Visual Monitoring: compare a screenshot taken during a canary run with a baseline screenshot.
- Canary Recorder: used with CloudWatch Synthetics Recorder is a way to record your actions on a website and automatically generate a script for that.
- GUI Workflow Builder: verifies that actions can be taken on your webpage (e.g., test a webpage with a login form).

## 17.12 EventBridge (Previously CloudWatch Events)

With EventBridge, you can:
- **Schedule cron jobs** (scheduled scripts).
    ![EventBridge Schedule Cron Jobs](/assets/aws-certified-developer-associate/eventbridge_schedule_cron_jobs.png "EventBridge Schedule Cron Jobs")
- **Use event pattern**: use event rules to react to a service doing something. For example, send an SNS email when someone signs in as the root user.
    ![EventBridge Event Pattern](/assets/aws-certified-developer-associate/eventbridge_event_pattern.png "EventBridge Event Pattern")
- **Use triggers**: trigger Lambda functions, SQS, SNS, Kinesis Data Streams, etc.

The way it works is that you have a **source that generates events** and these **events are routed to destinations** using **rules**:

![EventBridge Working](/assets/aws-certified-developer-associate/eventbridge_working.png "EventBridge Working")

**EventBridge is the default event bus** where AWS services push events. However, there are also **partner event buses** that can be used to receive events from SaaS partners, thus reacting to events happening outside of AWS. The same is possible for **custom event buses**, where you can have custom applications pushing events.
- **Event buses can be accessed by other AWS accounts using resource-based policies**.

![EventBridge Buses](/assets/aws-certified-developer-associate/eventbridge_buses.png "EventBridge Buses")

Useful features of EventBridge:
- You can **archive events** (all/filtered) sent to an event bus indefinitely or for a period.
- You can **replay archived events**, for example, to test your application with historical data.

### 17.12.1 EventBridge Schema Registry

EventBridge can analyze the events in your bus and infer the schema. The **schema registry allows you to generate code for your application that will know in advance how data is structured in the event bus**.
- Schema can be versioned.

![EventBridge Schema Registry](/assets/aws-certified-developer-associate/eventbridge_schema_registry.png "EventBridge Schema Registry")

### 17.12.2 EventBridge Resource-based Policies

Resource-based policies allow you to **manage permissions for a event bus**.
- You can **allow/deny events from another AWS account or AWS region**.
- By default, only the account that owns the event bus can send events to it.

A **use case** is to aggregate all events from your AWS Organization in a single AWS account or region for auditing purposes.

![EventBridge Resource-based Policies Use Case](/assets/aws-certified-developer-associate/eventbridge_resource_based_policies.png "EventBridge Resource-based Policies Use Case")

What we can do is defining a **resource-based policy on a central event bus to allow another account to send events to it**:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "events:PutEvents",
            "Principal": {
                "AWS": "111122223333"
            },
            "Resource": "arn:aws:events:us-east-1:123456789012:event-bus/central-event-bus"
        }
    ]
}
```

### 17.12.3 Multi-Account Aggregation with EventBridge

The **target of an event rule in an account can be an event bus in another account**, which **allows you to aggregate events from multiple accounts in a single account**.
- In the single destination account, you can then create an event rule to trigger something based on the aggregated events.
- You need a resource-based policy on the target event bus to allow the source accounts to send events to it.

![EventBridge Multi-Account Aggregation](/assets/aws-certified-developer-associate/eventbridge_multi_account_aggregation.png "EventBridge Multi-Account Aggregation")

## 17.13 Using EventBridge

### 17.13.1 Accessing and Creating Event Buses

In the EventBridge console, you can go into the *Event buses* tab to see the **default event bus** and any **custom event buses** that were created:

![EventBridge Event Buses](/assets/aws-certified-developer-associate/eventbridge_event_buses.png "EventBridge Event Buses")

In the same page, you can also create a new custom event bus by clicking on *Create Event Bus*. You start by giving it a **name**:

![EventBridge Create Event Bus](/assets/aws-certified-developer-associate/eventbridge_create_event_bus.png "EventBridge Create Event Bus")

If you need cross-account access, you can define a **resource-based policy**:

![EventBridge Resource-based Policy](/assets/aws-certified-developer-associate/eventbridge_resource_based_policy.png "EventBridge Resource-based Policy")

Create the bus and it will appear in the list of custom event buses.

### 17.13.2 Using Partner Event Sources

In the EventBridge console, you can go into the *Partner Event Sources* tab to see the **partner event buses** that are available:

![EventBridge Partner Event Sources](/assets/aws-certified-developer-associate/eventbridge_partner_event_sources.png "EventBridge Partner Event Sources")

For example, you can use the one to catch all the events from Auth0:

![EventBridge Auth0 Partner Event Source](/assets/aws-certified-developer-associate/eventbridge_auth0_partner_event_source.png "EventBridge Auth0 Partner Event Source")

### 17.13.3 Creating an EventBridge Rule

To create an EventBridge rule, go into the *Rules* tab and click on *Create Rule*:

![EventBridge Create Rule](/assets/aws-certified-developer-associate/eventbridge_create_rule.png "EventBridge Create Rule")

Then, give the rule a **name** and **description**, select the **event bus**, and specify the **rule type**:

![EventBridge Rule Name Description](/assets/aws-certified-developer-associate/eventbridge_rule_name_description.png "EventBridge Rule Name Description")

**Based on the rule type, the second step is either to build the event pattern or to schedule the rule**. In this casem we need to build the event pattern.

First, specify the **event source**:

![EventBridge Rule Event Source](/assets/aws-certified-developer-associate/eventbridge_rule_event_source.png "EventBridge Rule Event Source")

Then, define the **event pattern**. In this case, we are looking for events from EC2 instances with a state change to `stopped` or `terminated`:

![EventBridge Rule Event Pattern](/assets/aws-certified-developer-associate/eventbridge_rule_event_pattern.png "EventBridge Rule Event Pattern")

As you can see in the image above, you **can test the pattern against some sample events to see if it matches** what you want.

In the next step, **select target(s)** for the rule. In this case, we are going to send the event to an SNS topic:

![EventBridge Rule Targets](/assets/aws-certified-developer-associate/eventbridge_rule_targets.png "EventBridge Rule Targets")

Lastly, you can add some **tags** and create the rule. After creation, the rule will appear in the list of rules.

### 17.13.4 Accessing EventBridge Archives and Replays

In the EventBridge console, you can go into the *Archives* tab to see the **event archives** that were created or create new ones:

![EventBridge Archives](/assets/aws-certified-developer-associate/eventbridge_archives.png "EventBridge Archives")

From the console, you can also go into the *Replays* tab to see the **event replays** that were created or create new ones.

### 17.13.5 Accessing the EventBridge Schema Registry

In the EventBridge console, you can go into the *Schema Registry* tab to see the **schemas** that were created or create new ones:

![EventBridge Access Schema Registry](/assets/aws-certified-developer-associate/eventbridge_access_schema_registry.png "EventBridge Access Schema Registry")

You can also search for specific schemas and see the schema details. For example, the schema for an EC2 instance state change event.

![EventBridge Schema Registry EC2 Schema](/assets/aws-certified-developer-associate/eventbridge_schema_registry_ec2_schema.png "EventBridge Schema Registry EC2 Schema")

And **download the bindings for the schema in different languages**:

![EventBridge Schema Registry Bindings](/assets/aws-certified-developer-associate/eventbridge_schema_registry_bindings.png "EventBridge Schema Registry Bindings")

## 17.14 X-Ray

X-Ray is a tracing service that **provides you a visual analysis of your applications**.

X-Ray service **collects data from all the different services** and **computes a service map from all the traces** (discussed in [17.14.2 X-Ray Leverages Tracing](#17142-x-ray-leverages-tracing)).

![X-Ray Visual Analysis](/assets/aws-certified-developer-associate/xray_visual_analysis.png "X-Ray Visual Analysis")

Benefits and usages:
- Troubleshoot performance (bottlenecks).
- Understand dependencies in a microservice architecture.
- Pinpoint service issues.
- Review request behaviors.
- Find errors and exceptions.
- Evaluate compliancy with the SLA?
- Identify users that are impacted by errors.

### 17.14.1 X-Ray Compatibility

X-Ray is compatible with:
- AWS Lambda.
- Elastic Beanstalk.
- ECS.
- ELB.
- API Gateway.
- EC2 instances or any application server, even on premise.

### 17.14.2 X-Ray Concepts

- **Segments**: each application/service sends them.
- **Subsegments**: if you need more details in your segment.
- **Trace**: segments collected together to form an end-to-end trace.
- **Sampling**: decrease the amount of requests sent to X-Ray to reduce cost.
- **Annotations**: key/value pairs used to index traces and use with filters.
- **Metadata**: key/value pairs but not indexed, so they cannot be used for searching.

### 17.14.3 X-Ray Leverages Tracing

Tracing is the **process of end-to-end following a request as it travels through your application**.

Each **component dealing with the request adds its own trace segment to the request**. So, tracing is made of segments, which can contain subsegments.

**Annotations can be added to traces to provide extra-information**.

**X-Ray can trace**:
- Every request.
- A sample of requests: for exampple, a percentage or a rate per minute.

**X-Ray security**:
- IAM for authorization.
- KMS for encryption at rest.

### 17.14.4 How to Enable X-Ray

Two important steps to enable X-Ray. **This is very important for the exam**.

1. **Your code must import the AWS X-Ray SDK**.
    - This requires little code modification.
    - The application SDK will then capture:
        - Calls to AWS services.
        - HTTP/HTTPS requests.
        - Database calls (e.g., MySQL, PostgreSQL, DynamoDB).
        - Queue calls (SQS).

2. **Install the X-Ray daemon or enable X-Ray AWS integration**.
    - X-Ray daemon works as a low level UDP packet interceptor.
    - AWS Lambda (and other AWS services) already run the X-Ray daemon for you.
    - **Each application must have the IAM rights to write data to X-Ray**.
    - If everything works on your local machine but not when on EC2, check that the deamon is installed/running.

![X-Ray Working](/assets/aws-certified-developer-associate/xray_working.png "X-Ray Working")

### 17.14.5 X-Ray Troubleshooting and Tips

If X-Ray is **not working on EC2**:
- Ensure the EC2 IAM role has the proper permissions.
- Ensure the EC2 instance is running the X-Ray daemon.

To **enable X-Ray on AWS Lambda**:
- Ensure the function has an IAM execution role with proper policy: `AWSX-RayWriteOnlyAccess`.
- Ensure that X-Ray is imported in the code.
- Enable the *Lambda X-Ray Active Tracing* option.

The **X-Ray daemon has a config to send traces cross account**:
- Make sure the IAM permissions are correct: the agent will assume the role.
- This allows you to **have a central account for all application tracing**.

## 17.15 Tracing Using X-Ray

X-Ray console is now part of the CloudWatch console. This is because it is helpful to have it alongside metrics, logs, and alarms.

You can access it by going into the *X-Ray Traces* section and there you will find the *Service Map* menu entry to **see the service map**.
- To see a service map, you need at least an application that uses X-Ray already running.

![X-Ray Service Map](/assets/aws-certified-developer-associate/xray_service_map.png "X-Ray Service Map")

By clicking on nodes, you can access its **details**, such as metrics and alerts:

![X-Ray Service Map Node Details](/assets/aws-certified-developer-associate/xray_service_map_node_details.png "X-Ray Service Map Node Details")

In case there are are **issues** with a node, you can see it highlighted and in its details. For example, the SNS node has 100% errors:

![X-Ray Service Map Node Issues](/assets/aws-certified-developer-associate/xray_service_map_node_issues.png "X-Ray Service Map Node Issues")

And, to gather more information, you can click on the *View Traces* button to get to a console in CloudWatch where you can **run queries to see the traces** (also filtering by node of interest):

![X-Ray Service Map Node Traces](/assets/aws-certified-developer-associate/xray_service_map_node_traces.png "X-Ray Service Map Node Traces")

After running a query, this **how traces will look like**:

![X-Ray Service Map Node Traces Results](/assets/aws-certified-developer-associate/xray_service_map_node_traces_results.png "X-Ray Service Map Node Traces Results")

If you click on any of the traces, you can see the **trace details in terms of segments and logs**. For example, you can see how a POST request generated many requests to DynamoDB:

![X-Ray Service Map Node Trace Details](/assets/aws-certified-developer-associate/xray_service_map_node_trace_details.png "X-Ray Service Map Node Trace Details")

If you click on a segment, you can see the **segment details**:

![X-Ray Service Map Node Segment Details](/assets/aws-certified-developer-associate/xray_service_map_node_segment_details.png "X-Ray Service Map Node Segment Details")

## 17.16 X-Ray Instrumentation

Instrumentation means measuring a product’s performance, diagnosing errors, and writing trace information.

**Use the X-Ray SDK to instrument your application code**. For example, to instrument an Express application:

```javascript
const AWSXRay = require('aws-xray-sdk');
const express = require('express');

const app = express();
app.use(AWSXRay.express.openSegment('MyApp'));

app.get('/', (req, res) => {
    res.render('index');
});

app.use(AWSXRay.express.closeSegment());
```

As you can see from the example above, **many SDKs require only configuration changes**. However, you **can modify your application code to customize and annotate the data that the SDK sends to X-Ray**. For this, you can use interceptors, filters, handlers, middleware, and more.

## 17.17 X-Ray Sampling Rules

With sampling rules, you **control the amount of data that you record** because the more you record, the more you pay.

You can modify sampling rules without changing your code. The deamon automatically downloads the new rules.

**By default, the X-Ray SDK records the first request each second, and five percent of any additional requests**.
- One request per second is the **reservoir**, which ensures that at least one trace is recorded each second as long the service is serving requests.
- Five percent is the **rate** at which additional requests beyond the reservoir size are sampled.

### 17.17.1 Custom Sampling Rules

You can create your own rules with specified reservoir and rate.
- **Reservoir**: the number of maximum requests per second to record.
- **Rate**: the percentage of additional requests to record after exceeding the reservoir size.

For example, you can create the following rule where 10 requests per second will be sent to X-Ray (reservoir) and then 10% of the other ones will be sent (rate):

![X-Ray Custom Sampling Rule 1](/assets/aws-certified-developer-associate/xray_custom_sampling_rule_1.png "X-Ray Custom Sampling Rule 1")

Or, you can create a rule where all requests are sent to X-Ray by setting both the reservoir and rate to 1 (this is very expensive):

![X-Ray Custom Sampling Rule 2](/assets/aws-certified-developer-associate/xray_custom_sampling_rule_2.png "X-Ray Custom Sampling Rule 2")

### 17.17.2 Defining Sampling Rules

Go to the CloudWatch console and click on *Settings* and then on *Traces* to see the *Sampling Rules* property:

![X-Ray Sampling Rules](/assets/aws-certified-developer-associate/xray_sampling_rules.png "X-Ray Sampling Rules")

Click on *View Settings* to **see the current sampling rules**. In this case, there is only the default rule, which specifies 1 request per second (reservoir) and 5% of the other requests (rate):

![X-Ray Sampling Rules View Settings](/assets/aws-certified-developer-associate/xray_sampling_rules_view_settings.png "X-Ray Sampling Rules View Settings")

You can edit the default rule, to **change the reservoir and rate or other settings**:

![X-Ray Sampling Rules Edit](/assets/aws-certified-developer-associate/xray_sampling_rules_edit.png "X-Ray Sampling Rules Edit")

But you can also **create a new rule** by clicking on *Create Sampling Rule*. Start by giving the rule a **name** and **priority**:

![X-Ray Sampling Rules Create Rule](/assets/aws-certified-developer-associate/xray_sampling_rules_create_rule.png "X-Ray Sampling Rules Create Rule")

Then, define the **reservoir** and **rate**:

![X-Ray Sampling Rules Define Reservoir Rate](/assets/aws-certified-developer-associate/xray_sampling_rules_edit.png "X-Ray Sampling Rules Define Reservoir Rate")

Finally, set the **matching criteria** for the rule, which is **useful if you want to apply the rule only to specific requests** (e.g., only POST or GET requests, a specific URL path like `/api/`, etc.) **or services** (e.g., `MYSERVICE` service).

For example, in this case, we are matching POST requests to the `MYSERVICE` service:

![X-Ray Sampling Rules Matching Criteria](/assets/aws-certified-developer-associate/xray_sampling_rules_matching_criteria.png "X-Ray Sampling Rules Matching Criteria")

Create the rule and the X-Ray deamon will automatically start using it.

## 17.18 X-Ray APIs

It is important to know them because **the exam can test you about the right API to use for some tasks**.

### 17.18.1 X-Ray Write APIs

| API | Description |
| --- | ----------- |
| `PutTraceSegments` | Uploads segment documents to X-Ray. |
| `PutTelemetryRecords` | Used by the X-Ray daemon to send telemetry records: `SegmentsReceivedCount`, `SegmentsRejectedCounts`, `BackendConnectionErrors`, etc. |
| `GetSamplingRules` | Retrieves all sampling rules. This is crucial to allow the deamon to download the latest rules, so that it can sample correctly without the need to restart the application. |
| `GetSamplingTargets` | Retrieves information about the number of requests that X-Ray instrumented services received and the number of requests that were sampled. |
| `GetSamplingStatisticSummaries` | Retrieves information about the number of requests recorded, sampled, or dropped. |

The X-Ray daemon needs to have an **IAM policy authorizing it to use these APIs to function correctly**. The following is the `AWSXRayWriteOnlyAccess` policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "xray:PutTraceSegments",
                "xray:PutTelemetryRecords",
                "xray:GetSamplingRules",
                "xray:GetSamplingTargets",
                "xray:GetSamplingStatisticSummaries"
            ],
            "Resource": "*"
        }
    ]
}
```

### 17.18.2 X-Ray Read APIs

| API | Description |
| --- | ----------- |
| `GetServiceGraph` | Retrieves the service map that we saw before. |
| `BatchGetTraces` | Retrieves a list of traces specified by ID. Each trace is a collection of segments that originates from a single request. |
| `GetTraceSummaries` | Retrieves IDs and annotations for traces available for a specified time frame using an optional filter. To get the full traces, use `BatchGetTraces`. |
| `GetTraceGraph` | Retrieves a service map for one or more specific trace IDs. |

The following policy allows the X-Ray deamon to use the above APIs and more:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "xray:GetSamplingRules",
                "xray:GetSamplingTargets",
                "xray:GetSamplingStatisticSummaries"
                "xray:GetServiceGraph",
                "xray:GetTraceGraph"
                "xray:BatchGetTraces",
                "xray:GetTraceSummaries",
                "xray:GetGroups",
                "xray:GetGroup",
                "xray:GetTimeSeriesServiceStatistics",
            ],
            "Resource": "*"
        }
    ]
}
```

## 17.19 Integrate X-Ray with Elastic Beanstalk

Elastic Beanstalk platforms include the X-Ray daemon, so no need to manually install/configure it.

You can **run the daemon by setting an option** in the Elastic Beanstalk console **or with a configuration file in** `.ebextensions/xray-daemon.config`:

```yaml
option_settings:
  aws:elasticbeanstalk:xray:
    XRayEnabled: true
```

For it to work properly, you need to make sure:
- To assign to your EC2 instances an instance profile with the correct IAM permissions, so that the X-Ray daemon can function correctly.
- That your application code is instrumented with the X-Ray SDK.

**Note**: the X-Ray daemon is not provided for multi-container Docker, so you need to install it manually.

### 17.19.1 Elastic Beanstalk X-Ray Configuration

When creating an application, find the *Software* configuration section and click on *Edit*:

![Elastic Beanstalk X-Ray Configuration](/assets/aws-certified-developer-associate/eb_xray_configuration.png "Elastic Beanstalk X-Ray Configuration")

And **enable the X-Ray daemon**:

![Elastic Beanstalk X-Ray Configuration Enable the Deamon](/assets/aws-certified-developer-associate/eb_xray_configuration_enable.png "Elastic Beanstalk X-Ray Configuration Enable the Deamon")

Then, find the *Security* configuration section and click on *Edit* to **add a proper instance profile for EC2 instances**:

![Elastic Beanstalk X-Ray Configuration Instance Profile](/assets/aws-certified-developer-associate/eb_xray_configuration_instance_profile.png "Elastic Beanstalk X-Ray Configuration Instance Profile")

If you look at the roles in the instance profile in the IAM console, you will see that the **instance profile has a role with the permissions to access the X-Ray APIs** discussed before.

## 17.20 Integrate X-Ray with ECS

You have three options to do so:
1. **X-Ray Container as a Daemon in ECS Cluster**: X-Ray daemon running on each ECS instance.
![X-Ray Container as a Daemon in ECS Cluster](/assets/aws-certified-developer-associate/xray_container_daemon_ecs_cluster.png "X-Ray Container as a Daemon in ECS Cluster")

2. **X-Ray Container as a Sidecar in ECS Cluster**: X-Ray daemon running as a sidecar container in each ECS task (side-to-side with application containers).
![X-Ray Container as a Sidecar in ECS Cluster](/assets/aws-certified-developer-associate/xray_container_sidecar_ecs_cluster.png "X-Ray Container as a Sidecar in ECS Cluster")

3. **X-Ray Container as a Sidecar in Fargate Cluster**: X-Ray daemon running as a sidecar container in each Fargate task (side-to-side with application containers).
![X-Ray Container as a Sidecar in Fargate Cluster](/assets/aws-certified-developer-associate/xray_container_sidecar_fargate_cluster.png "X-Ray Container as a Sidecar in Fargate Cluster")

### 17.20.1 Example of X-Ray Container as a Sidecar in ECS Cluster

Full example is at [https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon-ecs.html](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon-ecs.html).

**Important to know for the exam**, you need to:
- Expose the X-Ray daemon port (e.g., 2000) over UDP in the task definition using the `portMappings` field.
- Set the `AWS_XRAY_DAEMON_ADDRESS` environment variable in the application container to the X-Ray daemon address (e.g., `xray-daemon:2000`).
- Link the X-Ray daemon container to the application container using the `links` field.

```json
{
    "name": "xray-daemon",
    "image": "123456789012.dkr.ecr.us-east-2.amazonaws.com/xray-daemon",
    "cpu": 32,
    "memoryReservation": 256,
    // this is crucial to allow other containers
    // to communicate with the X-Ray daemon
    "portMappings": [
        {
            "hostPort": 0,
            "containerPort": 2000,
            "protocol": "udp"
        }
    ]
},
{
    "name": "scorekeep-api",
    "image": "123456789012.dkr.ecr.us-east-2.amazonaws.com/scorekeep-api",
    "cpu": 192,
    "memoryReservation": 512,
    "environment": [
        {
            "name": "AWS_REGION",
            "value": "us-east-2",
        },
        {
            "name": "NOTIFICATION_TOPIC",
            "value": "arn:aws:sns:us-east-2:123456789012:scorekeep-notifications"
        },
        // this indicates the X-Ray daemon address to the application
        // uses the port 2000 (as specified above) to communicate
        {
            "name": "AWS_XRAY_DAEMON_ADDRESS",
            "value": "xray-daemon:2000"
        }
    ],
    "portMappings": [
        {
            "hostPort": 5000,
            "containerPort": 5000
        }
    ],
    // this links the X-Ray daemon container to the application container
    "links": [
        "xray-daemon"
    ]
}
```

## 17.21 AWS Disto for OpenTelemetry

Secure and production-ready AWS-supported distribution of the open-source OpenTelemetry project.

**OpenTelemetry is a set of APIs, libraries, agents, and instrumentation to collect distributed traces and metrics for application monitoring**. It also allows you to:
- Collect metadata from your AWS resources and services.
- Send traces and metrics to multiple AWS services and partner solutions: X-Ray, CloudWatch, Prometheus, etc.
- Instrument your applications running on AWS (e.g., EC2, ECS, EKS, Fargate, Lambda, etc.) as well as on-premises and use the OpenTelemetry standard to send traces and metrics to AWS or other third-party solutions.

It is very similar to X-Ray but open-source. It offers **auto-instrumentation agents to collect traces without changing your code**.

You may want to **use AWS Distro for OpenTemeletry over X-Ray if you want to standardize with open-source APIs or send traces to multiple destinations simultaneously**, which is not possible with X-Ray.

![AWS Distro for OpenTelemetry](/assets/aws-certified-developer-associate/aws_disto_opentelemetry.png "AWS Distro for OpenTelemetry")

## 17.22 CloudTrail

CloudTrail is a service that **provides governance, compliance, and audit for your AWS Account**.

CloudTrail is enabled by default and allows you to **get an history of events/API calls made within your AWS account** via:
- Console.
- SDK.
- CLI.
- AWS services.

You can put logs from CloudTrail into CloudWatch Logs or S3.

A **trail can be applied to all regions (default) or a single region**, if you want to group all the events across all regions in a single place (e.g., S3 bucket).

**If a resource is deleted in AWS and you want to investigate how and why did it happen, check CloudTrail**.

All actions from SDK, CLI, console, and other AWS services are recorded in CloudTrail and sent to CloudWatch Logs or S3:

![CloudTrail Working](/assets/aws-certified-developer-associate/cloudtrail_working.png "CloudTrail Working")

### 17.22.1 CloudTrail Events

**Management events**: they **represent operations that are performed on resources in your AWS account**:
- By default, trails are configured to log management events.
- You can separate them between **read events** that do not modify resources from **write events** that may modify resources.
- A few examples of management events are:
    - Configuring security: IAM `AttachRolePolicy` API call.
    - Configuring rules for routing data: EC2 `CreateSubnet` API call.
    - Setting up logging: CloudTrail `CreateTrail` API call.

**Data events**:
- By default, data events are not logged because **they are high volume operations**.
- **S3 object-level activity**: `GetObject`, `DeleteObject`, `PutObject`. You **can separate read and write events**.
- Lambda function execution activity using the `Invoke` API call.

**CloudTrail Insights events**: more on this in [17.22.2 CloudTrail Insights](#17222-cloudtrail-insights).

### 17.22.2 CloudTrail Insights

Enable CloudTrail Insights to **detect unusual activity in your account**. For example:
- Inaccurate resource provisioning.
- Hitting service limits.
- Bursts of AWS IAM actions.
- Gaps in periodic maintenance activity.

CloudTrail Insights **analyzes normal management events to create a baseline of expected activity**. Then, it **continuously analyzes write events to detect unusual patterns**.
- **Anomalies generate CloudTrail Insights events** and appear in the CloudTrail console.
- Events are sent to Amazon S3.
- An **EventBridge event is generated in case you have automation needs**, such as sending an email.

![CloudTrail Insights](/assets/aws-certified-developer-associate/cloudtrail_insights.png "CloudTrail Insights")

### 17.22.3 CloudTrail Events Retention

Events are stored for 90 days in CloudTrail, by default. To keep events beyond this period, log them to S3 and use Athena to query them.
- Athena is a serverless service to query data in S3 using SQL.

![CloudTrail Events Retention](/assets/aws-certified-developer-associate/cloudtrail_events_retention.png "CloudTrail Events Retention")

### 17.22.4 Using CloudTrail

Go to the CloudTrail console and click on *Event History* to see the **history of events** in the last 90 days:

![CloudTrail Event History](/assets/aws-certified-developer-associate/cloudtrail_event_history.png "CloudTrail Event History")

For example, if you terminate an EC2 instance, you will see that a `TerminateInstances` event was recorded and appears in the event history:

![CloudTrail Event History Terminate Instances](/assets/aws-certified-developer-associate/cloudtrail_event_history_terminate_instances.png "CloudTrail Event History Terminate Instances")

If you click on the event, you can see the **details of the event** (e.g., who did it, when, from where, etc.):

![CloudTrail Event Details](/assets/aws-certified-developer-associate/cloudtrail_event_details.png "CloudTrail Event Details")

You can also see the resorces referenced in the event, such as the EC2 instance that was terminated:

![CloudTrail Event Resources](/assets/aws-certified-developer-associate/cloudtrail_event_resources.png "CloudTrail Event Resources")

And you can see the whole event in JSON format.

### 17.22.5 Integrate CloudTrail with EventBridge

This integration allows you to **intercep any API call**.

For example, you can create a rule to watch for the `DeleteTable` DynamoDB API call and sent an email via SNS every time someone uses it to delete a table in DynamoDB:

![CloudTrail EventBridge Integration](/assets/aws-certified-developer-associate/cloudtrail_eventbridge_integration.png "CloudTrail EventBridge Integration")

Another example is to be notified every time a user assumes an IAM role with the `AssumeRole` API call:

![CloudTrail EventBridge Integration Assume Role](/assets/aws-certified-developer-associate/cloudtrail_eventbridge_integration_assume_role.png "CloudTrail EventBridge Integration Assume Role")

Yet another example is to be notified every time a user changes the security group of an EC2 instance. For instance, by changing the inbound rules using the EC2 `AuthorizeSecurityGroupIngress` API call:

![CloudTrail EventBridge Integration Security Group](/assets/aws-certified-developer-associate/cloudtrail_eventbridge_integration_security_group.png "CloudTrail EventBridge Integration Security Group")

## 17.23 CloudWatch vs CloudTrail vs X-Ray

**CloudWatch** is for monitoring and allows you to monitor your applications, respond to system-wide performance changes, optimize resource utilization, and get a unified view of operational health.
- CloudWatch Metrics over time for monitoring.
- CloudWatch Logs for storing application logs.
- CloudWatch Alarms to send notifications in case of unexpected metrics.

**CloudTrail** is for auditing and allows you to log, continuously monitor, and retain account activity related to actions across your AWS infrastructure.
- Provides the event history of your AWS account activity.
- Audit API calls made by users, services, and AWS console.
- Useful to detect unauthorized calls or root cause of changes (e.g., changes to a resource that make it not compliant with organization rules).

**X-Ray** is a trace-oriented service for distributed applications.
- Automated trace analysis and central service map visualizatio, making it useful when you have distributed systems.
- Request tracking across distributed systems.
- Latency, errors and fault analysis.