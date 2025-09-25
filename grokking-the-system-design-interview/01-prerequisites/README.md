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
