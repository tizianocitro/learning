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
