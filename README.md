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

### How did we improvise on the application's availability/ reliability?

**1 --> We've set reserved concurrency for our important Lambdas.**         
Our critical lambdas _should_ always have access to sufficient compute for operational functionality / Service continuity purposes,
We're ensuring we've got a certain quota of concurrency apportioned to the lambda. this'll enable a very fair distribution of compute amongst all the lambdas. 

</br>
 
> This helps us prevent critical Lambdas from being throttled during peak times. I'd say that we're "allocating" a portion of the total concurrency to the spciifc lambda function. It'll always have the resources it needs to run/ function smoothly 👍

</br>

 **2 -->  We need to be cognizant of the data durability aspect as well.** 
 
 </br>
 
> We'll enable PITR for our DynamoDB Table --> Point-In-Time Recovery. Reason? It'll enable us to restore data to any second in the last 35 days. This means we're improvising on the data's availability and fault-tolerance capabilities, Wherein there might've been situations wherein data is accidentally overwritten or is deleted. In such scenarios, we shouldn't rely upon traditional backup methods, (that're scheduled at a fixed time of the day). Once, we've enabled PITR - It'll capture changes to the dynamo table, continuously, allowing us to retrieve up-to-date information, --> Benefit? We've negated any potential data loss that might occur. Plus, this doesn't involve operational overhead or costs associated with capacity planning/ over-provisioning.
> It essentially _simplifies_ the entire process of reverting back to our desired state👍

 </br>

**3 --> Our gateway should be capable of sustaining backpressure scenarios**. Our backend services won't be overwhelmed.  (Because we've limited the rate of incoming connections through both API Throttling & Rate Limiting.

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


### Code optimisations that'll help enhancing lambda from a reliability standpoint

We had to ensure we're handling errors gracefully plus having some retry mechanisms for transient errors. 

╰┈➤ Number one, We had to ensure we've incorporated some error handling mechanisms within our function logic, This will make sure we're preventing any potential errors/ issues from cascading down, --> Errors can be gracefully handled by our lambda, --> Application's stability ++ 👍

╰┈➤ We felt it would be necessary for our critical lambda - SendMessageHandler to recover from transient/ temporary errors, for instance, network communication errors, or DB Operations, This way we'd be increasing the probability of a successful message delivery.

</br>

> Because, it'll re-attempt the operations multiple times, befor it's considered failed

</br>

╰┈➤ Incessant retries would lead us to a scenario wherein the function might enter into an infinite loop, and we'd want to prevent this, Hence, we had to make usre we've got some exponential backoffs in place too, along with the retry mechanisma, This would then iteratively increase the time interval between two subsequent retries. 

</br>

> So, we're not only giving the error more time to resolve, we're also reducing backpressure on downstream systems 👍

</br>

╰┈➤ An additional enhancement, we'd consider making in the second iteration, as a part of refining my architecture further, would be to set up a DLQ - dead Letter Queue to store failed delivery messages --> this would mean zero data loss, and also provides a potential opportunity to re-process and analyse these messages further. 

</br>

## If I were to improvise on API Gateway's availability further:-

Even though API Gateway is a managed service, as is inherently resilient to zonal failures, there might be situations wherein we'd like to implement regional redundancy for API Gateway. This could be done by deploying the gateway in multiple regions, and then utilising Route 53 for a DNS Failover. 

</br>

> I mean configuring a DNS health check to automatically failover to the API Gateway in the secondary region.
> (We'd also have to ensure that the supporting components too are up and running in another region!)
> More of a cost-redundancy tradeoff here, Will need to weigh in the benefits against the potential costs incurred, and it really justifies against the current needs of the application 👍

</br>

## Cost-effective Scalability. How?

1 --> I'd come across adaptive auto-scaling for DynamoDB, and I knew I had to utilise this            
_Benefit it brings in:-_ Helps us control on the costs, Dynamo would automatically adjust the workload based on the fluctauting requirements. This means it's making resource utilisation all the more efficient. 👍

</br>

#### We had to choose between On-demand throughput versus Provisoned Throughput + Auto-scaling. Which one did we opt while designing this?

 i. We had to consider the price point here. The "Per-unit cost" was turning out to be expensive for the on-demand mode than the provisioned counterpart + Autoscaling.

</br>

 > Yes, this was the _catch_ here. It's absolutely wonderful when you've got unpredictable access patterns, and capacity planning looks difficult. But since it charges you on the actual read/ writes, the cost per unit capacity, turns out to be way higher. If we'd be in a scenario, where there's consistent traffic coming in, levels are pretty much predictable, I'll advise it'll be way more cost-effective to go with a baseline set up for provisioned read./ write units plus auto-scale based on utilisation thresholds.

</br>

--

2 --> We had to eliminate lambda cold starts for improvising on the performance plus scalability of the application

> The simple equation, I often mention :-        
> **Prewarming a set of lambda instances = Reduces cold starts = Reduces latency** 👍

</br>

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

--> We've added a new Lambda function specifically designed to warm up our critical functions. ➤ Configured to invoke the critical functions **in a manner that "mimics typical user interactions" without altering my application state.**

 --> **Configured a CloudWatch event that triggers the warmer function**           
That could be either every 5 minutes, or fixed, at a time when peak usage is anticipated.

 --> We needed IAM Role and Policy that grants the warmer function permission to invoke other Lambda functions and log to CloudWatch

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

Point 2 --> We've pruned down IAM policies for the service roles attached to the lambdas, dynamo; strictly to what the component _actually_ needs for its functioning/ access.

> Finer the permissions, lesser the ridk of Privilege Escalation 👍

Point 3 -->  Implementing throttling / rate limiting mitigates a potential DDoS, We've mentioned this above in the availability section too --> this is because we're controlling the number of requests that a user / bot can hit the gateway ✔️


### **What kind of refinements could make my current design even more secure ?**

I'd consider using a WAF - Web Application Firewall,  on top of API gateway. By configuring WAF ACLs and associating them with the API, we can mitigate common web exploits and protect against malicious WebSocket requests.


**How can I achieve a better Performance Optimisation, while maintaining costs?**

A consistent high-volume read/write traffic would be a signal for me to explore DynamoDB batch operations. Particularly, for bulk write or delete operations, such as managing multiple connections in the ConnectHandler and DisconnectHandler Lambda functions. For individual, sporadic requests, where the application rarely receives bursts of traffic, the system would need to wait for a certain number of requests to accumulate before processing them together. In a scenario with sporadic requests, this delay might be noticeable. Not recommended for applications with low, sporadic traffic.

</br>

### Contributions 

Contributions are most welcome!

--> If you've got  suggestions on how I could further improvise on the architectural / configurational aspects, please feel free to drop a message on tanishka.marrott@gmail.com. I'd love to hear your thoughts on this!

</br>

### Credit Attribution

I'm grateful to [AWS](https://aws.amazon.com/) for providing an excellent blog that served as the foundation for this project. This -> [https://docs.aws.amazon.com/apigateway/latest/developerguide/websocket-api-chat-app.html], was instrumental in guiding the implementation of the base architecture.


