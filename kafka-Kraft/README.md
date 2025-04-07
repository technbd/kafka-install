
## Kafka with KRaft:

Installing Apache Kafka with KRaft (Kafka Raft Metadata Mode) on Rocky Linux requires a few steps, including downloading Kafka, configuring it for KRaft mode, and starting the service. Zookeeper is NOT needed when using KRaft mode (Kafka Raft Metadata Mode).



### Why KRaft?
- KRaft (Kafka Raft) replaces Zookeeper as the metadata management system.
- It allows Kafka to manage its metadata internally using the Raft consensus protocol.
- This makes Kafka deployment simpler and more scalable, especially in cloud environments.

#### When is Zookeeper Required?
- If you are using older Kafka versions (<3.3.0), then Zookeeper is required.
- If you are running a legacy Kafka cluster that hasn’t migrated to KRaft yet.



### Prerequisites:

Kafka requires Java to run. Check if Java is installed:

```
java -version
```



### Kafka Download: 

```
wget https://dlcdn.apache.org/kafka/3.8.1/kafka_2.13-3.8.1.tgz
```


```
tar -xvf kafka_2.13-3.8.1.tgz
```


```
mv kafka_2.13-3.8.1 kafka
cd kafka
```



### Generate a Cluster UUID:

Kafka needs to be configured for **KRaft mode instead of Zookeeper**. Run the following command to generate a unique cluster UUID:

```
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
```


```
echo $KAFKA_CLUSTER_ID
```



#### Configure `server.properties`:

Edit Kafka’s server configuration file:

```
vim config/kraft/server.properties


process.roles=broker,controller

node.id=1

controller.quorum.voters=1@localhost:9093


listeners=PLAINTEXT://:9092,CONTROLLER://:9093

advertised.listeners=PLAINTEXT://localhost:9092

controller.listener.names=CONTROLLER


log.dirs=/tmp/kraft-combined-logs
```




### Format Log Directories:

```
bin/kafka-storage.sh format --standalone -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties

or,

bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties
```


```
### Output:

Formatting /tmp/kraft-combined-logs with metadata.version 3.8-IV0.
```


### Start Kafka in KRaft Mode:

```
bin/kafka-server-start.sh config/kraft/server.properties
```




### Verify:


_Create a test topic:_
```
bin/kafka-topics.sh --create --topic test-topic --partitions 1 --replication-factor 1 --bootstrap-server localhost:9092
```


_List topics:_
```
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```


_Produce or Send a test message:_
```
bin/kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092
```


_Consume the message:_
```
bin/kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server localhost:9092
```



### Links:
- [Kafka KRaft](https://medium.com/@atakan.dnmz/3-node-kafka-cluster-installation-on-rocky-linux-969d0efb34c3)



You have successfully installed and configured Kafka with KRaft mode on Rocky Linux.



