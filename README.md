## DynamoWave Chat - Serverless Real-time Chat Application  🚀

DynamoWave Chat is a serverless, real-time chat application powered by AWS Lambda, DynamoDB, and WebSocket API.
DynamoWave Chat leverages the WebSocket API to facilitate seamless real-time communication, delivering users an interactive and engaging chat experience. This project is designed with a focus on System Design Principles, ensuring high availability, scalability, security, cost-optimization, and top-notch performance.


### System Design Principles

#### Serverless Architecture:
The entire architecture is Serverless, utilizing AWS Lambda for Compute, DynamoDB for a NoSQL database, and WebSocket API for handling Real-time Connections. No infrastructure Provioning / Management Overhead involved.

#### High Availability 

_Multi-AZ Deployments:_
DynamoDB supports Multi-AZ deployments, automatically replicating data across multiple Availability Zones (AZs) within a region.
Lambda runs your function in multiple Availability Zones to ensure that it is available to process events in case of a service interruption in a single zone.
The core services used here are implictly resilent to Zonal Failures.

_Lambda Reserved Concurrency:_ Helps in controlling the maximum number of concurrent invocations of a Lambda Function, Helps in throttling and prevents Resource Exhaustion


#### Scalability 

_Lambda Provisioned Concurrency:_ Reduces cold start latency, ensuring consistent & predictable performance. Pre-warming Lambda function instances helps improve Responsiveness & Scalability during traffic spikes

_DynamoDB Provisioned Throughput with Automatic Scaling:_ Configured DynamoDB Provisioned Throughput with RCUs and WCUs, implementing Dynamic scaling through Scaling Targets and Policies for automatic, workload-responsive adjustments. Optimizing resource-utilization and responsiveness.


#### Security 
_API Gateway Authorization & Authentication:_ using Cognito Authoriser

IAM roles and policies are used to grant the least privilege access to resources. Lambda functions and DynamoDB tables are secured with fine-grained permissions, ensuring data integrity and confidentiality.

#### Cost-Optimization 
By adopting a serverless architecture, DynamoWave Chat optimizes costs through a pay-as-you-go model. DynamoDB's on-demand capacity and Lambda's event-driven model contribute to cost efficiency.

#### Performance Optimization 
Web-Socket APIs optimises performance, by establishing long-lived, persistent connections. It reduces the performance bottleneck that a stateless protocol poses. DynamoDB provides low-latency data access, hence and ideal Data Store here.

### System Architecture

_WebSocket API:_ Helps in Bi-directional communication, without the client having to poll for messages. Ultra-low latency for Real-Time Communication, 

_AWS Lambda:_ Executes functions in response to WebSocket events, managing chat-related logic.

_DynamoDB_: Stores and retrieves chat messages, ensuring scalability and low-latency access.

### Contributing 🤝
Contributions are welcome! Feel free to submit issues, feature requests, or pull requests. 

#### Acknowledgments
Special thanks to [AWS](https://aws.amazon.com/) for providing an excellent tutorial that served as the foundation for this project. The original tutorial, [https://docs.aws.amazon.com/apigateway/latest/developerguide/websocket-api-chat-app.html], was instrumental in guiding the implementation of the base architecture.


