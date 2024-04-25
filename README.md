
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


To create a topic in Apache Kafka:

- **--bootstrap-server** - specifies the Kafka broker(s) to use for bootstrapping the client connection. Here is IP, 192.168.0.8:9092 


```
./bin/kafka-topics.sh --create --topic test-topic --bootstrap-server 192.168.0.8:9092 --partitions 2 --replication-factor 1 
```


```
./bin/kafka-topics.sh --bootstrap-server 192.168.0.8:9092 --describe --topic test-topic
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


Remember to consult the official Kafka documentation and relevant resources for detailed instructions specific to your use case and environment.




