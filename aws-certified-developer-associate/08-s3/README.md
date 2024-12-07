# 8 Simple Storage Service (S3)

Amazon S3 is one of the main building blocks of AWS as many websites use S3 as a backbone and also many AWS services use Amazon S3 as an integration.

So many use cases for S3, such as:
- Backup and storage.
- Disaster recovery: move your data to a different region in case of disaster.
- Archive: archive your data to S3 Glacier (archive service of S3) for cheap storage.
- Hybrid Cloud storage.
- Application hosting: static website hosting.
- Media hosting.
- Data lakes and big data analytics.
- Software delivery.
- Static website.

## 8.1 Buckets

Amazon S3 allows people to **store objects (files) in buckets (directories)**.

Buckets must have a **globally unique name across all regions and all accounts**. Still, buckets are defined at the region level, even though S3 looks like a global service. **Buckets are created in a region**.

**Naming convention** for buckets:
- No uppercase.
- No underscore.
- 3-63 characters long.
- Not an IP.
- Must start with lowercase letter or number.
- Must not start with the prefix *xn--*.
- Must not end with the suffix *-s3alias*.

## 8.2 Objects

**Objects (files) have a key**, which is the **full path of the file**:
- In *s3://my-bucket/my_file.txt*, the key is *my_file.txt*.
- In *s3://my-bucket/my_folder1/another_folder/my_file.txt*, the key is *my_folder1/another_folder/my_file.txt*.

So, the key has a **prefix** (folder structure) and a **name**:
- In *s3://my-bucket/my_file.txt*, the prefix is empty and the name is *my_file.txt*.
- In *s3://my-bucket/my_folder1/another_folder/my_file.txt*, the prefix is *my_folder1/another_folder/* and the name is *my_file.txt*.

It is important to note that **S3 is a flat storage system**. There is **no concept of directories within buckets** (although the UI will trick you to think otherwise). Keys are just very long names that contain slashes (/).

## 8.3 Object Properties

- **Object values are the content of the body**:
    - Max object size is 5TB (5000GB).
    - If uploading more than 5GB, you must use the **multi-part upload** to upload that file in multiple parts. For example, if you are uploading a 15GB file, you can upload it in 3 parts of 5GB each.
- **Metadata**: list of text key/value pairs that can be set by the system or user.
- **Tags**: Unicode key/value pair (up to 10), which are useful for security/lifecycle.
- **Version ID**: if versioning is enabled.

## 8.4 Creating an S3 Bucket

To create an S3 bucket, go to the Amazon S3 in the AWS Console and click on the *Create bucket* button. Then, you must provide a **unique name** for your bucket that will be created in the region you are currently in (the one selected in the console). Eventually, you can also replicate settings from an existing bucket. **For the exam, only general purpose buckets are important**, directory buckets are not required.

![Create S3 Bucket](/assets/aws-certified-developer-associate/create_s3_bucket.png "Create S3 Bucket")

You also have some security/ownership settings:

![S3 Bucket Settings](/assets/aws-certified-developer-associate/s3_bucket_settings.png "S3 Bucket Settings")

Next, you can enabled **bucket versioning**, which will keep all versions of an object in the bucket:

![S3 Enable Versioning](/assets/aws-certified-developer-associate/s3_enable_versioning.png "S3 Enable Versioning")

Then, you can add **tags** and configure the **default encryption**:

![S3 Default Encryption](/assets/aws-certified-developer-associate/s3_default_encryption.png "S3 Default Encryption")

Finally, you can create the bucket and it will appear in the list of buckets. Clicking on the bucket name will show the bucket details:

![S3 Bucket Details](/assets/aws-certified-developer-associate/s3_bucket_details.png "S3 Bucket Details")

### 8.4.1 Uploading Objects to an S3 Bucket

Go into the bucket's details and, from the *Objects* tab, you can **upload files to the bucket** (directly from the console):

![S3 Upload File](/assets/aws-certified-developer-associate/s3_upload_file.png "S3 Upload File")

And it will appear in the list of objects in the bucket:

