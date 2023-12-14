## DynamoWave Chat - Serverless Real-time Chat Application  

DynamoWave Chat is a serverless, real-time chat application powered by AWS Lambda, DynamoDB, and WebSocket API.
It leverages the WebSocket API to facilitate seamless real-time communication, delivering users an interactive and engaging chat experience. This project is designed with a focus on System Design Principles, ensuring high availability, scalability, security, cost-optimization, and top-notch performance.


### System Design Principles

#### Serverless Architecture:
Follows a Serverless Architecture Pattern. Utilised AWS Lambda for Compute, DynamoDB for a NoSQL database, and WebSocket API for handling Real-time Connections. No infrastructure Provioning / Management Overhead involved. Makes it massively scalable and reduces associated costs.

#### High Availability 

_Multi-AZ Deployments (Built-in) :_
DynamoDB supports Multi-AZ deployments, automatically replicating data across multiple AZs within a region.
Lambda & API Gateway are also resilent to zonal failures.

_Reserved Concurrency in Lambda:_ 
Helps in controlling the maximum number of Concurrent Invocations of a Lambda Function. We won't lose requests due to other functions consuming all of the available concurrency.

_Throttling in API Gateway:_ Configured Throttling for API Gateway. This helps in controlling both the volume of API requests hitting the API Gateway, preventing abuse & mitigating a DDoS Attack. The APIs thus wouldn't be overwhelmed by too many requests.

#### Scalability 

_Provisioned Concurrency for Lambda:_ Reduces cold start latency, ensuring consistent & predictable performance. Pre-warming a set of Lambda function instances helps improve Responsiveness & Scalability during traffic spikes

_Provisioned Throughput for DynamoDB :_ Configured DynamoDB Provisioned Throughput with RCUs and WCUs, for a consistent and predictable performance.

 _Automatic Scaling for DynamoDB:_ Implemented Dynamic Scaling through Scaling Targets and Policies for automatic, workload-responsive adjustments. Optimizing resource-utilization and responsiveness.


#### Security 

_Lambda Authoriser for API Gateway Authorization:_  
_Fine Grained Access Control using IAM Roles & Policies:_ Have granted the least privilege access to resources. Lambda functions and DynamoDB tables are secured with fine-grained permissions, ensuring data integrity and confidentiality.

#### Cost-Optimization 

Pay-as-You-Go compute offered by Lambda enables us to optimize on costs, 
An Event-Driven Architecture pattern & Auto-Scaling would ensure we reduce idle resources, especially in the case of sporadic workloads. 
Abstracting out the underlying infra helps in reducing associated costs. 


#### Performance Optimization 

_Rate Limiting in API Gateway:_ Message rate limiting enables the control of the load on the downstream systems that process the messages.

_Choice of WebSocket APIs over REST APIs:_ Web-Socket API optimises performance by establishing a long-lived, persistent connections. Eliminating the overhead involved in establishing connections frequently.  

_NoSQL Database as a connection registry:_  

_

### System Architecture

_WebSocket API:_ Helps in Bi-directional communication, without the client having to poll for messages. Ultra-low latency for Real-Time Communication, 

_AWS Lambda:_ Executes functions in response to WebSocket events, managing chat-related logic.

_DynamoDB_: Stores and retrieves chat messages, ensuring scalability and low-latency access.

### Enhancements for the Current Architecture -  Thoughts and Strategies for Improvement

How can I make my current architecture resilent to regional failures? 
Use of CDNs for geographically dispersed users, deploy critical components in multiple regions, enable PITR for Data Stores as a Backup-and-Restore Mechanism, Cross-region replicas, DynamoDB Global Tables, 

Use Route 53 health checks to control DNS failover from an API Gateway API in a primary region to an API Gateway API in a secondary regionConfigure Route53 DNS Health check to failover to an API Gateway in a secondary region, (We will have to make sure we've got all the required resources in that regions), Cost-Redundancy Tradeoff.

What other strategies can be implemented to make this architecture even more scalable?
Custom Auto-Scaling Logic using CloudWatch Alarms

How can I make this even more secure & withstand potential threats?

How might I further optimize costs within my architecture while maintaining performance?

What operational efficiencies can be introduced?



### Contributing 
Contributions are most welcome, feel free to submit issues, feature requests, or pull requests. 

#### Acknowledgments
Special thanks to [AWS](https://aws.amazon.com/) for providing an excellent tutorial that served as the foundation for this project. The original tutorial, [https://docs.aws.amazon.com/apigateway/latest/developerguide/websocket-api-chat-app.html], was instrumental in guiding the implementation of the base architecture.


