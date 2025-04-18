# 16 Integration and Messaging: Simple Queue Service, Simple Notification Service, and Kinesis

This section is really important because **the exam asks a lot of questions on these services, particularly SQS**.
When we start deploying multiple applications, they will inevitably need to communicate with one another. There are **two patterns of application communication**:

- **Synchronous communication**: application A sends a request to application B and waits for a response.
- **Asynchronous/event-based communication**: application A sends a message to a queue and application B processes the message from the queue.

![Two Patterns](/assets/aws-certified-developer-associate/two_communication_patterns.png "Two Patterns")

**Synchronous communication between applications can be problematic if there are sudden spikes of traffic** and **one application could overwhelm the other**. For example, suddenly there is a need to encode 1000 videos but usually it is 10, the encoding application could be overwhelmed and outages could occur.

To avoid this issue, we can **decouple our applications** using:
- **Simple Queue Service (SQS)** for queue model.
- **Simple Notification Service (SNS)** for pub/sub model.
- **Kinesis** for real-time streaming model.

The important features is that these services can scale independently from our application.

## 16.1 Simple Queue Service (SQS)

**Simple Queue Service (SQS)** is a fully managed message queuing service that provides a distributed queue system that enables web service components to communicate in a loosely coupled way.

An **SQS queue is a buffer** between the application components that receive data and the components that process the data.
With SQS queues, you have:
- **Producers**: one or more applications that send messages to the queue.
- **Consumers**: one or more applications that poll and process messages from the queue.

![SQS Queues](/assets/aws-certified-developer-associate/sqs_queues.png "SQS Queues")

## 16.2 SQS Standard Queues

It is the oldest offering of SQS (over 10 years ago) and it is the default queue type.

It is a **fully managed service used to decouple applications**, so think of SQS whenever you read "decoupling" during the exam.

**Attributes** of SQS Standard Queues:
- **Unlimited throughput and unlimited number of messages** in a queue.
- Default retention of messages: 4 days, up to a maximum of 14 days.
- **Low latency**: <10 ms on publish and receive.
- Limitation of 256KB per message sent.
- **At least once delivery**: can have duplicate messages, occasionally. For example, if a consumer does not process a message fast enough. Another offering of SQS, FIFO queues, guarantees exactly once delivery to solve this issue.
- **Best effort ordering**: can have out of order messages.

## 16.3 Producing Messages to SQS

Messages are produced to SQS using the SDK or API via the `SendMessage` API.

A **message is persisted in SQS until a consumer reads and deletes it** to indicate that the message has been processed.

For example, you may send a message to process an order, including the order ID, customer ID, and the items to be processed. The consumer will read the message, process the order, and then delete the message from the queue.

## 16.4 Consuming Messages from SQS

Consumers, which can run on EC2 instances, servers, or AWS Lambda, do:
1. **Poll SQS for messages**: they can **receive up to 10 messages at a time**.
2. **Process the messages**: for example, they can store the content of each message into an RDS database.
3. **Delete the messages**: they use the `DeleteMessage` API to indicate that the message has been processed, thus **guaranteeing that no other consumer will process the same message**.

![SQS Consumers](/assets/aws-certified-developer-associate/sqs_consumers.png "SQS Consumers")

### 16.4.1 Multiple EC2 Instances Consumers

An SQS queue can scale by having multiple consumers:
- These **consumers receive and process messages in parallel**.
- If somehow one consumer does not process a message fast enough, another consumer can process the message. Thus, you have at least once delivery and best effort ordering.
- Consumers delete messages after processing them to avoid other consumers seeing the same messages.
- We can **scale consumers horizontally to improve throughput of processing**.

![Multiple EC2 Instances Consumers](/assets/aws-certified-developer-associate/sqs_multiple_ec2_instances_consumers.png "Multiple EC2 Instances Consumers")

