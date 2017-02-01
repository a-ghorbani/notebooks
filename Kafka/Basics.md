## Create A topics

```bash
> kafka-topics --create --zookeeper node-1.cluster:2181 --replication-factor 1 --partitions 1 --topic test
```

## List of topics

```bash
> kafka-topics --zookeeper node-1.cluster:2181 --list
```
