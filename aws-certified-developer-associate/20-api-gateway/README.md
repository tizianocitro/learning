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

## 20.5 Deployment Stages

Making changes in the API Gateway does not mean they are effective. In fact, you need to do a deployment for them to be in effect.

**Changes are deployed to stages**. Stages can be as many as you want, and with the naming you like (e.g., dev, test, prod).
- Each stage has its own configuration parameters.
- **Stages can be rolled back as a history of deployments is kept**.

For instance, you can use stages to manage API versions. You can create a new stage for each version of the API you want to deploy. This way, you can have multiple versions of the API running at the same time and switch between them easily:

![API Gateway Stages](/assets/aws-certified-developer-associate/ag_stages.png "API Gateway Stages")

### 20.5.1 Stage Variables

Stage variables **are like environment variables for stages**. They can be used in:
- Lambda function ARN.
- HTTP endpoint.
- Parameter mapping templates.

Use cases:
- Configure HTTP endpoints your stages talk to.
- Pass configuration parameters to Lambda through mapping templates.

To **access stage variables within the API Gatewa**y, use the `${stageVariables.variableName}` syntax. For example, if you have a stage variable called `myVariable`, you can access it in the API Gateway using `${stageVariables.myVariable}`.

Regarding Lambda integration, **stage variables are passed to the `context` object in Lambda**. For example, if you have a stage variable called `myVariable`, you can access it in the Lambda function using `event.requestContext.stageVariables.myVariable`.

### 20.5.2 Stage Variables for Lambda Aliases

This is a very common pattern to use the API Gateway. The idea is to use stage variables to point to the right Lambda alias. This way, you can have different versions of the Lambda function for different stages (e.g., dev, test, prod) and the API Gateway will automatically invoke the right version based on the stage variable.

![API Gateway Stage Variables for Lambda Alias](/assets/aws-certified-developer-associate/ag_stage_variables_lambda_alias.png "API Gateway Stage Variables for Lambda Alias")

To **replicate the pattern** in the image above, you need to:
1. Create a Lambda function.
2. Publish two versions of the function: `v1`a nd `v2`.
3. Update the Lambda function to be the `LATEST` version.
4. Create a `dev` alias that points to the `LATEST` version.
5. Create a `test` alias that points to the `v2` version.
6. Create a `prod` alias that points to the `v1` version.
7. Create an API in the API Gateway.
8. Create a `/stage-variables` resource in the API.

After doing all the steps above, **create a `GET` method** for the `stage-variables` resource and **point the integration to the Lambda function using the function ARN with the stage variable** `${stageVariables.lambdaAlias}` included. For example, the ARN should look like this `arn:aws:lambda:us-east-1:123456789012:function:my-function:${stageVariables.lambdaAlias}`.

![API Gateway ARN with Stage Variable](/assets/aws-certified-developer-associate/ag_arn_stage_variable.png "API Gateway ARN with Stage Variable")

As you can see from the image above, you will get a warning to ensure that all functions that could be targeted by the ARN have the same permissions. This is because the API Gateway will use the ARN to invoke the Lambda function, and if the permissions are not set correctly, it will fail. The console will also provide you a command to set the permissions correctly:

![API Gateway ARN Permissions Command](/assets/aws-certified-developer-associate/ag_arn_permissions_command.png "API Gateway ARN Permissions Command")

Be aware that the command in the image above has a bug that causes the ARN to be repeated twice. The correct command should have it only once.

Before executing the command, you need to replace the `${stageVariables.lambdaAlias}` with the actual stage variable name you used in the ARN.
- If you are pointing the `dev` alias in `dev` stage, `arn:aws:lambda:us-east-1:123456789012:function:my-function:${stageVariables.lambdaAlias}` should become `arn:aws:lambda:us-east-1:123456789012:function:my-function:dev`.

Doing that, will add a resource-based policy statement to the corresponding Lambda function, allowing the API Gateway to invoke it:

![API Gateway Lambda Function Policy Statement](/assets/aws-certified-developer-associate/ag_lambda_policy_statement.png "API Gateway Lambda Function Policy Statement")

After running the command for all the aliases, create the method.

To **test** that everything is working, you can go to the API Gateway console and click on the `GET` method you created under the `/stage-variables` resource. Then, click on the *Test* tab and you should see that now there is an **input element for the stage variable** called `lambdaAlias` where you can enter the alias you want to point to:

![API Gateway Test Method with Stage Variable](/assets/aws-certified-developer-associate/ag_test_method_stage_variable.png "API Gateway Test Method with Stage Variable")

Because we are providing `prod` as alias, the Lambda function will be invoked with the `v1` version. If you provide `test`, the Lambda function will be invoked with the `v2` version. If you provide `dev`, the Lambda function will be invoked with the `LATEST` version.

Now, you can **deploy the API**. Click on the *Deploy API* button in the API console and create three stages: `dev`, `test`, and `prod`.

To **point each stage to the proper alias, create a stage variable called `lambdaAlias` and set it to the alias you want** to point to. Scroll the stage page and find the *Stage Variables* section:

![API Gateway Stage Variables](/assets/aws-certified-developer-associate/ag_stage_variables.png "API Gateway Stage Variables")

From this section, you can easily add the `lambdaAlias` stage variable and set it to the alias you want:

![API Gateway Stage Variables Add](/assets/aws-certified-developer-associate/ag_stage_variables_add.png "API Gateway Stage Variables Add")

Save the changes and you will see the stage variable in the list:

