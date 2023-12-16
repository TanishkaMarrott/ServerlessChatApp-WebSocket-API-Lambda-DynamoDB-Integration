# DynamoWave Chat - Serverless Real-time Chat Application  

DynamoWave Chat is a modern and scalable serverless real-time chat application. It is built on AWS Lambda, DynamoDB, and WebSocket API, to deliver a seamless communication experience. This is specifically designed, considering key System Design Principles.

## Table of Contents

1. [System Architecture and Components](#system-architecture-and-components)
2. [Project Workflow](#project-workflow)
3. [Design Considerations](#design-considerations)
4. [Setup](#setup)
5. [Usage](#usage)
6. [Enhancements for the Current Architecture](#enhancements-for-the-current-architecture)
7. [Contributions](#contributions)
8. [Acknowledgements](#acknowledgements)


## System Architecture And Components

The CloudFormation Template defines the AWS Architecture for handling WebSocket Connections, managing them in a DynamoDB table, & enabling communication between connected clients using Lambda. 

API Gateway for the Websocket API, will be separately defined on the console.

**_Components Breakdown:-_**

_Connections Table_
Primary Key: connectionId
Provisioned Throughput: 5 RCUs, 5 WCUs
Auto-Scaling for DynamoDB Write Capacity:

IAM Role: AutoScalingRole with permissions for describe and update table actions
Scalable Target: WriteAutoScalingTarget for write capacity auto scaling
Scaling Policy: WriteAutoScalingPolicy configured for the write auto scaling target

Auto Scaling for DynamoDB Read Capacity:

Scalable Target: ReadScalingTarget for read capacity auto scaling
Scaling Policy: ReadAutoScalingPolicy configured for the read auto scaling target
Connect Lambda Function:

IAM Role: ConnectHandlerServiceRole with permissions for DynamoDB actions
Lambda Function: ConnectHandler for handling WebSocket connections
Adds a new connectionId to ConnectionsTable when a WebSocket connection is established
Disconnect Lambda Function:

IAM Role: DisconnectHandlerServiceRole with permissions for DynamoDB actions
Lambda Function: DisconnectHandler for handling WebSocket disconnections
Removes a connectionId from ConnectionsTable when a WebSocket connection is closed
Send Message Lambda Function:

IAM Role: SendMessageHandlerServiceRole with permissions for DynamoDB and API Gateway actions
Lambda Function: SendMessageHandler for sending messages to connected clients
Retrieves all connectionIds from ConnectionsTable
Sends a message to each connected client using ApiGatewayManagementApi
Default Lambda Function:

IAM Role: DefaultHandlerServiceRole with permissions for API Gateway actions
Lambda Function: DefaultHandler provides information to a client when a WebSocket connection is established
Manage Connections Policy:

IAM Policy: manageConnections allowing the Lambda function SendMessageHandler to execute API Gateway actions related to managing connections
Lambda Versions and Aliases

 manage deployment and rollback.


## Project Workflow

Step 1- A WebSocket connection is established, triggering the _ConnectHandler_ Lambda function.

Step 2- The _ConnectHandler_ Lambda function adds the _connectionId_ to the _ConnectionsTable_ in DynamoDB.

Step 3- If a WebSocket connection is closed, the _DisconnectHandler_ Lambda function removes the _connectionId_ from the _ConnectionsTable_.

Step 4- The _SendMessageHandler_ Lambda function can be invoked to send messages to all connected clients by iterating through _connectionIds_ in the _ConnectionsTable_.

Step 5- The _DefaultHandler_ Lambda function provides information to a client when a WebSocket connection is established.

The Auto-Scaling configurations ensure that DynamoDB read & write capacities scale based on predefined metrics. The IAM roles and policies control access to DynamoDB and API Gateway actions for the Lambda functions.

## Design Considerations

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

**_Event-Driven Architecture pattern / Pay-as-You-Go compute :_** Saves on Idle resources, and thus ensuring an efficient resource utilisation.

**_Fine-Tuning Auto-Scaling Configurations:_** Especially in the case of sporadic workloads, it has the ability to scale down as well. 


#### Performance Optimization 

_Rate Limiting in API Gateway:_ Message rate limiting enables the control of the load on the downstream systems that process the messages.

_Choice of WebSocket APIs over REST APIs:_ Web-Socket API optimises performance by establishing a long-lived, persistent connections. Eliminating the overhead involved in establishing connections frequently.  

_NoSQL Database as a connection registry:_  DynamoDB would be well-suited to handle Connection Metadata here, the low latency access and 


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


