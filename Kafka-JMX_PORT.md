
## Kafka `JMX_PORT` Configure:

Apache Kafka disables remote JMX by default. You can enable remote monitoring using JMX by setting the environment variable `JMX_PORT` for processes started using the CLI or standard Java system properties to enable remote JMX programmatically.  

You must enable security when enabling remote JMX in production scenarios to ensure that unauthorized users cannot monitor or control your broker or application as well as the platform on which these are running. 

Note that authentication is disabled for JMX by default in Kafka and security configs must be overridden for production deployments by setting the environment variable `KAFKA_JMX_OPTS` for processes started using the CLI or by setting appropriate Java system properties. 

- Enable and configure JMX access to Apache Kafka. 




_Default setting:_

```
# JMX settings
if [ -z "$KAFKA_JMX_OPTS" ]; then
  KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
fi
```



_Change to:_

```
vim /apps/kafka_2.13/bin/kafka-run-class.sh


# JMX settings [line: 209]
if [ -z "$KAFKA_JMX_OPTS" ]; then
KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
fi
```




Or,



_Run Zookeeper:_

```
./bin/zookeeper-server-start.sh config/zookeeper.properties
```



_Run Kafka with `JMX_PORT`:_ [Worked]

```
./bin/kafka-server-start.sh config/server.properties

Or,

JMX_PORT=12345 ./bin/kafka-server-start.sh config/server.properties
```



```
netstat -tlpn | grep 12345

tcp6       0      0 :::12345          :::*       LISTEN      13761/java
```



Now Kafka will be accessible on the configured JMX_PORT for monitoring with compatible tools.

