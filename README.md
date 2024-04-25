
## Kafka Install: 
Apache Kafka is an open-source distributed event streaming platform originally developed by LinkedIn and later donated to the Apache Software Foundation. It is designed to handle large volumes of data in real-time, making it a popular choice for building real-time data pipelines and streaming applications.


### Pre-requisite:
- Kafka binary package
- Java


### Download and Install Apache Kafka: 

First, download the Apache Kafka binaries from the official website (https://kafka.apache.org/downloads) and follow the installation instructions for your operating system. 

**Binary downloads:**
- Scala 2.12  - kafka_2.12-3.7.0.tgz
- Scala 2.13  - kafka_2.13-3.7.0.tgz

We build for multiple versions of Scala. This only matters if you are using Scala and you want a version built for the same Scala version you use. Otherwise any version should work (2.13 is recommended).


```
java -version
echo $JAVA_HOME
```


```
cd /opt/
wget https://downloads.apache.org/kafka/3.7.0/kafka_2.13-3.7.0.tgz

tar -xvf kafka_2.13-3.7.0.tgz
mv kafka_2.13-3.7.0 kafka
```


```
mkdir -p /opt/kafka_data/zookeeper
mkdir -p /opt/kafka_data/kafka_logs
```


### Configure zookeeper:

```
vim /opt/kafka/config/zookeeper.properties

dataDir=/opt/kafka_data/zookeeper
clientPort=2181

maxClientCnxns=0

admin.enableServer=false
# admin.serverPort=8080


## Added this (optional):
###https://library.humio.com/kb/kb-zookeeper-4lw-commands.html

#4lw.commands.whitelist=*
4lw.commands.whitelist=stat, mntr


save and exit
```



### Configure kafka:

```
vim /opt/kafka/config/server.properties


#### Server Basics #######
broker.id=1

##### Socket Server Settings #######
#listeners=PLAINTEXT://:9092
listeners=PLAINTEXT://192.168.0.8:9092

#advertised.listeners=PLAINTEXT://your.host.name:9092
advertised.listeners=PLAINTEXT://192.168.0.8:9092

num.network.threads=3
num.io.threads=8

socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

##### Log Basics ######
log.dirs=/opt/kafka_data/kafka_logs

num.partitions=1
num.recovery.threads.per.data.dir=1

#### Log Retention Policy #######
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000

###### Zookeeper ########
zookeeper.connect=localhost:2181
zookeeper.connection.timeout.ms=18000

#### Group Coordinator Settings ######
group.initial.rebalance.delay.ms=0


### Add this, By default Kafka doesn’t allow us to delete the topics:
delete.topic.enable = true


save and exit
```


### Start zookeeper:

```
./bin/zookeeper-server-start.sh config/zookeeper.properties
```


```
netstat -tlpn | grep 2181
tcp        0      0   0.0.0.0:2181      0.0.0.0:*      LISTEN      31914/java
```


### Start kafka:

```
./bin/kafka-server-start.sh config/server.properties
```


```
netstat -tlpn | grep 9092
tcp        0      0   192.168.0.8:9092     0.0.0.0:*       LISTEN      314/java
```


### Check Version:

```
./bin/kafka-topics.sh --version
```


```
./bin/kafka-broker-api-versions.sh --bootstrap-server   192.168.0.8:9092 --version
```


### Status:
```
echo stat | nc localhost 2181
echo "status" | nc  localhost 2181 | head -n 1

    Zookeeper version: 3.8.3-6ad6d364c7c0bcf0de452d54ebefa3058098ab56, built on 2023-10-05 10:34 UTC
```


```
telnet localhost 2181

stat
```


### Configure Topics:
- **replication-factor**: specifies the number of servers (number of copies) a message will be written to. For example, if you have a replication factor of 3 for a topic and you publish a message, Kafka will store three copies of that message across different brokers in the cluster. If one broker goes down, the other two copies ensure that the message is still accessible and no data is lost.

- **partitions**: controls how many event logs the topic are sharded into. Partitions are the basic unit of parallelism and scalability. A partition is an ordered, immutable sequence of records that is continually appended to. Each topic in Kafka is divided into one or more partitions, and each partition can be hosted on a different broker in the Kafka cluster.

- **--bootstrap-server**:  specifies the Kafka broker(s) to use for bootstrapping the client connection. Here is IP, 192.168.0.8:9092 
- If Kafka v2.2+, use the Kafka hostname and port e.g., --bootstrap-server localhost:9092
- If older version of Kafka, use the Zookeeper URL and port e.g. --zookeeper localhost:2181
- The topic name must contain only ASCII alphanumerics, '.', '_' and '-'


To create a topic in Apache Kafka:

```
./bin/kafka-topics.sh --create --topic test-topic --bootstrap-server 192.168.0.8:9092 --partitions 2 --replication-factor 1 
```


```
./bin/kafka-topics.sh --bootstrap-server 192.168.0.8:9092 --describe --topic test-topic

Topic: test-topic       TopicId: 6vsL0ssRRIyKIbQ5TfL66A PartitionCount: 2       ReplicationFactor: 1    Configs:
        Topic: test-topic       Partition: 0    Leader: 1       Replicas: 1     Isr: 1
        Topic: test-topic       Partition: 1    Leader: 1       Replicas: 1     Isr: 1

```


List all topics in a Kafka: 

```
./bin/kafka-topics.sh --bootstrap-server 192.168.0.8:9092 --list
```


To delete a topic in Apache Kafka:

```
./bin/kafka-topics.sh --bootstrap-server 192.168.0.8:9092 --delete --topic test-topic
```



**Note:** Newer versions(2.2 and above) of Kafka no longer require ZooKeeper connection string ie **"--zookeeper localhost:2181"** and throw Exception in thread "main".

In newer versions of Kafka (2.2 and above) the ZooKeeper connection string is no longer required, i.e. "--zookeeper localhost:2181". Use Kafka Broker's **"--bootstrap-server localhost:9092"** instead of --zookeeper localhost:2181.



### Some of Kafka CLI command:

Create a Topic:
```
./bin/kafka-topics.sh --create --topic my-topic1 --bootstrap-server  192.168.0.8:9092 --partitions 1 --replication-factor 1
./bin/kafka-topics.sh --create --topic my-topic2 --bootstrap-server  192.168.0.8:9092 --partitions 2 --replication-factor 1
./bin/kafka-topics.sh --create --topic my-topic3 --bootstrap-server  192.168.0.8:9092 --partitions 3 --replication-factor 1
```


Add/Increase Partitions to a Topic:
```
./bin/kafka-topics.sh --bootstrap-server  192.168.0.8:9092 --alter --topic my-topic1 --partitions 2
```


View Details Topic:
```
./bin/kafka-topics.sh --bootstrap-server  192.168.0.8:9092 --describe --topic my-topic1

Topic: my-topic1        TopicId: faPE2TbASxW4XsbIoqSmSw PartitionCount: 2       ReplicationFactor: 1    Configs:
        Topic: my-topic1        Partition: 0    Leader: 1       Replicas: 1     Isr: 1
        Topic: my-topic1        Partition: 1    Leader: 1       Replicas: 1     Isr: 1
```


List of Topics:
```
./bin/kafka-topics.sh --bootstrap-server  192.168.0.8:9092 --list

my-topic1
my-topic2
my-topic3
```


Delete a Topic:
```
./bin/kafka-topics.sh --bootstrap-server  192.168.0.8:9092 --delete --topic my-topic3
```


### Change Message Retention Period:

When the producer client sends an event to the Kafka broker, that event is appended to the end of one of the commit logs. By default, the event is retained for 168 hours or 7 days after which it’s deleted to free up disk space.


Based on your application’s requirements, you might want to bypass this behavior. This sets a retention period of 05 (5x60x1000=300000) minutes for all messages appended to the topic "my-topic4". For example:


```
./bin/kafka-topics.sh --create --topic my-topic4 --bootstrap-server  192.168.0.8:9092 --partitions 2 --replication-factor 1 --config retention.ms=300000
```


The **kafka-configs** tool to change and describe topic, client, user, broker, or IP configuration settings. To change a property, specify the entity-type to the desired entity (topic, broker, user, etc), and use the alter option.


```
./bin/kafka-configs.sh --bootstrap-server  192.168.0.8:9092 --entity-type topics --entity-name my-topic4 --alter --add-config retention.ms=-1,retention.bytes=524288000

./bin/kafka-configs.sh --bootstrap-server  192.168.0.8:9092 --entity-type topics --entity-name my-topic4 --alter --add-config delete.retention.ms=172800000
```



### Kafka Producer CLI:

Publish Events to a Topic:

```
### Kafka v2.5+:

./bin/kafka-console-producer.sh --bootstrap-server  192.168.0.8:9092 --topic my-topic1

>this kafka
>this kafka2
>this kafka3
```


```
### Kafka v2.4 or less:

./bin/kafka-console-producer.sh --broker-list  192.168.0.8:9092 --topic my-topic1

>this my-kafka
>this my-kafka2
>this my-kafka3
```

You have now published 3 events to the my-topic topic. Press **"Ctrl+C**" to stop the producer client.
After running this command, a prompt will open. Type your messages and click enter to publish them to the Kafka topic. Each time you click to enter a new message is submitted.


```
### Produce messages from a file:

./bin/kafka-console-producer.sh --bootstrap-server  192.168.0.8:9092 --topic my-topic2 < topic2_input.txt
```


### Kafka Consumer CLI:

Consume Data in a Kafka Topics:
```
./bin/kafka-console-consumer.sh --bootstrap-server  192.168.0.8:9092 --from-beginning --topic my-topic1


this my-kafka
this my-kafka2
this my-kafka3
this kafka
this kafka2
this kafka3
```


Consuming only the future messages of a Kafka topic:
```
./bin/kafka-console-consumer.sh --bootstrap-server  192.168.0.8:9092 --topic my-topic1
```


Formatter:
```
./bin/kafka-console-consumer.sh --bootstrap-server  192.168.0.8:9092 --topic my-topic1 --formatter kafka.tools.DefaultMessageFormatter --property print.timestamp=true --property print.key=true --property print.value=true --from-beginning

CreateTime:1714061985195        null    this kafka
CreateTime:1714062018824        null    this kafka2
CreateTime:1714062065688        null    this kafka3
```





### Create consumers Group:

```
./bin/kafka-topics.sh --create --topic first-topic --bootstrap-server  192.168.0.8:9092 --partitions 3 --replication-factor 1

./bin/kafka-topics.sh --bootstrap-server  192.168.0.8:9092 --list
```


Then launch a consumer in a consumer group named "my_group1":
```
### Terminal-1:

./bin/kafka-console-consumer.sh --bootstrap-server  192.168.0.8:9092 --topic first-topic --group my_group1
```


```
### Terminal-2:

./bin/kafka-console-consumer.sh --bootstrap-server  192.168.0.8:9092 --topic first-topic --group my_group1
```


Produce message to first-topic:

```
### Terminal-3:

./bin/kafka-console-producer.sh --bootstrap-server  192.168.0.8:9092 --topic first-topic

>first message
>second message
>third message
>fourth message
```



### Kafka Consumer Group Management:

```
./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --list

./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --list --state

GROUP                 STATE
my_group1             Stable
console-consumer-7799 Stable
```


```
./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --describe --group my_group1

Consumer group 'my_group1' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
my_group1       first-topic     0          0               0               0               -               -               -
my_group1       first-topic     1          5               5               0               -               -               -
```


```
./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --describe --all-groups --state
```


Reset the offsets: Disconnect the consumer then apply reset. 
```
./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --group my_group1 --topic first-topic --reset-offsets --to-earliest --execute

./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --group my_group1 --topic first-topic --reset-offsets --to-latest --execute

./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --group my_group1 --topic first-topic --reset-offsets --to-datetime 2024-04-25T00:00:00.000 --execute
```


Delete offsets for a specific topic:
```
./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --delete-offsets --group my_group1 --topic first-topic
```


Delete group: 
```
./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --list
./bin/kafka-consumer-groups.sh --bootstrap-server  192.168.0.8:9092 --delete --group my_group1

    Deletion of requested consumer groups ('my_group1') was successful.
```



### kcat (kafkacat):
kcat is the project formerly known as as kafkacat. Kafkacat offers a simple command line interface to interact with Kafka. You can use kcat to produce, consume, and list topic and partition information for Kafka. Here is, (https://github.com/edenhill/kcat). 


```
### Ubuntu:

apt-get install kafkacat

### list all topics currently in kafka:
kafkacat -b localhost:9092 -L

### Producer message publish:
# kafkacat -b localhost:9092 -t test-topic -P

### Consumer message consume: 
kafkacat -b localhost:9092 -t test-topic -C
```


```
### Centos:

yum update -y
yum group install "Development Tools" -y
yum install -y librdkafka-devel yajl-devel avro-c-devel -y

yum install libcurl-devel -y
yum install jq -y
```


```
git clone https://github.com/edenhill/kafkacat.git
cd kafkacat/
./bootstrap.sh
./kcat -h
```


```
### Metadata listing:
./kcat -b <broker> -L [-t <topic>]
./kcat -b  192.168.0.8:9092 -L
./kcat -b  192.168.0.8:9092 -L -t my-topic1


### JSON metadata listing:
./kcat -b  192.168.0.8:9092 -L -J
./kcat -b  192.168.0.8:9092 -L -J | jq .


### Producer mode:
./kcat -P -b  192.168.0.8:9092 -t my-topic1


### Consumer mode:
./kcat -C -b  192.168.0.8:9092 -t my-topic1


### Query offset by timestamp:
./kcat -Q -b  192.168.0.8:9092 -t <topic>:<partition>:<timestamp>
./kcat -Q -b  192.168.0.8:9092 -t my-topic3:0:1714062065688
```


Remember to consult the official Kafka documentation and relevant resources for detailed instructions specific to your use case and environment.



