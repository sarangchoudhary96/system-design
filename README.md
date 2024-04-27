# Table of Contents
- [Distributed message queue (like : KAFKA, RMQ)](#distributed-message-queue)
  + [KAFKA](#kafka)
  + [RabbitMQ](#rabbitmq)
- [Proxy](#proxy)
  + [Forward proxy](#forward-proxy)
  + [Reverse proxy](#reverse-proxy)
- [Caching](#caching)
  + 


### Distributed message queue

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
  - Producer
  - Consumer
  - Consumer group
  - Topic
  - Partition
  - offset
  - Broker (Kafka server like mysql server)
  - Cluster
  - Zookeeper
 
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


#### Kafka vs RMQ
* kafka is pull based approach and RMQ is a push based approach.


#### RabbitMQ
https://medium.com/cwan-engineering/rabbitmq-concepts-and-best-practices-aa3c699d6f08


## Proxy
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


## Caching

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

#### 1) Cache Aside
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










