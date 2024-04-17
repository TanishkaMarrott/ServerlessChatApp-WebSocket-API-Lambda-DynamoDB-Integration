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
| _API Gateway_  | _`Web-socket-api`_      | ➡️ **Real-time communication** in our application|                     
| _DynamoDB_     | _`ConnectionsTable`_    | **Our connection registry** for tracking & managing connections  |
| _AWS Lambda_   | _`ConnectHandler`_      | --> Every new connection must be recorded --> **Helps us ensure operational Integrity** |
|                | _`DisconnectHandler`_   | ▶️ Removing entries from our table wrt inactive connections |
|                | _`SendMessageHandler`_  | --> Needed for reliable communication|
|                | _`DefaultHandler`_      | Helps notify the client when we're through with establishing a connection   |


</br>

## **How does the workflow look like?**

                  Establishes the websocket connection
                        ⬇️
                  Triggers ConnectHandler
                        ⬇️
                  ConnectHandler inserts connectionId into ConnectionsTable
                        ⬇️
                  WebSocket connection closes
                        ⬇️
                  DisconnectHandler automatically removes connectionId from ConnectionsTable
                        ⬇️
                  SendMessageHandler iterates through connectionIds and sends messages to connected clients
                        ⬇️
                  DefaultHandler --> notifies the client on establishment of the connection

</br>

## How did we improvise on the design considerations?

### _Availability:-_


1 - I've set some **Reserved Concurrency for key lambdas.** 📌 For service continuity 


</br>

> We wanted to ensure that such Lambdas have the necessary resources they need for smooth operations,          
>  --> client requests aren't lost due to other Lambdas consuming all available capacity 👍👍

</br>

2 - The services we've used here are resilient to zonal failures.        
  **Multi-AZ Deployments --> Resilience + Reliability** 

</br>

3- We've implemented **Request Throttling in API Gateway** --> Helps manage the rate of incoming requests                        
 - **I wanted the gateway to be capable of sustaining backpressure scenarios**, --> Prevents my system from being overwhelmed      
 - **Helps us safeguard against a DDoS attack**  --> This means that my API will remain responsive to legit users

</br>

--

## _Scalability_ 

1 - We've configured <ins>**Provisioned Concurrency**</ins> for Lambdas. This ensures that my critical Lambdas will keep a specified number of instances always available at all times, --> Highly Responsive 👍 

> _Reason:-_                                             
> ▶️ **Prewarming a set of lambda instances 🟰 Reduces cold Starts 🟰 Reducing latency**
> 
</br>

2-  We wanted things to _scale dynamically_ such thet we're workload-responsive always            
--> Implemented **dynamic Auto-Scaling for DynamoDB** ▶️ **Cost-Optimisation**.  It'll be able to handle huge variations in load, in case of traffic spikes or quiter periods

> This means Dynamo would automatically adjust based on actual usage _only_. We save on infra-costs.

3- **We've provisioned throughput for DynamoDB with RCUs and WCUs** ➡️ a consistent + predictable read/write performance.

</br>

### How exactly is Provisoned Concurrency different from the reserved counterpart?

✅ _Difference 1_ --> **When I'm talking about Provisioned Concurrency, it all about eliminating cold starts**, reducing _the initialisation latency_. While **reserved Concurrency is about ensuring you've got a certain portion of the Total Concurrency dedicated** to this lambda.

✅ _Difference 2_ --> **Provisioned concurrency is geared towards enhancing performance**, while  **reserved counterpart is about managing resource limits.** 

> --> Prevents a lambda function from consuming too many resources. 👍

✅ _Difference 3_ --> **PC means you're incurring costs of keeping such instances ready at all times**, while **RC means you've sanctioned limits, no costs per se** 



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


