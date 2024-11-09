
## Kafka `SASL_PLAIN` Authentication: 

To configure `SASL_PLAIN` authentication in Kafka, you'll need to adjust both the Kafka broker and client configurations.




### Kafka Broker Configuration:

Edit `server.properties` on each Kafka broker to enable SASL/PLAIN authentication. Locate the file (usually in `config/server.properties`).

- `listeners`: Defines the protocol, in this case,` SASL_PLAINTEXT` (non-encrypted) or `SASL_SSL` (encrypted).
- `inter.broker.listener.name` :  Must Match a Defined Listener and the value of `inter.broker.listener.name` should match one of the listeners defined in the listeners configuration.
- `sasl.enabled.mechanisms`: Specifies available SASL mechanisms. like `PLAIN`, `SCRAM-SHA-256`, `SCRAM-SHA-512`. 
- `sasl.mechanism.inter.broker.protocol`: Sets the authentication method used for broker-to-broker communication. like `PLAIN`, `SCRAM-SHA-256`, `SCRAM-SHA-512`. 



_Syntax:_

```
## Enable SASL on the broker: 
listeners=SASL_PLAINTEXT://your.kafka.server:9092  			  # or SASL_SSL for encrypted communication
advertised.listeners=SASL_PLAINTEXT://your.kafka.server:9092

inter.broker.listener.name=SASL_PLAINTEXT

## Enable SASL mechanisms:
sasl.enabled.mechanisms=PLAIN,SCRAM-SHA-256,SCRAM-SHA-512  	# Add SASL mechanisms you want to support
sasl.mechanism.inter.broker.protocol=PLAIN  				# or SCRAM-SHA-256 or SCRAM-SHA-512
```


```
vim config/server.properties


## Enable SASL on the broker: 
#listeners=PLAINTEXT://:9092
listeners=SASL_PLAINTEXT://192.168.10.192:9092

#advertised.listeners=PLAINTEXT://192.168.10.192:9092
advertised.listeners=SASL_PLAINTEXT://192.168.10.192:9092

inter.broker.listener.name=SASL_PLAINTEXT

## Enable SASL mechanisms:
sasl.enabled.mechanisms=PLAIN
sasl.mechanism.inter.broker.protocol=PLAIN

```



#### Add user credentials:

- **Purpose**: This file is specifically designed for **Java Authentication and Authorization Service** (JAAS) configurations, including handling multiple users and different security mechanisms.

- **For Production Environments**: `kafka_server_jaas.conf` is generally preferred for production due to its flexibility, separation of concerns, and suitability for handling multiple users or complex authentication setups.


 In the `kafka_server_jaas.conf` file, this configuration specifies the SASL/PLAIN authentication settings for Kafka, where multiple users are defined with individual credentials. 


_Here's a breakdown:_

- `KafkaServer { ... };` Block : This section applies to the Kafka server, where authentication settings for the Kafka broker are defined.

- `org.apache.kafka.common.security.plain.PlainLoginModule required` : Specifies the authentication mechanism to use, in this case, PlainLoginModule, which handles SASL/PLAIN authentication.

- `username="admin"` and `password="secret"` : These credentials are primarily used by the Kafka broker itself when it needs to authenticate with other brokers in a Kafka cluster. This can occur during **inter-broker communication** (like when brokers share metadata about topics, partitions, and offsets).

- `user_admin="secret"`: This line creates a user with the username `admin` and the password `secret`.
- `user_zabbix="zabbix"`: This line creates another user with the username `zabbix` and the password `zabbix`.



```
vim config/kraft/kafka_server_jaas.conf


KafkaServer {
 org.apache.kafka.common.security.plain.PlainLoginModule required
 username="admin"
 password="secret"
 user_admin="secret"
 user_zabbix="zabbix";
 };

```





_Export `KAFKA_OPTS` before start kafka:_

```
vim config/kraft/kafka-env.sh


export KAFKA_OPTS="-Djava.security.auth.login.config=/apps/kafka_2.13/config/kraft/kafka_server_jaas.conf"
```



#### Start Kafka: 


_Run Zookeeper:_

```
./bin/zookeeper-server-start.sh config/zookeeper.properties
```



_Run Kafka with `JMX_PORT`:_

```
./bin/kafka-server-start.sh config/server.properties

Or,

JMX_PORT=12345 ./bin/kafka-server-start.sh config/server.properties
```



```
netstat -tlpn | grep 12345
```




#### Client Configuration:

For Kafka clients (like producers and consumers), specify SASL/PLAIN authentication in their properties.


_Sample client configure:_

```
vim config/kraft/client.properties

or,

vim config/kraft/admin.config


sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="secret";
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
```



### Verify: 


_Create Kafka Topic:_
```
./bin/kafka-topics.sh --bootstrap-server 192.168.10.192:9092 --create --topic topic1 --partitions 2  --command-config /apps/kafka_2.13/config/kraft/admin.config
```


_List all topics in a Kafka:_
```
./bin/kafka-topics.sh --bootstrap-server 192.168.10.192:9092 --list --command-config /apps/kafka_2.13/config/kraft/admin.config

topic1
```


_Delete a Topic:_
```
./bin/kafka-topics.sh --bootstrap-server 192.168.10.192:9092 --delete --topic topic1 --command-config /apps/kafka_2.13/config/kraft/admin.config
```






_Publish Events to a Topic:_
```
./bin/kafka-console-producer.sh --bootstrap-server 192.168.10.192:9092 --topic my-topic1 --producer.config /apps/kafka_2.13/config/kraft/admin.config

>hello
>bangladesh
```


_Consume Data in a Kafka Topics:_
```
./bin/kafka-console-consumer.sh --bootstrap-server 192.168.10.192:9092 --from-beginning --topic my-topic1 --consumer.config /apps/kafka_2.13/config/kraft/admin.config

hello
bangladesh
```


This configuration should enable SASL/PLAIN authentication on your Kafka setup.

