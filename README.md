# DynamoWave Chat - A serverless, real-time chat Application

DynamoWave Chat is a modern and scalable serverless real-time chat application. 

It's built on top of AWS Services --> Lambda, DynamoDB & API Gateway

➡️ **Our core focus here:** _Enhancing the application to ensure a  high-performance delivery._

</br>


## System Architecture & Components

<img width="927" alt="image" src="https://github.com/TanishkaMarrott/ServerlessChatApp-WebSocket-API-Lambda-DynamoDB-Integration/assets/78227704/afed5865-ebe0-4292-b402-b74216650655">

</br>

## **How does the workflow look like?**

                  Establishes the websocket connection
                        ⬇️
                  Triggers ConnectHandler
                        ⬇️
                  Inserts connectionId into ConnectionsTable 
                        ⬇️
                   Notifies the client when we're done with establishing a connection
                        ⬇️
                  SendMessageHandler - iterates through connectionIds + sends messages to connected clients
                        ⬇️
                  When triggered, DisconnectHandler removes the connectionId from ConnectionsTable
                        ⬇️
                  Connection closes
                        

</br>

### Services we've used plus their purpose

We'll quickly define the purpose of each component in our architecture:-

</br>

| Services we've used        | Identifiers    | Purpose --> Why we've used?                       |
|--------------------|---------------------|-------------------------------|
| _API Gateway_  | _`Web-socket-api`_      | ➡️ **Real-time communication** in our application|                     
| _DynamoDB_     | _`ConnectionsTable`_    | **Our connection registry**. This is for tracking & managing connections  |
| _AWS Lambda_   | _`ConnectHandler`_      | --> Every new connection must be recorded --> **Helps us ensure we're good from an operational standpoint** |
|                | _`DisconnectHandler`_   | ▶️ Removing entries from our table wrt inactive connections |
|                | _`SendMessageHandler`_  | --> We'll need this for reliable communication across|
|                | _`DefaultHandler`_      | Helps notify our client when we're through with establishing a connection   |


</br>

## Design considerations

## _How did we improvise on the application's availability?_

</br>

**1 --> We've set reserved concurrency for important Lambdas.**     

Rationale:-        
Our critical lambdas would always have access to sufficient compute for operational functionality / Service continuity

</br>
 
> ➡️ **I wanted to prevent critical Lambdas from being throttled during peak times.  There shouldn't be any sidelining due to resource contention among other running lambdas**👍

</br>


 **2 -->  We need to be cognizant of the data durability aspect as well.** in event of accidental deletes. Our recovery mechanism to retrieve data within the last 35 days.    🏁  Hence, **have enabled Point-In-Time-Recovery for our DynamoDB** Table

</br>


**3 --> Our gateway should be capable of sustaining backpressure scenarios**. Our backend services won't be overwhelmed.  (Because we've limited the rate of incoming connections) 💡

>   ➡️ This means that our API will remain responsive to legit users. **Helps us safeguard against a DDoS**

</br>

**4 --> Multi-AZ Deployments => Data Redundancy => High Availability**
--> DynamoDB automatically replicates data across AZs  

</br>

---

</br>

## _Cost-effective Scalability. How?_

1 --> One of our objective was that our data store should automatically adjust to match the workload. This means configuring Auto-Scaling was essential for DynamoDB      

 **We've included <ins>Auto-scaling policies for both RCUs and WCUs.</ins>** I mean the Read and Write Capacity units
 
> --> It scales up to handle the increased traffic and down to reduce our costs, when there's a lower demand. 👍

</br>

2 --> **We had initially configured Provisioned Concurrency for lambdas** as well. We had to keep some number of execution environments pre-ready, That's actually called "Warming up the Function instances" ✅               

> ▶️ **Prewarming a set of lambda instances 🟰 Reduces cold Starts 🟰 Reducing latency**

but I had to answer this question..

</br>

###  How could we actually optimise for performance in Lambda (while still taking the costs into consideration)

We had two options:-

####  _Approach 1 :- Through Provisioned Concurrency_

 This actually means that a certain number of exec. environments - or rather, lambda instances would be running **at all times**

</br>

> 🚩Potential Red Flag:- **We're incurring charges for uptime irrespective of the actual usage.** 

</br>

**_Scenario where this would work:-_**     

Where **absolutely zero cold starts** are essential, and we need to minimize latency at all costs. Also, **in cases where we've got predictable and consistent traffic** patterns.

</br>

--

#### _Our approach -> Implementing a custom Lambda Warmer_

Why?

1 ➔ **Our application had sporadic usage patterns** 

2 ➔ I **could not compromise on my performance-critical aspects.** For me, application execution is equally important. 

### How did we solve this challenge?

Implementing a custom Lambda Warmer. 💡

Step 1 --> We've added a new Lambda function specifically designed to warm up our critical functions. ➤ Configured to invoke the critical functions **in a manner that "mimics typical user interactions" without altering my application state.**

Step 2 --> **Configured a CloudWatch event that triggers the warmer function**           
That could be either every 5 minutes, or fixed, at a time when peak usage is anticipated.

Step 3 --> We needed IAM Role and Policy that grants the warmer function permission to invoke other Lambda functions and log to CloudWatch

</br>

### _How exactly is Provisoned Concurrency different from the reserved counterpart?_

</br>

 Point 1 --> **When I'm talking about Provisioned Concurrency, it all about eliminating cold starts**, reducing _the initialisation latency_. While **reserved Concurrency is about ensuring we've got a certain portion of the Total Concurrency dedicated** to this lambda.

 Point 2  --> **Provisioned concurrency is geared towards enhancing performance**, while  **reserved counterpart is about managing resource limits.** 

>  Prevents a lambda function from consuming too many resources. 👍

 Point 3 --> **PC means you're incurring costs of keeping such instances ready at all times**, while **RC means you've sanctioned limits, no costs per se** 


## Security 

_**Lambda Authorizer - API Gateway Authorization:**_
IAM authorization is implemented for the $connect method using Lambda Authorizer in API Gateway, ensuring secure and controlled access.

_**Fine-grained Access Control:**_ Have granted the least privilege access to resources. Lambda functions and DynamoDB tables are secured with fine-grained permissions, ensuring data integrity and confidentiality.

</br>





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


