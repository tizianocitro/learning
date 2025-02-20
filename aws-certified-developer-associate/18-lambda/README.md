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
