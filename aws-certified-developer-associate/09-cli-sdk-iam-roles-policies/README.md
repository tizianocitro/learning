# 9 CLI, SDK, and IAM Roles and Policies

## 9.1 EC2 Instance Metadata Service (IMDS)

The IMDS service allows AWS EC2 instances to learn about themselves without using an IAM Role for that purpose.

You can use the URL http://169.254.169.254/latest/meta-data to get the following information:
- **Metadata**: info about the EC2 instance.
- **Userdata**: launch script of the EC2 instance.

You **can retrieve the IAM role name** from the metadata (also the instance ID, public IP, some credentials, etc). However, you **cannot retrieve the IAM policy**.

### 9.1.1 IMDSv2 vs IMDSv1

You can use **IMDSv1** by accessing http://169.254.169.254/latest/meta-data directly.

**IMDSv2** is more secure version and requires a token to access the metadata. You need two steps:
1. Get the session token (limited validity) using a PUT request with a TTL header:
    ```bash
    TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
    ```
2. Use the session token in IMDSv2 requests by passing it in the headers:
    ```bash
    curl -H "X-aws-ec2-metadata-token: $TOKEN" "http://169.254.169.254/latest/meta-data/profile"
    ```

## 9.2 Using IMDS

First, you need to create an EC2 instance. During creation, into the advanced details, you can find configurations about metadata:

![EC2 Metadata](/assets/aws-certified-developer-associate/ec2_metadata.png "EC2 Metadata")

After launching the instance, you can SSH into it and run the commands to get metadata. Mind that V1 does not work on all instances depending on the AMI. You should get an output like this:

![EC2 Metadata Response](/assets/aws-certified-developer-associate/ec2_metadata_response.png "EC2 Metadata Response")

