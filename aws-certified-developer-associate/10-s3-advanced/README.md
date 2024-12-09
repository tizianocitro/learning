## 10 S3 Advanced

## 10.1 Moving between Storage Classes

You can transition objects between storage classes.
- For infrequently accessed object, move them to Standard IA.
- For archive objects that you do not need fast access to, move them to Glacier or Glacier Deep Archive.

Following is the possible transitions:

![S3 Storage Classes Transitions](/assets/aws-certified-developer-associate/s3_storage_classes_transitions.png "S3 Storage Classes Transitions")

**Moving objects can be automated using lifecycle rules**.

## 10.2 Lifecycle Rules

Lifecycle rules can be created for certain:
- **Prefixes**: for example, s3://mybucket/mp3/*.
- **Objects tags**: for example, Department: Finance.

Rules contain **actions**:
- **Transition actions**: configure objects to transition to another storage class.
    - Move objects to Standard IA class 60 days after creation.
    - Move to Glacier for archiving after 6 months.
- **Expiration actions**: configure objects to expire (delete) after some time.
    - Access log files can be set to delete after a 365 days.
    - Can be used to **delete old versions of files (if versioning is enabled)**.
    - Can be used to delete incomplete multi-part uploads (for example, if they are older than 7 days because they are stuck, otherwise they would have already been finished).

All actions are here:

![S3 Lifecycle Rule Actions](/assets/aws-certified-developer-associate/s3_lifecycle_rule_actions.png "S3 Lifecycle Rule Actions")

### 10.2.1 Scenario 1 for Lifecycle Rules

**Question**:

Your application on EC2 creates images thumbnails after profile photos are uploaded to Amazon S3. These thumbnails can be easily recreated, and only need to be kept for 60 days. The source images should be able to be immediately retrieved for these 60 days, and afterwards, the user can wait up to 6 hours. How would you design this?

**Answer**:

- S3 source images can be on Standard, with a lifecycle configuration to transition them to Glacier after 60 days.
- S3 thumbnails can be on One-Zone IA, with a lifecycle configuration to expire them (delete them) after 60 days.

### 10.2.2 Scenario 2 for Lifecycle Rules

**Question**:

A rule in your company states that you should be able to recover your deleted S3 objects immediately for 30 days, although this may happen rarely. After this time, and for up to 365 days, deleted objects should be recoverable within 48 hours.

**Answer**:

- Enable S3 Versioning in order to have object versions, so that deleted objects are in fact hidden by a delete marker and can be recovered.
- Transition the noncurrent versions of the object to Standard IA.
- Transition afterwards the noncurrent versions to Glacier Deep Archive.

## 10.3 S3 Analytics

S3 Analytics provides insights into storage access patterns and can be used to optimize storage class and lifecycle policies.

It helps you decide when to transition objects to the right storage class by providing **recommendations for Standard and Standard IA but does not work for One-Zone IA or Glacier**. It is a good first step to put together lifecycle rules (or improve them).

The report is updated daily but it can take from 24 to 48 hours to start seeing data analysis.

![S3 Analytics](/assets/aws-certified-developer-associate/s3_analytics.png "S3 Analytics")

## 10.4 Creating Lifecycle Rules

You can have **multiple actions in a single lifecycle rule**.

![S3 Lifecycle with Multiple Rule Actions](/assets/aws-certified-developer-associate/s3_lifecycle_with_multiple_rule_actions.png "S3 Lifecycle with Multiple Rule Actions")

The console will show you a summary of how the rule will be applied:

![S3 Lifecycle Rule Summary](/assets/aws-certified-developer-associate/s3_lifecycle_rule_summary.png "S3 Lifecycle Rule Summary")

### 10.4.1 Move Current Versions of Objects between Storage Classes

This was shown in the previous S3 section. For example, the actions in the following image will **transition objects to the Standard-IA storage class after 30 days** and **transition objects to the Intelligent-Tiering storage class after 60 days**:

![S3 Lifecycle Rule Actions Example 1](/assets/aws-certified-developer-associate/s3_lifecycle_rule_actions_example.png "S3 Lifecycle Rule Actions Example 1")

### 10.4.2 Move Non-Current Versions of Objects between Storage Classes

This is useful when versioning is enabled. For example, the actions in the following image will **transition non-current versions to the Glacier Flexible Retrieval storage class after 90 days**:

![S3 Lifecycle Rule Actions Example 2](/assets/aws-certified-developer-associate/s3_lifecycle_rule_actions_example2.png "S3 Lifecycle Rule Actions Example 2")

### 10.4.3 Expire Current Versions of Objects

This is useful when you want to delete objects after a certain period of time. For example, the actions in the following image will **expire objects/delete after 700 days**:

![S3 Lifecycle Rule Actions Example 3](/assets/aws-certified-developer-associate/s3_lifecycle_rule_actions_example3.png "S3 Lifecycle Rule Actions Example 3")

