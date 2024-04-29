# Table of Contents
- [Microservices design patterns](#microservices-design-patterns)
- [Distributed message queue (like : KAFKA, RMQ)](#distributed-message-queue)
  + [KAFKA](#kafka)
  + [RabbitMQ](#rabbitmq)
- [Proxy](#proxy)
  + [Forward proxy](#forward-proxy)
  + [Reverse proxy](#reverse-proxy)
- [Caching](#caching)
- [Distributed Caching](#distributed-caching)
- [Caching Strategies](#caching-strategies)
  + [Cache Aside](#1-cache-aside)
  + [Read through cache](#2-read-through-cache)
  + [Write around cache](#3-write-around-cache)
  + [Write through cache](#4-write-through-cache)
  + [Write back or behind cache](#5-write-back-or-behind-cache)
- [JSON web token](#json-web-token)
- [OAuth](#oauth)


> # Microservices design patterns
#### ```Strangler Design Pattern```
* As we know that using decompostion and DDD pattern we can divide our monolithic service to micrtservices. so, to do this we can use strangler pattern to convert monolithic to microservices pattern we can take a controller in some place and this controller will divide the traffic to microservices and monolithic service. Gradually we will move the traffic from monolithic to microservices like 10%, 20%, 30%... so on.

#### ```SAGA design pattern```
* When we divide the monolithic to microservice then, we divide the database also, so there are to ways to divide the database either take a single DB for all services or take a seperate DB per service. But, there are some disadvantages of using a single DB that may be out of all service only one service data is very huge as comapared to other and if we want to delete a column in any table then it can be difficult as we need to care of other services, but it has some advantages, that it maintains ACID properties and also query join is easy, but its advantages are difficult for other approach like DB per service.
* So to solve these to cases in DB per service approach we use SAGA for ACID and CQRS for joins.
S<img width="1149" alt="Screenshot 2024-04-29 at 11 48 59 PM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/5652141e-668b-40cc-9d84-e92710690735">
* SAGA pattern is of two types:
  - ```Choreography``` : let say there are three services s1, s2, and s3. and there are two queues success anf fail queues. s1 updates the data and push into the success queue and then s2 listens to success queue and updates the data and so on. And if let say s2 get fails so s2 will pusblish only fail event in fail queue and s1 will roll back.
  - ```Orchestrator``` : here we have a orchestrator which is responsible for updating all the services. so, s1 will call orch. for success or failure if success then it will call s2 and s2 again will call orch. with success or failure, if let day calls with failure then orch will call s1 and with failed event and s1 will rolles back.


#### ```CQRS Pattern```
* CQRS stands for : Command(Create, Update, Delete), Query(Select), Request, Segregation.
* to overcome joins challanges, what we can do, All services will create, update and delete into their respective DBs and there will be a common DB of all data from which all service will read the data so here we perform joins. The common DB also needs to be updated to we can place a event listener or create a procedure which will trigger where there is any changes on service's DBs so it will update commond DB basically syncing.



> # Distributed message queue

#### What is message queue & why it is needed ? 

* Asynchronous nature : We can use it for sending notifications asynchronously.
* Retry capability : If consumer goes gown then we can retry the message again from the queue.
* pace matching : Lets say we have three services s1, s2 and s3. All three services wants to send messages for their respective use cases. and we also have an notification application which sends messages. s1 sends data at 10 req/sec, s2 sends data at 20 req/sec and s3 sends data at 30 req/sec, but notification service can consume 15 req/sec. So this will create a problem. To solve this problem of pace matching we can use queue in between s1, s2, s3 and notification service.
* Lets say we have N number of cars and they send their location and car_id in every 10 seconds and there is one more service which takes that data and create a dashboard out of it, but it cannot accept frequent numnber of requests as there can be huge number of cars. so for this use case we can use queue.

#### What is point-2-point and what is pub/sub ? 
<img width="1171" alt="Screenshot 2024-04-27 at 12 47 29 PM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/95c98cd3-921d-4e4a-87da-dd95565cb1d8">

* As we can see in the fig. above in P2P, a message can be consumer either by consumer 1 or consumer 2.

<img width="1170" alt="Screenshot 2024-04-27 at 12 47 36 PM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/597db4f6-d2c3-4643-933a-f8a46b1bb44e">

* As we can see in the fig. above in pub/sub, a message is consumed by the exchange then exchange send the same message to all queues attached with it and comsumer consumes messages accordingly.


### How messaging queue works ? 
#### KAFKA
* In Kafka we have below components
  ```- Producer
  - Consumer
  - Consumer group
  - Topic
  - Partition
  - offset
  - Broker (Kafka server like mysql server)
  - Cluster
  - Zookeeper```
 
<img width="983" alt="Screenshot 2024-04-27 at 1 11 10 PM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/73ea80f4-223a-4b18-87d2-6f92de555efd">

* Data is stored inside partitions and in a consumer group a consumer can consume data from one partion per topic so if there are two topics and both topics have 2 partitions then a consumer can consumer from 2 partitions one partition from every topic.
* Let say we have 4 kafka brokers and running on different machines i.e Node1, Node2, Node3, Node 4 then, a group of kafka brokers is known as cluster, and this cluster is managed by zookeeper. All brokers communicate with each other using zookeeper.

#### How message goes into the partition ? 
* Let say we have a TOPIC - A which has three partitions - p0, p1, p2 then how messages will go into these partions.
* A message can have below attributes
  - Key : String/Id (Not mandatory) -
  - Value : Actual message (Mandatory)
  - Partition : partition name (Not mandatory)
  - Topic : Topic Name (Mandatory)
* If we pass the key which is id then kafka create hash out if it and according to has it puts data in the partiton and if we don't pass the Key then it looks for partition and push data to the given partition and if partition is also not given then it uses round-robin algo to push data in partitions.

#### Message retention
* Kafka supports different message retention policies such as time based rentention and size based retention.
* Messages can be retained for a specific duration or until log segment reaches a certain size.
* This allows kafka to store messages for a configurable period, enabling consumers to replay messages if needed.

#### Use of offset ?
* Continuing with above example, Let say we have a consumer group A which only one consumer initially and it is reading from topic A - partition0.
* Zookeeper maintains this info - like consumer group A has consumer 1 reading from TOPIC A - partition 1 and it also has a committed offset.
* Now let say consumer 1 have successfully read messages upto index 3 then committed offset will become 3 in zookeeper for a consumer group A- consumer 1.
* Now if anyhow consumer 1 goes down then consumer 2 will come up inside consumer group A and it will start reading from index 4 by communicating with zookeeper and zookeeper will tell commited offset is 3 you can start from index 4 of partition 0 of TOPIC - A


* Now let say we have a TOPIC - A with partitions p0 and p1 and we have three brokers b1, b2 and b3 (i.e cluster). broker b1 has TOPIC - A with partion p0 and broker b2 has TOPIC - A with partition 1 . Even with same topic partitions can reside into different kafka servers. here b1 hosting partion p0 and broker 2 hosting partition p1.
* Now If partition p0 goes down then what will happen to the message inside it? so for it we have replica. so here for TOPIC - A partition p0 a replica is placed inside broker b1 and for TOPIC - A partition p1 a replica is placed inside broker b3. Those who are not replica are known as leaders and their replicas are known as followers.
* Read/Write happens only through a leader and when a leader goes gown then a follower takes over and starts work on it.
* When a data is written in a leader then followers continously checks in leader for dataa they keep synching.

#### What happens when queue size reached ?
* we can have multiple brokers. Limit of patition is equal to limit of machine.

#### What happens to messages when queue goes down ? 
* Follower becomes the new leader when a leader goes down.

#### What happens when consumer goes down ?
* Another consumer takes over from the consumer group at commited offset from zookeeper.

#### What happens when consumer not able to process it
* Let say we have a partition p0 and a consumer c1 and c1 has consumed upto index 6 and when it is processing 7th index data then it goes failed. so for it we can add retry mechanism and retry can be 2, 3, 4 .. n no of times and now of retry has reached then a consumer c1 and push that message into another queue know as dead letter queue. and with the help of another process which will consume the message from dead letter queue and will fix the message and push back to the partition p0.

#### In kafka can a consumer in a consumer group can connect to more than one partition of same topic ?
* Yes, in Apache Kafka, a consumer within a consumer group can indeed be assigned to multiple partitions of the same topic. The assignment of partitions to consumers is handled by Kafka's group coordinator, which ensures that each partition of a topic is consumed by only one consumer in the group at any time. However, a single consumer can definitely consume messages from more than one partition.
* How Partition Assignment Works:
  - Partition Distribution: Kafka distributes the partitions of a topic across different consumers in a consumer group such that each partition is assigned to only one consumer of the group at a time. If there are more partitions than consumers, some consumers will end up consuming from multiple partitions.
  - Consumer Scalability: If there are fewer consumers than partitions, some consumers will consume from multiple partitions. Conversely, if there are more consumers than partitions, some consumers will remain idle without any partitions to consume from.
  - Balancing: Kafka periodically rebalances partitions across consumers in a consumer group. This rebalancing can occur when new consumers join the group, existing consumers leave, or when the topics/partitions themselves change.
  - Consumer Load: Assigning multiple partitions to a single consumer can increase the load on that consumer, potentially leading to performance bottlenecks if not properly managed.
  - Fault Tolerance: Having multiple partitions on a single consumer can also affect fault tolerance; if that consumer fails, a larger volume of data (multiple partitions) must be redistributed to other consumers.

#### who creates consumer group ?
* In Apache Kafka, consumer groups are created implicitly by the consumers themselves. When a consumer connects to a Kafka broker, it specifies the consumer group it belongs to as part of its configuration. If the specified consumer group does not already exist, it is automatically created by the Kafka broker.
* Here's how the process typically works:
  - Consumer Configuration: When writing a Kafka consumer application, you configure each consumer with various settings. One of these settings is the group.id, which specifies the consumer group to which the consumer belongs.
  - Starting the Consumer: When you start the consumer application, it connects to the Kafka cluster and registers itself with the specified group.id. If this consumer group does not yet exist in the Kafka cluster, Kafka creates it at this point.
  - Group Management: Kafka brokers manage the consumer groups. This includes keeping track of which consumers are part of which groups, which partitions each consumer is currently reading, and handling rebalances in case consumers are added or removed from the group.
  - Multiple Consumers: If additional consumers connect to the same Kafka cluster specifying the same group.id, they are automatically added to the existing consumer group. Kafka then may rebalance the partitions among all consumers in the group to ensure an efficient and fair distribution of work.
  - Persistence: The information about consumer groups, such as their offsets (i.e., their position within each partition of a topic), is stored in a special Kafka internal topic named __consumer_offsets. This allows Kafka to maintain state across consumer restarts and rebalances.

#### does every consumer within a consumer group has its own offset ?
* Yes, every consumer within a consumer group in Apache Kafka maintains its own offset for each partition it is consuming from.
* Here is how it works:
  - Partition Assignment: In Kafka, each consumer group is assigned a set of partitions from the topics it subscribes to. This assignment is exclusive; that is, each partition is consumed by exactly one consumer in the group at any given time.
  - Offset Tracking: Each consumer within the group tracks its own offset for each partition it is consuming. The offset represents the position of the consumer in the partition, indicating which messages have been consumed. It is crucial for ensuring that each message is processed once and only once by the group, even if consumers crash or new consumers join the group.
  - Offset Storage: Offsets are typically stored in Kafka's internal topic named __consumer_offsets. When a consumer processes a message from a partition, it can commit the offset of that message to Kafka. This commit tells Kafka that all previous messages up to that offset have been processed by that consumer. When the consumer restarts, it begins consuming from the last committed offset, ensuring no message is missed or redundantly processed.
  - Independent Offsets: Since each consumer manages its own offset for the partitions it reads, consumers in the same group do not affect each other's position in their respective partitions. This independent management allows for flexible scaling of consumers within a group and robust fault tolerance.
  - Rebalancing: If a consumer leaves the group or new consumers join the group, Kafka may reassign partitions among the remaining or new consumers. During this rebalancing, offsets play a crucial role in ensuring continuous and accurate processing. Consumers will start consuming their new partitions from the offset stored in __consumer_offsets.
 
#### If any consumer is consuming messages and if I change the consumer group name for that consumer in a configuration then from where it will start process the data from same topic?
* When you change the consumer group name for a Kafka consumer, effectively creating a new consumer or moving it to a different group, the behavior in terms of where it starts consuming messages depends on several configuration settings:
  - Offset Reset Configuration: Kafka consumers have a configuration setting called auto.offset.reset that determines what to do when there is no initial offset in Kafka or if the current offset does not exist any more (e.g., because it has been deleted):
    + earliest: The consumer starts reading at the earliest offset (the oldest available message in the log).
    + latest: The consumer starts reading just past the latest offset (it will only consume messages that are produced after it has started).
    + none: If no previous offset is found for the consumer's group, it throws an exception to the consumer.
  - New Consumer Group: When you assign a new group name to a consumer, Kafka treats this as a completely new consumer group. Since this new group has no committed offsets, it will rely on the auto.offset.reset policy:
    + If auto.offset.reset is set to earliest, the consumer will start from the oldest message available in each partition of the topic.
    + If set to latest, the consumer will start consuming new messages from the point it starts up, ignoring any messages sent before.
  - Practical Impact: This setting is critical when setting up new consumers or changing consumer group names because it controls the visibility of historical data to the consumer. Choosing between earliest and latest depends on whether you need to process previously unprocessed messages or just new messages moving forward.
  - Example: Suppose a consumer is part of a group consuming messages from a topic. If you change the group name and set auto.offset.reset to earliest, the new group’s consumers will start processing from the oldest message available in the topic. If set to latest, they will only see messages published after they start.
 
#### If there is a consumer group with one consumer and it has its own offset and if any new consumer comes in a consumer group then from where this consumer will start reading messages?
* When a new consumer joins an existing consumer group in Kafka, the handling of where this new consumer starts reading messages depends on a few factors, primarily focusing on whether there are existing committed offsets for this consumer and how partitions are reassigned among consumers in the group:
  - Partition Assignment: Kafka uses a group coordination protocol to assign partitions to consumers within a group. When a new consumer joins an existing group, a re-balance occurs. This re-balance process redistributes the partitions among all consumers in the group to evenly distribute the workload.
  - Starting Point After Rebalance:
    + Existing Offsets: If the partitions assigned to the new consumer have committed offsets (from previous consumers who handled these partitions before the re-balance), the new consumer will start consuming from the next message after the last committed offset. This ensures that no messages are missed or doubly processed, maintaining consistency.
    + No Existing Offsets: If the partitions assigned to the new consumer do not have committed offsets (which can occur if these partitions were not previously assigned to any consumer), the new consumer's starting point will depend on the auto.offset.reset configuration:
      - earliest: Starts from the earliest available message in the partition.
      - latest: Starts consuming messages that arrive after the consumer starts.
      - none: Throws an exception if no offset is saved.
  - Effect of Adding a New Consumer:
    + When the new consumer joins, and the group is rebalanced, each consumer may end up with a different set of partitions than they had before. This means the new consumer might take over some partitions previously handled by others, starting from where the last consumer left off (based on committed offsets).
    + The existing consumers will adjust to handle their new set of partitions, continuing from the committed offsets of any new partitions they receive.
  - Considerations: It's important to manage consumer group changes carefully because re-balances, while necessary for scaling and fault tolerance, can cause temporary disruptions. During a re-balance, consumers cannot consume messages, which may lead to a slight delay in message processing.
 
#### what should be the ratio of partition per consumer ?
* In Kafka, the ideal ratio of partitions per consumer in a consumer group depends on several factors, including the throughput requirements of your application, the processing capabilities of each consumer, and the overall architecture of your Kafka deployment.
* Here are some general guidelines and considerations for determining the appropriate ratio:
  - ```Parallelism Needs```:
    + ```More Partitions than Consumers```: Having more partitions than consumers allows your application to scale by adding more consumers up to the number of partitions. This can improve parallel processing as each consumer can process data from multiple partitions if needed.
    + ```Equal or Fewer Partitions than Consumers```: If there are fewer partitions than consumers, some consumers will remain idle. This scenario is generally not desirable as it wastes resources.
  - ```Throughput and Performance```:
    + ```High Throughput```: If your application requires high throughput, having more partitions can help because it allows distributing the load more effectively across more consumers.
    + ```Consumer Capacity```: Each consumer has a limit to how much data it can process efficiently. If a consumer is assigned too many partitions, it may not keep up with the flow of data, leading to increased latency or backlog in processing.
  - ```Fault Tolerance and Availability```: Multiple partitions also contribute to better fault tolerance. If one consumer fails, only the partitions assigned to that consumer are affected. Other consumers can continue processing the remaining partitions.
  - ```Operational Simplicity```: While having a large number of partitions increases parallelism and fault tolerance, it also comes with overhead in terms of management and possibly increased latency due to more frequent consumer rebalances.
 
#### if there is an application running with more than one instances then will consumers in a consumer group will be considered equal to number of instances ?
* In the context of Kafka and application architecture, whether consumers in a consumer group are considered equal to the number of instances of an application depends on how the application is designed and how it integrates with Kafka.
* Here are a few common scenarios and considerations:
  - ```One Consumer Per Application Instance``` : In many scenarios, especially when using Kafka in microservices architectures, each instance of an application runs its own Kafka consumer. This setup is typical because it aligns the lifecycle of the consumer with the lifecycle of the application instance, making scaling straightforward:
    + ```Scaling Up/Down```: When you scale the application by adding more instances, you inherently increase the number of consumers in the consumer group. This helps in distributing the load more evenly across more consumers.
    + ```Fault Tolerance```: If an instance of the application fails, only the consumer in that instance is affected. Other instances (and their consumers) continue processing messages, which enhances the resilience of the system.
  -  ```Multiple Consumers Per Application Instance``` : Some applications may run multiple consumers in a single instance, either in different threads or as part of a more complex consumer setup:
    + ```Use Case Specific```: This is less common but can be useful in cases where different threads or components of the application need to handle different types of messages or to consume from different topics.
    + ```Resource Utilization```: This approach can maximize the utilization of the application instance’s resources but requires careful management of threading and resource allocation.
  - ```Single Consumer for Multiple Instances```: It’s less common, but in some architectures, you might have a single consumer shared across multiple application instances. This setup is complex and generally used when the application instances are stateless, and the consumer’s processing logic is entirely separate from the main application logic.
  - ```Best Practices```
    + ```One Consumer Per Instance```: Generally, running one consumer per application instance is recommended for simplicity, scalability, and fault tolerance.
    + ```Monitoring and Management```: Use tools and metrics (like Kafka's built-in metrics or external monitoring tools) to observe consumer behavior, performance, and the rebalancing of consumers and partitions.


  



#### Kafka vs RMQ
* kafka is pull based approach and RMQ is a push based approach.


#### RabbitMQ
* [Rabbit MQ](https://medium.com/cwan-engineering/rabbitmq-concepts-and-best-practices-aa3c699d6f08)


> # Proxy
### What is Proxy server ? 
* Let us understand it with an example, let say their is a child and there is a chocolate shop and child wants a chocolate so child will ask his mon for the chocolate and his mom on behalf of him will go to chocolate shop will get it and given will chocolate to the child so mon here is proxy.
* So in real case let say we have client1 and client2 and a server so we can place a proxy server in between clients and a server that will connect to a server on behalf of clients.

### Types of Proxy
#### Forward proxy
* In general when we talk about proxy it is called forward proxy like in above examples.

##### Advantages
* It hides the client from outside world, How ? let say we have two clients c1 and c2 and they have their own IPs so when they hit a server through forward proxy then server will have a IP of proxy server but here client is the one who is making a call. Basically it provides anonymous.
* Grouping of requests : let say client c1 and c2 making a request google.com at same time so proxy server will club all similar requests and will hit the server with one request only.
* Access restricted data/content.
* It brings security : we can certain kind of security like you cannot access data from facebook.com or anything like that.
* Caching : Let say there are 100 clients they ask some static content so proxy will get the data from server and will push it into its cache and next time when req comes for same static content this will return the data from cache.

##### Disadvantages
* lets say there n number of applications then we have to set n number of proxy server.
  
#### Reverse proxy
* It is just a reverse in direction of forward proxy.
* Let say there are 3 servers and a request comes from the internet then request cannot go directly to any server there is a reverse proxy server placed in front of these servers which will hit the server.

##### Advantage
* Security : Outside world cannot hit server directly their IPs are hide for outside world as there is only reverse proxy server placed. for example CDN, CDN is a reverse proxy.
* Caching : For example CDN has its own cache.
* Latency : CDN is placed at near user location so reduces latency.
* Load Balancer capability : reverse proxy can be used for load balancing to servers.

### Proxy vs VPN
* Proxy cannot do encryption and decrption of data where as VPN can do encryption and decryption. Let say there is a VPN client and somewhere a VPN server and there is VPN tunnel between then through which data flows so data go from VPN client to VPN server in encrypted form and VPN server decrypt and call the main server.

### Proxy(reverse proxy) vs Load Balancer
* Reverse proxy can act as LB , but LB cannot act as a proxy.
* Proxy can do much more like caching, IP anonymity, logging, but LB does not have these kind of capabilitities.
* If we have only one server then we don't need LB, but reverse proxy might be required due its other capabilities.

### Proxy vs Firewall
* What Firewall is ? In firewall, we put certain holes and each hole defines a rule that what data can pass to the outside internet. So it works on packet scanning, in which it checks header which contains port no, IP address, source and destination. so based on these attributes it check wheather it should allow it to pass or not.
* But in case of proxy we have data and we can setup rules on the basis of data also.
* Proxy can act as a firewall also, but the way they work is know as traditional firewall is known as packet


> # Caching

* Caching is a technique to store frequently used data in a fast memory rather than accessing data every time from slow access memory.
* This makes our system very fast.
* It helps to reduce the latency.
* It also helps to achieve the fault tolerance.
* There are different types of caching present at different layer of system like:
  - client side caching (Browser caching)
  - CDN (used to store the static data)
  - Load Balancer
  - Server side Application caching (like Redis etc.)
  - etc.
* At server side cache site between app server and DB.

### Distributed Caching
* Before understanding this, let say we have three app servers app1, app2 and app3 and there is a single cache server in which all app servers are calling. so this will create a problem of scalability at any particular point of time we can't scale it more due to limited resources and single point of failure, if this cache server gone then caching capability gone. so thats where distributed caching comes in to the picture.
* So here we a have cache pool and in cache pool we have N number of cache servers. cache server 1, cache server 2 , cache server 3 and so on. and also we have a cache client. so each app server uses this cache client to connect to a particular cache server and each cache server has its own cache client. How cache server is alloted to a particular app server ? So for this it uses consistent hashing technique.

### Caching Strategies

### 1) Cache Aside
* Application First check the cache
* If data found in cache, its called cache hit and data is returned to the client.
* If data is not found in cache, its called cache miss. Application fetch the data from DB, store it back to cache and data is returend to the client.

##### Advantages
* Good approach read heavy applications.
* Even cache is down, request will not fail, as it will fetch the data from the DB.
* Cache Document data structure can be different than the data present in DB.

##### Disadvantages
* For new data read, there will always be CACHE-MISS first. (to resolve this, generally we can pre-heat the cache).
* Without appropriate caching is not used during write operation. There is a chance of inconsistency between cache and DB.

### 2) Read through cache
* Application first check the cache.
* If data found in cache, it's called cache hit and data is returned to the client.
* If data is not found in the cache, its called cache miss. Cache library itself fetch data from the DB, store it back to cache and data is returned to the application.

##### Advantages
* Good approach for heavy read application.
* Logic of fetching the data from DB and updating is seperated from the application.

##### Disadvantages
* For new data read, there will always be CACHE-MISS first. (to resolve this, generally we can pre-heat the cache).
* Without appropriate caching is not used during write operation. There is a chance of inconsistency between cache DB.
* Cache document structure should be same as DB table.

### 3) Write around cache
* Directly writes data into the DB.
* It do not update the cache.
* Let say PUT request comes and it directly writes into the DB let say earlier the value is 10 now it is changed to 11. then it will make the cache dirty, make dirty flag to true. so when get request comes it will see dirty flag is true so it miss the cache and will the DB and will update the cache with updated data.

##### Advantages
* Good approach for heavy read application.
* Resolved inconsistency problem between cache and DB.

##### Disadvantages
* For new data read, there will always be CACHE-MISS first.(to resolve this, generally we can pre-heat the cache).
* If DB is down, write operation will fail.

### 4) Write through cache
* First writes data into the cache.
* Then in synchronous writes data into DB.

##### Advantages
* Cache and DB always remain consistent.
* Cache hit chance increase a lot.

##### Disadvantages
* Alone its not useful it will increase the latency.(that's why its always used with read through or cache aside cache).
* 2 phase commit, need to be supported with this. To maintain the transactional property.
* If DB is down, write operation will fail.

### 5) Write back or behind cache
* First writes data into the cache.
* Then in asynchronous writes data into the DB.

##### Advantages
* Good for write heavy application.
* Improves the write operation latency. As writing into the DB happens asynchronously.
* Cache hit chance increase a lot.
* Gives much more performance when used with read through cache.
* Even when DB fails, write operation willl still works.

##### Disadvantages
* If data is removed from the cache and DB write still not happen happens, then there is a chance of an issue. (it is handled by keeping the TAT of cache little higher like 2 days).


> # JSON web token
* It provides a secure way of transmitting information between parties as a JSON object.
* This information can verified its digitally signed using RSA(Public/Private key pair) etc.

#### Advantages
* ```Compact``` : Because of its size, it can be send inside an HTTP header itself. And, due to its size its transmission is fast.
* ```Self contained / stateless``` : The payload contain all the required information about the user, thus it avoid querying the database.
* Can be signed using symmetric(HMAC) or Asymmetric(RSA).
* Built in expiry mechanism.
* Custom claim (additional data) can be added in the JWT.

#### Where to use JWT ? 
* Used for AUTHENTICATING (confirming the user identity).
* Used for AUTHORIZATION (checks the user permission before providing access to resources).
* Used for SSO (Single Sign On) i.e Authenticate once and access multiple applications.

<img width="1274" alt="Screenshot 2024-04-28 at 10 49 53 AM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/4bd7008b-3f96-4885-b396-1b8fdeecfe97">

### Before we understand more about JWT, lets first understand, what was popular before JWT and what are problems with it ?
### ```Session ID```
<img width="1150" alt="Screenshot 2024-04-28 at 10 58 17 AM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/382dc1a5-49b0-4f0e-b51a-07a393a4cb21">

### ```Disadvantages```
  - Stateful : It rely on server side state management, it cause problem in distributed systems.
  - Its just a unique randon string, when server get this id, it has to perform DB query to fetch the details.


### ```JWT Structure```
<img width="1239" alt="Screenshot 2024-04-28 at 11 04 19 AM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/1ca24791-8142-4471-b0b3-3050e6d56ce0">

#### ```Header```
* Contains metadata information of the token.
* typ : Type of the token, generally JWT always we add here.
* alg : Signing algorithm used like RSA or HMAC etc.

#### ```Payload```
* Contains Claims (or in simole terms, user information or any additional information is kept here).
* There are three types of claims inside the payload:-
  - Registered claims
 <img width="708" alt="Screenshot 2024-04-28 at 11 11 07 AM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/daa50aa4-a568-4b5e-a035-85cb6b8fc1e9">
 
  - Public claims
 <img width="425" alt="Screenshot 2024-04-28 at 11 11 14 AM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/9ec7ad89-f13b-457d-89a3-37dbb7e865ba">

  - Private claims
 <img width="414" alt="Screenshot 2024-04-28 at 11 12 08 AM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/51f83688-1fc3-415d-abc5-96e0e6321f40">

 
#### ```Signature```
* Encode JWT header and payload seperately using Base64 encoding.
* Concatenate the encoded header and payload strings using "." (ex :  ```xxxxxx.yyyyyy```) This is known as message.
* Use RSA(Asymmetric cryptography) or HMAC(symmetric cryptography) to create digital signature.
* Encode the signature generated in previous step.
* Concatenation using "."
* ```Dummy JWT : eyu32d2kbdiknwuid.23di2u3nfuiwndo2i3fu2uhefo2ijwcoiq.imqcnkqwecoq9jcoui2n19o32bd```
                      base64(header) .          base64(payload)          .       base64(signature)
### ```Challanges```
* ```Token Invalidation``` : Lets sat, I have blacklisted one user, how to invalidate its token before expiration ?
  - a) server need to keep the list of blacklisted tokens and then DB/cache lookup is required while validating.
  - b) Or, change the secret key, but this will make the JWT invalidate for all users
  - c) Or, token should be very short lived.
  - d) Or, token should be used only once.
* JWT token is encoded, not encrypted, so its less secure. use JWE, means encrypt the payload part.
* Unsecured JWT with "alg" : none, such JWT should be rejected.
* ```jwk exploit``` : public key shared in this, should not be used to verify the signature.
  - e stands for "exponent" and n stands for "modulus" and combined together they form public key.
    ```json
    {
      "typ": "JWT",
      "alg": "RSA",
      "jwk" : {
          "n" : "jhbeqo8wcho8q192d1i2hdu19",
          "e" : "ABC4ED",
          "kid" : "cbuqwih83742h3oj1ndijhoid"
      }
    }
    ```
* Use ```"kid"``` in the header to lookup up the https://{Auth server domain/.well-known/jwks.json} to find the public key


> # OAuth
* Open authorization (Oauth) a authorization framework.
* Enables secure third-party access to user protected data.
* We've seen that a user go to any website and there is a option of signin with google, so here we grant an access to the website that you can go get my info which are already present in gmail.
* Four important roles or actors involve in it:
  - Resource owner (a user)
  - client (a website where a user trying to log in)
  - Authrization server (gmail authorization server which take care of authorization)
  - Resource Hosting server (like gmail server from where gmail give data like name etc.)

* Authorization grant types(Mechanism used by client to obtain access token):
  - Authorization code grant
  - Implicit grant
  - Resource owner password credentials grant
  - Client credential grant
  - Refresh token grant

#### Lets understand the end to end flow using authorization code grant ?
<img width="641" alt="Screenshot 2024-04-28 at 11 52 40 PM" src="https://github.com/sarangchoudhary96/system-design/assets/42025130/362ec705-2196-47e1-9df1-254a337a5429">








