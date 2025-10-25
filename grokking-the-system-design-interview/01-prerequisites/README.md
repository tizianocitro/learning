# 1 Prerequisites

## 1.1 Functional Requirements

Requirements that **define what a system is supposed to do by describing the various functions that the system must perform**.

For example:
- A user authentication system must validate user credentials and provide access levels.
- An e-commerce website should allow users to browse products, add them to a cart, and complete purchases.
- A report generation system must collect data, process it, and generate timely reports.

## 1.2 Non-functional Requirements

Requirements that **describe how the system performs a task, rather than what tasks it performs**. They are related to the quality attributes of the system.

For example:
- Scalability: The system should handle growth in users or data.
- Performance: The system should process transactions within a specified time.
- Availability: The system should be up and running a defined percentage of time.
- Security: The system must protect sensitive data and resist unauthorized access.

## 1.3 Functional and Non-functional Requirements in Interviws

When you are in a system design interview, you can handle these requirements by following these guidelines:
1. **Clarify requirements**: Start by asking questions to understand both functional and non-functional requirements. Interviewers often leave these vague to see if you will ask for more details.
2. **Prioritize**: Not all requirements are equally important. Identify which ones are critical for the system's success.
3. **Trade-offs**: Discuss trade-offs related to different architectural decisions, especially concerning non-functional requirements. For example, a system highly optimized for read operations might have slower write operations.
4. **Use real-world examples**: If you can, relate your points to real-world systems or your past experiences. This shows practical understanding.
5. **Balance**: Ensure you are not focusing too much on one type of requirement over the other. A well-rounded approach is often necessary. You should demonstrate that you can design a system that not only meets its functional goals but also performs effectively, securely, and reliably.

In system design interviews, interviewers are often interested in seeing how you think and approach problems, not just the final solution. Demonstrating a clear understanding of both functional and non-functional requirements is key to showing your comprehensive knowledge in system design.

## 1.4 Back-of-the-Envelope Estimations

Back-of-the-envelope estimations are like quick, rough calculations you might do on a napkin during lunch. They are not detailed or exact but give you a starting figure. These rough calculations help you quickly assess the feasibility of a proposed solution, estimate its performance, and identify potential bottlenecks.

**Back-of-the-envelope estimation is a technique used to quickly approximate values and make rough calculations using simple arithmetic and basic assumptions**. This method is particularly useful in system design interviews because interviewers expect candidates to make informed decisions and trade-offs based on rough estimates.

### 1.4.1 Why is Estimation Important in System Design Interviews?

The ability to make quick estimations is essential for several reasons:
- Indicate system scalability: highlights your understanding of how the system can grow or adapt.
- Validate proposed solutions: estimation helps you ensure that your proposed architecture meets the requirements and can handle the expected load.
- Identify bottlenecks: quick calculations help you identify potential performance bottlenecks and make necessary adjustments to your design.
- Demonstrate your thought process: estimation showcases your ability to make informed decisions and trade-offs based on a set of assumptions and constraints.
- Communicate effectively: providing estimates helps you effectively communicate your design choices and their implications to the interviewer.
- Quick decision making: reflects your ability to make swift estimations to guide your design decisions.

## 1.5 Estimation Techniques

### 1.5.1 Rule of Thumb

Rules of thumb are general guidelines or principles that can be applied to make quick and reasonably accurate estimations. They are **based on experience and observation, and while not always precise, they can provide valuable insights in the absence of detailed information**.

For instance, estimating that a user will generate 1 MB of data per day on a social media platform can serve as a starting point for capacity planning.

### 1.5.2 Approximation

**Approximation involves simplifying complex calculations by rounding numbers or using easier-to-compute values**. This technique can help derive rough estimates quickly and with minimal effort.

For example, assuming 1000 users instead of 1024 when estimating storage requirements can simplify calculations and still provide a reasonable approximation.

### 1.5.3 Breakdown and Aggregation

**Breaking down a problem into smaller components and estimating each separately can make it easier to derive an overall estimate**. This technique involves identifying the key components of a system, estimating their individual requirements, and then aggregating these estimates to determine the total system requirements.

For instance, estimating the storage needs for user data, multimedia content, and metadata separately can help in identifying the overall storage requirements of a social media platform.

### 1.5.4 Sanity Check

**A sanity check is a quick evaluation of an estimate to ensure its plausibility and reasonableness**. This step helps finding potential errors or oversights in the estimation process and can lead to more accurate and reliable results.

For example, comparing the estimated storage requirements for a messaging service with the actual storage used by a similar existing service can help validate the estimate.

## 1.6 Types of Estimations in System Design Interviews

There are several types of estimations you may need to make:
- **Load estimation**: predict the expected number of requests per second, data volume, or user traffic for the system.
- **Storage estimation**: estimate the amount of storage required to handle the data generated by the system.
- **Bandwidth estimation**: determine the network bandwidth needed to support the expected traffic and data transfer.
- **Latency estimation**: predict the response time and latency of the system based on its architecture and components.
- **Resource estimation**: estimate the number of servers, CPUs, or memory required to handle the load and maintain desired performance levels.

### 1.6.1 Process

1. **Understand the scope**: clarify the scale of the problem in terms of how many users, how much data, etc.
2. **Use simple math**: use basic arithmetic to estimate the scale of data and resources.
3. **Round numbers for simplicity**: use round numbers to make calculations easier and faster.
4. **Be logical and reasonable**: ensure your estimations make sense given the context of the problem.

