## 11 S3 Security

## 11.1 S3 Encryption

You can encrypt objects in S3 buckets using one of **4 methods**:

1. **Server-Side Encryption with Amazon S3-Managed Keys (SSE-S3)**: 
    - Enabled by Default.
    - Encrypts S3 objects using keys handled, managed, and owned by AWS.
2. **Server-Side Encryption with KMS Keys stored in AWS KMS (SSE-KMS)**:
    - Leverage AWS Key Management Service (AWS KMS) to manage encryption keys yourself within AWS.
3. **Server-Side Encryption with Customer-Provided Keys (SSE-C)**:
    - When you want to manage your own encryption keys.
4. **Client-Side Encryption**:
    - When you want to encrypt data client-side and upload the encrypted data to S3.

It is **important to understand which ones are for which situation for the exam**.

There is also a new method called **Dual-layer Server-Side Encryption with KMS Keys stored in AWS KMS (DSSE-KMS)**, which simplifies the process of applying two layers of encryption to your data (for now, this does not come up in the exam).

### 11.1.1 Server-Side Encryption with Amazon S3-Managed Keys (SSE-S3)

Objects are encrypted server-side using AES-256 encryption. **Encryption using keys handled, managed, and owned by AWS**. 

Enabled by default for new buckets and new objects.

When you upload an object to S3, you can request that S3 encrypt the object using an AWS managed key. If you don't provide a key, S3 encrypts the object using a default key.

**To request S3 to encrypt the object using an AWS managed key, you can add the header** `"x-amz-server-side-encryption": "AES256"`. So, **S3 will pair the object with a S3-owned key, encrypt the object, and then store the encrypted object**. With this approach, **you do not need to access the key to retrieve the object**.

![SSE-S3](/assets/aws-certified-developer-associate/sses3.png "SSE-S3")

### 11.1.2 Server-Side Encryption with KMS Keys stored in AWS KMS (SSE-KMS)

Objects are encrypted server-side and the encryption is performerd using keys handled and managed by AWS KMS (Key Management Service) but **you manage the keys within AWS KMS and you can audit key usage using *CloudTrail***.

**To request S3 to encrypt the object using a KMS key, you can add the header**` "x-amz-server-side-encryption": "aws:kms"`. So, **S3 will pair the object with a KMS key, encrypt the object, and then store the encrypted object**. In this case, **you must access the KMS key to retrieve the object**, which adds an extra layer of security.

![SSE-KMS](/assets/aws-certified-developer-associate/ssekms.png "SSE-KMS")

SSE-KMS has some **limitations**:
- If you use SSE-KMS, you **may be impacted by the KMS rate limits and costs**.
- When you upload, it calls the GenerateDataKey KMS API.
- When you download, it calls the Decrypt KMS API.
- Each of **these API calls count towards the KMS quota per second** (5500, 10000, 30000 req/s based on region), so **you may incur in throttling issues (the exam may ask you about this)**.
- You can request a quota increase using the Service Quotas console.

![SSE-KMS Limitations](/assets/aws-certified-developer-associate/ssekms_limitations.png "SSE-KMS Limitations")

### 11.1.3 Server-Side Encryption with Customer-Provided Keys (SSE-C)

Objects are encrypted server-side using **keys fully managed by the customer outside of AWS**. **S3 does not store the encryption key you provide**, so you are responsible for managing the encryption key.

**HTTPS must be used because the key is sent as a header with the HTTPS request (upload, download)**, it must be send for every request made.

![SSE-C](/assets/aws-certified-developer-associate/ssec.png "SSE-C")

### 11.1.4 Client-Side Encryption

To implement client-side encryption, use client libraries such as **Amazon S3 Client-Side Encryption Library**.

Clients must encrypt data themselves before sending to Amazon S3 and must decrypt data themselves when retrieving from Amazon S3. **Customer fully manages the keys and encryption cycle**.

