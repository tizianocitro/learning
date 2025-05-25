# 20 Serverless: API Gateway

API Gateway is a **fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale**. It acts as a front door for applications to access data, business logic, or functionality from your backend services, such as applications running on Amazon EC2, AWS Lambda, or any web application.

![API Gateway to Expose Lambda Functions](/assets/aws-certified-developer-associate/ag_expose_lambda.png "API Gateway to Expose Lambda Functions")

## 20.1 Overview of Features and Integrations

A few **features** of API Gateway that are not include when you a simpler ALB:
- Lambda and API Gateway guarantee no infrastructure to manage.
- Support for the WebSocket protocol.
- Handle API versioning (e.g., v1, v2).
- Handle different environments (e.g., dev, test, prod).
- Handle security via authentication and authorization.
- Create API keys.
- Handle request throttling.
- Swagger/Open API import to quickly define APIs.
- Transform and validate requests and responses.
- Generate SDK and API specifications.
- Cache API responses.

Available **integrations** at high level:
- **Lambda**: easy way to expose REST APIs backed by Lambda functions.
- **HTTP**:
    - Expose HTTP endpoints in the backend.
    - For example, internal HTTP APIs on premise or an ALB.
    - The **reason why we add the gateway is to add rate limiting, caching, user authentication, API keys, etc**.
- **Other AWS service**:
    - Expose any AWS APIs through the API Gateway.
    - For example, start a Step Function workflow, post a message to SQS, and more.
    - The **reason why we add the gateway is to add authentication, deploy publicly, rate control, etc**.

### 20.1.1 API Gateway to Expose Kinesis Data Streams

An example of integration is to allow users to send HTTP requests to an API Gateway endpoint, which then sends them to Kinesis Data Streams. In this way, you do not need to provides Kinesis Data Streams endpoint to the users. Instead, you can provide a single API Gateway endpoint that is easier to manage and secure. The API Gateway can also handle authentication, authorization, and request validation for the Kinesis Data Streams integration.

![API Gateway to Expose Kinesis Data Streams](/assets/aws-certified-developer-associate/ag_expose_kinesis.png "API Gateway to Expose Kinesis Data Streams")

## 20.2 Different Ways of Deploying API Gateway Endpoints

- **Edge-Optimized (default)**:
    - For global clients.
    - Requests are routed through the CloudFront Edge locations to improve latency.
    - The API Gateway is still deployed in only one region.
- **Regional**:
    - For clients within the same region.
    - You can manually combine it with your own CloudFront distribution to get more control over the caching strategies and the distribution in general.
- **Private**:
    - Can only be accessed from your VPC using an interface VPC endpoint (ENI).
    - Use a resource policy to define access.

## 20.3 API Gateway Security

**User authentication** through:
- IAM roles: useful for internal applications.
- Cognito: it provides identity for external users (e.g., mobile users).
- Custom authorizer: your own logic provided by a Lambda function.

**Custom Domain Name HTTPS security through integration with AWS Certificate Manager (ACM)**:
- If using a Edge-Optimized endpoint, the certificate must be in us-east-1.
- If using a Regional endpoint, the certificate must be in the same region where the API Gateway is deployed.
- You must setup CNAME or A-alias record in Route 53 to point to the API Gateway endpoint.

## 20.4 Creating APIs

Go to the API Gateway console:

![API Gateway Console](/assets/aws-certified-developer-associate/ag_console.png "API Gateway Console")

And choose an **API type**. The options are:
- **HTTP API**: for REST APIs with low latency and cost. It supports only a subset of features compared to the REST API.
- **REST API**: for public REST APIs with all features.
- **REST API Private**: for REST APIs with all features, but only accessible from your VPC.
- **WebSocket API**: for WebSocket APIs.

![API Gateway Console Choose API Type](/assets/aws-certified-developer-associate/ag_console_choose_api.png "API Gateway Console Choose API Type")

Choose REST API and then click *Build*. First thing is to choose between creating a new API, importing or cloning one, and using a sample API. Select *New API* and give it a **name** and optionally **description**:

![API Gateway Create API](/assets/aws-certified-developer-associate/ag_create_api.png "API Gateway Create API")