Where you see a traling slash (*/*), it means that it is a directory and you can access more information by adding the directory name to the URL. Instead if you want to get the content of a file, you can add the file name to the URL.

For example, to get the hostname:

![EC2 Metadata Hostname](/assets/aws-certified-developer-associate/ec2_metadata_hostname.png "EC2 Metadata Hostname")

The same is true for IP address, instance ID, etc.

An **important directory is identity-credentials**, where you can find a **security-credentials directory with the role name**. If you do not have a role assigned, you will get a 404 error when trying to access this directory. However, if you have a role, you will find and **ec2-instance** file with the credentials. For example:

![EC2 Metadata Credentials](/assets/aws-certified-developer-associate/ec2_metadata_credentials.png "EC2 Metadata Credentials")

As you can see, you get the access key, secret key, and a token with an expiration time. This is how the EC2 instance gets its credentials to access AWS services.

## 9.3 AWS CLI Profiles

You can create **profiles in the AWS CLI to store multiple sets of credentials**. This is useful when you have multiple accounts or roles.

To create a profile, you can use the `aws configure --profile <profile-name>` command. You will be asked for the access key, secret key, region, and output format. You can also edit the `~/.aws/credentials` file to add more profiles.

The **credentials file** has the following format:

```ini
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
[second-profile]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET
```

The **default profile is used when you do not specify a profile**. To use another profile, you can use the `--profile` flag in the AWS CLI commands. For example:

```bash
aws s3 ls --profile second-profile
```

## 9.4 How to Use MFA with the CLI

To use MFA with the CLI, you must create a temporary session and, to do so, you must run the STS GetSessionToken API call. **STS GetSessionToken is the API you want to call to get temporary credentials with a MFA device**.

**The command is (this is important for the exame)**:

```bash
aws sts get-session-token --serial-number arn-of-the-mfa-device --token-code code-from-token --duration-seconds 3600
```

For example:

```bash
aws sts get-session-token --serial-number arn:aws:iam::123456789012:mfa/my-user --token-code 123456 --duration-seconds 3600
```

And this is the response containing the temporary credentials:

![STS Token](/assets/aws-certified-developer-associate/sts_token.png "STS Token")

The `--serial-number` can be obtained from the IAM console where you can find the ARN of the MFA device you assigned to the user. The `--token-code` is the code from the MFA device (e.g., the code in an authenticator app like Authy).

You can **use these temporary credentials to create a new profile for the CLI** either by editing the `~/.aws/credentials` file or by using the `aws configure --profile <profile-name>` command.

An example of how the temporary credentials should look like in the `~/.aws/credentials` file:

```ini
[mfa-profile]
aws_access_key_id = TEMP_ACCESS_KEY
aws_secret_access_key = TEMP_SECRET_KEY
aws_session_token = TEMP_SESSION_TOKEN
```

## 9.5 SDK

You can use an SDK to perform actions on AWS directly from your applications code (without using the CLI).

Official SDKs are: java, .NET, node.js, PHP, python, Ruby, Go, C++, ...

We have to use the AWS SDK when coding against AWS Services such as DynamoDB. The AWS CLI uses the python SDK (boto3).

**The exam expects you to know when you should use an SDK**, e.g., for Lambda functions.

**Important for the exam: if you do not specify or configure a default region, then us-east-1 will be chosen by default**.

## 9.6 AWS Limits (Quotas)

**AWS has limits for each service**. For example, you can only have 100 S3 buckets per account. **These limits are called quotas**.

Two types of quotas:
1. **API Rate Limits**:
    - **DescribeInstances** API for EC2 has a limit of 100 calls per seconds.
    - **GetObject** on S3 has a limit of 5500 GET per second per prefix.
    - **For intermittent errors: implement exponential backoff**.
    - **For consistent errors: ask AWS for an API throttling limit increase**.
2. **Service Quotas** (service limits, how many resources we can run of something):
    - **Running on-demand standard instances**: max 1152 vCPU.
    - **You can request a service limit increase by opening a ticket to AWS**.
    - **You can request a service quota increase by using the Service Quotas API**.

### 9.6.1 Exponential Backoff

You use **exponential backoff when you get ThrottlingException (rised for too many API calls) intermittently**.

Expontential backoff is a strategy where you retry an operation with an increasing amount of time between retries. For example, you can retry an operation after 1s, 2s, 4s, 8s, 16s, etc.

The retry mechanism is already included in AWS SDK API calls. However, if you are using the AWS API as-is, you can implement your own retry mechanism.

The **exam ask you which kind of errors should you retry on exponential backoff**:
- **Must only implement the retries on 5xx server errors and throttling**.
- **Do not implement on the 4xx client errors** because it means something is wrong with the request sent by the client, so retrying will not help.

## 9.7 AWS Credentials Provider Chain

### 9.7.1 AWS CLI Credentials Provider Chain

The CLI will look for credentials in this order (**can come up at the exam**):
1. **Command line options**: `--region`, `--output`, `--profile`, and more.
2. **Environment variables**: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`, and more.
3. **CLI credentials file**: `~/.aws/credentials` on Linux/macOS and `C:\Users\user\.aws\config` on Windows.
4. **CLI configuration file**: `~/.aws/config` on Linux/macOS and `C:\Users\USERNAME\.aws\config` on Windows.
5. **Container credentials**: for ECS tasks.
6. **Instance profile credentials**: for EC2 instance profiles.

### 9.7.2 AWS SDK Default Credentials Provider Chain

For example, the java SDK will look for credentials in this order:
1. **Java system properties**: `aws.accessKeyId` and `aws.secretKey`.
2. **Environment variables**: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
3. **Default credential profiles file**: for example, at `~/.aws/credentials`, shared by many SDKs.
4. **Amazon ECS container credentials**: for ECS containers.
5. **Instance profile credentials**: used on EC2 instances.

### 9.7.3 AWS Credentials Scenario

**Important for the exam**.

**Question**:

Suppose you have an application deployed on an EC2 instance that is using environment variables with credentials from an IAM user to call the Amazon S3 API (bad practice!).

The IAM user has S3FullAccess permissions.

The application only uses one S3 bucket, so according to best practices:
- An IAM role and EC2 instance profile was created for the EC2 instance.
- The role was assigned the minimum permissions to access that one S3 bucket.

The IAM instance profile was assigned to the EC2 instance, but it still has access to all S3 buckets. Why?

**Answer**:

The credentials chain is still giving priorities to the environment variables. The environment variables will always take precedence over the instance profile. To solve this issue, you should remove the environment variables from the EC2 instance to make the credentials chain work by using the instance profile.

## 9.8 AWS Credentials Best Practices

- Never ever store AWS credentials in your code.
- Best practice is for credentials to be inherited from the credentials chain.
- **If you are working within AWS, use IAM roles**:
    - EC2 instances roles for EC2 Instances.
    - ECS roles for ECS tasks.
    - Lambda roles for Lambda functions.
- **If working outside of AWS, use environment variables/named profiles**.
    - For example, you are running an application on an **on-premises server**. The application needs to perform API calls to an S3 bucket. You can **create a dedicated IAM user for the application and store the credentials in environment variables or a named profile**.

## 9.9 Signing AWS API requests

When you call the AWS HTTP API, you sign the request so that AWS can identify you, using your AWS credentials (i.e., access key and secret key).

Some requests to Amazon S3 do not need to be signed but for most requests, you need to sign them.

If you use the SDK or CLI, the HTTP requests are signed for you by the SDK or CLI.

However, it is important to know that you should **sign an AWS HTTP request using Signature v4 (SigV4)**.

![SigV4](/assets/aws-certified-developer-associate/sigv4.png "SigV4")

You do not need to know how to **sign a request for the exam**, but you should know that it is **done using SigV4** and that there are **2 ways to send your signature to AWS once computed**:
1. **HTTP Header (signature in Authorization header)**, this what the CLI does:
    ![SigV4 Header](/assets/aws-certified-developer-associate/sigv4_header.png "SigV4 Header")
2. **Query String (signature in X-Amz-Signature parameter)**, S3 pre-signed URLs use this method:
    ![SigV4 Query String](/assets/aws-certified-developer-associate/sigv4_query_string.png "SigV4 Query String")