![S3 Client-Side Encryption](/assets/aws-certified-developer-associate/s3_client_side_encryption.png "S3 Client-Side Encryption")

### 11.1.5 Encryption in Transit

**Encryption in transit/flight is also called SSL/TLS**. It protects your data as it travels to and from S3.

S3 exposes **two endpoints**:
- HTTP Endpoint: non encrypted.
- HTTPS Endpoint: encryption in flight.

**HTTPS is fully recommended and is mandatory for SSE-C**. Most clients would use the HTTPS endpoint by default.

To **force encryption in transit, you can use an S3 bucket policy to deny all S3 operations that are not using HTTPS**.

![S3 Encryption in Transit](/assets/aws-certified-developer-associate/s3_encryption_in_transit.png "S3 Encryption in Transit")

In the above image, the **policy uses the** `aws:SecureTransport` **condition key to allow only requests that are made using HTTPS**. If the request is made using HTTP, the request is denied. The `aws:SecureTransport` will be `true` whenever the request is made using HTTPS and `false` whenever the request is made using HTTP.

### 11.1.6 Default Encryption vs Bucket Policies

By default, SSE-S3 encryption is automatically applied to new objects stored in S3 buckets. However, you ca change it to another encryption option.

Optionally, you can force encryption using a bucket policy and refuse any API call to PUT an S3 object without encryption headers (SSE-KMS or SSE-C).

Note: **Bucket Policies are evaluated before Default Encryption settings**.

