# DynamoWave Chat - A serverless, real-time chat Application

DynamoWave Chat is a modern and scalable serverless real-time chat application. 

It's built on top of AWS Services --> Lambda, DynamoDB & API Gateway

➡️ **Our core focus here:** _Enhancing the application from a non-functional standpoint --> Making it Scalable + Performant._

</br>

## System Architecture & Components

<img width="500" alt="image" src="https://github.com/TanishkaMarrott/ServerlessChatApp-WebSocket-API-Lambda-DynamoDB-Integration/assets/78227704/afed5865-ebe0-4292-b402-b74216650655">

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

</br>

> Please make sure to check out the cloudFormation template above, for the initial configurations plus the deployment

</br>

| Services we've used        | Identifiers    | Purpose --> Why we've used?                       |
|--------------------|---------------------|-------------------------------|
| _API Gateway_ | Web-socket-api      | ➡️ **Real-time communication** in our application|                     
| _DynamoDB_     | ConnectionsTable    | **Our connection registry**. This is for tracking & managing connections  |
| _AWS Lambda_   | ConnectHandler      | --> Every new connection must be recorded --> **Helps us ensure we're good from an operational standpoint** |
|                | DisconnectHandler   | ▶️ Removing entries from our table wrt inactive connections |
|                | SendMessageHandler  | --> We'll need this for reliable communication across|
|                | DefaultHandler      | Helps notify our client when we're through with establishing a connection   |

</br>

Now, that we're through with the functionality, let's now shift our attention to the NFRs


</br>

## Design considerations:-

### How did we improvise on the application's availability/ reliability?

**1 --> We've set reserved concurrency for our important Lambdas.**        
 
Our critical lambdas _should_ always have access to sufficient compute for _operational functionality / Service continuity purposes,_
We're ensuring we've got a certain "quota" of concurrency apportioned for critical lambdas. This'll enable a very _fair distribution of compute_ amongst all the lambdas. 

</br>
 
> This helps us prevent critical Lambdas from being throttled during peak times. I'd say that we're "allocating" a portion of the total concurrency to the speciifc lambda function. It'll always have the resources it needs to run/ function smoothly 👍

</br>

 **2 -->  We need to be cognizant of the data durability aspect as well.** 
 
 </br>
 
> We'll enable PITR for our DynamoDB Table --> Point-In-Time Recovery. Reason? It'll enable us to restore data to any second in the last 35 days.
>  This means we're improvising on the data's availability and fault-tolerance capabilities, Wherein there might've been situations wherein data is accidentally overwritten or is deleted.
> 
> In mission-critical scenarios, we shouldn't rely upon traditional backup methods, (that're scheduled at a fixed time of the day). Once, we've enabled PITR - It'll capture changes to the dynamo table, continuously, allowing us to retrieve up-to-date information, --> Benefit? We've negated any potential data loss that might occur.
> Plus, this doesn't involve operational overhead or costs associated with capacity planning/ over-provisioning. It essentially _simplifies_ the entire process of reverting back to our desired state👍

 </br>

**3 --> Our gateway should be capable of sustaining backpressure scenarios**. Our backend services won't be overwhelmed.  (Because we've limited the rate of incoming connections through both API Throttling & Rate Limiting.

</br>

> Why exactly did we implement this?
>          
> --> We're defining a threshold on the maximum number of requests that can hit the gateway per second --> max number of requests per second)
> --> Plus, a cap on the total number of requests originating from a particular client --> in a specific time window. There's an advantage to it, from an availability standpoint, Requests from a single client won't bog down/ overwhelm downstream services.
> 
> ➡️ So, The API would stay responsive to legitimate users, Plus, helps us avert a potential DDoS, that might bring down our entire system.

</br>

**4 --> Multi-AZ Deployments => Data Redundancy => High Availability**

--> We're resilient to zonal failures _in a region_ 👍

</br>

> This is a very use-case specific pointer. Had we been dealing with production systems, wherein you need super-high availability across AWS Regions,
>
>  I'd suggest opting in for DynamoDB global tables.  --> For scenarios like:-                     
> i. You've got a geographically distributed user base, and you'd want the data to be positioned near your users          
> ii. it's a mission critical application, and needs to be available even in the event of a regional outage.     
> iii. There're some regulatory compliance commitments, due to which data should not leave a region

> Something important that I'd like to highlight:- 👉 it _necessitates_ the need of deploying other supporting components in multiple regions as well --> for you to have a full secondary failover mechanism in another region, up and running.
>
> I'd say that this is a _pure_ cost - availability tradeoff. We should be carefully weigh if the expenses actually justify/ weigh against the needs of our application 📍

</br>

### Code optimisations that'll help enhancing Lambda from a reliability standpoint


╰┈➤ We've incorporated some error handling mechanisms within our function logic, This will make sure we're preventing any potential errors/ issues from cascading down, --> Errors can be gracefully handled by our lambda, --> Application's stability ++ 👍

