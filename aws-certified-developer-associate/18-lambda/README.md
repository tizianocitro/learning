# 18 Serverless: Lambda

In AWS, serverless was pioneered by Lambda but now includes anything that is managed, so also databases, messaging, storage:
- Lambda.
- DynamoDB.
- Cognito.
- API Gateway.
- S3.
- SNS and SQS.
- Amazon Data Firehose.
- Aurora Serverless.
- Step Functions.
- Fargate.

Below there is a **reference architecture for serverless applications**:

![Serverless Reference Architecture](/assets/aws-certified-developer-associate/serverless_reference_architecture.png "Serverless Reference Architecture")

The **exam tests heavily on serverless knowledge**, so it is important to understand it deeply.

## 18.1 Lambda

Lambda is a **compute service where you can upload your code and create a Lambda function (serverless function)**.
- Lambda takes care of provisioning and managing the servers that you use to run the code.
- You do not have to worry about operating systems, patching, scaling, etc.

### 18.1.1 EC2 vs Lambda

With **EC2**:
- Virtual servers in the cloud.
- Limited by RAM and CPU.
- Continuously running.
- Scaling means intervention to add/remove servers.

But with **Lambda**:
- Virtual functions: no servers to managem as AWS provides and manages them.
- Limited by time: short executions (up to 15 minutes).
- **Run on-demand**: when you do not use them, Lambda functions are not running.
- **Automatic scaling**: scales with the number of requests.

### 18.1.2 Benefits of Lambda