In the screen above, you have to choose the **Endpoint Type** by selecting one of the option presented in [20.2 Different Ways of Deploying API Gateway Endpoints](#202-different-ways-of-deploying-api-gateway-endpoints).

Then, click on *Create API* to create the API. You will be redirected to the API Gateway console where you can see the API you just created:

![API Gateway API Created](/assets/aws-certified-developer-associate/ag_api_created.png "API Gateway API Created")

### 20.4.1 Creating Methods

To create a method, select the resource you want to add the method to. In this case, we will select the root resource `/`, which is the default resource created when you create a new API. Then, click on *Create Method* and select the **method type** (HTTP verbs, e.g., GET, POST, DELETE) and **integration type**:

![API Gateway Create Method](/assets/aws-certified-developer-associate/ag_create_method.png "API Gateway Create Method")

You **can integrate with any AWS service by selecting the AWS service integration type**, you just need to select the service you want to in the dropdown list:

![API Gateway Create Method Integration Type](/assets/aws-certified-developer-associate/ag_create_method_integration_type.png "API Gateway Create Method Integration Type")

In this case, we will **choose the Lambda Function integration type**, so we need to select the region where the Lambda function is deployed and then select the Lambda function we want to integrate with:

![API Gateway Create Method Lambda Function](/assets/aws-certified-developer-associate/ag_create_method_lambda_function.png "API Gateway Create Method Lambda Function")

You can **customize the timeout**, which is the time the API Gateway will wait for a response from the Lambda function before returning an error.
- The default timeout is 29 seconds.

Finally, click on *Create Method* to create the method.

As a result, the gateway will appear in the function as a trigger.
- The gateway also gains the permission to invoke the Lambda function.
- You can see this in the Lambda function's *Configuration* tab, under *Permissions*.

![API Gateway Lambda Function Trigger](/assets/aws-certified-developer-associate/ag_lambda_trigger.png "API Gateway Lambda Function Trigger")

### 20.4.2 Accessing and Testing Methods

In the API Gateway console, you can **see the created method** by clicking on it under the resource you selected:

![API Gateway Method Created](/assets/aws-certified-developer-associate/ag_method_created.png "API Gateway Method Created")

And **test the method** by going to the *Test* tab:

![API Gateway Test Method](/assets/aws-certified-developer-associate/ag_test_method.png "API Gateway Test Method")

Which will show you the response:

![API Gateway Test Method Response](/assets/aws-certified-developer-associate/ag_test_method_response.png "API Gateway Test Method Response")

You can debug and see what the gateway is sending to the Lambda function by accessing the CloudWatch logs of the Lambda function:

![API Gateway Function CloudWatch Logs](/assets/aws-certified-developer-associate/ag_function_cloudwatch_logs.png "API Gateway Function CloudWatch Logs")

### 20.4.3 Creating Resources

To create a resource, select the API you want to add the resource to and click on *Create Resource*:

![API Gateway Console Create Resource](/assets/aws-certified-developer-associate/ag_api_created.png "API Gateway Console Create Resource")

Configure the **resource path** and **resource name**.
- You can also choose to enable CORS for the resource, which will enable access to the resource from different domains.

![API Gateway Create Resource](/assets/aws-certified-developer-associate/ag_create_resource.png "API Gateway Create Resource")

Click on *Create Resource* to create the resource. You will be redirected to the API Gateway console where you can **see the resource you just created**:

![API Gateway Resource Created](/assets/aws-certified-developer-associate/ag_resource_created.png "API Gateway Resource Created")

And you can create methods for this new resource by clicking on *Create Method*, just like in [20.4.1 Creating Methods](#2041-creating-methods). This is how it will appear in the console:

![API Gateway Resource Created with Method](/assets/aws-certified-developer-associate/ag_resource_created_method.png "API Gateway Resource Created with Method")

### 20.4.4 Deploying APIs in Stages

To deploy the API, click on the *Deploy API* button in the API console:

![API Gateway Deploy API](/assets/aws-certified-developer-associate/ag_resource_created_method.png "API Gateway Deploy API")

Then select the **stage** you want to deploy the API to. You can create a new stage or select an existing one. A stage is a logical reference to a lifecycle state of your API (e.g., dev, test, prod). Then, give the stage a **name** and optionally a **description**:

![API Gateway Deploy API Stage](/assets/aws-certified-developer-associate/ag_deploy_api_stage.png "API Gateway Deploy API Stage")

And click on *Deploy* to deploy the API. You will be redirected to the API Gateway console where you can **see the created stage**:

![API Gateway Stage Created](/assets/aws-certified-developer-associate/ag_stage_created.png "API Gateway Stage Created")

Use the **invoke URL** to access the API. The URL will be in the format:

```bash
https://{restapi_id}.execute-api.{region}.amazonaws.com/{stage}
```

If you created a resource called `houses` and deployed it to the `dev` stage, the URL will be:

```bash
https://{restapi_id}.execute-api.{region}.amazonaws.com/dev/houses
```