╰┈➤ We felt it would be necessary for our critical lambda - "SendMessageHandler" to recover from transient/ temporary errors, for instance, network communication errors, or DB Operations, This way we'd be increasing the probability of a successful message delivery.

</br>

> Because, it'll *re-attempt* the operations multiple times, before it's considered failed. 

</br>

╰┈➤ Incessant retries would create a scenario wherein the function might enter into an infinite loop of faiures. And we should prevent this at all costs. Hence, we had to incorporate "exponential backoffs", as a part of our code, along with the retry mechanisms. This means we'd be iteratively increasing the time interval between two subsequent retries. 👍 

</br>

> So, we're not only giving the error more time to resolve, we're also reducing backpressure on our downstream systems. This is what I call a Graceful Error Retry mechanism 👍

</br>

╰┈➤ An additional enhancement we'd consider making in future (as a part of refining my architecture even further), would be to set up a DLQ - Dead Letter Queue to store failed delivery messages.

 --> This would mean **Zero data loss**. Plus, provide us with a potential opportunity to re-process and analyse these messages at a later point in time. 

</br>

## If I were to improvise on API Gateway's availability further:-

</br>

> The API Gateway might turn out to be a Single Point of Failure (SPOF), in your system... And if you're like me, who aims for a truly resilient system, you won't let that happen 🙂

</br>

Solution:-- Even though API Gateway is a managed service [ its inherently resilient to zonal failures ], there might be situations wherein it would be imperative to embed regional redundancy for the API Gateway. 

📌 This could be done by deploying the API gateway in multiple regions, and then utilising Route 53 for a DNS Failover. 

</br>