First benefit is **easy pricing** (more at [https://aws.amazon.com/lambda/pricing](https://aws.amazon.com/lambda/pricing/)):
- **Pay per request and compute time** (how long the code executes for).
- Free tier of 1,000,000 Lambda requests and 400,000 GBs of compute time.
- It is usually very cheap to run Lambda, so it is very popular.
- **Pay per calls**:
    - First 1,000,000 requests are free
    - 0.20 per 1 million requests thereafter, so 0.0000002 per request.
- **Pay per duration** in increments of 1 ms:
    - 400,000 GB-seconds of compute time per month for free. It means you get 400,000 seconds free if the function has 1 GB of RAM. While you get 3,200,000 seconds if the function has 128 MB of RAM.
    - After that, it costs 1.00 for 600,000 GB-seconds.

**Other benefits**:
- Integrated with the whole AWS suite of services.
- Integrated with many programming languages.
- Easy monitoring through CloudWatch.
- Easy to get more resources per functions with up to 10 GBs of RAM.
- Increasing RAM will also improve CPU and network.

### 18.1.3 Lambda Language Support

Many supported languages:
- Node.js.
- Python.
- Java.
- C#.
- Ruby.
- **Custom Runtime API**: community supported, for example, Rust or Golang.

You **can use container images on Lambda**. However, from an exam perspective, ECS and Fargate are preferred for running arbitrary Docker images.
- The container image must implement the *Lambda Runtime API*.

### 18.1.4 Integrations with Other AWS Services

Main services that Lambda functions integrate with:
- API Gateway: create RESTful APIs to trigger Lambda functions.
- Kinesis: use Lambda functions to do data transformations on the fly.
- DynamoDB: trigger Lambda functions when data is inserted/updated or somethign happens in the database.
- S3: create triggers to run Lambda functions when objects are created or other events happen.
- CloudFront: run Lambda functions at edge locations.
- EventBridge: create rules to react to changes by triggering Lambda functions.
- CloudWatch Logs: stream logs to Lambda to process them/send them to another service.
- SNS: trigger Lambda functions when a message is sent to an SNS topic.
- SQS: trigger Lambda functions when a message is sent to an SQS queue.
- Cognito: trigger Lambda functions when a user authenticates.

For example, consider the following **architecture to create a thumbnail when an image is uploaded to S3**:

![Lambda S3 Thumbnail Architecture](/assets/aws-certified-developer-associate/lambda_s3_thumbnail_architecture.png "Lambda S3 Thumbnail Architecture")

Another example is to have **serverless CRON jobs** using EventBridge and Lambda:

![Lambda CRON Jobs](/assets/aws-certified-developer-associate/lambda_cron_jobs.png "Lambda CRON Jobs")

## 18.2 Using Lambda

Go to the Lambda service in the console:

![Lambda Console](/assets/aws-certified-developer-associate/lambda_service.png "Lambda Console")

### 18.2.1 Testing and How It Works

From the console, you can already **test a function** in the *How it works* section (this section also offers indications on what Lambda is and how it works):

![Lambda Test](/assets/aws-certified-developer-associate/lambda_test.png "Lambda Test")

And also **simulate events to a function** (by clicking on clients on the left) and **see how the function scales** (number of boxes on the right):

![Lambda Events](/assets/aws-certified-developer-associate/lambda_events.png "Lambda Events")

You can also get information about the **traffic** and the **cost** of the function:

![Lambda Traffic and Cost](/assets/aws-certified-developer-associate/lambda_traffic_cost.png "Lambda Traffic and Cost")

### 18.2.2 Creating a Lambda Function using a Blueprint

To create a Lambda function, click on *Create a Function* in the console and **choose one of the following creation options**:

![Lambda Create Function](/assets/aws-certified-developer-associate/lambda_create_function.png "Lambda Create Function")

In this case, we will **use a blueprint to quickly spin up a function**. For example, the `Hello World` function in python. So, we will:
- Give it a **name**: `demo-lambda`.
- Choose to **create a new role with basic Lambda permissions**.

![Lambda Blueprint](/assets/aws-certified-developer-associate/lambda_blueprint.png "Lambda Blueprint")

Then, we can have a look at the **function code**:

![Lambda Function Code](/assets/aws-certified-developer-associate/lambda_function_code.png "Lambda Function Code")

Finally, create the function.

### 18.2.3 Accessing the Details of a Lambda Function

After creating the function, it appears in the console and by clicking on it, you can see the **function details**:

![Lambda Function Details](/assets/aws-certified-developer-associate/lambda_function_details.png "Lambda Function Details")

Like the **function code** in a code editor (so that you can also modify it):

![Lambda Function Code in Details](/assets/aws-certified-developer-associate/lambda_function_code_details.png "Lambda Function Code in Details")

If you change the code, you can **deploy the changes** by clicking on *Deploy*.

In the *Code* tab, you can check the **runtime settings**, which also **indicate the function that is executed and in which file it is**. In this case, it is the `lambda_handler` function in the `lambda_function.py` file:

![Lambda Function Runtime Settings](/assets/aws-certified-developer-associate/lambda_function_runtime_settings.png "Lambda Function Runtime Settings")


By clicking on *Test* in the *Code* tab, you can also **test the function**. It will prompt you to **create and give a name to a new test event** (the format depends on the funtion):

![Lambda Test Function in Details](/assets/aws-certified-developer-associate/lambda_test_function_details.png "Lambda Test Function in Details")

Then, you can see the **execution result** from the same tab with also information about the **duration, memory used, and build time**:

![Lambda Test Function Result in Details](/assets/aws-certified-developer-associate/lambda_test_function_result_details.png "Lambda Test Function Result in Details")

Another important tab is the **Configuration** tab, where you can see the **function configuration**:

![Lambda Function Configuration in Details](/assets/aws-certified-developer-associate/lambda_function_configuration_details.png "Lambda Function Configuration in Details")

From the *Configuration* tab, you can also **edit the function** and configure the **memory**, **timeout**, and **execution role**:

![Lambda Edit Function Configuration](/assets/aws-certified-developer-associate/lambda_edit_function_configuration.png "Lambda Edit Function Configuration")

However, you can also **configure triggers** and more.

Next tab is the *Monitoring* tab, where you can see the **function's metrics, logs, and traces**. For example, logs look like this and when you click on a stream, you can see it in CloudWatch:

![Lambda Monitoring in Details](/assets/aws-certified-developer-associate/lambda_monitoring_details.png "Lambda Monitoring in Details")

The **reason we can write logs to CloudWatch is that the function has an execution role that allows it to write logs to CloudWatch**.

## 18.3 Lambda Synchronous Invocation

You are making a **synchronous invocation** when you call the Lambda function directly. For example, via CLI, SDK, API Gateway, and Application Load Balancer.

With synchronous invocation:
- Results are returned right away.
- **Error handling must happen client-side**: retries, exponential backoff, etc.

![Lambda Synchronous Invocation](/assets/aws-certified-developer-associate/lambda_synchronous_invocation.png "Lambda Synchronous Invocation")

### 18.3.1 When Is a Lambda Function Invoked Synchronously

Every time it is **user invoked**:
- Elastic Load Balancing: Application Load Balancer.
- API Gateway.
- CloudFront: Lambda@Edge.
- S3 Batch.
- Testing in the console: for example, the *Test* button in functions' *Code* tab.

**Service invoked**:
- Cognito.
- Step Functions.

Other services:
- Amazon Lex
- Amazon Alexa
- Amazon Data Firehose

#### 18.3.2 Synchronous Invocation via CLI

Use the following command to **list all functions**:

```bash
aws lambda list-functions --region us-east-1
```

This will return a response with all the functions in the region:

![Lambda List Functions CLI](/assets/aws-certified-developer-associate/lambda_list_functions_cli.png "Lambda List Functions CLI")

To **synchronously invoke a function**, use the following command:

```bash
aws lambda invoke \
    --function-name demo-lambda \
    --cli-binary-format raw-in-base64-out \ 
    --region us-east-1 \
    --payload '{"key1": "value1", "key2": "value2", "key3": "value3"}' \
    response.json
```

The response of the command will be written to the `response.json` file, which you can read using `cat response.json`.

## 18.4 Lambda Integration with ALB

To **expose a Lambda function as an HTTP(S) endpoint**, you can **use an ALB or API Gateway**.
- The Lambda function must be registered in a target group.

**With the ALB**, clients will send requests to the ALB, which will then synchronously invoke the Lambda function within a target group. The Lambda function will then send the response back to the ALB, which will send it back to the client:

![Lambda ALB Integration](/assets/aws-certified-developer-associate/lambda_alb_integration.png "Lambda ALB Integration")

### 18.4.1 How the ALB Transforms HTTP Requests into Lambda Invocation

**When a client sends an HTTP request to the ALB, the ALB will transform the request into a JSON object that is sent to the Lambda function**.

The JSON object will contain the following information:

![Lambda ALB Request](/assets/aws-certified-developer-associate/lambda_alb_request.png "Lambda ALB Request")

The crucial thing is that **query string parameters, headers, and body are all converted**.

### 18.4.2 Lambda Function Response to the ALB

The **function must return a JSON to the ALB, so that the ALB can transform it back into an HTTP response to the client**.

The response must contain the following information:

![Lambda ALB Response](/assets/aws-certified-developer-associate/lambda_alb_response.png "Lambda ALB Response")

### 18.4.3 ALB Multi-Value Headers

**ALB can support multi-value headers**, which can be enabled in the ALB settings.
- It is called multi-value headers but applies to query string parameters as well.
- It is a **feature that you need to enable in the target group**.

When you enable multi-value headers, **HTTP headers and query string parameters that are sent with multiple values are shown as arrays within the Lambda event and response objects**.

For example, the following image shows how the `name` parameter in the query string is sent as an array in the Lambda event's `queryStringParameters` object:

![Lambda ALB Multi-Value Headers](/assets/aws-certified-developer-associate/lambda_alb_multi_value_headers.png "Lambda ALB Multi-Value Headers")

## 18.5 Creating a Lambda Function Integrating with ALB

### 18.5.1 Creating the Lambda Function

Go to the Lambda console and create a new function by choosing the option *Author from scratch* and giving it a nime of `lambda-alb`:

![Lambda ALB Create Function](/assets/aws-certified-developer-associate/lambda_alb_create_function.png "Lambda ALB Create Function")

And create the function.

### 18.5.2 Creating the ALB and Its Target Group

Create a `demo-lambda-alb` ALB that we will need to integrate with the function.

The crucial step is to create the **target group** that will be used to register the Lambda function. To do so, **select *Lambda Function* as the target type** and give the target group a name of `demo-tg-lambda`:

![Lambda ALB Create Target Group](/assets/aws-certified-developer-associate/lambda_alb_create_target_group.png "Lambda ALB Create Target Group")

Then, **register the `lambda-alb` function as the target of the target group**:

![Lambda ALB Register Target](/assets/aws-certified-developer-associate/lambda_alb_register_target.png "Lambda ALB Register Target")

And **assign this target group to the ALB**:

![Lambda ALB Assign Target Group](/assets/aws-certified-developer-associate/lambda_alb_assign_target_group.png "Lambda ALB Assign Target Group")

After that, you can **see the ALB in the function's details**:

![Lambda ALB in Function Details](/assets/aws-certified-developer-associate/lambda_alb_in_function_details.png "Lambda ALB in Function Details")

And also see the ALB in the *Configuration* tab of the function:

![Lambda ALB in Function Configuration](/assets/aws-certified-developer-associate/lambda_alb_in_function_configuration.png "Lambda ALB in Function Configuration")

### 18.5.3 Invoking the Function via the ALB

After the ALB is created, you can access its DNS name and check that the returned response is the one from the Lambda function. However, **to have the response shown in the browser, the function must to return a proper response in JSON**, which format is shown in section [18.4.2 Lambda Function Response to the ALB](#1842-lambda-function-response-to-the-alb). For example, to have the response shown as HTML it should be:

```json
{
    "statusCode": 200,
    "statusDescription": "200 OK",
    "headers": {
        "Content-Type": "text/html"
    },
    "body": "<h1>Hello from Lambda!</h1>",
    "isBase64Encoded": false,
}
```

All of this **works because the ALB has a resource-based policy that allows it to invoke the Lambda function**.

From the *Monitoring* tab of the function, you can also see the logs to check the events that the ALB sends to the function:

![Lambda ALB Monitoring](/assets/aws-certified-developer-associate/lambda_alb_monitoring.png "Lambda ALB Monitoring")

And you can see it has the fields indicated in section [18.4.1 How the ALB Transforms HTTP Requests into Lambda Invocation](#1841-how-the-alb-transforms-http-requests-into-lambda-invocation).

### 18.5.4 Configure Multi-Value Headers in the Target Group

To enable multi-value headers, go to the target group and **edit the target group attributes**:

![ALB Target Group Multi-Value Headers](/assets/aws-certified-developer-associate/alb_target_group_multi_value_headers.png "ALB Target Group Multi-Value Headers")

Then, **enable the multi-value headers**:

![ALB Target Group Enable Multi-Value Headers](/assets/aws-certified-developer-associate/alb_target_group_enable_multi_value_headers.png "ALB Target Group Enable Multi-Value Headers")

Keep in mind that enabling multi-value headers will require you to change the Lambda function to handle the multi-value headers both in the request and the response.

## 18.6 Lambda Asynchronous Invocation

This is **for services that invoke Lambda functions asynchronously via events**. For example, S3, SNS, and EventBridge.
- The events are placed in an **internal event queue**.

Lambda attempts to **retry when an error happens during processing**.
- 3 tries total with 1 minute wait after the first try, then 2 minutes wait.
- **Make sure the function processing is idempotent, so that it can be retried without side effects**.
- If the function is retried, you will see duplicate logs entries in CloudWatch Logs.
- You can define a **dead-letter queue via SNS or SQS for failed processing**, but this needs the correct IAM permissions to work.

Asynchronous invocations allow you to **speed up the processing of events through parallel processing if you do not need to wait for the response**.
- For example, if you have a large number of files to process in S3, you can trigger a Lambda function for each file and process them in parallel.

### 18.6.1 Asynchronous Invocation for S3 Event

S3 can trigger a Lambda function when an object event happens. In that case, a new event is generated and sent to an internal event queue from which the Lambda function is reading for events to process.

If something goes wrong, the funtion will retry a few times. If it fails, the event is sent to a dead-letter queue (via SQS or SNS)

![Lambda Asynchronous Invocation](/assets/aws-certified-developer-associate/lambda_asynchronous_invocation.png "Lambda Asynchronous Invocation")

### 18.6.2 Services that Invoke Lambda Asynchronously

- **S3**: react to objects being created, modified, or deleted.
- **SNS**: react to messages being sent to a topic.
- **EventBridge**: react to events from AWS services.
- CodeCommit: react to events such as new branch, new tag, and new push.
- CodePipeline: invoke a Lambda function during the pipeline, Lambda must callback.
- CloudWatch Logs: for log processing.
- Simple Email Service.
- CloudFormation.
- Config.
- IoT.
- IoT Events.

**For the exam, it is crucial to know how Lambda integrates with S3, SNS, and EventBridge**.

### 18.6.3 Asynchronous Invocation via CLI

Use the following command to **list all functions** (the output is like the one shown in section [18.3.2 Synchronous Invocation via CLI](#1832-synchronous-invocation-via-cli)):

```bash
aws lambda list-functions --region us-east-1
```

To **asynchronously invoke a function**, use the following command with the `--invocation-type Event` flag that makes the invocation asynchronous:

```bash
aws lambda invoke \
    --function-name demo-lambda \
    --cli-binary-format raw-in-base64-out \ 
    --region us-east-1 \
    --payload '{"key1": "value1", "key2": "value2", "key3": "value3"}' \
    --invocation-type Event \
    response.json
```

The response of the command will have a `StatusCode: 202`, which means that the invocation was successful. To see the response, you can use CloudWatch Logs, for example.

### 18.6.4 Setting Up a Dead-Letter Queue for Asynchronous Invocation

To do so, go into the function's *Configuration* tab and scroll down to the *Asynchronous invocation* section, where you can see the *Dead-letter Queue Service* option:

![Lambda Dead-Letter Queue](/assets/aws-certified-developer-associate/lambda_dead_letter_queue.png "Lambda Dead-Letter Queue")

Click *Edit* to change the asynchronous invocation settings:

![Lambda Dead-Letter Queue Edit](/assets/aws-certified-developer-associate/lambda_dead_letter_queue_edit.png "Lambda Dead-Letter Queue Edit")

Then select the **dead-letter queue** that you want to use (you need to create it first on SQS or SNS). In this case, we have a SQS queue called `lambda-dlq`:

![Lambda Dead-Letter Queue Selection](/assets/aws-certified-developer-associate/lambda_dead_letter_queue_selection.png "Lambda Dead-Letter Queue Selection")

However, before you can save these changes, you need to **add the correct permissions to the Lambda function to send messages to the dead-letter queue**. To do so, you can attach a policy to the function's execution role that allows it to send messages to the dead-letter queue. This role can be found in the *Permissions* section in the *Configuration* tab of the function:

![Lambda Dead-Letter Queue Permissions](/assets/aws-certified-developer-associate/lambda_dead_letter_queue_permissions.png "Lambda Dead-Letter Queue Permissions")

If you do not do this, you will get an error when trying to set the dead-letter queue.

In case a message fails a number of times greater than the **retry attempts** property specified in the function's asynchronous settings, it will be sent to the dead-letter queue.

## 18.7 Lambda Integration with EventBridge

1. Create an **EventBridge rule to trigger a function on a time-based schedule or when an event happens**.
![Lambda EventBridge Time-based Rule](/assets/aws-certified-developer-associate/lambda_eventbridge_time_based_rule.png "Lambda EventBridge Time-based Rule")

2. Create a **CodePipeline EventBridge rule to trigger a function when a pipeline state changes**.
![Lambda EventBridge CodePipeline Rule](/assets/aws-certified-developer-associate/lambda_eventbridge_codepipeline_rule.png "Lambda EventBridge CodePipeline Rule")

## 18.8 Creating a Lambda Function Integrating with EventBridge

Create a new function called `lambda-demo-eventbridge`.

Create a new EventBridge **rule that triggers the function every minute** and call it `invokeLambdaEveryMinute`:

![Lambda EventBridge Rule](/assets/aws-certified-developer-associate/lambda_eventbridge_rule.png "Lambda EventBridge Rule")

And define the **rule's schedule**:

![Lambda EventBridge Rule Schedule](/assets/aws-certified-developer-associate/lambda_eventbridge_rule_schedule.png "Lambda EventBridge Rule Schedule")

Next, select the **target for the rule**, which is the `lambda-demo-eventbridge` function:

![Lambda EventBridge Rule Target](/assets/aws-certified-developer-associate/lambda_eventbridge_rule_target.png "Lambda EventBridge Rule Target")

**EventBridge will automatically configure the necessary permissions for the targets to be triggered by the rule**. You can also configure some additional settings for the target, such as the retry policy and the dead-letter queue:

![Lambda EventBridge Rule Target Settings](/assets/aws-certified-developer-associate/lambda_eventbridge_rule_target_settings.png "Lambda EventBridge Rule Target Settings")

Finally, create the rule and go to the `lambda-demo-eventbridge` function to **see the new trigger**:

![Lambda EventBridge Trigger](/assets/aws-certified-developer-associate/lambda_eventbridge_trigger.png "Lambda EventBridge Trigger")

And see the trigger's details in the *Configuration* tab:

![Lambda EventBridge Trigger Details](/assets/aws-certified-developer-associate/lambda_eventbridge_trigger_details.png "Lambda EventBridge Trigger Details")

All the invocations can be see in the function's log stream in CloudWatch Logs. In the function's code, you can **log the input event to see what is being sent to the function**. For example, you can see the time the function was invoked.

![Lambda EventBridge Log Stream](/assets/aws-certified-developer-associate/lambda_eventbridge_log_stream.png "Lambda EventBridge Log Stream")

You can also see that the `lambda-demo-eventbridge` **function has a resource-based policy that allows EventBridge to invoke it** (`lambda:invokeFunction` permission).

## 18.9 Lambda Integration with S3 Event Notifications

**When an object is created, modified, or deleted in an S3 bucket, an event notification can be sent to a Lambda function.**
- A few examples of S3 events are S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, and S3:Replication.
- It is possible to filter the events based on the object name (e.g., only trigger the function when the object name contains `.jpg`).

![Lambda S3 Event Notifications](/assets/aws-certified-developer-associate/lambda_s3_event_notifications.png "Lambda S3 Event Notifications")

Notes:
- S3 event notifications **typically deliver events in seconds but can sometimes take a minute or longer**.
- If **two writes are made to a single non-versioned object at the same time**, it is **possible that only one event notification will be sent**.
- **Enable versioning on the bucket to ensure that an event notification is sent for every write**.

**Use case**: generate thumbnails of images uploaded to an S3 bucket.

### 18.9.1 Metadata Sync by Integrating Lambda with S3

When an object is uploaded to an S3 bucket, you can use a Lambda function to process the object, read its metadata, and sync them with a database. For example, you can sync the metadata of an image with a DynamoDB table or RDS table.

![Lambda S3 Metadata Sync](/assets/aws-certified-developer-associate/lambda_s3_metadata_sync.png "Lambda S3 Metadata Sync")

## 18.10 Creating a Lambda Function Integrating with S3 Event Notifications

Create a new function called `lambda-s3`. 

Create an S3 bucket called `demo-s3-event`. Then, go to the bucket's *Properties* tab and **configure an event notification to trigger the function when an object is created**. To do so, find the *Event Notifications* section and click on *Create Event Notification*. Give it a name of `invokeLambda`:

![Lambda S3 Event Notification](/assets/aws-certified-developer-associate/lambda_s3_event_notification.png "Lambda S3 Event Notification")

Then, for the **event types**, select the *All object create events* option:

![Lambda S3 Event Notification Event Types](/assets/aws-certified-developer-associate/lambda_s3_event_notification_event_types.png "Lambda S3 Event Notification Event Types")

Next is to configure the `lambda-s3` function as the **destination**:

![Lambda S3 Event Notification Destination](/assets/aws-certified-developer-associate/lambda_s3_event_notification_destination.png "Lambda S3 Event Notification Destination")

Save the event notification see it appear as a trigger in the function's details:

![Lambda S3 Event Notification Trigger](/assets/aws-certified-developer-associate/lambda_s3_event_notification_trigger.png "Lambda S3 Event Notification Trigger")

To **test that it is working**, upload an object to the bucket and see the function being triggered. Like for EventBridge, you can see the invocations in the function's log stream in CloudWatch Logs. In the function's code, you can **log the input event to see what is being sent to the function**. For example, you can see the bucket name, object key, and event name (e.g., `ObjectCreated:Put`).

![Lambda S3 Event Notification Log Stream](/assets/aws-certified-developer-associate/lambda_s3_event_notification_log_stream.png "Lambda S3 Event Notification Log Stream")

Also in this case, the `lambda-s3` **function has a resource-based policy that allows S3 to invoke it** (`lambda:invokeFunction` permission).

## 18.11 Lambda Event Source Mapping

The **exam may ask very precise and deep questions about Lambda event source mapping**, so it is important to know very well sections:
- [18.11 Lambda Event Source Mapping](#1811-lambda-event-source-mapping).
- [18.12 Streams (a.k.a Kinesis/DynamoDB) with Lambda](#1812-streams-aka-kinesisdynamodb-with-lambda).
- [18.13 Queues (a.k.a SQS/SNS) with Lambda](#1813-queues-aka-sqssns-with-lambda).

This is the last way Lambda can process events. It can be **used by**:
- Kinesis Data Streams.
- SQS and SQS FIFO queue.
- DynamoDB streams.

**Common denominator of this feature is that records need to be polled from the source**, meaning that the Lambda function asks one of the services to get the records.
- The **function polls the records and processes them**.
- The **function is invoked synchronously**.

### 18.11.1 Lambda Event Source Mapping with Kinesis Data Streams

**With Kinesis Data Streams**, an internal event source mapping is created and is responsible for polling the records from the stream and getting them back as batches. When the event source mapping has an event batch, it invokes the function synchronously and sends the event batch to it.

![Lambda Event Source Mapping](/assets/aws-certified-developer-associate/lambda_event_source_mapping.png "Lambda Event Source Mapping")

## 18.12 Streams (a.k.a Kinesis/DynamoDB) with Lambda

In case of streams, **an event source mapping creates an iterator for each shard and processes items in order at shard level**.
- **Processed items are not removed from the stream, meaning that other consumers can read them**.

Read operations can start:
- With new items.
- From the beginning.
- From timestamp.

Depeding on the stream's traffic:
- If you have a **low traffic stream**, you can use a batch window to accumulate records before processing. In this way, you can ensure to invoke the function efficiently.
- If you have a **high traffic stream**, you can process multiple batches in parallel at shard level using *record processors*.
    - You can have up to 10 batches per shard.
    - For each batch, in-order processing is still guaranteed at partition key level.
![Lambda Event Source Mapping High Traffic](/assets/aws-certified-developer-associate/lambda_event_source_mapping_high_traffic.png "Lambda Event Source Mapping High Traffic")

### 18.12.1 Error Handling with Streams

By default, **if your function returns an error, the entire batch is reprocessed until the function succeeds, or the items in the batch expire**.
- So, having an error in a batch can block processing.

**Processing for the affected shard is paused until the error is resolved to ensure in-order processing**.

To handle errors, you can configure the the event source mapping to:
- Discard old events: **discarded events can go to a specified destination**.
- Restrict the number of retries.
- **Split the batch on errors**: this to **work around Lambda timeout issues** when your function does not have enough time to process the whole batch, you can split the batch into smaller ones.

## 18.13 Queues (a.k.a SQS/SNS) with Lambda

In case of queues, **an event source mapping polls the queue and sends messages to the function synchronously**.
- The event source mapping polls the queue using long polling for efficiency.
- You can specify batch size from 1 to 10 messages per batch.

![Lambda Event Source Mapping Queues](/assets/aws-certified-developer-associate/lambda_event_source_mapping_queues.png "Lambda Event Source Mapping Queues")

The **configuration** to use is very simple, you just need to specify the queue or topic and the batch size:

![Lambda Event Source Mapping Queues Configuration](/assets/aws-certified-developer-associate/lambda_event_source_mapping_queues_configuration.png "Lambda Event Source Mapping Queues Configuration")

**Recommendations**:
- Set the queue visibility timeout to 6 times the timeout of the Lambda function.
- To use a DLQ to capture failed messages, set up it on the source queue and not on the Lambda function because the DLQ on functions is only for asynchronous invocations (and this is a synchronous invocation).
- Alternatively to a DQL, use a Lambda destination to capture failed messages.

### 18.13.1 More on Queues and Lambda

Order of processing and scaling:
- Lambda supports in-order processing for FIFO queues, scaling up to the number of active message groups.
- For standard queues, items are not necessarily processed in order.
- Lambda scales up to process a standard queue as quickly as possible.

Errors and delivery:
- When an error occurs in a queue, batches are returned to the queue as individual items and might be processed in a different grouping than the original batch.
- Occasionally, the event source mapping might receive the same item from the same queue twice, even if no function error occurred. So, make sure to idempotent processing in Lambda functions.
- Lambda deletes items from the queue after they are processed successfully.
- You can configure the source queue to send items to a dead-letter queue if
they cannot be processed.

### 18.13.2 Event Source Mapping Scaling

**Kinesis Data Streams** and **DynamoDB Streams**:
- One Lambda invocation per stream shard.
- If you use parallelization, you can have up to 10 batches processed per shard simultaneously.

**SQS Standard**:
- Lambda adds 60 more instances per minute to scale up.
- Up to 1000 batches of messages processed simultaneously.

**SQS FIFO**:
- Messages with the same `GroupID` will be processed in order.
- The Lambda function scales to the number of active message groups.

## 18.14 Using Event Source Mapping for Lambda and SQS Integration

Create a new function called `lambda-sqs` and a new standard SQS queue called `lambda-demo-sqs`.

Then, in the `lambda-sqs` function, click *Add Trigger* to add a new trigger. The first thing is to select the **trigger configuration**:

![Lambda Trigger Configuration](/assets/aws-certified-developer-associate/lambda_trigger_configuration.png "Lambda Trigger Configuration")

Select SQS, so you will need to enter the **SQS queue**, **batch size (how many messages to poll at once)**, and **maximum batch window (how long to gather records before invoking the function)**, and **enable the trigger**:

![Lambda SQS Trigger Configuration](/assets/aws-certified-developer-associate/lambda_sqs_trigger_configuration.png "Lambda SQS Trigger Configuration")

Keep in mind that to add the trigger successfully, you need to **add the necessary permissions to the function's execution role to allow it to poll the queue** (`ReceiveMessage` permission). Otherwise, you will get an error:

![Lambda SQS Trigger Permissions Error](/assets/aws-certified-developer-associate/lambda_sqs_trigger_permissions_error.png "Lambda SQS Trigger Permissions Error")

After adding the permissions and saving, the trigger gets added to the function and can be seen in the function's details:

![Lambda SQS Trigger Added](/assets/aws-certified-developer-associate/lambda_sqs_trigger_added.png "Lambda SQS Trigger Added")

To **test the integration**, you can send a message to the `lambda-sqs` queue. For example, you can send:

![Lambda SQS Send Message](/assets/aws-certified-developer-associate/lambda_sqs_send_message.png "Lambda SQS Send Message")

Since the function is long polling the `lambda-sqs` queue, it will process the message at a certain point. To check it, go into the function's log stream in CloudWatch Logs and see the message being processed. Notice the properties:
- `body: 'Hello world!'`: it contains the content of the message.
- `messageAttributes`: it contains the `foo` attribute sent in the message.
- `eventSource: 'aws:sqs'`: it indicates that the event source is SQS.

![Lambda SQS Log Stream](/assets/aws-certified-developer-associate/lambda_sqs_log_stream.png "Lambda SQS Log Stream")

After the message is processed, it is deleted from the queue, so you will see that the number of messages available is 0 (supposing you only sent one message).

To **disable the trigger**, go into the function's *Configuration* tab and click on the *SQS* trigger. Then, click on *Disable*:

![Lambda SQS Trigger Disable](/assets/aws-certified-developer-associate/lambda_sqs_trigger_disable.png "Lambda SQS Trigger Disable")

Disabling it can be useful to avoid costs due to the function continuosly polling the queue.

## 18.15 Using Event Source Mapping for Lambda and Kinesis Integration

If you want to integrate a Lambda function with a Kinesis Data Stream, you can do it by creating an event source mapping. This is the same process as with SQS, but you need to **select Kinesis as the trigger configuration**.

![Lambda Kinesis Trigger Configuration](/assets/aws-certified-developer-associate/lambda_kinesis_trigger_configuration.png "Lambda Kinesis Trigger Configuration")

You need to specify the:
- **Kinesis stream**: the stream that the function will poll.
- **Consumer**: if you have an enhanced fan-out consumer.
- **Batch size**: how many records to get in a batch (read at once).
- **Starting position**: you have three options to start reading records from the stream:
    - `TRIM_HORIZON`: oldest records.
    - `LATEST`: newest records.
    - `AT_TIMESTAMP`: records from a specific timestamp.

**Enable the trigger** and, optionally, configure some **additional settings**, among which you have the ones in the image below and the **concurrent batches per shard (how many batches to process in parallel)**:

![Lambda Kinesis Trigger Additional Settings](/assets/aws-certified-developer-associate/lambda_kinesis_trigger_additional_settings.png "Lambda Kinesis Trigger Additional Settings")

## 18.16 Event and Context Object in Lambda

When a Lambda function is invoked, it receives two arguments:
- **Event**: the data that is passed to the function when it is invoked.
- **Context**: information about the invocation, function, and execution environment.

The image below shows an example of both objects:

![Lambda Event and Context](/assets/aws-certified-developer-associate/lambda_event_context.png "Lambda Event and Context")

### 18.16.1 Event Object

It is a JSON-formatted document that **contains data for the function to process**.
- It contains information from the invoking service (e.g., EventBridge).
- For example, it contains input arguments, invoking service arguments, etc.

The Lambda runtime converts the event to an object (e.g., a `dict` type in Python).

### 18.16.2 Context Object

It **provides methods and properties that provide information about the invocation, function, and runtime environment**.
- For example, you can get `aws_request_id`, `function_name`, `memory_limit_in_mb`, etc.

It is passed to functions by Lambda at runtime.

### 18.16.3 Accessing the Event and Context Objects

To access the event and context objects in a Lambda function, you can use the following code in Python:

```python
def lambda_handler(event, context):
    # Access the event object
    print("Event source: ", event.source)
    print("Event region: ", event.region)

    # Access the context object
    print("Request ID: ", context.aws_request_id)
    print("Function name: ", context.function_name)
    print("Function ARN: ", context.invoked_function_arn)
    print("Function memory limit (MB): ", context.memory_limit_in_mb)
    print("CloudWatch log group name: ", context.log_group_name)
    print("CloudWatch log stream name: ", context.log_stream_name)
```

## 18.17 Lambda Destinations

**Destinations are for sending the result or failures of an asynchronous invocation or event source mapping somewhere**.
- They are an alternative to DLQs.
- Since their introduction, AWS recommends using destinations instead of DLQs because they allow you to send the results to multiple destinations instead of just SQS or SNS (as with DLQs).
- You can use both destinations and DLQs at the same time.

### 18.17.1 Destinations for Asynchronous Invocations

For **asynchronous invocations**, you can **define destinations for both successful and failed events**. Destinations can be:
- SQS.
- SNS.
- Lambda.
- EventBridge.

![Lambda Destinations for Asynchronous Invocations](/assets/aws-certified-developer-associate/lambda_destinations_async.png "Lambda Destinations for Asynchronous Invocations")

### 18.17.2 Creating a Destination for a Lambda Function

For **event source mapping**, you can **define destinations for discarded event batches**. Destinations can be:
- SQS.
- SNS.

![Lambda Destinations for Event Source Mapping](/assets/aws-certified-developer-associate/lambda_destinations_mapping.png "Lambda Destinations for Event Source Mapping")

## 18.18 Creating Destinations for a Lambda Function Asynchronous Invocation

To add a destination to a function, go to the function's details console and click on *Add Destination*. In this case, we will **create a destination for successful invocations and one for failed invocations** for a function integrating with S3 called `lambda-s3`:

![Lambda Add Destination](/assets/aws-certified-developer-associate/lambda_add_destination.png "Lambda Add Destination")

Next step is to create two queues: a success queue called `s3-success` and a failure queue called `s3-failure`.

### 18.18.1 Creating the Failure Destination

For the failure destination, set the **condition** to `On failure` and the **destination type** to `SQS`. Then, select the `s3-failure` queue:

![Lambda Failure Destination](/assets/aws-certified-developer-associate/lambda_failure_destination.png "Lambda Failure Destination")

If your function's execution role does not have the necessary permissions to send failures to the destination, **saving the destination will attempt to add the necessary permissions to the role** for you.

### 18.18.2 Creating the Success Destination

To create the success destination, set the **condition** to `On success` and the **destination type** to `SQS`. Then, select the `s3-success` queue:

![Lambda Success Destination](/assets/aws-certified-developer-associate/lambda_success_destination.png "Lambda Success Destination")

Once saved, **destinations appear in the function's details**. In this case you will see 2 SQS destinations: one for success and one for failure.

![Lambda Destinations Created](/assets/aws-certified-developer-associate/lambda_destinations_created.png "Lambda Destinations Created")

But you can also see them in the *Configuration* tab:

![Lambda Destinations Configuration](/assets/aws-certified-developer-associate/lambda_destinations_configuration.png "Lambda Destinations Configuration")

### 18.18.3 Testing the Destinations

To **test the success destination**, you can upload a file to S3 and see that in case of success, the message is sent to the `s3-success` queue (1 message available):

![Lambda Destinations Success Message](/assets/aws-certified-developer-associate/lambda_destinations_success_message.png "Lambda Destinations Success Message")

So, you can poll messages from the `s3-success` queue and see the message that was sent by the function and also that the event source is S3.
- The **message contains the response body of the function**.

![Lambda Destinations Success Message Received](/assets/aws-certified-developer-associate/lambda_destinations_success_message_received.png "Lambda Destinations Success Message Received")

To **test the failure destination**, you can modify the function to raise an exception and see that in case of failure, the message is sent to the `s3-failure` queue (1 message available).
- Keep in mind that **the function has to fail all the retry attempts before the message is sent to the failure destination**.
- The message is similar to the one sent to the success destination, but it contains also a property called `approximateInvokeCount` that indicates the number of times the message was retried.
- The response body contains all the error information you return in the function.

## 18.19 Lambda Execution Role for Lambda to Access Other Services

The execution role is the IAM role that the Lambda service assumes when it executes the function. It is the **role that grants the Lambda function permissions to access AWS services/resources**.
- The **best practice is to create one execution role per function**.

**Lambda uses the execution role to get the permissions to read from the stream/queue when you use an event source mapping** to invoke your function.
- This is because the function polls the stream/queue via the event source mappings, it is not the service that invokes the function.
- This is why when you create an event source mapping, you need to specify the execution role but do not need a resource-based policy.

There are sample managed policies for Lambda:
- `AWSLambdaBasicExecutionRole`: upload logs to CloudWatch, this is why it is called *basic*.
- `AWSLambdaKinesisExecutionRole`: read from Kinesis.
- `AWSLambdaDynamoDBExecutionRole`: read from DynamoDB Streams.
- `AWSLambdaSQSQueueExecutionRole`: read from SQS.
- `AWSLambdaVPCAccessExecutionRole`: deploy Lambda function in VPC.
- `AWSXRayDaemonWriteAccess`: upload trace data to X-Ray.

You can see a function's execution role in the *Permissions* section of the function's *Configuration* tab:

![Lambda Execution Role](/assets/aws-certified-developer-associate/lambda_execution_role.png "Lambda Execution Role")

## 18.20 Lambda Resource-based Policies for Other Services to Invoke Lambda

When you use other services to invoke a Lambda function, you need to **use a resource-based policy to grant permissions to the services to invoke the function**.
- It is similar to an S3 bucket policy for S3 buckets.
- When an AWS service like S3 calls your Lambda function, the resource-based policy needs to give it permission to invoke the function.

An **IAM principal can access you Lambda function if one of these two conditions is met**:
- The IAM policy attached to the principal authorizes it to access the function (e.g. user access).
- The resource-based policy authorizes access to the function (e.g. service access).

You can see a function's resource-based policy in the *Permissions* section of the function's *Configuration* tab:

![Lambda Resource-based Policy](/assets/aws-certified-developer-associate/lambda_resource_based_policy.png "Lambda Resource-based Policy")

And you can see it in details by clicking on the *View Policy*:

![Lambda Resource-based Policy Details](/assets/aws-certified-developer-associate/lambda_resource_based_policy_details.png "Lambda Resource-based Policy Details")

The policy above allows the `s3.amazonaws.com` service to invoke the function `lambda-s3` using the `lambda:InvokeFunction` permission.