For example, a **policy to block PUT request missing the SSE-KMS header** would look like this:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyRequestsMissingKMSEncryptionHeader",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::mybucket/*",
            "Condition": {
                // StringNotEquals means the header is not equal to the value
                "StringNotEquals": {
                    "s3:x-amz-server-side-encryption": "aws:kms"
                }
            }
        },
        {
            "Sid": "DenyRequestsMissingEncryptionHeader",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::mybucket/*",
            "Condition": {
                // Null means the header is not present
                "Null": {
                    "s3:x-amz-server-side-encryption": "true"
                }
            }
        }
    ]
}
```

Another example, a **policy to block PUT request missing the SSE-C header** would look like this:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyRequestsMissingCustomerEncryptionHeader",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::mybucket/*",
            "Condition": {
                // Null means the header is not present
                "Null": {
                    "s3:x-amz-server-side-encryption-customer-algorithm": "true"
                }
            }
        }
    ]
}
```

## 11.2 Using Encryption in S3

To practice encryption in S3, create a bucket and in it, you can configure default encryption for the bucket. The default encryption can be changed later in the bucket properties.

Default encryption settins will apply to all objects uploaded to the bucket. **You do not need to tell AWS that you are encrypting objects client-side, so the Client-Side Encryption is not an option in the console**.

![S3 Test Encryption](/assets/aws-certified-developer-associate/s3_test_encryption.png "S3 Test Encryption")

You can check if an uploaded object is encrypted by accessing the object's properties under *server-side encryption settings*, where you can also edit it. If you edit the encryption settings, it will create a new version of the object.

![S3 Test Encryption Object](/assets/aws-certified-developer-associate/s3_test_encryption_object.png "S3 Test Encryption Object")

So, if you choose to **override the encryption settings** for that object, you will be prompted to choose the new encryption settings for the object:

![S3 Test Encryption Object Override](/assets/aws-certified-developer-associate/s3_test_encryption_object_override.png "S3 Test Encryption Object Override")

If you choose to use SSE-KMS, you can select the KMS key to use for the encryption (you can also create a new one):

![S3 Test Encryption Object Override KMS](/assets/aws-certified-developer-associate/s3_test_encryption_object_override_kms.png "S3 Test Encryption Object Override KMS")

**By default, there is a default KMS key available that will not cost any money**.

If you select a key and confirm, now the object's encryption settings will look like this:

![S3 Test Encryption Object Override KMS Selected](/assets/aws-certified-developer-associate/s3_test_encryption_object_override_kms_selected.png "S3 Test Encryption Object Override KMS Selected")

**You can select encryption settings also when you upload an object to the bucket**.

## 11.3 Cross-Origin Resource Sharing (CORS)

**CORS can come up in 1 question in the exam**.

**Origin**: scheme (protocol) + host (domain) + port.
- For example, https://www.example.com (implied port is 443 for HTTPS, 80 for HTTP).

CORS is a web browser-based security mechanism to allow requests to other origins while visiting the main origin.
- **Same origin**: http://example.com/app1 and http://example.com/app2.
- **Different origins**: http://www.example.com and http://other.example.com.

The **requests won't be fulfilled unless the other origin allows for the requests using CORS Headers** (e.g., *Access-Control-Allow-Origin*).

How CORS works:

![CORS](/assets/aws-certified-developer-associate/cors.png "CORS")

CORS is important for S3 because **if a client makes a cross-origin request on our S3 bucket, we need to enable the correct CORS headers**.

This is a **popular exam question**, you can allow for a specific origin or for * (all origins).

For example, we request a website deployed on S3 and the website needs an image that is stored in another S3 bucket. The website will make a request to the other S3 bucket to get the image. **If the other S3 bucket does not have the correct CORS settings, the request will be blocked**.

![S3 CORS](/assets/aws-certified-developer-associate/s3_cors.png "S3 CORS")

## 11.4 Using CORS in S3

To define CORS on an S3 bucket, you need to go under the *Permissions* tab and then find the *Cross-Origin Resource Sharing* configuration box.

A **CORS configuration is a JSON document that defines the rules that allow the web page to make requests from a different domain**. For example, the above CORS configuration allows for GET requests from all origins and allows the Authorization header:

```json
[
    {
        "AllowedHeaders": [
            // you can also allow all headers with "*"
            "Authorization"
        ],
        "AllowedMethods": [
            // you can also allow all methods with "*"
            // or multiple methods, for example, ["GET", "POST"]
            "GET"
        ],
        "AllowedOrigins": [
            // * means all origins
            // alternatively, you can specify a specific origin
            // for example, "https://www.example.com"
            // for multiple origins, you can use a list
            // for example, ["https://www.example.com", "https://other.example.com"]
            // remember, no trailing slash!
            "*"
        ],
        "ExposeHeaders": [],
        "MaxAgeSeconds": 3000
    }
]
```

To test this, place an `index.html` file in a main bucket and a `extra-page.html` file in another bucket. Both buckets must have the website hosting enabled and have public access.

The `index.html` file will make a request to the `extra-page.html` file. If the CORS settings are not correct, the request will be blocked.

The `index.html` file will look like this:

```html
<html>
    <head>
        <title>Coffee Lovers Webpage</title>
    </head>

    <body>
        <h1>We love coffee!!!</h1>
    </body>

    <img src="coffee.jpg" width=500 />

    <!-- CORS demo -->
    <div id="tofetch" />

    <script>
        var tofetch = document.getElementById("tofetch");

        fetch('http://<enter bucket website URL here>/extra-page.html')
        .then((response) => { 
            return response.text();
        })
        .then((html) => {
            tofetch.innerHTML = html     
        });
    </script>
</html>
```

The `extra-page.html` file will look like this:

```html
<p>Our <strong>coffee</strong> has been made successfully!</p>
```

## 11.5 S3 MFA Delete

MFA Delete is an **extra protection to prevent accidental permanent deletions of specific object versions in versioned S3 buckets**.

This features alows you to employ MFA to **force users to generate a code on a device** (usually a mobile phone or hardware) **before doing important operations on S3**.

**Only the bucket owner (root account) can enable/disable MFA Delete**.

**Versioning must be enabled on the bucket to use MFA Delete**.

MFA will be required to (dangerous operations):
- Permanently delete an object version.
- Suspend Versioning on the bucket.

MFA will not be required to (not dangerous operations):
- Enable Versioning.
- List deleted versions.

## 11.6 How to Use MFA Delete

To use MFA Delete, add an MFA device on your account and create a bucket with versioning enabled. Then check MFA Delete status on the bucket by going into *Properties* and then into the *Bucket Versioning* settings.

![S3 MFA Delete Status](/assets/aws-certified-developer-associate/s3_mfa_delete_status.png "S3 MFA Delete Status")

As you can see from the image above, MFA Delete is disabled. As of now, it is not possible to enable MFA Delete using the console, you must use the AWS CLI.

Set the CLI to add a profile with root account credentials (e.g., `root-account-for-mfa-delete`) and then run the following command to enable MFA Delete (`MFADelete=Enabled`):

```bash
aws s3api put-bucket-versioning \
    --bucket mybucket \
    --versioning-configuration Status=Enabled,MFADelete=Enabled \
    --mfa "arn:aws:iam::123456789012:mfa/root-account-mfa-device 123456" \
    --profile root-account-for-mfa-delete
```

After doing so, if you access the bucket properties again, you will see that MFA Delete is enabled.

With MFA Delete enabled, **if you try to delete an object version (permanent delete operation) from the console, you will get an error**:

![S3 MFA Delete Error](/assets/aws-certified-developer-associate/s3_mfa_delete_error.png "S3 MFA Delete Error")

To delete the object version, you must use the AWS CLI and provide the MFA token:

```bash
aws s3api delete-object \
    --bucket mybucket \
    --key myobject \
    --version-id myversionid \
    --mfa "arn:aws:iam::123456789012:mfa/root-account-mfa-device 123456" \
    --profile root-account-for-mfa-delete
```

To disable MFA Delete, you can use the following command (`MFADelete=Disabled`):

```bash
aws s3api put-bucket-versioning \
    --bucket mybucket \
    --versioning-configuration Status=Enabled,MFADelete=Disabled \
    --mfa "arn:aws:iam::123456789012:mfa/root-account-mfa-device 123456" \
    --profile root-account-for-mfa-delete
```

## 11.7 S3 Access Logs

For **audit purpose, you may want to log all access to S3 buckets**. 

Any request made to S3, from any account, authorized or denied, will be logged in a file into another S3 bucket. That data can be analyzed using data analysis tools, such as Amazon Athena.

The **target logging bucket must be in the same AWS region**.

![S3 Access Logs](/assets/aws-certified-developer-associate/s3_access_logs.png "S3 Access Logs")

**Do not set your logging bucket to be the monitored bucket as it will create a logging loop**, and your bucket will grow exponentially and cost you a lot of money.

![S3 Access Logs Loop](/assets/aws-certified-developer-associate/s3_access_logs_loop.png "S3 Access Logs Loop")

### 11.8 How to Use S3 Access Logs

Create two buckets: *s3-bucket* and *s3-access-logs*.

Then, go to the *Properties* of *s3-bucket* and find *Server Access Logging*:

![S3 Access Logs Settings](/assets/aws-certified-developer-associate/s3_access_logs_settings.png "S3 Access Logs Settings")

**Enable the logging and select the target bucket (destination)** for the logs:

![S3 Access Logs Enable](/assets/aws-certified-developer-associate/s3_access_logs_enable.png "S3 Access Logs Enable")

Enabling the loggind will also upddate the policy for the target bucket to allow the source bucket to write logs to it. For example:

```json
{
    "Version": "2012-10-17",
    "Id": "S3-Console-Auto-Gen-Policy-1649330814237",
    "Statement": [
        {
            "Sid": "S3PolicyStmt-DO-NOT-MODIFY-1649330814124",
            "Effect": "Allow",
            "Principal": {
                "Service": "logging.s3.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::s3-access-logs/*"
        }
    ]
}
```

You **can also enable logs only for a prefix** (e.g., *logs/*). For example, instead of `s3://s3-access-logs`, you could use `s3://s3-access-logs/logs/`.

Finally, set the **log object key format**:

![S3 Access Logs Format](/assets/aws-certified-developer-associate/s3_access_logs_format.png "S3 Access Logs Format")

Now, if you access the *s3-access-logs* bucket, you will see the **log files containing the logs** of the operations made on *s3-bucket*:

![S3 Access Logs Log Files](/assets/aws-certified-developer-associate/s3_access_logs_log_files.png "S3 Access Logs Log Files")

And if you open one of the log files, you will see the actual log entries.

## 11.9 S3 Pre-signed URLs

Pre-signed URLs are URLs that you can generate using the S3 Console, AWS CLI or SDK. They have an **expiration that depends on the method used to generate them**: 
- Console: from 1 minute up to 720 minutes (12 hours).
- AWS CLI: expiration configured in seconds with the `--expires-in` parameter (default: 3600 seconds, maximum: 604800 seconds ~ 168 hours).

Users given a pre-signed URL inherit the permissions of the user who generated the URL for GET/PUT of an object. **You can generate a pre-signed URL for an object if you want to share the object with someone else without making the object public**. Pre-signed URLs are useful for temporary access to objects.

They are **called pre-signed URLs because they are signed with the user's security credentials and they carry the user's permissions**.

![S3 Pre-signed URLs](/assets/aws-certified-developer-associate/s3_presigned_urls.png "S3 Pre-signed URLs")

A few examples of **use cases**:
- Allow only logged-in users to download a premium video from your S3 bucket.
- Allow an ever-changing list of users to download files by generating URLs dynamically.
- Allow temporarily a user to upload a file to a specific location in your S3 bucket.

### 11.9.1 How to Use S3 Pre-signed URLs

Go into a private bucket and select an object (in the image below, *coffee.jpg*).

Then, under *Object Actions*, select *Share with a pre-signed URL* to create a pre-signed URL for the object:

![S3 Pre-signed URL Creation](/assets/aws-certified-developer-associate/s3_presigned_url_creation.png "S3 Pre-signed URL Creation")

Create the URL and then you will be able to copy it and share it with someone else, who can use it until it expires.

## 11.10 S3 Access Points

Access points simplify security management for S3 buckets. Each access point has:
- Its own DNS name (Internet Origin or VPC Origin).
- An **access point policy** (similar to bucket policy) to manage security at scale.

![S3 Access Points](/assets/aws-certified-developer-associate/s3_access_points.png "S3 Access Points")

As you can see from the image above, you can **use access points to scale access to S3 buckets**. You can create an access point for each application, team, or department, and then manage the access to the bucket using the access point policies.

### 11.10.1 Access Point VPC Origin

You can define the **access point to be accessible only from within the VPC** (so, making it private without going through the internet).

To do so, you must **create a VPC Endpoint to privately access the access point (Gateway or Interface Endpoint)**. The **VPC Endpoint Policy must allow access to the target bucket and access point**.

![S3 Access Points VPC Origin](/assets/aws-certified-developer-associate/s3_access_points_vpc_origin.png "S3 Access Points VPC Origin")

## 11.11 S3 Object Lambda

Use **AWS Lambda Functions to change objects before they are retrieved to the requester**.

**Only one S3 bucket is needed**, on top of which we create an **S3 access point** and **S3 object lambda access points**.

**Object lambda access points are associated with a Lambda function** that will be triggered when an object is requested through the access point.

![S3 Object Lambda](/assets/aws-certified-developer-associate/s3_object_lambda.png "S3 Object Lambda")

**Use cases**:
- Redacting personally identifiable information for analytics or non-production environments.
- Converting across data formats, such as converting CSV to JSON or XML to JSON.
- Resizing images on the fly.
- Watermarking images on the fly using caller-specific information, such as the user who requested the image.