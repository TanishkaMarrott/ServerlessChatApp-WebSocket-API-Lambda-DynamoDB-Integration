# DynamoWave Chat - Serverless Real-time Chat Application  

DynamoWave Chat is a modern and scalable serverless real-time chat application. It is built on AWS Lambda, DynamoDB, and WebSocket API, to deliver a seamless communication experience. This is specifically designed, considering key System Design Principles.

## Table of Contents
1. 
2. [Project Architecture and Components](#project-architecture-and-components)
3. [Project Workflow](#project-workflow)
4. [Design Considerations](#design-considerations)
5. [Setup](#setup)
6. [Usage](#usage)
7. [Enhancements for the Current Architecture](#enhancements-for-the-current-architecture)
8. [Contributions](#contributions)
9. [Acknowledgements](#acknowledgements)


## Project Architecture and Components
1- 
## Project Workflow

1- A WebSocket connection is established, triggering the _ConnectHandler_ Lambda function.

2- The _ConnectHandler_ Lambda function adds the _connectionId_ to the _ConnectionsTable_ in DynamoDB.

3- If a WebSocket connection is closed, the _DisconnectHandler_ Lambda function removes the _connectionId_ from the _ConnectionsTable_.

4- The _SendMessageHandler_ Lambda function can be invoked to send messages to all connected clients by iterating through _connectionIds_ in the _ConnectionsTable_.

5- The _DefaultHandler_ Lambda function provides information to a client when a WebSocket connection is established.

The auto scaling configurations ensure that DynamoDB read and write capacities scale based on predefined metrics.
The IAM roles and policies control access to DynamoDB and API Gateway actions for the Lambda functions.

The main aim of the template is to provide a scalable and maintainable architecture for handling WebSocket connections and managing communication between clients.




## Design Considerations

### Serverless Architectural Pattern

Utilises AWS Lambda for Compute, DynamoDB for a NoSQL database, and WebSocket API for handling Real-time Connections. No infrastructure Provioning / Management Overhead involved. Makes it massively scalable and reduces associated costs.

### Availability 

_**Multi-AZ Deployments (Built-in) :**_
The core services used here are implicitly resilent to Zonal Failures.

_**Set Reserved Concurrency in Lambda:**_ 
Helps in controlling the maximum number of Concurrent Invocations of a Lambda Function. We won't lose requests due to other functions consuming all of the available concurrency.

_**Implemented Throttling in API Gateway:**_ This helps in controlling the volume of API requests hitting the API Gateway, preventing abuse & mitigating a DDoS Attack. The APIs thus wouldn't be overwhelmed by too many requests.

### Scalability 

_**Configured Provisioned Concurrency for Lambda:**_ Reduces cold start latency, ensuring consistent & predictable performance. Pre-warming a set of Lambda function instances helps improve Responsiveness & Scalability during traffic spikes

_**Provisioned Throughput for DynamoDB:**_ Configured DynamoDB Provisioned Throughput with RCUs and WCUs, for a consistent and predictable read/write performance.

 _**Implemented Automatic Scaling for DynamoDB:**_ Dynamic Auto-Scaling through Targets and Policies for automatic, workload-responsive adjustments. Aids in resource-utilization & helps in cost optimisation


### Security 

_**Lambda Authoriser for API Gateway Authorization:**_  

_**Fine Grained Access Control using IAM Roles & Policies:**_ Have granted the least privilege access to resources. Lambda functions and DynamoDB tables are secured with fine-grained permissions, ensuring data integrity and confidentiality.

#### Cost-Optimization 

**_Pay-as-You-Go compute - lambda:_** Enables us to optimize on costs. Serverless also helps in abstracting out the underlying infra and saving on associated costs.

**_Event-Driven Architecture pattern:_** Saves on Idle resources, and thus ensuring an efficient resource utilisation.

**_Fine-Tuning Auto-Scaling Configurations:_** Especially in the case of sporadic workloads, it has the ability to scale down as well. 


#### Performance Optimization 

_Rate Limiting in API Gateway:_ Message rate limiting enables the control of the load on the downstream systems that process the messages.

_Choice of WebSocket APIs over REST APIs:_ Web-Socket API optimises performance by establishing a long-lived, persistent connections. Eliminating the overhead involved in establishing connections frequently.  

_NoSQL Database as a connection registry:_  DynamoDB would be well-suited to handle Connection Metadata here, the low latency access and 

_

### System Architecture

_WebSocket API:_ Helps in Bi-directional communication, without the client having to poll for messages. Ultra-low latency for Real-Time Communication, 

_AWS Lambda:_ Executes functions in response to WebSocket events, managing chat-related logic.

_DynamoDB_: Stores and retrieves chat messages, ensuring scalability and low-latency access.

### Enhancements for the Current Architecture

How can I make my current architecture resilent to regional failures? 
Use of CDNs for geographically dispersed users, deploy critical components in multiple regions, enable PITR for Data Stores as a Backup-and-Restore Mechanism, Cross-region replicas, DynamoDB Global Tables, 

Use Route 53 health checks to control DNS failover from an API Gateway API in a primary region to an API Gateway API in a secondary regionConfigure Route53 DNS Health check to failover to an API Gateway in a secondary region, (We will have to make sure we've got all the required resources in that regions), Cost-Redundancy Tradeoff.

What other strategies can be implemented to make this architecture even more scalable?
Custom Auto-Scaling Logic using CloudWatch Alarms

How can I make this even more secure & withstand potential threats?

How might I further optimize costs within my architecture while maintaining performance?

What operational efficiencies can be introduced?


### Contributions 
Contributions are most welcome, feel free to submit issues, feature requests, or pull requests. 
If you've got  suggestions on how I could further improvise on the architectural / configurational aspects, please feel free to drop a message on tanishka.marrott@gmail.com. I'd love to hear your thoughts on this!

### Acknowledgements
Special thanks to [AWS](https://aws.amazon.com/) for providing an excellent tutorial that served as the foundation for this project. The original tutorial, [https://docs.aws.amazon.com/apigateway/latest/developerguide/websocket-api-chat-app.html], was instrumental in guiding the implementation of the base architecture.


