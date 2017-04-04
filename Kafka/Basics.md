## Create a topic

```bash
> kafka-topics --create --zookeeper zookeeper-host:2181 --replication-factor 1 --partitions 1 --topic testTopic
```

## List of topics

```bash
> kafka-topics --zookeeper zookeeper-host:2181 --list
```

## Kafka Producer/Consumer

### Console

Produce messages just by typing in the console:
```bash
> kafka-console-producer --broker-list kafka-broker-host:9092 --topic testTopic
```

Consume messages of a topic from the **beginning**, i.e. consumes all the messages.
```bash
> kafka-console-consumer --from-beginning --bootstrap-server  kafka-broker-host:9092 --topic testTopic
```

Consume messages of a topic's **first** partition from **offset** 4 on:
```bash
> kafka-console-consumer --offset 4 --partition 0 --bootstrap-server  kafka-broker-host:9092 --topic testTopic
```
Consume **latest** messages of the topic :
```bash
> kafka-console-consumer --bootstrap-server  kafka-broker-host:9092 --topic testTopic
```

Consume message of a topic given specific deserializer: 
```
> kafka-console-consumer \
      --formatter kafka.tools.DefaultMessageFormatter \
      --property print.key=true \
      --property print.value=true \
      --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
      --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer \
      --from-beginning \
      --bootstrap-server  kafka-broker-host:9092 \
      --topic testTopic
```

