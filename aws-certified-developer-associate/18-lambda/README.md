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