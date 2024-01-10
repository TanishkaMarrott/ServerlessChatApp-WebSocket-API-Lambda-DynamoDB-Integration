# DynamoWave Chat - A Serverless Real-time Chat Application

DynamoWave Chat is a modern and scalable serverless real-time chat application. It is built on top of AWS Lambda, DynamoDB and WebSocket API, to deliver a seamless communication experience. The emphasis here is on the optimising System Design Considerations.

</br>

## Table of Contents

1. [System Architecture and Components](#system-architecture-and-components)
2. [The Workflow](#the-workflow)
3. [Design Considerations](#design-considerations)
4. [Setup](#setup)
5. [Usage](#usage)
6. [Enhancements for the Current Architecture](#enhancements-for-the-current-architecture)
7. [Contributions](#contributions)
8. [Credit Attribution](#credit-attribution)

</br>

## System Architecture And Components

The CF Template defines the architecture responsible for:-  handling WebSocket Connections, managing them in a DynamoDB table, & enabling communication between connected clients using Lambda.

The WebSocket API (via the API Gateway) has been built using the AWS Console. (To be Shared Shortly)


#### Architectural Diagram

<img width="416" alt="image" src="https://github.com/TanishkaMarrott/ServerlessChatApp-WebSocket-API-Lambda-DynamoDB-Integration/assets/78227704/afed5865-ebe0-4292-b402-b74216650655">
</br>

### API Gateway 

**_web-app-api:_** 
The WebSocket APi via the API Gateway. Helps establish bidirectional, persistent data connections. This allows for a seamless exchange of messages and data between Clients and Serverless Backends.
</br>

### DynamoDB 

**_ConnectionsTable:_**  The Connection registry. This database table serves as a repository for storing essential metadata related to connections. It includes information such as connection identifiers, facilitating efficient tracking and management of connections within the system.
</br>

### AWS Lambda 

Four Lambdas have been used in the solution, Functions written below:-

**1- _ConnectHandler:_** Handles WebSocket connections, Adds a new connectionId to ConnectionsTable when a WebSocket connection is established

**2 -_DisconnectHandler:_** Removes a connectionId from ConnectionsTable when a WebSocket connection is closed.

**3 -_SendMessageHandler:_** Sends messages to connected clients. It retrieves all connectionIds from ConnectionsTable, and sends a message to each connected client using ApiGatewayManagementApi.

**4 -_DefaultHandler_**: Provides information to a client when a WebSocket connection is established

</br>

## The Workflow

**Step 1**- A WebSocket connection is established, triggering the _ConnectHandler_ Lambda function.

**Step 2**- The _ConnectHandler_ Lambda function adds the _connectionId_ to the _ConnectionsTable_ in DynamoDB.

**Step 3**- If a WebSocket connection is closed, the _DisconnectHandler_ Lambda function removes the _connectionId_ from the _ConnectionsTable_.

**Step 4**- The _SendMessageHandler_ Lambda function can be invoked to send messages to all connected clients by iterating through _connectionIds_ in the _ConnectionsTable_.

**Step 5**- The _DefaultHandler_ Lambda function provides information to a client when a WebSocket connection is established.

Scaling policies, and targets would help ensure that DynamoDB read & write capacities scale based on predefined metrics. The IAM roles and policies control access to DynamoDB and API Gateway actions for the Lambda functions.

</br>

## Design Considerations

### Availability 

_**Multi-AZ Deployments (Built-in) :**_
The core services used here are implicitly resilent to Zonal Failures.

_**Set Reserved Concurrency in Lambda:**_ 
Helps in controlling the maximum number of Concurrent Invocations of a Lambda Function. We won't lose requests due to other functions consuming all of the available concurrency.

_**Implemented Throttling in API Gateway:**_ This helps in controlling the volume of API requests hitting the API Gateway, preventing abuse & mitigating a DDoS Attack. The APIs thus wouldn't be overwhelmed by too many requests.

</br>

### Scalability 

_**Configured Provisioned Concurrency for Lambda:**_ Reduces cold start latency, ensuring consistent & predictable performance. Pre-warming a set of Lambda function instances helps improve Responsiveness & Scalability during traffic spikes

_**Provisioned Throughput for DynamoDB:**_ Configured DynamoDB Provisioned Throughput with RCUs and WCUs, for a consistent and predictable read/write performance.

 _**Implemented Automatic Scaling for DynamoDB:**_ Dynamic Auto-Scaling through Targets and Policies for automatic, workload-responsive adjustments. Aids in resource-utilization & helps in cost optimisation

</br>

### Security 

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


