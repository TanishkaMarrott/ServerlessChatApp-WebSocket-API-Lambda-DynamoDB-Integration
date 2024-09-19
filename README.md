# DynamoWave Chat - A Serverless, Real-time Chat Application

**DynamoWave Chat** is a modern, scalable, serverless real-time chat application built using **AWS Lambda**, **DynamoDB**, and **API Gateway**.

➡️ **Core Focus**: Enhancing scalability, performance, and security from a non-functional standpoint.

---

## System Architecture & Components

<img width="500" alt="System Architecture" src="https://github.com/TanishkaMarrott/ServerlessChatApp-WebSocket-API-Lambda-DynamoDB-Integration/assets/78227704/afed5865-ebe0-4292-b402-b74216650655">

---

## Workflow Overview

1. **Establish WebSocket connection**: Two-way communication between the client and server.
2. **ConnectHandler Lambda**: Triggered when a connection is established, inserting the `connectionId` into **ConnectionsTable**.
3. **Notification to Client**: Once the connection is established, the client is notified.
4. **SendMessageHandler Lambda**: Iterates through `connectionIds` and sends messages to connected clients.
5. **DisconnectHandler Lambda**: Cleans up by removing inactive `connectionIds` after the session ends.

---

## Services & Purpose

| **Service**           | **Identifier**       | **Purpose**                                                            |
|-----------------------|----------------------|------------------------------------------------------------------------|
| **API Gateway**        | WebSocket API        | Enables real-time communication.                                        |
| **DynamoDB**           | ConnectionsTable     | Tracks and manages active connections.                                  |
| **AWS Lambda**         | ConnectHandler       | Records new connections for operational monitoring.                     |
|                       | DisconnectHandler    | Removes inactive connections from the registry.                         |
|                       | SendMessageHandler   | Handles reliable communication among connected clients.                 |
|                       | DefaultHandler       | Notifies clients when the connection has been established.              |

---

## Design Considerations

### Availability & Reliability Improvements

1. **Reserved Concurrency for Critical Lambdas**: Critical Lambda functions have reserved concurrency quotas to ensure compute availability during peak times and prevent throttling.
   
   ➡️ Allocating resources ensures the application remains functional even during high load periods.

2. **Data Durability via Point-In-Time Recovery (PITR)**: Enabled PITR for **DynamoDB** to restore data to any second in the past 35 days, ensuring data availability and fault tolerance, even in case of accidental overwrites or deletions.

   ➡️ Simplifies data recovery without operational overhead or over-provisioning costs.

3. **Backpressure Resilience**: Implemented API Gateway **throttling** and **rate limiting** to ensure backend services aren't overwhelmed during peak usage.

   ➡️ Defines a maximum threshold of incoming requests and caps client requests to avoid service downtime or DDoS attacks.

4. **Resilience to Zonal Failures**: While the application is resilient to zonal outages, to improve availability in production systems, **DynamoDB Global Tables** and regional redundancy with Route 53 DNS failover could be implemented.

   ➡️ Provides higher availability for mission-critical applications and geographically distributed user bases.

---

## Code Optimizations for Lambda Reliability

1. **Error Handling**: Incorporated error handling mechanisms in Lambda functions to prevent cascading failures and ensure system stability.
   
2. **Retry Mechanisms**: Implemented **exponential backoff** for critical functions to recover from transient errors such as network failures or database operations.

   ➡️ Increases the likelihood of successful message delivery without overwhelming the system.

3. **Graceful Error Recovery**: Configured a Dead Letter Queue (DLQ) to capture and reprocess failed messages, ensuring **zero data loss**.

---

## Performance Optimizations

1. **Dynamic Auto-scaling for DynamoDB**: Dynamically adjusts capacity based on fluctuating workloads to ensure cost-effective scalability.

   ➡️ Automatically scales based on demand, reducing costs during idle times and increasing capacity during peak times.

2. **Custom Lambda Warmer**: Implemented a Lambda warmer function to reduce cold starts and improve performance for sporadically used functions.

   ➡️ Configured a **CloudWatch event** to trigger the warmer function, maintaining low-latency performance.

---

## Security Features

1. **API Gateway Resource Policies**: Enforced HTTPS-only requests to ensure secure transport for all connections.
   
2. **KMS Encryption for DynamoDB**: Secures all data at rest with **KMS encryption**, protecting sensitive user information.

3. **Least Privilege IAM Roles**: Pruned down **IAM policies** for Lambda service roles to enforce least-privilege access, minimizing risks of privilege escalation.

4. **Throttling for DDoS Mitigation**: API throttling prevents potential DDoS attacks by limiting the number of requests a user or bot can send in a given time.

---

## Future Enhancements

1. **WAF Integration**: Adding a Web Application Firewall (WAF) on top of API Gateway to protect against excessive resource consumption and ensure application availability.

   ➡️ Managed and custom rules will prevent potential security threats.

2. **CloudWatch Alarms & SNS Integration**: Configuring **real-time alerts** for abnormal API usage patterns or security incidents via CloudWatch alarms and SNS notifications.

   ➡️ Improves monitoring and operational insights.

---

## Contributions

We welcome suggestions to further improve the architecture or performance of **DynamoWave Chat**. Feel free to contact me at **tanishka.marrott@gmail.com**.

---

## Credit Attribution

Special thanks to [AWS](https://aws.amazon.com/) for providing the foundational architecture guidelines for this project: [AWS WebSocket API Chat App Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/websocket-api-chat-app.html).
