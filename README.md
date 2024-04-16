# DynamoWave Chat - A serverless, real-time chat Application

DynamoWave Chat is a modern and scalable serverless real-time chat application. 

It is built on top of AWS Services - Lambda, DynamoDB & API Gateway

➡️ **Focus:** Enhancing the application to ensure high-performance delivery.


</br>

## Table of Contents

1. [System Architecture & Components](#system-architecture--components)
2. [The Workflow](#the-workflow)
3. [How did we improvise on the design considerations?](#how-did-we-improvise-on-the-design-considerations)
4. [Setup](#setup)
5. [Usage](#usage)
6. [Enhancements for the Current Architecture](#enhancements-for-the-current-architecture)
7. [Contributions](#contributions)
8. [Credit Attribution](#credit-attribution)

</br>

## System Architecture & Components

<img width="927" alt="image" src="https://github.com/TanishkaMarrott/ServerlessChatApp-WebSocket-API-Lambda-DynamoDB-Integration/assets/78227704/afed5865-ebe0-4292-b402-b74216650655">

</br>

### Services we've used, with their purpose

To provide clarity, we'll define the purpose of each component in our architecture:-

</br>

| Service        | Identifier we're using     | Purpose - Why we've used?                       |
|--------------------|---------------------|-------------------------------|
|||                               |
| _API Gateway_  | `Web-socket-api`      | ➡️ Real-time communication in our application|                     
| _DynamoDB_     | `ConnectionsTable`    | **_Purpose?_** Acts as a connection registry to efficiently track connections  |
| _AWS Lambda_   | `ConnectHandler`      | --> **_Every new connection must be recorded --> Operational Integrity_** |
|                | `DisconnectHandler`   | Updates the connection table by **removing inactive connections** |
|                | `SendMessageHandler`  | --> Needed for reliable communication across all active connections|
|                | `DefaultHandler`      | Helps notify the client at connection setup   |


</br>

## **How does the workflow look like?**

         Establishes the websocket connection     
         
               ↓     
               
         Triggers `ConnectHandler`     
         
               ↓    
               
         `ConnectHandler` inserts `connectionId` into `ConnectionsTable`    
         
               ↓    
               
         WebSocket connection closes       
         
               ↓    
               
         `DisconnectHandler` automatically removes `connectionId` from `ConnectionsTable`    
         
               ↓    
               
         `SendMessageHandler` iterates through `connectionIds` and send msgs to connected clients    
         
               ↓    
               
         `DefaultHandler` --> notifies the client on establishmenet of the connection    
         

</br>

## How did we improvise on the design considerations?

### The availability aspect

1 - The services we've used here are **Multi-AZ - resilent to Zonal Failures.** 👍

2 - **We've set some reserved concurrency in lambda.** --> Helps us in controlling the maximum number of concurrent invocations 

> We won't lose requests due to other functions consuming all of the available concurrency.

3 - **Implemented Throttling in API Gateway.** We had to control the volume of API requests hitting the gateway --> Mitigating a DDoS Attack. ➡️ The APIs thus wouldn't be overwhelmed by too many requests.

</br>

### Scalability 

1 - **We've configured provisioned concurrency for Lambda**    
_Purpose?_                                             
**Reduces cold start latency 🟰 Consistent & predictable performance** 

> **Pre-warming a set of lambda instances** helps us in improvising responsiveness & Scalability during traffic spikes 👍

2- **We've provisioned throughput for DynamoDB with RCUs and WCUs** ➡️ a consistent and predictable read/write performance.

3-  We wanted things to scale dynamically such thet we're workload-responsive always, **hence we implemented dynamic auto-scaling for DynamoDB through targets and policies** ▶️ **Cost optimisation**

</br>

## Security 

_**Lambda Authorizer - API Gateway Authorization:**_
IAM authorization is implemented for the $connect method using Lambda Authorizer in API Gateway, ensuring secure and controlled access.

_**Fine-grained Access Control:**_ Have granted the least privilege access to resources. Lambda functions and DynamoDB tables are secured with fine-grained permissions, ensuring data integrity and confidentiality.

</br>

### Cost-Optimization 

**_Event-Driven architectural pattern:_** Pay-as-you-go model helps save on idle resources, cutting on unnecessary costs.

**_Auto-Scaling configurations:_** Especially in the case of sporadic workloads, it has the ability to scale down as well. 

</br>

### Performance Optimization 

_**Rate Limiting in API Gateway:**_ Message rate limiting enables the control of the load on the downstream systems that process the messages.

_**Choice of WebSocket APIs over REST APIs:**_ Web-Socket API optimises performance by establishing a long-lived, persistent connections. Eliminating the overhead involved in establishing connections frequently.  

_**NoSQL Database as a connection registry:**_  DynamoDB would be well-suited to handle Connection Metadata here, the low latency access and 

</br>

### Enhancements for the Current Architecture

**How can I make my current architecture resilent to regional failures?**

If high availability and failover capabilities are critical, the regional setup with Route 53 offers better control over traffic distribution during regional outages. Configure Route53 DNS Health check to failover to an API Gateway in a secondary region, (We will have to make sure we've got all the required resources in that regions), Cost-Redundancy Tradeoff

Deploy critical components in multiple regions, enable PITR for Data Stores as a Backup-and-Restore Mechanism, Cross-region replicas, DynamoDB Global Tables, 



**What other strategies can be implemented to make this architecture even more scalable?**
Custom Auto-Scaling Logic using CloudWatch Alarms

**How can I make this even more secure & withstand potential threats?**
Use WAF on top of API gateway for enhancing the security of the current architecture. By configuring WAF ACLs and associating them with the API, we can mitigate common web exploits and protect against malicious WebSocket requests.

**How can I achieve a better Performance Optimisation, while maintaining costs?**

If we're looking at a geographically dispersed audience, and need to reduce connection times, I'd go in for the Edge-Optimised API Gateway. With Regional API Gateway, the requests traverse through the public internet, and then reach the regional endpoint. While with an Edge-optimised API gateway, the API requests travel through the CloudFront Points of Presence (PoP), before hitting the API Endpoint. The first one is thus a simpler, and an effective solution, with minimal costs & configuration. If I need more control over the routing logic, or global load balancing, and I'm more concerned about the DR Capabilities, the latter, Multiple Regional API Endpoints with Route 53 Latency-Based Routing would be my alternative here.

DAX (DynamoDB Accelerator) wouldn't be something we'd be taking about here. Caching would be ideal in cases where we need to optimise read-performance / access to some frequent data. Introducing caching in a real-time application doesn't serve the purpose - we'd be introducing unnecessary complexities and costs, without achieving something fruitful. Features like Auto-Scaling, Provisioned Throughput, have already been incorporated here. 

If the chat application spans multiple AWS regions, I would consider using DynamoDB Global Tables for multi-region replication, for ensuring low-latency access for users across the globe.

A consistent high-volume read/write traffic would be a signal for me to explore DynamoDB batch operations. Particularly, for bulk write or delete operations, such as managing multiple connections in the ConnectHandler and DisconnectHandler Lambda functions. For individual, sporadic requests, where the application rarely receives bursts of traffic, the system would need to wait for a certain number of requests to accumulate before processing them together. In a scenario with sporadic requests, this delay might be noticeable. Not recommended for applications with low, sporadic traffic.

I would monitor and adjust Provisioned / Reserved Concurrency (Lambda) & DynamoDB throughput based on real-time demand - through Performance Insights / CW. This would help reduce unnecessary standing costs for over-provisioning.



### Contributions 
Contributions are most welcome, feel free to submit issues, feature requests, or pull requests. 
If you've got  suggestions on how I could further improvise on the architectural / configurational aspects, please feel free to drop a message on tanishka.marrott@gmail.com. I'd love to hear your thoughts on this!

### Credit Attribution
Special thanks to [AWS](https://aws.amazon.com/) for providing an excellent blog that served as the foundation for this project. This -> [https://docs.aws.amazon.com/apigateway/latest/developerguide/websocket-api-chat-app.html], was instrumental in guiding the implementation of the base architecture.