> I mean configuring a DNS health check to automatically failover to the API Gateway in the secondary region. --> a full secondary failover
> (We'd also have to ensure that the supporting components are up and running in the secondary region!)
> 
> More of a cost-redundancy tradeoff here.
>  Will need to weigh in the benefits against the potential costs incurred and if it really justifies against the current needs of the application 👍

</br>

## Cost-effective scalability. How?

1 --> The first thing that comes to my head :- Adaptive / Dynamic auto-scaling for our DynamoDB. 

_Benefit it brings in:-_ Reduced Costs 👍 DynamoDB would automatically adjust its capacity in response to the fluctuating workload. This means it's making resource utilisation all the more efficient. 👍

</br>

A common question I've often heard:-

### Q - We need to choose between On-Demand Throughput v/s Provisoned Throughput + Auto-scaling. Which combination should we opt for and why?

</br>

My answer to this :-

 i. We would need to consider the price point here. 
The "Per- Unit cost" for the On-Demand Mode turns out to be more expensive, than it's provisioned counterpart + Autoscaling.

</br>

 > Yes, this was the _catch_ here.
> It's absolutely wonderful when you've got unpredictable access patterns, and capacity planning looks difficult.
>
> But since it charges you on the actual read/ writes, the cost per unit capacity, turns out to be way higher.
>
> If we'd be in a scenario, where there's consistent traffic coming in, levels are pretty much predictable, I'll advise it'll be way more cost-effective to go with a baseline set up for provisioned read./ write units plus auto-scale based on utilisation thresholds. 💡

</br>

--

2 --> We had to eliminate lambda cold starts for improvising on the performance plus scalability of the application

> The simple equation, I often mention :-
>   
> **Prewarming a set of lambda instances = Reduces cold starts = Reduces latency** 👍

</br>

###  How could we actually optimise for performance in Lambda (while taking costs into consideration)?

We had two options at hand:-

###  Approach I --> Through setting provisioned concurrency

We'd have a certain number of execution environments - or rather, lambda instances would be running **at all times**

</br>

> 🚩Potential Red Flag:- **We're incurring charges for uptime irrespective of the actual usage.** 

</br>

**Scenario where this would work:-**     

Where **absolutely zero cold starts** are essential, and we need to minimize latency at all costs. Also, **in cases where we've got predictable and consistent traffic** patterns.

</br>

> This is something I'd definitely recommend, when we're dealing with production systems. However, given our usage pattern and subsequent impact on the price point, we'd go in for a custom lambda warmer for now

</br>

### Our approach --> Implementing a custom Lambda Warmer

Why?

1 ➔ **Our application had sporadic usage patterns** 

2 ➔ I **could not compromise on my performance-critical aspects.** For me, application execution is equally important. 

</br>

### How did we solve this challenge?

Implementing a custom Lambda Warmer. 💡

--> We've added a new Lambda function specifically designed to warm up our critical functions. ➤ Configured to invoke the critical functions **in a manner that "mimics typical user interactions" without altering my application state.**

 --> **Configured a CloudWatch event that triggers the warmer function**           
That could be either every 5 minutes, or fixed, at a time when peak usage is anticipated.

 --> We needed IAM Role and Policy that grants the warmer function permission to invoke other Lambda functions and log to CloudWatch

</br>

### How exactly is Provisoned Concurrency different from the reserved counterpart?

 Point 1 --> **When I'm talking about Provisioned Concurrency, it all about eliminating cold starts**, reducing _the initialisation latency_. While **reserved Concurrency is about ensuring we've got a certain portion of the Total Concurrency dedicated** to this lambda.

 Point 2  --> **Provisioned concurrency is geared towards enhancing performance**, while  **reserved counterpart is about managing resource limits.** 

</br>

>  Prevents a lambda function from consuming too many resources. 👍 --> Fair resource utilisation amongst functions

</br>

 Point 3 --> **PC means you're incurring costs of keeping such instances ready at all times**, while **RC means you've sanctioned limits, no costs per se** 

</br>

## From a security standpoint

Point 1 --> We've defined API Gateway resource Policies to enforce requests by denying requests that do not use HTTPS.

</br>

>We had a couple of options here, first through resource-based policies, (specifying the gateway ARN for the resource) This denies requests upfront if "aws:securetransport" = false.
>
>Second, we could have a custom domain, attach a ssl/ tls certificate from certificate manager, and update the DNS Settings. It does achieve the objective of allowing only HTTPS requests.
>
>But too much of a roundabout, We'd rather go in for the first alternative.
          
Point 2 --> Data Encryption at rest through KMS Encryption (DynamoDB)

</br>

 We decided against implementing FGAC or fine-grained access control for DynamoDB, It's main purpose is controlling access to specific attributes, or specific database items.        
 
> --> Our schema is pretty simple and straightforward. A single-attribute schema not at all warrants the kind of complexity FGAC brings in. We went ahead with standard IAM policies with table-level access controls
> 
>  --> But, yes, it's super helpful when we've got  multiple attributes or maybe need specific teams/roles to access only certain partitions of the data. Or we're seeking absolutely locked down security at a very granular level - (Something that IAM Policies lack - IAM Policies operate only at the table level --> FGAC on the other hand operates at the db item / attribute level) - Useful when we're working under strict compliance requirements 👍

</br>

Point 3 --> We've pruned down IAM policies for the service roles attached to the lambdas, dynamo; strictly to what the component _actually_ needs for its functioning/ access. Lesser risk of Privilege Escalation

Point 4 -->  Implementing throttling / rate limiting mitigates a potential DDoS, We've mentioned this above in the availability section too --> this is because we're controlling the number of requests that a user / bot can hit the gateway ✔️

> If we'd be looking at amping up the security aspects of our application, Cognito User Pools would be a better bet, It's not just a solution for user creation/ managaement, it lets you authenticate requests, has MFA, password recovery. 👍

</br>

### **What kind of refinements could make my current design even more secure ?**

I'd consider implementing a WAF on top of the gateway. A significant enhancement this brings along is that it protect's application availability, --> it prevents excessive consumption of resources downstream + averts potential security compomises 👍

</br>

> We could utilise AWS managed rules here, they are pre-configured web security rules, they're designed + maintained by AWS plus they're automatically updated from time to time, This means it abstracts out the need of manual maintenance of IP sets, plus simplifies deployment of such ACLs, redcuing teh operational burden of maintaing these sets.      
> I feel there might be certain situations, wherein we need to explicitly block specific IP addresses, this does necessitate the need of using a more comprehensive solution having both Custom Rules plus Managed Rule groups in a Web ACL,        
> This would then make up for a perfect security enhancement, wherein both security and operational overhead associated, have been considered 👍

On a side note, I'd recommend logging these metrics to CloudWatch, And  incrementing the CW metric , in case, it matches specific regex/ attack patterns, You could also configure a CloudWatch Alarm to trigger off a notification via a SNS topic / mail/ message, in the event of the threshold being breached. --> Real -Time Alerting 📌

</br>

### **How can I achieve a better Performance Optimisation, while maintaining costs?**

A consistent high-volume read/write traffic would be a signal for me to explore DynamoDB batch operations. Particularly, for bulk write or delete operations, such as managing multiple connections in the ConnectHandler and DisconnectHandler Lambda functions. For individual, sporadic requests, where the application rarely receives bursts of traffic, the system would need to wait for a certain number of requests to accumulate before processing them together. In a scenario with sporadic requests, this delay might be noticeable. Not recommended for applications with low, sporadic traffic.

</br>

## Contributions 

Contributions are most welcome!

--> If you've got  suggestions on how I could further improvise on the architectural / configurational aspects, please feel free to drop a message on tanishka.marrott@gmail.com. I'd love to hear your thoughts on this!

</br>

## Credit Attribution

I'm grateful to [AWS](https://aws.amazon.com/) for providing an excellent blog that served as the foundation for this project. This -> [https://docs.aws.amazon.com/apigateway/latest/developerguide/websocket-api-chat-app.html], was instrumental in guiding the implementation of the base architecture.


