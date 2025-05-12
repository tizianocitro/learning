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
