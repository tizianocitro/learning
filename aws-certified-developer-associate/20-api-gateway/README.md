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

## 20.7 Different Ways to Integrate API Gateway with Backend

There are different integration types you can use to integrate the API Gateway with the backend:
- **MOCK**: used to test the API Gateway without any backend integration. It will return a static response without invoking any backend service.
- **HTTP/AWS (e.g., Lambda and AWS services)**: used to integrate with any HTTP endpoint or AWS service. It will invoke the backend service and return the response to the client.
    - You must configure both the integration request and integration response.
    - You can setup **data mapping using mapping templates for the request and response**, so that you can change the data format before sending it to the backend and after receiving it from the backend.
    ![API Gateway HTTP/AWS Integration](/assets/aws-certified-developer-associate/ag_http_aws_integration.png "API Gateway HTTP/AWS Integration")
- **AWS_PROXY (Lambda Proxy)**: incoming request from the client is the input to the Lambda function because we cannot modify the request.
    - The function is responsible for the logic of request/response only.
    - It is **not possible to use mapping template because headers and query string parameters are passed as arguments**.
    - The API Gateway only role is to works as a proxy.
- **HTTP_PROXY**: same as AWS_PROXY, but for HTTP endpoints (e.g., ALB).
    - The HTTP request is passed to the backend without any modification.
    - The backend is responsible for the logic of request/response only.
    - The HTTP response from the backend is forwarded by API Gateway
    - It is **not possible to use mapping template**, but it is **possible to add HTTP headers that are not visible to the client** if needed (e.g., API key)
    ![API Gateway HTTP_PROXY Integration](/assets/aws-certified-developer-associate/ag_http_proxy_integration.png "API Gateway HTTP_PROXY Integration")

## 20.8 Mapping Templates

Mapping templates can be **used to modify requests and responses**:
- Rename/modify query string parameters.
- Add/modify headers.
- Modify the body content.
- Filter output results: for example, to remove unnecessary data.
- The `Content-Type` header can be set to `application/json` or `application/xml`.

Mapping templates are based on the Velocity Template Language (VTL), which provides operations like for loop, if, etc.

### 20.8.1 Integrating a Rest Client with a SOAP Service

SOAP APIs are XML based, whereas REST APIs are JSON based. So, you need a way to convert the JSON request to XML and the XML response to JSON. This is where API Gateway with mapping templates come in handy.

![API Gateway SOAP Integration](/assets/aws-certified-developer-associate/ag_soap_integration.png "API Gateway SOAP Integration")

The API Gateway should:
- Extract data from the request: either path, payload or header.
- Build a SOAP message based on request data using a mapping template.
- Call the SOAP service and receive its XML response.
- Transform the XML response to the desired format (in this case, JSON), and respond to the user with the transformed data.

### 20.8.2 Mapping Query String Parameters

You can map query string parameters to the backend using mapping templates.

In the example below, we are mapping the `name` query string parameter to the `my_variable` body variable in the backend. The `other` query string parameter is mapped to the `other_variable` body variable in the backend.
- You could also change their case or add a prefix to the variable name, and more.

![API Gateway Mapping Query String Parameters](/assets/aws-certified-developer-associate/ag_mapping_query_string_parameters.png "API Gateway Mapping Query String Parameters")

## 20.9 Using Mapping Templates to Transform Integration Responses

To use mapping templates, you need to:
1. Create a Lambda function.
2. Create an API.
3. Create a resource (e.g., `/mapping`).
4. Create a method (e.g., `GET`) for the resource and integrate it with the function without enabling the *Lambda Proxy Integration* option.

In the method details page, go into the *Integration Response* tab and scroll to the *Mapping Templates* section:

![API Gateway Mapping Integration Response](/assets/aws-certified-developer-associate/ag_mapping_integration_response.png "API Gateway Mapping Integration Response")

Click on *Create Template* to **configure a new mapping template** for the integration response by entering the `Content-Type` header value and the **template body**.
- The **template body is the VTL code that will be used to transform the response from the backend** to the desired format.
- The template body can be a static response or a dynamic response based on the input from the backend.
- The **input from the backend is available in the `$input` variable**, which is a VTL object that contains the request and response data.

![API Gateway Mapping Integration Response Template](/assets/aws-certified-developer-associate/ag_mapping_integration_response_template.png "API Gateway Mapping Integration Response Template")

Create the template and you will see it in the *Mapping Templates* section:

![API Gateway Mapping Integration Response Template Created](/assets/aws-certified-developer-associate/ag_mapping_integration_response_template_created.png "API Gateway Mapping Integration Response Template Created")

If you **test the API**, you will see that the response is transformed according to the mapping template you created:

![API Gateway Mapping Integration Response Test](/assets/aws-certified-developer-associate/ag_mapping_integration_response_test.png "API Gateway Mapping Integration Response Test")

## 20.10 OpenAPI Specification in API Gateway

The **OpenAPI Specification (OAS) is a standard for defining APIs in a machine-readable format**. It allows you to describe the API endpoints, request and response formats, authentication methods, and more.
- OpenAPI specifications can be written in YAML or JSON.
- Using OpenAPI, it is possible to generate SDKs for applications.

You **can import existing OpenAPI 3.0 specifications** to API Gateway that define: 
- Method.
- Method request.
- Integration request.
- Method response.
- AWS extensions for API Gateway and setup every single option.

You **can export current APIs as OpenAPI specifications** as well. This is useful for documentation purposes or to share the API definition with other developers.

### 20.10.1 Importing OpenAPI Specifications

Go into the API Gateway console and select the option to create a REST API. Then, you can:
1. Select *Import API* and provide the OpenAPI specification file in the *API Definition* field.
2. Select *Example API* to use a sample OpenAPI specification file provided by AWS.

We will use the second option:

![API Gateway Import OpenAPI Specification](/assets/aws-certified-developer-associate/ag_import_openapi_specification.png "API Gateway Import OpenAPI Specification")

Create the API and you will be redirected to the API Gateway console where you can see the API you just created:

![API Gateway OpenAPI Specification Created](/assets/aws-certified-developer-associate/ag_openapi_specification_created.png "API Gateway OpenAPI Specification Created")

### 20.10.2 Exporting OpenAPI Specifications

To export the OpenAPI specification of an API, go to the API Gateway console and select the API you want to export. Then, go to a stage and click on *Export* under the *Stage Actions* dropdown menu:

![API Gateway Export OpenAPI Specification](/assets/aws-certified-developer-associate/ag_export_openapi_specification.png "API Gateway Export OpenAPI Specification")

And configure the export format:

![API Gateway Export OpenAPI Specification Configuration](/assets/aws-certified-developer-associate/ag_export_openapi_specification_configuration.png "API Gateway Export OpenAPI Specification Configuration")

### 20.10.3 Generating SDKs

Go to the API Gateway console and select the API you want to export. Then, go to a stage and click on *Generate SDK* under the *Stage Actions* dropdown menu:

![API Gateway Generate SDK](/assets/aws-certified-developer-associate/ag_export_openapi_specification.png "API Gateway Generate SDK")

And select the **destination platform**:

![API Gateway Generate SDK Configuration](/assets/aws-certified-developer-associate/ag_generate_sdk_configuration.png "API Gateway Generate SDK Configuration")

## 20.11 API Gateway Request Validation

You can configure API Gateway **to perform basic validation of an API request** before proceeding with the integration request. **When the validation fails, API Gateway immediately fails the request and returns a `400`-error response** to the caller, reducing the load on the backend service.

You can check:
- The inclusion of required request parameters in the URI, query string, and headers of an
incoming request.
- The request payload adheres to the configured JSON schema request model of the method.

### 20.11.1 Request Validation using OpenAPI

You can setup request validation by importing OpenAPI definitions file.

You can **define request validators in the OpenAPI file** and then import it to API Gateway. The request validators will be created automatically in API Gateway and you can use them in your API methods.

The following is an example of an OpenAPI file with request validators definition:

```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "Request Validation Sample",
    "version": "1.0.0"
  },
  "servers" [/*...*/],
  /*...*/
  "x-amazon-apigateway-request-validators": {
    "all": {
      "validateRequestBody": true,
      "validateRequestParameters": true
    },
    "params-only": {
      "validateRequestBody": false,
      "validateRequestParameters": true
    }
  }
}
```

To **use the request validators in your API methods**, you need to specify the `x-amazon-apigateway-request-validator` property in the method definition:

```json
{
  /*...*/
  "paths": {
    "/validation": {
      "get": {
        "x-amazon-apigateway-request-validator": "params-only",
        "responses": {/*...*/}
      },
        "post": {
            "x-amazon-apigateway-request-validator": "all",
            "responses": {/*...*/}
        }
    }
  }
}
```

## 20.12 API Gateway Caching Capabilities

Caching reduces the number of calls made to the backend. The API Gateway **caches the responses from the backend and returns them to the client when the same request is made again**.

![API Gateway Caching](/assets/aws-certified-developer-associate/ag_caching.png "API Gateway Caching")

**Caches are defined at stage level**, so you defined the cache size and TTL for each stage.
- But it is **possible to override cache settings per method**.
- Default **TTL** is 300 (5 minutes) seconds with a minimum of 0s and a maximum 3600s (1 hour).
- Cache content can be encrypted.
- Cache capacity is between 0.5 GB to 237 GB.