### 10.4.4 Permanently Delete Non-Current Versions of Objects

This is useful when you want to delete non-current versions of objects after a certain period of time. For example, the actions in the following image will **permanently delete non-current versions after 700 days but you can choose to retain a certain number of versions**:

![S3 Lifecycle Rule Actions Example 4](/assets/aws-certified-developer-associate/s3_lifecycle_rule_actions_example4.png "S3 Lifecycle Rule Actions Example 4")

## 10.5 S3 Event Notifications

**S3 can publish events** (e.g. new object created). For example:
- S3:ObjectCreated.
- S3:ObjectRemoved.
- S3:ObjectRestore.
- S3:Replication.

It is also possible to **filter events based on prefixes and suffixes** (e.g., **.jpg*).

You can **create as many S3 events as desired and send them to different targets (SNS, SQS, Lambda, Event Bridge) that can react to them**. S3 event notifications typically deliver events in seconds but can sometimes take a minute or longer.

![S3 Event Notifications](/assets/aws-certified-developer-associate/s3_event_notifications.png "S3 Event Notifications")

**Use case**: generate thumbnails of images uploaded to S3.

### 10.5.1 IAM Permissions for S3 Event Notifications

You need to create and attach **resource access policies** to the AWS services you want to use (e.g., Lambda, SQS, SNS) to allow S3 to send events to them.

![S3 Event Notifications IAM Permissions](/assets/aws-certified-developer-associate/s3_event_notifications_iam_permissions.png "S3 Event Notifications IAM Permissions")

**Using EventBridge as target for S3 events you can also send S3 events to other 18 AWS services**.

![S3 Event Notifications EventBridge](/assets/aws-certified-developer-associate/s3_event_notifications_eventbridge.png "S3 Event Notifications EventBridge")

Thanks to EventBridge, you also get:
- **Advanced filtering options with JSON rules**: metadata, object size, name, and more.
- **Multiple destinations**: e.g., Step Functions, Kinesis Streams/Firehose, and more.
- **EventBridge capabilities**: for example, the possibility to archive and replay Events, or the reliable delivery.

## 10.6 Using Event Notifications

Create a bucket to proceed. Go to the bucket, click on *Properties*, and then scroll down to *Event notifications*:

![S3 Event Notifications Create](/assets/aws-certified-developer-associate/s3_event_notifications_create.png "S3 Event Notifications Create")

As you can see, you have **two options: create an event notification** or **enable Amazon EventBridge integration (to send all the events from the S3 bucket to EventBridge)**.

### 10.6.1 Enabling Amazon EventBridge Integration

To **enable Amazon EventBridge integration**, just set it to *on*:

![S3 Event Notifications EventBridge Integration](/assets/aws-certified-developer-associate/s3_event_notifications_eventbridge_integration.png "S3 Event Notifications EventBridge Integration")

### 10.6.2 Creating an Event Notification

To **create an event notification**, click on *Create event notification*. Add a name and optionally the prefix and suffix:

![S3 Event Notifications Create Event Notification](/assets/aws-certified-developer-associate/s3_event_notifications_create_event_notification.png "S3 Event Notifications Create Event Notification")

Then, select the **event types** you want to be notified about (e.g., object creation, object removal, object restore, object tagging, ...), you can also be granular:

![S3 Event Notifications Event Types](/assets/aws-certified-developer-associate/s3_event_notifications_event_types.png "S3 Event Notifications Event Types")

Then, select the **destination** (e.g., SNS, SQS, Lambda), in this case an SQS queue:

![S3 Event Notifications Destination](/assets/aws-certified-developer-associate/s3_event_notifications_destination.png "S3 Event Notifications Destination")

So, we need to **create an SQS queue** (more in following sections). Go to *Services*, *SQS*. and then click *Create Queue*:

![S3 Events SQS Create Queue](/assets/aws-certified-developer-associate/s3_events_sqs_create_queue.png "S3 Events SQS Create Queue")

Then, go into *Access Policy* and **enhance permissions to the policy** (for example, using the policy generator) **to allow S3 to send messages to the queue** (*SendMessage* action):

![S3 Events SQS Access Policy](/assets/aws-certified-developer-associate/s3_events_sqs_access_policy.png "S3 Events SQS Access Policy")

Otherwise, we will get an error when trying to create the event notification because the SQS queue does not allow S3 to send messages to it:

![S3 Events SQS Error](/assets/aws-certified-developer-associate/s3_events_sqs_error.png "S3 Events SQS Error")

After setting everything up, go to the SQS queue and **poll for messages in the queue**:

![S3 Events SQS Poll Messages](/assets/aws-certified-developer-associate/s3_events_sqs_poll_messages.png "S3 Events SQS Poll Messages")

You will see that **S3 has sent a message with a test event to the SQS queue**:

![S3 Events SQS Message](/assets/aws-certified-developer-associate/s3_events_sqs_message.png "S3 Events SQS Message")

