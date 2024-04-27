# Table of Contents
- [Distributed message queue (like : KAFKA, RMQ)](#distributed-message-queue)
- [KAFKA](#kafka)
- [RabbitMQ](#rabbitmq)


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