![API Gateway Stage Variables List](/assets/aws-certified-developer-associate/ag_stage_variables_list.png "API Gateway Stage Variables List")

In this way, the value of the `lambdaAlias` stage variable is fixed for the deployment stage. **When using the invoke URL of the stage, the API Gateway will always invoke the Lambda function with the alias you set in the stage variable**.
- Using the invoke URL `https://{restapi_id}.execute-api.{region}.amazonaws.com/dev/stage-variables`, the API Gateway will invoke the Lambda function with the `dev` alias, which means the `LATEST` version.
- Using the invoke URL `https://{restapi_id}.execute-api.{region}.amazonaws.com/test/stage-variables`, the API Gateway will invoke the Lambda function with the `test` alias, which means the `v2` version.
- Using the invoke URL `https://{restapi_id}.execute-api.{region}.amazonaws.com/prod/stage-variables`, the API Gateway will invoke the Lambda function with the `prod` alias, which means the `v1` version.

### 20.5.3 Stage Configuration

To edit the stage configuration, click on the stage you want to edit in the API Gateway console and click on *Edit*:

![API Gateway Stage Configuration](/assets/aws-certified-developer-associate/ag_stage_configuration.png "API Gateway Stage Configuration")

From the editing page, you can change the stage **name** and **description**:

![API Gateway Stage Configuration Name and Description](/assets/aws-certified-developer-associate/ag_stage_configuration_name_desc.png "API Gateway Stage Configuration Name and Description")

Or **additional settings** like caching, throttling, firewall, and certificate:

![API Gateway Stage Configuration Additional Settings](/assets/aws-certified-developer-associate/ag_stage_configuration_additional_settings.png "API Gateway Stage Configuration Additional Settings")

Back to the stage details page, you have the *Logging and Tracing* section where you can enable **logging** and **tracing** for the stage:

![API Gateway Stage Configuration Logging and Tracing](/assets/aws-certified-developer-associate/ag_stage_configuration_logging_tracing.png "API Gateway Stage Configuration Logging and Tracing")

You also have a *Deployment History* tab where you can **see the history of deployments** for the stage, but also other tabs for **documentation**, **stage variables**, **canary**, and **tags**:

![API Gateway Stage Configuration Deployment History](/assets/aws-certified-developer-associate/ag_stage_configuration_deployment_history.png "API Gateway Stage Configuration Deployment History")

## 20.6 Canary

API Gateway offers the possibility to enable canary deployments for any stage (usually prod) for when you want to test new features before rolling them out to all users.
- You choose the percentage of traffic the canary channel receives.

They way it works is that you **create a stage for the canary deployment** and then you can **configure canary on the stage**. Whenever you redeploy on the stage, a percentage of traffic will be sent to the updated API Gateway endpoint, while the rest will be sent to the previous version of the API Gateway endpoint.
- You will be able to **choose when the canary is not needed anymore by promoting the canary**.
- Metrics and logs are separate for better monitoring.
- Possibility to override stage variables for the canary stage.
- This is the equivalent of blue/green deployment with Lambda and API Gateway.

![API Gateway Canary Deployment](/assets/aws-certified-developer-associate/ag_canary_deployment.png "API Gateway Canary Deployment")

### 20.6.1 Using Canary

To use canary stages, you need to:
1. Create a Lambda function.
2. Publish two versions of the function: `v1` and `v2`.
3. Create a new API in the API Gateway.
4. Create a `/canary-demo` resource in the API.

Create a `GET` method for the `/canary-demo` resource and point the integration to the Lambda function using the function ARN with the version included.
- For example, the ARN should look like this `arn:aws:lambda:us-east-1:123456789012:function:my-function:1` to point to the `v1` version of the function.

![API Gateway Canary Stage Method](/assets/aws-certified-developer-associate/ag_canary_stage_method.png "API Gateway Canary Stage Method")

Then, deploy the API to a new stage called `canary`:

![API Gateway Deploy Stage Method](/assets/aws-certified-developer-associate/ag_deploy_stage_method.png "API Gateway Deploy Stage Method")

And go to the *Canary* tab to **configure canary** on the stage by clicking on *Create Canary*:

![API Gateway Enable Canary Deployment](/assets/aws-certified-developer-associate/ag_enable_canary_deployment.png "API Gateway Enable Canary Deployment")

And configure the canary by defining the **request distribution (percentage of traffic)**:

![API Gateway Canary Deployment Configuration](/assets/aws-certified-developer-associate/ag_canary_deployment_configuration.png "API Gateway Canary Deployment Configuration")

Save the changes and you will see the canary settings in the stage details page:

![API Gateway Canary Deployment Settings](/assets/aws-certified-developer-associate/ag_canary_deployment_settings.png "API Gateway Canary Deployment Settings")

Now, whenever you deploy the API to the `canary` stage, a percentage of traffic will be sent to the updated API Gateway endpoint, while the rest will be sent to the previous version of the API Gateway endpoint.

When you are confident that the new version is working correctly, you can **promote the canary** by clicking on *Promote Canary*:

![API Gateway Promote Canary](/assets/aws-certified-developer-associate/ag_promote_canary.png "API Gateway Promote Canary")

This will make 100% of the traffic go to the stage and not the canary anymore, as you can see in the two *Requests directed to X* values in the image below:

![API Gateway Promote Canary Confirmation](/assets/aws-certified-developer-associate/ag_promote_canary_confirmation.png "API Gateway Promote Canary Confirmation")