To test that events are correctly sent to the SQS queue, **upload a file to the S3 bucket** because we set the event notification to trigger on object creation. **Poll again for messages in the SQS queue and you will see that S3 has sent a message with the object creation event** (you can see the name of the uploaded object in the message):

![S3 Events SQS Message Object Creation](/assets/aws-certified-developer-associate/s3_events_sqs_message_object_creation.png "S3 Events SQS Message Object Creation")

## 10.7 S3 Performance

### 10.7.1 S3 Baseline Performance

S3 automatically scales to high request rates with low latency of around 100/200 ms to get the first byte out of S3 (**time to first byte**).

Your application can achieve **at least 3,500 PUT/COPY/POST/DELETE** or **5,500 GET/HEAD requests per second per prefix** in a bucket.

There are **no limits to the number of prefixes in a bucket**.

Consider the following example (object path -> prefix):
1. bucket/folder1/sub1/file -> /folder1/sub1/
2. bucket/folder1/sub2/file -> /folder1/sub2/
3. bucket/1/file            -> /1/
4. bucket/2/file            -> /2/

If you spread reads across all four prefixes evenly, you can achieve 22,000 requests per second for GET and HEAD, and 14,000 requests per second for PUT, COPY, POST, and DELETE.

### 10.7.2 S3 Performance Optimization for Uploads

To optimize performance, you can use the following features/strategies.

**Multi-part upload**: it can help parallelize uploads (speed up transfers) as files are uploaded in parts and then S3 is smart enough to reconstruct them into a single file.
- Recommended for files bigger than 100MB.
- Must be used for files bigger than 5GB.

![S3 Multi-Part Upload](/assets/aws-certified-developer-associate/s3_multi_part_upload.png "S3 Multi-Part Upload")

**S3 Transfer Acceleration**: increases transfer speed by transferring file to an AWS edge location which will forward the data to the S3 bucket in the target region.
- Compatible with multi-part upload.
- It reduces the amount of time we use the public internet to transfer files to take advantage of AWS's fast private network.
- For example, you have a file in the US and you want to upload it to an S3 bucket in Europe. **The file will be uploaded to the edge location in the US via the public internet and then transferred to the S3 bucket in Europe via AWS's fast private network**.

![S3 Transfer Acceleration](/assets/aws-certified-developer-associate/s3_transfer_acceleration.png "S3 Transfer Acceleration")

### 10.7.3 S3 Performance Optimization for Downloads (S3 Byte-Range Fetches)

**S3 Byte-Range Fetches** allows you to parallelize GETs by requesting specific byte ranges.

They provide **better resilience in case of failures** because you can retry only the failed byte ranges, which is more efficient than retrying the whole file.

You can use it to **speed up downloads by making parallel requests for different parts of the same object**.

![S3 Byte-Range Fetches](/assets/aws-certified-developer-associate/s3_byte_range_fetches.png "S3 Byte-Range Fetches")

You can use it to **retrieve only partial data** (e.g., the head of a file)

![S3 Byte-Range Fetches Partial Data](/assets/aws-certified-developer-associate/s3_byte_range_fetches_partial_data.png "S3 Byte-Range Fetches Partial Data")

## 10.8 S3 User-defined Object Metadata and S3 Objects Tags

**S3 User-Defined Object Metadata**:
- When uploading an object, you can also assign metadata.
- They are **name-value (key-value) pairs**.
- User-defined metadata names **must begin with "x-amz-meta-"** becuase without this prefix, the metadata will be treated as AWS-added metadata.
- S3 stores user-defined metadata keys in lowercase.
- Metadata can be retrieved while retrieving the object.

**S3 Object Tags**:
- **Key-value pairs** for objects in S3.
- **Useful for fine-grained permissions (only give access to specific objects with specific tags)**.
- **Useful for analytics purposes** (using S3 Analytics to group by tags).

You **cannot search the object metadata or object tags**. Instead, **you must use an external DB as a search index**, such as DynamoDB.

**Important for the exam**: if you want to search your S3 buckets, you must put all tags and metadata into a searcheable index in DynamoDB. Then perform searches on DynamoDB and the results of the searches will be extracted as objects from S3.

![S3 User-Defined Object Metadata and S3 Objects Tags](/assets/aws-certified-developer-associate/s3_user_defined_object_metadata_and_s3_objects_tags.png "S3 User-Defined Object Metadata and S3 Objects Tags")

**Example question**:

You are looking to build an index of your files in S3, using Amazon RDS PostgreSQL. To build this index, it is necessary to read the first 250 bytes of each object in S3, which contains some metadata about the content of the file itself. There are over 100,000 files in your S3 bucket, amounting to 50 TB of data. how can you build this index efficiently?

**Answer**:

Create an application that will traverse the S3 bucket, issue a Byte-Range Fetch for the first 250 bytes of each object, and then write this data to Amazon RDS PostgreSQL.