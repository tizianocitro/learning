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