Cache is expensive, it makes sense in production but may not make sense in dev/test environments.

### 20.12.1 Cache Invalidation

You can flush the entire cache (invalidate it) immediately from the UI.

Clients can invalidate the cache with the header `Cache-Control: max-age=0` but they need proper IAM authorization. For example, the policy below allows the `InvalidateCache` action on all API resources:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "execute-api:InvalidateCache"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

If you do not impose an `InvalidateCache` policy (or choose the *Require* authorization check box in the console), any client can invalidate the API cache.
- This can be a serious security issue.

### 20.12.2 Configuring Caching on Stages

Go into a stage and click *Edit*. Scroll until you find the *Additional Settings* section to configure caching:

![API Gateway Caching Configuration](/assets/aws-certified-developer-associate/ag_caching_configuration.png "API Gateway Caching Configuration")

From the image above, you can:
- **Enable API caching**: enable or disable caching for the stage.
- **Cache capacity**: set the cache size for the stage.
- **Enable cache encryption**: enable or disable cache encryption for the stage.
- **Cache TTL**: enable or disable cache TTL for the stage.

Another important configuration is the **per-key cache invalidation**. This option allows you to **require authorization for cache invalidation**. This is useful to prevent unauthorized users from invalidating the cache and react accordingly (e.g., ignore the cache invalidation request):

![API Gateway Per-Key Cache Invalidation](/assets/aws-certified-developer-associate/ag_per_key_cache_invalidation.png "API Gateway Per-Key Cache Invalidation")

### 20.12.3 Configuring Caching on Methods

To do that, click on the method you want to configure caching for and click *Edit*. Then, scroll until you find the *Enable Method Cache* option:

![API Gateway Enable Method Cache](/assets/aws-certified-developer-associate/ag_enable_method_cache.png "API Gateway Enable Method Cache")

**Enable method caching to override the stage cache settings** and configure the method cache settings:

![API Gateway Method Caching Configuration](/assets/aws-certified-developer-associate/ag_method_caching_configuration.png "API Gateway Method Caching Configuration")

## 20.13 Usage Plans and API Keys

If you want to **make an API available as an offering (under paid plans) to your customer**s, you can use usage plans and API keys.

**Usage plans**:
- Define who can access one or more deployed API stages and methods.
- Define how much and how fast they can access them.
- Use API keys to identify API clients and meter access.
- Configure throttling limits and quota limits that are enforced on individual client.

**API keys**:
- Alphanumeric string values to distribute to your customers: for example, `WBjHxNtoAb4WPKBC7cGm64CBibIb24b4jt8jJHo9`.
- Can be used with usage plans to control access.
- Throttling limits are applied to the API keys.
- Quota limits indicate the overall number of maximum requests.

### 20.13.1 Correct Order for API keys

It is **important to remember the steps to configure a usage plan**:
1. Create one or more APIs, configure the methods to require an API key, and deploy the APIs to stages.
2. Generate or import API keys to distribute to application developers (your customers) who will be using your API.
3. Create the usage plan with the desired throttle and quota limits.
4. Associate API stages and API keys with the usage plan.

**Callers of the API must supply an assigned API key in the `x-api-key` header in requests** to the API.

## 20.14 Logging, Tracing, and Monitoring

**CloudWatch Logs integration**:
- Logs contain information about request/response body.
- CloudWatch logging is **enabled at the stage level** with log levels: `ERROR`, `DEBUG`, and `INFO`.
- You can override settings on a per API basis.

![API Gateway CloudWatch Logs Integration](/assets/aws-certified-developer-associate/ag_cloudwatch_logs_integration.png "API Gateway CloudWatch Logs Integration")

**X-Ray integration**:
- Enable tracing to get extra information about requests in API Gateway.
- Enabling you X-Ray for both API Gateway and Lambda, you get the full picture for your APIs.

**CloudWatch Metrics integration**:
- **Metrics are by stage** with the possibility to enable detailed metrics.
- Important metrics:
  - `CacheHitCount` and `CacheMissCount`: efficiency of the cache.
  - `4XXError` and `5XXError`: errors returned client-side and server-side, respectively.
  - `Count`: the total number API requests in a given period.
  - `IntegrationLatency`: the time between when API Gateway relays a request to the backend and when it receives a response from the backend.
    - This does not include the API Gateway overhead.
  - `Latency`: The time between when API Gateway receives a request from a client and when it returns a response to the client.
    - The latency includes the integration latency and other API Gateway overhead (e.g., mapping templates and request validation).
    - If the latency is higher than 29 seconds, the API Gateway will return an error to the client.
