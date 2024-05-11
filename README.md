# DynamoWave Chat - A serverless, real-time chat Application

DynamoWave Chat is a modern and scalable serverless real-time chat application. 

It's built on top of AWS Services --> Lambda, DynamoDB & API Gateway

➡️ **Our core focus here:** _Enhancing the application to ensure a  high-performance delivery._

</br>

## System Architecture & Components

<img width="927" alt="image" src="https://github.com/TanishkaMarrott/ServerlessChatApp-WebSocket-API-Lambda-DynamoDB-Integration/assets/78227704/afed5865-ebe0-4292-b402-b74216650655">

</br>

## **How does the workflow look like?**

                  We'll first establish the websocket connection
                        ⬇️
                  It'll trigger the ConnectHandler lambda
                        ⬇️
                  This function would insert connectionId into the ConnectionsTable 
                        ⬇️
                   The client would be notified when we're done with establishing a connection
                        ⬇️
                  SendMessageHandler lambda would iterate through connectionIds + sends messages to the connected clients
                        ⬇️
                  Once the session ends --> DisconnectHandler function would remove the connectionId from the registry
                        ⬇️
                  The connection closes now
                        

</br>

## Services we've used plus their purpose

We'll quickly define the purpose of each component in our architecture:-

> Please make sure to check out the cloudFormation template above, for the initial configurations plus the deployment
Now, that we're through with the functionality, let's now shift our attention to the NFRs


| Services we've used        | Identifiers    | Purpose --> Why we've used?                       |
|--------------------|---------------------|-------------------------------|
| _API Gateway | _Web-socket-api_      | ➡️ **Real-time communication** in our application|                     
| _DynamoDB     | _ConnectionsTable_    | **Our connection registry**. This is for tracking & managing connections  |
| _AWS Lambda   | _`ConnectHandler`_      | --> Every new connection must be recorded --> **Helps us ensure we're good from an operational standpoint** |
|                | _`DisconnectHandler`_   | ▶️ Removing entries from our table wrt inactive connections |
|                | _`SendMessageHandler`_  | --> We'll need this for reliable communication across|
|                | _`DefaultHandler`_      | Helps notify our client when we're through with establishing a connection   |

</br>

## Design considerations:-

### How did we improvise on the application's availability?

**1 --> We've set reserved concurrency for our important Lambdas.**         
Our critical lambdas _should_ always have access to sufficient compute for operational functionality / Service continuity purposes,
We're ensuring we've got a certain quota of concurrency apportioned to the lambda. this'll enable a very fair distribution of compute amongst all the lambdas. 

</br>
 
> This helps us prevent critical Lambdas from being throttled during peak times. I'd say that we're "allocating" a portion of the total concurrency to the spciifc lambda function. It'll always have the resources it needs to run 👍

</br>

 **2 -->  We need to be cognizant of the data durability aspect as well.** 
We've enabled Point-In-Time recovery -- Add about  PITR

 </br>

**3 --> Our gateway should be capable of sustaining backpressure scenarios**. Our backend services won't be overwhelmed.  (Because we've limited the rate of incoming connections) We've implemented both API Throttling & Rate Limiting.

</br>

> By having both of these defined:-                   
> --> We're predefining a threshold on the maximum number of requests that can hit the gateway per second --> max number of requests per second) 
> --> Plus, a cap on the total number of requests originating from a particular client --> in a specific time window. This has a strategic advantage to it, from an availability standpoint, This'll prevent downstream services from being overwhelmed.          
> ➡️ The API stays responsive to legitimate users, Plus, helps us avert a potential DDoS, that might bring down our entire system.

</br>

**4 --> Multi-AZ Deployments => Data Redundancy => High Availability**
--> DynamoDB automatically replicates data across AZs. This means we're resilient to zonal failures _in a region_

</br>

> This is a very use-case specific pointer. Had we been dealing with production systems, wherein you need super-high availability across AWS Regions, I'd suggest opting in for DynamoDB global tables.  --> For scenarios like:-                     
> i. You've got a geographically distributed user base, and you'd want the data to be positioned near your users          
> ii. it's a mission critical application, and needs to be available even in the event of a regional outage.     
> iii. There're some regulatory compliance commitments, due to which data should not leave a region
>
> Something important to note here, it _necessitates_ the need of deploying other supporting components in multiple regions as well --> for you to have a full secondary failover mechanism in another region, up and running.
>
> I'd say that this is a _pure_ cost - availability tradeoff. We should be carefully weigh if the expenses actually justify/ weigh against the needs of our application

</br>

### Cost-effective Scalability. How?

1 --> I'd come across adaptive auto-scaling for DynamoDB, and I knew I had to utilise this

>  Our application, has sporadic usage patterns, 

 To achieve this, we implemented auto-scaling policies for both Read Capacity Units (RCUs) and Write Capacity Units (WCUs) on DynamoDB.**

   > 

2 --> We had to eliminate lambda cold starts for improvising on the performance plus scalability of the application

> The simple equation, I often mention :-        
> Prewarming a set of lambda instances = Reduces cold starts = Reduces latency 👍


###  How could we actually optimise for performance in Lambda (while still taking the costs into consideration)?

We had two options at hand:-

####  Approach I was through setting provisioned concurrency

We'd have a certain number of execution environments - or rather, lambda instances would be running **at all times**

</br>

> 🚩Potential Red Flag:- **We're incurring charges for uptime irrespective of the actual usage.** 

</br>

**Scenario where this would work:-**     

Where **absolutely zero cold starts** are essential, and we need to minimize latency at all costs. Also, **in cases where we've got predictable and consistent traffic** patterns.