![S3 Object List](/assets/aws-certified-developer-associate/s3_object_list.png "S3 Object List")

By clicking on the object name, you can see the object details, such as the Object URL, ARN, and S3 URI.

The **Object URL** is the URL that you can use to access the object from the web. It is in the format [https://s3-<region>.amazonaws.com/<bucket_name>/<object_name>](https://s3-<region>.amazonaws.com/<bucket_name>/<object_name>). **This is an S3 pre-signed URL that contains a signature that allows you to access the object (only if you are logged in)**.

### 8.4.2 Creating a Folder in an S3 Bucket

As mentioned before, S3 is a flat storage system, so there are no folders. However, **you can create a folder by creating an object with a key that ends with a slash (/)**. For example, if you create an object with the key *my_folder/*, it will look like a folder in the console.

In the AWS Console, you can create a folder by clicking on the *Create folder* button:

![S3 Create Folder](/assets/aws-certified-developer-associate/s3_create_folder.png "S3 Create Folder")

After you create the folder, it appears in the list of objects in the bucket:

![S3 Folder](/assets/aws-certified-developer-associate/s3_folder.png "S3 Folder")

Then, you can upload objects to the folder. To do so, click on the folder name and then click on the *Upload* button. What will change, is that, in this case, the **destination will be the folder**:

![S3 Upload File to Folder](/assets/aws-certified-developer-associate/s3_upload_file_to_folder.png "S3 Upload File to Folder")

After you upload the file, it will appear in the list of objects within the folder in the same way it appears in the bucket.

## 8.5 S3 Security

- **User-based**:
    - **IAM policies**: which API calls should be allowed for a specific user from IAM.
- **Resource-based**:
    - **Bucket policies**: bucket wide rules from the S3 console. They also allow you to enabled cross account access, meaning you can access a bucket from another account.
    - **Object Access Control List (ACL)**: finer grain (can be disabled).
    - **Bucket Access Control List (ACL)**: less common (can be disabled).
- **Encryption**: encrypt objects in S3 using encryption keys.

The **most common ways to control access to S3 is through bucket policies or IAM policies (IAM permissions to IAM users or roles)**.

**An IAM principal can access an S3 object if the user's IAM permissions allow it or the resource policy allows it and there is no explicit deny**.

### 8.5.1 S3 Bucket Policies

They are **JSON-based policies**:
- Resources: buckets and objects.
- Effect: Allow/Deny.
- Actions: set of APIs to allow or deny.
- Principal: the account or user to apply the policy to.

Following an example of a bucket policy that allows public read access to the bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            // Apply the policy to all objects in the bucket
            // By using the wildcard (*) in the resource
            "Resource": [
                "arn:aws:s3:::my-bucket/*"
            ]
        }
    ]
}
```

Use S3 bucket policies to:
- Grant public access to the bucket.
- Force objects to be encrypted at upload.
- Grant access to another account (Cross account).

### 8.5.2 S3 Block Public Access

During bucket creation, you can enable **Block all public access** to the bucket. This will prevent any public access to the bucket and its objects. This setting can be changed later in the bucket settings. **This settings override any bucket policy that allows public access**.

![S3 Block Public Access](/assets/aws-certified-developer-associate/s3_bucket_settings.png "S3 Block Public Access")

These settings were created to **prevent company data leaks**:
- If you know your bucket should never be public, leave these on.
- Can be set at the account level.

## 8.6 Creating S3 Bucket Policies

Suppose we want to **create a policy that allows public read access to the bucket**. To do so, go to the bucket's details and click on the *Permissions* tab. 

First thing you need to do is to disable the **Block all public access** setting. Then, you can add a bucket policy by adding it into the **bucket policy editor** or using the **policy generator**

Following is an example of policy creation using the policy generator:

![S3 Bucket Policy Generator](/assets/aws-certified-developer-associate/s3_bucket_policy_generator.png "S3 Bucket Policy Generator")

**To apply the policy to the bucket, you need to apply it to the bucket ARN followed by `/*` because the policy is applied to objects in the bucket.**

The policy generator will give you the policy in JSON format, which you can copy and paste into the bucket policy editor:

![S3 Bucket Policy in JSON](/assets/aws-certified-developer-associate/s3_bucket_policy_json.png "S3 Bucket Policy in JSON")

Just copy and paste it into the bucket policy editor:

![S3 Bucket Policy Editor](/assets/aws-certified-developer-associate/s3_bucket_policy_editor.png "S3 Bucket Policy Editor")

After you save the policy, the bucket will have public read access. You can test it by trying to access the object URL in an incognito window or another browser where you are not logged in.

## 8.7 Static Website Hosting

S3 can **host static websites and have them accessible on the internet**.

The website URL will be (depending on the region):
- http://bucket-name.s3-website-aws-region.amazonaws.com
- or http://bucket-name.s3-website.aws-region.amazonaws.com

For example, if the bucket name is *demo-bucket* and the region is *us-west-2*, the website URL will be:
- http://demo-bucket.s3-website-us-west-2.amazonaws.com
- or http://demo-bucket.s3-website.us-west-2.amazonaws.com

If you get a **403 Forbidden error, make sure the bucket policy allows public reads**.

To enable static website hosting, go to the bucket's details and click on the *Properties* tab. Then, enable the **static website hosting** option. You must provide the **index document** (the default page) and the **error document** (the page to show when there is an error):

![S3 Static Website Hosting](/assets/aws-certified-developer-associate/s3_static_website_hosting.png "S3 Static Website Hosting")

Then, go back to the bucket's details and click on the *Objects* tab. **Upload the website files to the bucket**. By default, index.html and error.html (as you can see from the image above).

**To access the website, find the bucket website endpoint** in the bucket's *Properties* tab, under the *static website hosting* option.

## 8.8 S3 Versioning

You can **version your files in Amazon S3 by enabling the versioning at bucket level** either at creation time (image below) or by updating the bucket's properties.

![S3 Enable Versioning](/assets/aws-certified-developer-associate/s3_enable_versioning.png "S3 Enable Versioning")

**Same key overwrite will change the version** (from 1 to 2, from 2 to 3, ...) and not actually delete the file.

![S3 Versions](/assets/aws-certified-developer-associate/s3_versions.png "S3 Versions")

It is **best practice to version your buckets** because:
- It protects against unintended deletes as you do not actually delete files, you just attach a **delete marker** instead, which gives you the ability to restore a version.
- Of easy roll back to previous versions.

**Notes**:
- Any file that is not versioned prior to enabling versioning will have version *null*.
- Suspending versioning does not delete the previous versions.

If you have versioning enabled, you can **show versions** in the console:

![S3 Show Versions](/assets/aws-certified-developer-associate/s3_show_versions.png "S3 Show Versions")

For example, in the image above, the file index.html has 2 versions (the top one is the latest). You can see the **version ID** near each of its versions (as aforementioned, some versions are *null* because they were not versioned before enabling versioning).

**With versioning enabled, you delete versions not the actual file**. Instead, **if you delete the file, it will just add a delete marker** to the file. You can **delete the delete marker to restore the file**.

## 8.9 S3 Replication

To **enable replication, you must enable versioning in source and destination buckets**.

Two possible replication configurations:
- **Cross-Region Replication (CRR)**: replications across different regions.
- **Same-Region Replication (SRR)**: replication within the same region.

You must give proper IAM permissions to S3 because it needs to be able to read and write from the source and destination buckets. Buckets can be in different AWS accounts and **copying is asynchronous**.

![S3 Replication](/assets/aws-certified-developer-associate/s3_replication.png "S3 Replication")

Use cases:
- CRR: compliance, lower latency access, replication across accounts.
- SRR: log aggregation across multiple buckets, live replication between production and test accounts.

After you enable replication, only new objects are replicated. Optionally, you can replicate existing objects using **S3 Batch Replication**, which **replicates existing objects and objects that failed replication**.

For delete operations, you **can replicate delete markers from the source to the target bucket**. However, **deletions with a version ID (i.e., deleting a precise version) are not replicated to avoid malicious deletes**.

Moreover, there is **no chaining of replication**: if bucket 1 has replication into bucket 2, which has replication into bucket 3, then objects created in bucket 1 are not replicated to bucket 3.

## 8.10 Using Replication in S3

You need at least two buckets with versioning enabled to use replication. To enable replication, go to the **source bucket's details** and click on the *Management* tab. Then, in the *Replication Rules* tab, click on the *Create Replication Rule* button.

Insert the rule's name, status (enabled or disabled), and the priority (if you have multiple rules):

![S3 Replication Rule](/assets/aws-certified-developer-associate/s3_replication_rule.png "S3 Replication Rule")

Then, **configure the source bucket (the current bucket) and configure the scope of the replication**. You can replicate the entire bucket or just objects with a defined prefix:

![S3 Replication Source](/assets/aws-certified-developer-associate/s3_replication_source.png "S3 Replication Source")

Next is the destination bucket. You must first choose between a bucket in the same account or a bucket in another account. Then, you must provide the **destination bucket name** (destination region is automatically selected):

![S3 Replication Destination](/assets/aws-certified-developer-associate/s3_replication_destination.png "S3 Replication Destination")

Then, you must provide the **IAM role** that will be used by S3 to replicate the objects. You can create a new role or use an existing one. You also need to choose whether to replicate objects that are encrypted with AWS Key Management Services keys:

![S3 Replication IAM Role](/assets/aws-certified-developer-associate/s3_replication_iam_role.png "S3 Replication IAM Role")

Finally, some additional replication options are available, such as **replicating delete markers**:

![S3 Replication Options](/assets/aws-certified-developer-associate/s3_replication_options.png "S3 Replication Options")

Upon creation, you will get a message about **replicating existing objects** using **S3 Batch Replication**:

![S3 Replication Message](/assets/aws-certified-developer-associate/s3_replication_message.png "S3 Replication Message")

To test the replication, you can upload a file to the source bucket and check if it is replicated to the destination bucket (it might take some time). Important to notice is that **version ids are replicated as well**.

## 8.11 S3 Durability and Availability

**Durability** (defines how many times an object can be lost by S3):
- High durability (99.999999999%, **11 9s**) of objects across multiple AZ.
- 11 9s means that if you store 10,000,000 objects with S3, you can on average expect to incur a loss of a single object once every 10,000 years.
- Same for all storage classes.

**Availability** (measures how readily available a service is):
- Varies depending on storage class.
- Example: S3 standard has 99.99% availability, which means it is not available about 53 minutes a year.

## 8.12 S3 Storage Classes

The storage class defines how your data is stored and the price you pay for storing it. **S3 provides a range of storage classes**:
- Amazon S3 Standard General Purpose.
- Amazon S3 Standard-Infrequent Access (IA).
- Amazon S3 One Zone-Infrequent Access.
- Amazon S3 Glacier Instant Retrieval.
- Amazon S3 Glacier Flexible Retrieval.
- Amazon S3 Glacier Deep Archive.
- Amazon S3 Intelligent-Tiering.

You can **move between storage classes manually or using S3 lifecycle configurations**.

This image offers a comparison of storage classes (more details at [https://aws.amazon.com/s3/storage-classes/](https://aws.amazon.com/s3/storage-classes/)):

![S3 Storage Classes](/assets/aws-certified-developer-associate/s3_storage_classes.png "S3 Storage Classes")

This image offers a comparison of storage classes pricing (more details at [https://aws.amazon.com/s3/pricing/](https://aws.amazon.com/s3/pricing/)):

![S3 Storage Classes Pricing](/assets/aws-certified-developer-associate/s3_storage_classes_pricing.png "S3 Storage Classes Pricing")

### 8.12.1 S3 Standard General Purpose

It offers 99.99% availability and is **used for frequently accessed data**.

Provides **low latency and high throughput**, and can sustain 2 concurrent facility failures on AWS side.

Use cases: big data analytics, mobile and gaming applications, content distribution, etc.

### 8.12.2 S3 Infrequent Access

It is **used for data that is less frequently accessed (once a month), but requires rapid access when needed**. For example, you need to access your data once a month but still want millisecond access.

It offers lower cost than S3 Standard.

**Amazon S3 Standard-Infrequent Access (S3 Standard-IA)**:
- 99.9% availability.
- Use cases: Disaster recovery and backups.

**Amazon S3 One Zone-Infrequent Access (S3 One Zone-IA)**:
- High durability (99.999999999%) in a single AZ.
- **Data is lost when AZ is destroyed**.
- 99.5% availability.
- Use cases: storing secondary backup copies of on-premises data, or data you can recreate.

### 8.12.3 S3 Glacier

It is **low-cost object storage meant for archiving and backup**.

You are priced for storage and also with an object retrieval cost.

**Amazon S3 Glacier Instant Retrieval**:
- It offers **millisecond retrieval**.
- It is great **for data accessed once a quarter**.
- Minimum storage duration of 90 days.
- Use case: backups but you need to retrieve them quickly.

**Amazon S3 Glacier Flexible Retrieval (formerly Amazon S3 Glacier)**:
- 3 flexibilities:
    - **Expedited**: get the data back in 1 to 5 minutes.
    - **Standard**: get the data back in 3 to 5 hours.
    - **Bulk**: get the data back in 5 to 12 hours, but this is free.
- Minimum storage duration of 90 days.

**Amazon S3 Glacier Deep Archive**:
- It is **meant for long term storage**.
- 2 tiers:
    - **Standard**: get the data back in 12 hours.
    - **Bulk**: get the data back in 48 hours.
- Minimum storage duration of 180 days.

### 8.12.4 S3 Intelligent-Tiering

This class allows you to **move objects automatically between access tiers based on usage**.
- You will be **charged a small monthly monitoring and auto-tiering fee**.
- **No retrieval charges** in S3 Intelligent-Tiering.
- Useful when you do not know your data access patterns.

Tiers:
- **Frequent Access tier (automatic)**: the default tier.
- **Infrequent Access tier (automatic)**: for objects not accessed for 30 days.
- **Archive Instant Access tier (automatic)**: for objects not accessed for 90 days.
- **Archive Access tier (optional)**: configurable from 90 days to 700+ days.
- **Deep Archive Access tier (optional)**: configurable from 180 days to 700+ days.

## 8.13 Using Storage Classes in S3

When uploading a new object to S3, you can choose the storage class for that object. To do so, use the *Properties* tab when uploading the object:

![S3 Storage Class](/assets/aws-certified-developer-associate/s3_storage_class.png "S3 Storage Class")

You can also do the same when the object is already uploaded. Go to the object's details in the *Properties* tab and find the *Storage Class* box. From there you can change the storage class with a UI that is similar to the one above.

![S3 Storage Class after Upload](/assets/aws-certified-developer-associate/s3_storage_class_change_after_upload.png "S3 Storage Class Change after Upload")

## 8.14 Creating a Lifecycle Policy in S3

To create a lifecycle policy, go to the bucket's details and click on the *Management* tab. Then, in the *Lifecycle Rules* tab, click on the *Create lifecycle rule* button.

![S3 Lifecycle Rule](/assets/aws-certified-developer-associate/s3_lifecycle_rule.png "S3 Lifecycle Rule")

First thing is to set the **rule's name and scope (i.e., object's prefix to match or all objects in the bucket)**, applying to all objects in the bucket will require you to check an acknowledgment box:

![S3 Lifecycle Rule Scope](/assets/aws-certified-developer-associate/s3_lifecycle_rule_scope.png "S3 Lifecycle Rule Scope")

Then, you need to set the rule **actions**:

![S3 Lifecycle Rule Actions](/assets/aws-certified-developer-associate/s3_lifecycle_rule_actions.png "S3 Lifecycle Rule Actions")

For example, the actions in the following image will **transition objects to the Standard-IA storage class after 30 days** and **transition objects to the Intelligent-Tiering storage class after 60 days**:

![S3 Lifecycle Rule Actions Example](/assets/aws-certified-developer-associate/s3_lifecycle_rule_actions_example.png "S3 Lifecycle Rule Actions Example")