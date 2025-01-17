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