## 1.7 Practical Examples for Types of Estimations

### 1.7.1 Load Estimation

Suppose you are asked to design a social media platform with 100 million daily active users (DAU) and an average of 10 posts per user per day. To estimate the load, you need to calculate the total number of posts generated daily:

```text
100 million DAU * 10 posts per user = 1 billion posts per day
```

Then, you can estimate the request rate per second:

```text
1 billion posts per day / 86400 seconds per day ≈ 11574 requests per second
```

### 1.7.2 Storage Estimation

Consider a photo-sharing application with 500 million users and an average of 2 photos uploaded per user per day. Each photo has an average size of 2 MB. To estimate the storage required for one day worth of photos, you calculate:

```text
500 million users * 2 photos per user * 2 MB per photo = 2 billion MB = 2 PB
```

### 1.7.3 Bandwidth Estimation

For a video streaming service with 10 million users streaming 1080p videos at 4 Mbps, you can estimate the required bandwidth:

```text
10 million users * 4 Mbps = 40 million Mbps
```

So, the 1080p is not really relevant in this case.

### 1.7.4 Latency Estimation

Suppose you are designing an API that fetches data from multiple sources, and you know that the average latency for each source is 50 ms, 100 ms, and 200 ms, respectively. If the data fetching process is sequential, you can estimate the total latency as follows:

```text
50 ms + 100 ms + 200 ms = 350 ms
```

If the data fetching process is parallel, the total latency would be the maximum latency among the sources:

```text
max(50 ms, 100 ms, 200 ms) = 200 ms
```

### 1.7.5 Resource Estimation

Imagine you are designing a web application that receives 10000 requests per second, with each request requiring 10 ms of CPU time. To estimate the number of CPU cores needed, you can calculate the total CPU time per second:

```text
10,000 requests per second * 10 ms per request = 100000 ms per second
```

Assuming each CPU core can handle 1000 ms of processing per second, the number of cores required would be:

```text
100000 ms per second / 1000 ms per core = 100 cores
```

## 1.8 System Design Examples

### 1.8.1 Designing a Messaging Service

Imagine you are tasked with designing a messaging service similar to WhatsApp. To estimate the system's requirements, you can start by considering the following aspects:
- **Number of users**: estimate the total number of users for the platform. This can be based on market research, competitor analysis, or historical data.
- **Messages per user per day**: estimate the average number of messages sent by each user per day. This can be based on user behavior patterns or industry benchmarks.
- **Message size**: estimate the average size of a message, considering text, images, videos, and other media content.
- **Storage requirements**: calculate the total storage needed to store messages for a specified retention period, taking into account the number of users, messages per user, message size, and data redundancy.
- **Bandwidth requirements**: estimate the bandwidth needed to handle the message traffic between users, considering the number of users, messages per user, and message size.

### 1.8.2 Designing a Video Streaming Platform

Suppose you are designing a video streaming platform similar to Netflix. To estimate the system's requirements, consider the following aspects:
- **Number of users**: estimate the total number of users for the platform based on market research, competitor analysis, or historical data.
- **Concurrent users**: Estimate the number of users who will be streaming videos simultaneously during peak hours.
- **Video size and bitrate**: Estimate the average size and bitrate of videos on the platform, considering various resolutions and encoding formats.
- **Storage requirements**: Calculate the total storage needed to store the video content, taking into account the number of videos, their sizes, and data redundancy.
- **Bandwidth requirements**: Estimate the bandwidth needed to handle the video streaming traffic, considering the number of concurrent users, video bitrates, and user locations.

## 1.9 Tips for Successful Estimation in Interviews

### 1.9.1 Break Down the Problem

When faced with a complex system design problem, break it down into smaller, more manageable components. This will make it easier to estimate each component's requirements and help you understand how they interact with each other.

By identifying the key components and estimating their requirements separately, you can then aggregate your estimates to get a comprehensive view of the system's requirements.

### 1.9.2 Use Reasonable Assumptions

During an interview, you may not have all the necessary information to make precise estimations. In such cases, make reasonable assumptions based on your knowledge of similar systems, industry standards, or user behavior patterns. 

Clearly state your assumptions to the interviewer, as this demonstrates your thought process and enables them to provide feedback or correct your assumptions if necessary.

### 1.9.3 Leverage your Experience

Drawing from your past experiences can be beneficial when estimating system requirements. If you have worked on similar systems or have experience with certain technologies, use that knowledge to inform your estimations. This will not only help you make more accurate estimations but also showcase your expertise to the interviewer.

### 1.9.4 Be Prepared to Adjust Estimations

As you progress through the interview, the interviewer may provide additional information or challenge your assumptions, requiring you to adjust your estimations. Be prepared to adapt and revise your estimations accordingly.

This demonstrates your ability to think critically and shows that you can handle changing requirements in a real-world scenario.

### 1.9.5 Do Not Forget to Ask Clarifying Questions

Do not hesitate to ask the interviewer clarifying questions if you are unsure about a requirement or assumption. This will help you avoid making incorrect estimations.

### 1.9.6 Communicate your Thought Process

Throughout the estimation process, communicate your thought process clearly to the interviewer. Explain how you arrived at your estimations and the assumptions you made along the way. This allows the interviewer to understand your reasoning, provide feedback, and assess your problem-solving skills.