</br>

> This is something I'd definitely recommend, when we're dealing with production systems. However, given our usage pattern and subsequent impact on the price point, we'd go in for a custom lambda warmer for now

</br>

#### Our approach --> Implementing a custom Lambda Warmer

Why?

1 ➔ **Our application had sporadic usage patterns** 

2 ➔ I **could not compromise on my performance-critical aspects.** For me, application execution is equally important. 

</br>

#### How did we solve this challenge?

Implementing a custom Lambda Warmer. 💡

Step 1 --> We've added a new Lambda function specifically designed to warm up our critical functions. ➤ Configured to invoke the critical functions **in a manner that "mimics typical user interactions" without altering my application state.**

Step 2 --> **Configured a CloudWatch event that triggers the warmer function**           
That could be either every 5 minutes, or fixed, at a time when peak usage is anticipated.

Step 3 --> We needed IAM Role and Policy that grants the warmer function permission to invoke other Lambda functions and log to CloudWatch

</br>

#### How exactly is Provisoned Concurrency different from the reserved counterpart?

 Point 1 --> **When I'm talking about Provisioned Concurrency, it all about eliminating cold starts**, reducing _the initialisation latency_. While **reserved Concurrency is about ensuring we've got a certain portion of the Total Concurrency dedicated** to this lambda.

 Point 2  --> **Provisioned concurrency is geared towards enhancing performance**, while  **reserved counterpart is about managing resource limits.** 

</br>

>  Prevents a lambda function from consuming too many resources. 👍 --> Fair resource utilisation amongst functions

</br>

 Point 3 --> **PC means you're incurring costs of keeping such instances ready at all times**, while **RC means you've sanctioned limits, no costs per se** 

</br>

## From a security standpoint

**Lambda Authorizer - API Gateway Authorization:**
IAM authorization is implemented for the $connect method using Lambda Authorizer in API Gateway, ensuring secure and controlled access.

--> We've pruned down IAM policies for the service role
**Fine-grained Access Control:** Have granted the least privilege access to resources. Lambda functions and DynamoDB tables are secured with fine-grained permissions, ensuring data integrity and confidentiality.

</br>

## 


_**Choice of WebSocket APIs over REST APIs:**_ Web-Socket API optimises performance by establishing a long-lived, persistent connections. Eliminating the overhead involved in establishing connections frequently.  

_**NoSQL Database as a connection registry:**_  DynamoDB would be well-suited to handle Connection Metadata here, the low latency access and 

</br>

### Enhancements for the Current Architecture

**How can I make my current architecture resilent to regional failures?**

If high availability and failover capabilities are critical, the regional setup with Route 53 offers better control over traffic distribution during regional outages. Configure Route53 DNS Health check to failover to an API Gateway in a secondary region, (We will have to make sure we've got all the required resources in that regions), --> **Cost-Redundancy Tradeoff**




**What other strategies can be implemented to make this architecture even more scalable?**
Custom Auto-Scaling Logic using CloudWatch Alarms

**How can I make this even more secure & withstand potential threats?**
Use WAF on top of API gateway for enhancing the security of the current architecture. By configuring WAF ACLs and associating them with the API, we can mitigate common web exploits and protect against malicious WebSocket requests.

**How can I achieve a better Performance Optimisation, while maintaining costs?**

If we're looking at a geographically dispersed audience, and need to reduce connection times, I'd go in for the Edge-Optimised API Gateway. With Regional API Gateway, the requests traverse through the public internet, and then reach the regional endpoint. While with an Edge-optimised API gateway, the API requests travel through the CloudFront Points of Presence (PoP), before hitting the API Endpoint. The first one is thus a simpler, and an effective solution, with minimal costs & configuration. If I need more control over the routing logic, or global load balancing, and I'm more concerned about the DR Capabilities, the latter, Multiple Regional API Endpoints with Route 53 Latency-Based Routing would be my alternative here.

DAX (DynamoDB Accelerator) wouldn't be something we'd be taking about here. Caching would be ideal in cases where we need to optimise read-performance / access to some frequent data. Introducing caching in a real-time application doesn't serve the purpose - we'd be introducing unnecessary complexities and costs, without achieving something fruitful. Features like Auto-Scaling, Provisioned Throughput, have already been incorporated here. 


A consistent high-volume read/write traffic would be a signal for me to explore DynamoDB batch operations. Particularly, for bulk write or delete operations, such as managing multiple connections in the ConnectHandler and DisconnectHandler Lambda functions. For individual, sporadic requests, where the application rarely receives bursts of traffic, the system would need to wait for a certain number of requests to accumulate before processing them together. In a scenario with sporadic requests, this delay might be noticeable. Not recommended for applications with low, sporadic traffic.

I would monitor and adjust Provisioned / Reserved Concurrency (Lambda) & DynamoDB throughput based on real-time demand - through Performance Insights / CW. This would help reduce unnecessary standing costs for over-provisioning.



### Contributions 
Contributions are most welcome, feel free to submit issues, feature requests, or pull requests. 
If you've got  suggestions on how I could further improvise on the architectural / configurational aspects, please feel free to drop a message on tanishka.marrott@gmail.com. I'd love to hear your thoughts on this!

### Credit Attribution
Special thanks to [AWS](https://aws.amazon.com/) for providing an excellent blog that served as the foundation for this project. This -> [https://docs.aws.amazon.com/apigateway/latest/developerguide/websocket-api-chat-app.html], was instrumental in guiding the implementation of the base architecture.


