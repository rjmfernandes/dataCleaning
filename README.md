# Data Cleaning 

Start the repo:

```shell
docker compose up -d
```

Create a compacted topic:

```shell
kafka-topics --bootstrap-server localhost:29092 --topic test --create --partitions 1 --replication-factor 1 --config cleanup.policy=compact --config max.compaction.lag.ms=600  --config min.cleanable.dirty.ratio=0.01 --config segment.ms=5000 --config local.retention.ms=100 --config retention.ms=300 --config delete.retention.ms=100  
```

Let's write a message to it:

```shell
kafka-console-producer --broker-list localhost:29092 --topic test --property parse.key=true --property key.separator=:
```

Write:

```
Rui:Is writing to Kafka
```

One can consume it of course:

```shell
kafka-console-consumer --bootstrap-server localhost:29092 --topic test --from-beginning --property print.key=true --property key.separator=:
```

We can also run:

```shell
cat ./data/test-0/*.log
```

And confirm the event is there on our segments.

Now create the tombstone record:

```bash
echo "Rui:" | kcat -b localhost:29092 -t test -Z -K:
```

And consume again:

```shell
kafka-console-consumer --bootstrap-server localhost:29092 --topic test --from-beginning --property print.key=true --property key.separator=:
```

Try to write some other record:

```shell
kafka-console-producer --broker-list localhost:29092 --topic test --property parse.key=true --property key.separator=:
```

It's still there due to the min.cleanable.dirty.ratio. So write:

```
Somebody:Is also writing to Kafka
```

And now consume again:

```shell
kafka-console-consumer --bootstrap-server localhost:29092 --topic test --from-beginning --property print.key=true --property key.separator=:
```

And checking logs:

```bash
cat ./data/test-0/*.log
```

# Clean up

```shell
docker compose down -v
rm -fr data
```