Given that we can scale consumers horizontally, we can **combine SQS with an ASG to scale the number of EC2 instances based on the number of messages in the queue** (queue's lenght: `ApproximateNumberOfMessages` CloudWatch metric):

![Auto Scaling Group with SQS](/assets/aws-certified-developer-associate/sqs_auto_scaling_group_with_sqs.png "Auto Scaling Group with SQS")

**SQS with ASG is a common architecture pattern in the exam**.

## 16.5 SQS to Decouple between Application Tiers

Consider having an application that processes videos comprising two tiers: a front-end tier (which handles user interaction) and a back-end tier (which processes videos).

We can decouple these two tiers to avoid making the front-end tier wait for the back-end tier to process the videos because video processing can take a long time.

![SQS to Decouple between Application Tiers](/assets/aws-certified-developer-associate/sqs_decouple_between_application_tiers.png "SQS to Decouple between Application Tiers")

**This usage of SQS is important to know for the exam**.

## 16.6 SQS Security

**Encryption**:
- In-flight encryption: using HTTPS APIs.
- At-rest encryption: using SQS keys (SSE-SQS) or KMS keys.
- Client-side encryption: if the client wants to perform encryption/decryption itself but it is not native to SQS.

**Access controls**: IAM policies to regulate access to SQS APIs.

**SQS access policies** are similar to S3 bucket policies:
- Useful for cross-account access to SQS queues.
- Useful for allowing other services (e.g., SNS, S3, etc.) to write to an SQS queue.

## 16.7 Creating a Standard SQS Queue

To create an SQS queue, go to the SQS service in the console and click on *Create Queue*.

First things to do are selectin the **type (in this case queue)** and entering the **name**:

![SQS Queue Name and Type](/assets/aws-certified-developer-associate/sqs_queue_name.png "SQS Queue Name and Type")

Some **configuration options**: visibility timeout, message retention period, delivery delay, etc.

![SQS Queue Configuration](/assets/aws-certified-developer-associate/sqs_queue_configuration.png "SQS Queue Configuration")

Next are **encryption** settings (in this case SSE-SQS):

![SQS Queue Encryption](/assets/aws-certified-developer-associate/sqs_queue_encryption.png "SQS Queue Encryption")

Then, you can setup the **access policy**, which is a JSON document. You can define it in the editor (`Basic`) or use the policy generator (`Advanced`). The **policy defines**:
- Who can send messages to the queue.
- Who can receive messages from the queue.

![SQS Queue Access Policy](/assets/aws-certified-developer-associate/sqs_queue_access_policy.png "SQS Queue Access Policy")

Finally, you can configure some **advanced options** (which will be discussed in following sections) and **tags**:

![SQS Queue Advanced Options and Tags](/assets/aws-certified-developer-associate/sqs_queue_advanced_options_tags.png "SQS Queue Advanced Options and Tags")

Once the queue is created, if you have messages on it, you can delete all of them by clicking on *Purge*. You also have several tabs with information about the queue:

![SQS Queue Tabs](/assets/aws-certified-developer-associate/sqs_queue_tabs.png "SQS Queue Tabs")

### 16.7.1 Sending and Receiving Messages

To send a message to the queue, click on *Send and Receive Messages*:

![SQS Send and Receive Messages](/assets/aws-certified-developer-associate/sqs_send_and_receive_messages.png "SQS Send and Receive Messages")

From there, you can **send a message** to the queue by typing the content of the message and clicking on *Send Message*:

![SQS Send Message](/assets/aws-certified-developer-associate/sqs_send_message.png "SQS Send Message")

Then, you will notice that the message will be counted in the *Messages Available* section and to **receive the message**, click on *Poll for Messages*:

![SQS Poll for Messages](/assets/aws-certified-developer-associate/sqs_poll_for_messages.png "SQS Poll for Messages")

To **access the content of the message**, click on the message ID. Metadata will be under *Details* and the content of the message (`Hello world!` in the example above) will be under *Body*:

![SQS Message Content](/assets/aws-certified-developer-associate/sqs_message_content.png "SQS Message Content")

The *Receive Count* attribute indicates how many times a message has been received. If a message is received more than once, it is because it was not deleted after processing. To delete a message, select it and click on *Delete*.

## 16.8 SQS Queue Access Policy

An SQS queue access policy is a **JSON document that you attach to a queue and defines who can send messages to the queue and who can receive messages from the queue**.

There are **two good use cases**: cross-account access and allowing other services to write to an SQS queue (e.g., publish S3 event notifications to SQS queue).

**The two use cases described below will be tested in the exam**.

### 16.8.1 Cross-Account Access

This is useful **when you have a resource** (e.g., an EC2 instance) **in one account that needs to send messages to an SQS queue in another account**.

![Cross-Account Access](/assets/aws-certified-developer-associate/sqs_cross_account_access.png "Cross-Account Access")

### 16.8.2 Publish S3 Event Notifications to SQS Queue

This can be useful when you want to **process S3 events** (e.g., object created, object deleted), but you want to **decouple the processing by sending the events to an SQS queue**. So, any resource (e.g., Lambda) that needs to process the events can read from the queue.

![S3 Event Notifications to SQS Queue](/assets/aws-certified-developer-associate/sqs_s3_event_notifications_to_sqs_queue.png "S3 Event Notifications to SQS Queue")

## 16.9 Using an SQS Access Policy

To do so, create a policy and reach the **access policy** section. You can define the policy in the editor (`Basic`) or use the policy generator (`Advanced`). In the `Basic` editor, you can select options like this:

![SQS Queue Access Policy](/assets/aws-certified-developer-associate/sqs_queue_access_policy.png "SQS Queue Access Policy")

While in the `Advanced` editor, you can define the policy in JSON format:

![SQS Queue Access Policy JSON](/assets/aws-certified-developer-associate/sqs_queue_access_policy_json.png "SQS Queue Access Policy JSON")

### 16.9.1 Sending S3 Event Notifications to SQS Queue

Create a bucket named `demo-sqs-queue-access-policy`. Then, go to the bucket and click on *Properties* > *Event Notification* > *Create Event Notification*, and **select the SQS queue as the destination of the event notification**:

![S3 Event Notifications to SQS Queue](/assets/aws-certified-developer-associate/s3_event_notifications_to_sqs_queue.png "S3 Event Notifications to SQS Queue")

However, when you try to save the event notification, you will get an **error because the queue access policy that you see above does not allow the bucket to send messages** to the SQS queue.

To better see the queue access policy defined above, here it is again:

![SQS Queue Access Policy Better View](/assets/aws-certified-developer-associate/sqs_queue_access_policy_better_view.png "SQS Queue Access Policy Better View")

To allow S3 to send messages to the SQS queue, you need to update the policy to follow what is indicated in [16.8.2 Publish S3 Event Notifications to SQS Queue](#1682-publish-s3-event-notifications-to-sqs-queue):

![SQS Queue Access Policy Allow S3](/assets/aws-certified-developer-associate/sqs_queue_access_policy_allow_s3.png "SQS Queue Access Policy Allow S3")

Now, you can save the event notification and the bucket will be able to send messages to the SQS queue. You can see the messages sent to the queue, for example:

![SQS Queue Messages from S3](/assets/aws-certified-developer-associate/sqs_queue_messages_from_s3.png "SQS Queue Messages from S3")

## 16.10 SQS Message Visibility Timeout

![SQS Message Visibility Timeout](/assets/aws-certified-developer-associate/sqs_message_visibility_timeout.png "SQS Message Visibility Timeout")

- **After a message is polled by a consumer, it becomes invisible to other consumers**. This is where the **message visibility timeout starts**.
- By **default**, the message visibility timeout is 30 seconds (up to 12 hours), which means the message has 30 seconds to be processed.
- After the message visibility timeout is over, if the message has not been deleted, it becomes visible again in SQS.

If a **message is not processed within the visibility timeout, it will be processed twice**. A consumer can call the `ChangeMessageVisibility` API to **get more time** if it needs more to process a message.

**Configuring the visibility timeout** is important because if visibility timeout is:
- **High (hours)**: if the consumer crashes, re-processing will take a long time.
- **Low (seconds)**: we may get duplicates because consumers might not have enough time to process the message.

The visibility timeout can be configured when creating or editing a queue:

![SQS Queue Configuration](/assets/aws-certified-developer-associate/sqs_queue_configuration.png "SQS Queue Configuration")

## 16.11 SQS Dead Letter Queue (DLQ)

Consider that a consumer fails to process a message within the visibility timeout. Then,  the message goes back to the queue. If this happens multiple times, the message will keep going back to the queue and will never be processed, it will be stuck in the queue.

We can **set a threshold of how many times a message can go back to the queue**. After the `MaximumReceives` threshold is exceeded, the message goes into a dead letter queue. A **dead letter queue is a queue for messages that could not be processed** and is useful for debugging.

![SQS Dead Letter Queue](/assets/aws-certified-developer-associate/sqs_dead_letter_queue.png "SQS Dead Letter Queue")

Important to **keep in mind**:
- A DLQ of a FIFO queue must also be a FIFO queue.
- A DLQ of a standard queue must also be a standard queue.
- Ensure to process the messages in a DLQ before they expire. It is good to set long retention times (e.g., 14 days) for messages in the DLQ.

### 16.11.1 SQS Redrive to Source

Feature to help consume messages (for manual inspection and debugging) in a DLQ to understand what is wrong with them.

When our code is fixed, we can **redrive messages from a DLQ back into the source queue (or any other queue) in batches without writing custom code**. Now the consumers can process the messages successfully without even knowing that the messages were in the DLQ.

![SQS Redrive to Source](/assets/aws-certified-developer-associate/sqs_redrive_to_source.png "SQS Redrive to Source")

## 16.12 Using a Dead Letter Queue

Create a queue like in [16.7 Creating a Standard SQS Queue](#167-creating-a-standard-sqs-queue) but be sure to **set a long retention time**, such as 14 days. For example, call this queue `DemoQueueDLQ`.

Then, go to the queue you want to indicate a DLQ for (e.g., `DemoQueue`) and edit it to find the *Dead-letter Queue* section. In this section, you can:
- **Enable the DLQ**, which is disabled by default.
- **Select the queue to use as DLQ** (e.g., the `DemoQueueDLQ` created before) for the current queue.
- **Set the number of times a message can be received before going to the DLQ** (e.g., 10 times in the image below).

![SQS Enable Dead Letter Queue](/assets/aws-certified-developer-associate/sqs_enable_dead_letter_queue.png "SQS Enable Dead Letter Queue")

To **test that it is working**, you can send a message to the `DemoQueue` queue and then poll for the message from the queue. If you do not delete the message and let the visibility timeout expire, it will go back to the queue and be received again. After 10 times, the message will go to the `DemoQueueDLQ` dead letter queue and will not be polled anymore by the queue.

### 16.12.1 Redriving Messages from a Dead Letter Queue

To **redrive messages from a DLQ back into the source queue**, go to the DLQ queue (e.g., `DemoQueueDLQ`) and click on *Start DLQ Redrive*:

![SQS Start DLQ Redrive](/assets/aws-certified-developer-associate/sqs_start_dlq_redrive.png "SQS Start DLQ Redrive")

Then, you can **select to redrive to the source queue**:

![SQS Redrive to Source Queue](/assets/aws-certified-developer-associate/sqs_redrive_to_source_queue.png "SQS Redrive to Source Queue")

You can also inspect the messages in DLQ before redriving them, or eventually send them to another queue via the **redrive to custom destination** option.

Finally, click on *DLQ Redrive* to start the **redrive task**, which you can also inspect in the *Dead-letter Queue Redrive Tasks* tab:

![SQS DLQ Redrive Task](/assets/aws-certified-developer-associate/sqs_dlq_redrive_task.png "SQS DLQ Redrive Task")

## 16.13 Delay in SQS Queues

In SQS, you can **delay a message up to 15 minutes**, so that consumers do not see it immediately. The **default** is 0 seconds, so the message is available right away.

You can **set a default at queue level** using the `Delivery Delay` setting:

![SQS Delivery Delay at Queue Level](/assets/aws-certified-developer-associate/sqs_queue_configuration.png "SQS Delivery Delay at Queue Level")

Or can **override the default when sending the message** using the `DelaySeconds` parameter. In the console, you can set the delay when sending a message using the `Delivery Delay` setting:

![SQS Delivery Delay when Sending Message](/assets/aws-certified-developer-associate/sqs_send_message.png "SQS Delivery Delay when Sending Message")

## 16.14 SQS Long Polling

Long polling is when **a consumer requests messages from SQS but there are no messages in the queue, so the consumer can optionally wait for a message to arrive in the queue**. When the message arrives, it is sent to the consumer.
- The wait time can be between 1 second to 20 seconds but 20 seconds is preferable.
- It is preferable to short polling.
- It can be **enabled at queue level or API level** using the `ReceiveMessageWaitTimeSeconds` parameter.

You can **set long polling at queue level** using the `Receive Message Wait Time` setting:

![SQS Receive Message Wait Time at Queue Level](/assets/aws-certified-developer-associate/sqs_queue_configuration.png "SQS Receive Message Wait Time at Queue Level")

Long polling is **useful because it decreases the number of API calls made to SQS while increasing the efficiency and decreasing the latency of your application**. Use it when:
- A consumer is making too many requests to SQS.
- Your application has too much latency.

## 16.15 SQS Extended Client

The message size limit is 256KB but you may need to send larger messages (e.g. 1GB). To do so, you can use the **SQS Extended Client**, which is a Java library that allows you to send large messages to SQS.

This library does something very simple: it **uploads the message to S3 and sends a reference to the message in SQS**. The consumer will then retrieve the message from S3 using the reference in the message it consumes.

![SQS Extended Client](/assets/aws-certified-developer-associate/sqs_extended_client.png "SQS Extended Client")

## 16.16 SQS Must Know APIs

This table describes APIs that is important to know for the exam.

| API | Description |
| --- | ----------- |
| `CreateQueue` | Creates a new queue. You can use the `MessageRetentionPeriod` parameter to set the retention period of messages in the queue. |
| `DeleteQueue` | Deletes the queue and all the messages in the queue. |
| `PurgeQueue` | Deletes all the messages in the queue but not the queue itself. |
| `SendMessage` | Sends a message to the queue. You can use the `DelaySeconds` parameter to set the delay of the message. |
| `ReceiveMessage` | Receives messages from the queue (polling). You can use the **1.** `ReceiveMessageWaitTimeSeconds` parameter to set the long polling. Use the **2.** `MaxNumberOfMessages` parameter to receive up to 10 messages at a time. By default, it receives 1 message. |
| `DeleteMessage` | Deletes a message from the queue. |
| `ChangeMessageVisibility` | Changes the message visibility timeout in case the consumer needs more time to process the message. |

You can **use batch APIs** for `SendMessage`, `DeleteMessage`, and `ChangeMessageVisibility`, which helps decrease your costs.

## 16.17 SQS FIFO Queues

FIFO stands for First In First Out and corresponds to the ordering of messages in the queue that is guaranteed by these queues.

![SQS FIFO Queues](/assets/aws-certified-developer-associate/sqs_fifo_queues.png "SQS FIFO Queues")

FIFO queues have:
- **Limited throughput**:
    - 300 msg/s without batching.
    - 3000 msg/s with batching.
- **Exactly-once send capability**: duplicates are removed using a `MessageDeduplicationID` parameter.
- **Messages are processed in order by the consumer**: following the FIFO principle.
- **Ordering by `MessageGroupID`**: all messages in the same group are ordered and `MessageGroupID` is a mandatory parameter that you need to send every time you send a message on a FIFO queue.

### 16.17.1 Creating a FIFO SQS Queue

It is almost the same as creating a standard queue, but you need to select the `FIFO` queue type and **give a name that ends with `.fifo`**:

![SQS FIFO Queue Type](/assets/aws-certified-developer-associate/sqs_fifo_queue_type.png "SQS FIFO Queue Type")

You can also set the **content-based deduplication** feature, which is useful to avoid duplicates:

![SQS FIFO Queue Content-Based Deduplication](/assets/aws-certified-developer-associate/sqs_fifo_queue_content_based_deduplication.png "SQS FIFO Queue Content-Based Deduplication")

### 16.17.2 Sending Messages to a FIFO SQS Queue

When sending messages to a FIFO queue, you need to set the `MessageGroupId` and `MessageDeduplicationID` (this is mandatory only if content-based deduplication is not enabled) parameter to ensure that messages are ordered:

![SQS FIFO Queue Send Message](/assets/aws-certified-developer-associate/sqs_fifo_queue_send_message.png "SQS FIFO Queue Send Message")

If you send and then poll messages from this FIFO queue, you will see that they are ordered in a FIFO order.

## 16.18 FIFO SQS Queues Deduplication

There is a **deduplication interval of 5 minutes**, meaning that if you send the same message twice during this interval, the second message will be discarded.

**Two deduplication methods**:
1. Content-based deduplication: builds a `MessageDeduplicationID` by doing a SHA-256 hash of the message body to check for duplicates.
2. Explicitly provide a `MessageDeduplicationID` to uniquely identify a message, so that if the same deduplication ID is sent again, it will be discarded.

For example, with content-based deduplication:

![SQS FIFO Queue Content-Based Deduplication Example](/assets/aws-certified-developer-associate/sqs_fifo_queue_content_based_deduplication_example.png "SQS FIFO Queue Content-Based Deduplication Example")

## 16.19 FIFO SQS Message Grouping

If you specify the same value of `MessageGroupID` in an SQS FIFO queue, you can only have one consumer, and all the messages are in order for that one consumer.

**Specify different values for `MessageGroupID` to get ordering at the level of a subset of messages**:
- Messages that share a common `MessageGroupID` will be in order within the group.
- Ordering across groups is not guaranteed.
- Each `MessageGroupID` can have a different consumer, which **enables parallel processing**.

![SQS FIFO Queue Message Grouping](/assets/aws-certified-developer-associate/sqs_fifo_queue_message_grouping.png "SQS FIFO Queue Message Grouping")


## 16.20 Simple Notification Service (SNS)

SNS providers support for **push-based delivery of messages (pub/sub)**. It is a fully managed service that allows you to send messages to a large number of subscribers through topics.

![SNS Pub/Sub](/assets/aws-certified-developer-associate/sns_pub_sub.png "SNS Pub/Sub")

In SNS:
1. The **event producer** only sends message to one SNS topic.
2. As many **event receivers (subscribers)** as we want can listen to the SNS topic notifications.
3. Each subscriber to the topic will get all the messages (except if we use the feature to filter messages).

SNS offers up to 12,500,000 subscriptions per topic with a maximum of 100,000 topics.

With SNS you can **send messages to many destinations**:

![SNS Destinations](/assets/aws-certified-developer-associate/sns_destinations.png "SNS Destinations")

At the same time, you can **receive messages from many sources** as many AWS services can send data directly to SNS for notifications:

![SNS Sources](/assets/aws-certified-developer-associate/sns_sources.png "SNS Sources")

### 16.20.1 How to Publish on SNS

Topic publishing using the SDK:
- Create a topic.
- Create a subscription (or many).
- Publish to the topic and all subscribers will receive the message.

Direct publishing for mobile applications SDK:
- Create a platform application.
- Create a platform endpoint.
- Publish to the platform endpoint.
- Works with Google GCM, Apple APNS, Amazon ADM, etc.

### 16.20.2 SNS Security

**Encryption**:
- In-flight encryption: using HTTPS APIs.
- At-rest encryption: using KMS keys.
- Client-side encryption: if the client wants to perform encryption/decryption itself but it is not native to SNS.

**Access controls**: IAM policies to regulate access to SNS APIs.

**SNS access policies** are similar to S3 bucket policies:
- Useful for cross-account access to SNS topics.
- Useful for allowing other services (e.g., S3, etc.) to write to an SNS topic.

## 16.21 Fan Out Using SQS and SNS

With this pattern, you push once in SNS and receive in all SQS queues that are subscribed to the SNS topic.
- This is a **common pattern where only one message is sent to an SNS topic and then fan-out to multiple SQS queues**.

![Fan Out Using SQS and SNS](/assets/aws-certified-developer-associate/fan_out_using_sqs_and_sns.png "Fan Out Using SQS and SNS")

In this way, you can have:
1. A **fully decoupled model with no data loss** because:
    - SQS allows for data persistence, delayed processing and retries of work.
    - You can add more SQS subscribers over time.
    - The other queues can still process messages if one queue fails.
2. **Cross-region delivery**: the SNS topic can send messages to SQS queues in other regions.

For this pattern to work, you need to make sure that your **SQS queue access policy allows for SNS to write** to it.

### 16.21.1 Fan Out For S3 Events to Multiple Queues

You can have only one S3 event rule for the same combination of event type and prefix. If you want **to send the same S3 event to many SQS queues, use fan out**.

With fan out, you can have one S3 event rule that sends the event to an SNS topic and then have many SQS queues (but also other destinations, e.g., a Lambda function) subscribed to the SNS topic to receive the S3 event:

![Fan Out For S3 Events to Multiple Queues](/assets/aws-certified-developer-associate/fan_out_for_s3_events_to_multiple_queues.png "Fan Out For S3 Events to Multiple Queues")

### 16.21.2 Fan Out For SNS to S3 through Kinesis Data Firehose

SNS has direct integration with Kinesis Data Firehose, so SNS can send to Kinesis and then Kinesis can send to S3 (or any supported destination for Kinesis Data Firehose):

![Fan Out For SNS to S3 through Kinesis Data Firehose](/assets/aws-certified-developer-associate/fan_out_for_sns_to_s3_through_kinesis_data_firehose.png "Fan Out For SNS to S3 through Kinesis Data Firehose")

## 16.22 SNS FIFO Topics

**SNS FIFO topics are similar in features to SQS FIFO queues**:
- Ordering by `MessageGroupID`: all messages in the same group are ordered.
- Deduplication using a `DeduplicationI`D or content-based deduplication.
- The name of a SNS FIFO topic must end in `.fifo`.

![SNS FIFO Topics](/assets/aws-certified-developer-associate/sns_fifo_topics.png "SNS FIFO Topics")

**SNS FIFO topics can have SQS standard and FIFO queues as subscribers**.

SNS FIFO topics have **limited throughput** (same throughput as SQS FIFO).

### 16.22.1 Combining SNS FIFO Topics and SQS FIFO Queues

By combining SNS FIFO topics and SQS FIFO queues, you can **have fan out, ordering, and deduplication** all at the same time.

![Combining SNS FIFO Topics and SQS FIFO Queues](/assets/aws-certified-developer-associate/combining_sns_fifo_topics_and_sqs_fifo_queues.png "Combining SNS FIFO Topics and SQS FIFO Queues")

## 16.23 SNS Message Filtering

In SNS, you can use **JSON policies to filter messages sent to SNS topics' subscriptions**.
- If a subscription does not have a filter policy, it receives every message.

![SNS Message Filtering](/assets/aws-certified-developer-associate/sns_message_filtering.png "SNS Message Filtering")

## 16.24 Creating an SNS Topic

To do so, go to the SNS service in the console and click on *Create Topic* by entering a **name**.

![SNS Create Topic](/assets/aws-certified-developer-associate/sns_create_topic.png "SNS Create Topic")

Then, select whether the **topic type** is standard or FIFO:

![SNS Topic Type](/assets/aws-certified-developer-associate/sns_topic_type.png "SNS Topic Type")

Finally, you can configure other **advanced options**, such as encryption, access policy, tags:

![SNS Advanced Options](/assets/aws-certified-developer-associate/sns_advanced_options.png "SNS Advanced Options")

### 16.24.1 Subscribing to an SNS Topic

After you create a topic, you can subscribe to it by clicking on *Create Subscription*:

![SNS Create Subscription](/assets/aws-certified-developer-associate/sns_create_subscription.png "SNS Create Subscription")

Then, chose the **protocol** (e.g., email, SMS, Lambda, SQS, etc.). **Remember the protocols because they are tested in the exam**.

![SNS Subscription Protocol](/assets/aws-certified-developer-associate/sns_subscription_protocol.png "SNS Subscription Protocol")

Then, enter the **endpoint**, which depends on the protocol. For example, for an email, you need to enter the email address.

You can also configure a **subscription filter policy** to filter messages sent to the subscription:

![SNS Subscription Filter Policy](/assets/aws-certified-developer-associate/sns_subscription_filter_policy.png "SNS Subscription Filter Policy")

Finally, create the subscription and it will appear in the list of subscriptions for the topic.

To **realize the fan out pattern, you can just create multiple SQS queues and subscribe them to the SNS topic**.

### 16.24.2 Publishing to an SNS Topic

To publish a message to a topic, click on *Publish Message* with the topic selected:

![SNS Publish Message](/assets/aws-certified-developer-associate/sns_publish_message.png "SNS Publish Message")

And enter the **message content**:

![SNS Publish Message Content](/assets/aws-certified-developer-associate/sns_publish_message_content.png "SNS Publish Message Content")

Finally, add **attributes** or send the message:

![SNS Publish Message Attributes](/assets/aws-certified-developer-associate/sns_publish_message_attributes.png "SNS Publish Message Attributes")

When you send the message, all the subscribers to the topic will receive the message. For example, if have configured an email subscription, you will receive the message in your email.

## 16.25 Kinesis Data Streams

Kinesis Data Streams is a service that allows you to **collect and store streaming data in real-time**.
- **Real-time is the keyword to look for in the exam**.

**Real-time data** can be anything like website clickstreams, IoT telemetry data, application metrics and logs, etc.

To send real-time data to Kinensis Data Streams, you need **producers** like an application or Kinesis Agent (e.g., for application metrics and logs).
- Use *Kinesis Producer Library (KPL)* to write an optimized producer application.

To process the data, you need **consumers** like EC2 instances, Lambda functions, Kinesis Data Firehose, etc.
- Use *Kinesis Client Library (KCL)* to write an optimized consumer application.

![Kinesis Data Streams](/assets/aws-certified-developer-associate/kinesis_data_streams.png "Kinesis Data Streams")

Some **features**:
- Data retention is up to 365 days.
- You can reprocess (replay) data by consumers.
- Data cannot be deleted from Kinesis until it expires.
- You can send data up to 1MB but the **typical use case is to send lot of small real-time data**.
- **Data ordering is preserved** for data with the same `PartitionID`.
- You have KMS encryption at-rest and HTTPS encryption in-flight.
- **Shard splitting**: if you have a hot shard in a Kinesis Data Stream that is causing many `ProvisionedThroughputExceededException` exceptions, you can increase the number of shards to increase the throughput by splitting the shard.

### 16.25.1 Two Capacity Modes in Kinesis Data Streams

**Provisioned mode**:
- You need to choose the number of shards because the more shards you have, the higher the throughput:
    - Each shard gets 1MB/s in, or 1000 records per second.
    - Each shard gets 2MB/s out.
- You pay per shard provisioned per hour.
- You can manually scale to increase or decrease the number of shards.

**On-demand mode**:
- No need to provision or manage the capacity.
- You have a default provisioned capacity of 4 MB/s in, or 4000 records per second.
- It scales automatically based on observed throughput peak during the last 30 days.
- You pay per stream per hour, and data in/out per GB.

## 16.26 Creating a Kinesis Data Stream

To create a Kinesis Data Stream, go to the Kinesis service in the console and click on *Create Data Stream*.
- In this page, you also get some information about the service and pricing.

![Kinesis Create Data Stream](/assets/aws-certified-developer-associate/kinesis_create_data_stream.png "Kinesis Create Data Stream")

Then, enter the **name** and select the **capacity mode** (provisioned or on-demand):

![Kinesis Data Stream Name and Capacity Mode](/assets/aws-certified-developer-associate/kinesis_data_stream_name_capacity_mode.png "Kinesis Data Stream Name and Capacity Mode")

If you choose the **provisioned mode**, you need to enter the **number of shards** and you can use the estimator to help you:

![Kinesis Data Stream Provisioned Mode](/assets/aws-certified-developer-associate/kinesis_data_stream_provisioned_mode.png "Kinesis Data Stream Provisioned Mode")

Finally, create the data stream and it will appear in the list of data streams. You can see its details by clicking on it:

![Kinesis Data Stream Details](/assets/aws-certified-developer-associate/kinesis_data_stream_details.png "Kinesis Data Stream Details")

### 16.26.1 Sending Data to a Kinesis Data Stream

To send data to a Kinesis Data Stream, you can use the AWS CLI or the SDK.

For example, you can use the `PutRecord` API to **send a record to the stream**:

```bash
aws kinesis put-record \
    --stream-name DemoStream \
    --partition-key partition1 \
    --data "user signup" \
    --cli-binary-format raw-in-base64-out
```

If you run the above command, you will get a response with the `ShardID` and `SequenceNumber` of the record:

![Kinesis Put Record](/assets/aws-certified-developer-associate/kinesis_put_record.png "Kinesis Put Record")

### 16.26.2 Consuming Data from a Kinesis Data Stream

To consume data from a Kinesis Data Stream, you can use the AWS CLI or the SDK.

With the CLI, the first thing is to **get information about the stream**:

```bash
aws kinesis describe-stream \
    --stream-name DemoStream
```

Then, you can use the `GetShardIterator` API to **get a shard iterator**:

```bash
aws kinesis get-shard-iterator \
    --stream-name DemoStream \
    --shard-id shardId-000000000000 \
    --shard-iterator-type TRIM_HORIZON
```

Which will return a shard iterator in a response like this:

![Kinesis Get Shard Iterator](/assets/aws-certified-developer-associate/kinesis_get_shard_iterator.png "Kinesis Get Shard Iterator")

Finally, you can use the `GetRecords` API to **get records from the stream**. Instead of `<shard-iterator>`, use the shard iterator you got in the response before:

```bash
aws kinesis get-records \
    --shard-iterator <shard-iterator>
```

The response will contain the records in the shard.

## 16.27 Amazon Data Firehose (Previously Kinesis Data Firehose)

It is a **fully-managed serverless service to send data from sources to destinations**.
- It provides automatic scaling with a pay-as-you-go pricing model (you pay for what you use).
- It is a **near real-time service (this is the keyword to look for in the exam)** because of buffering capability based on size/time.
- It supports CSV, JSON, Apache Parquet, Avro, raw text, and binary data.
- It offers **data conversions** to Apache Parquet and Apache ORC but allows for custom conversions via Lambda functions (e.g., CSV to JSON).
- It comes with **data compression** with GZIP and Snappy.

You need **producers** to send data to Data Firehose, which can be AWS services, custom applications, etc.

**Destinations** can be:
1. **AWS services (important for the exam)**: S3, Redshift, OpenSearch.
2. Third party partner destinations: for example, Splunk, Datadog, MongoDB.
3. Custom destinations via HTTP endpoints.

![Amazon Data Firehose](/assets/aws-certified-developer-associate/amazon_data_firehose.png "Amazon Data Firehose")

Some **features** that you can see in the image above:
- You can perform **data transformation** before sending data to the destination via Lambda functions.
- You can **store all or failed data in S3 for backup and reprocessing**.
- **Data are accumulated in a buffer and then sent to the destination in batches**, thus making it near real-time.

## 16.28 Creating a Data Firehose

**Note**: some things have changed in the console since the images were taken, but the process is still the same. For example, delivery streams are now called data firehoses.

To do so, go to the Kinesis service in the console and click on *Create Delivery Stream*. You will be provided with an explanation of how it works:

![Create Delivery Stream](/assets/aws-certified-developer-associate/create_delivery_stream.png "Create Delivery Stream")

Then, choose the **source** and the **destination**:

![Delivery Stream Source and Destination](/assets/aws-certified-developer-associate/delivery_stream_source_destination.png "Delivery Stream Source and Destination")

And configure the **source settins (based on the selected source)**, while the **delivery stream name** can be autogenerated like in the image below:

![Delivery Stream Source Settings](/assets/aws-certified-developer-associate/delivery_stream_source_settings.png "Delivery Stream Source Settings")

Optionally, configure **data transformation**:

![Delivery Stream Data Transformation](/assets/aws-certified-developer-associate/delivery_stream_data_transformation.png "Delivery Stream Data Transformation")

Then configure **destination settings (based on the selected destination)**:

![Delivery Stream Destination Settings](/assets/aws-certified-developer-associate/delivery_stream_destination_settings.png "Delivery Stream Destination Settings")

In the destination settings, you can also configure **buffering hints**. Based on the configuration below Data Firehose will buffer data until the buffer size reaches 5MB or the buffer interval reaches 100 seconds if the buffer size is not reached.

![Delivery Stream Destination Buffering Hints](/assets/aws-certified-developer-associate/delivery_stream_destination_settings_2.png "Delivery Stream Destination Buffering Hints")

Again, in the destination settings, you can configure **data compression and encryption**:

![Delivery Stream Data Compression and Encryption](/assets/aws-certified-developer-associate/delivery_stream_data_compression_encryption.png "Delivery Stream Data Compression and Encryption")

In the **advanced settings**, you can configure **error logging** and **permissions**. As you can see, it automatically creates an IAM role for you to write to the destination:

![Delivery Stream Advanced Settings](/assets/aws-certified-developer-associate/delivery_stream_advanced_settings.png "Delivery Stream Advanced Settings")

Finally, create the delivery stream and it will appear in the list of delivery streams. You can see its details by clicking on it.

Since we have configured the delivery stream to send data from a Kinesis Data Stream to S3:
- To send data, we can just use the same CLI command as in [16.25.1 Sending Data to a Kinesis Data Stream](#16251-sending-data-to-a-kinesis-data-stream).
- Sent data can be seen in the S3 bucket that was configured as the destination after Data Firehose flushes the buffer.

## 16.29 Kinesis Data Streams vs Amazon Data Firehose

| Kinesis Data Streams | Amazon Data Firehose |
| -------------------- | -------------------- |
| Streaming data collection | Load streaming data into S3/Redshift/OpenSearch/3rd party/custom HTTP |
| Write producer/consumer code | Fully managed service |
| Real-time data processing | Near real-time data processing |
| Provisioned/on-demand capacity | Automatic scaling |
| Data storage up to 365 days | No data storage |
| Replay capability | No support for replay capability |

## 16.30 Amazon Managed Service for Apache Flink

Previously named *Kinesis Data Analytics for Apache Flink*. Flink (Java, Scala or SQL) is a framework for processing data streams in real-time.

**Run any Apache Flink application on a managed cluster on AWS**.
- It offers provisioned compute resources, parallel computation, automatic scaling,
- It provides application backups implemented as checkpoints and snapshots.
- Allows you to use any Apache Flink programming features to transform data.

Amazon Managed Service for Apache Flink can **read only from Kinesis Data Streams and Amazon Managed Streaming for Apache Kafka**. However, it **cannot read from Amazon Data Firehose (this can a tricky thing to look for in the exam**).

![Amazon Managed Service for Apache Flink](/assets/aws-certified-developer-associate/amazon_managed_service_for_apache_flink.png "Amazon Managed Service for Apache Flink")

## 16.31 SQS vs SNS vs Kinesis Data Streams

**SQS**:
- **Consumers pull data from queues**.
- **Data is deleted after being consumed**, so it is persisted until it is processed and deleted by the consumer.
- It can have as many workers (consumers) as we want.
- No need to provision throughput.
- Ordering guaranteed only on FIFO queues.
- Individual message delay capability.

**SNS**:
- **Pub/Sub**: push data to many subscribers.
- Up to 12,500,000 subscribers.
- Up to 100,000 topics.
- **Data is not persisted, so it can be lost if not delivered**.
- No need to provision throughput.
- It integrates with SQS for fan-out architecture pattern.
- FIFO capability for SQS FIFO.

**Kinesis Data Streams**:
- It is **meant for real-time big data, analytics and ETL**.
- **Standard mode**: consumers pull data from streams.
    - 2 MB per shard.
- **Enhanced-fan out mode**: push data to consumers.
    - 2 MB per shard per consumer.
- Possibility to **replay data because data are persisted** for up to X days.
- Ordering at the shard level.
- Data expires after X days.
- Two capacity modes: provisioned mode or on-demand mode.