# Bundled services: Kafka, ETCD, ZooKeeper

TigerGraph runs its own Kafka, ETCD and ZooKeeper and manages them through `gadmin`. Day-to-day work belongs in `gadmin` and `gsql`. Touch these services directly only to look, for example to see whether a loading topic has data or a consumer is lagging.

## Ground rules

- **Look, don't change.** Creating or deleting topics, changing consumer offsets, or writing keys can leave the cluster inconsistent in ways `gadmin` cannot repair.
- **Read configuration through `gadmin config get`.** ETCD and ZooKeeper hold cluster configuration, which can include credentials, so don't dump it from them.
- **Repairs:** see `SKILL.md`, "Safety".

## Locate the tools and ports

```
APP=$(gadmin config get System.AppRoot)
ls $APP/kafka/bin/
gadmin config list entry | grep -i port      # find port entry names, then:
gadmin config get Kafka.Port
```

These services listen on the node's IP, not on `localhost`. Use `$(gmyip)`.

## Kafka: read-only commands

```
BS=$(gmyip):$(gadmin config get Kafka.Port)
$APP/kafka/bin/kafka-topics.sh          --bootstrap-server $BS --list
$APP/kafka/bin/kafka-topics.sh          --bootstrap-server $BS --describe --topic <topic>
$APP/kafka/bin/kafka-consumer-groups.sh --bootstrap-server $BS --list
$APP/kafka/bin/kafka-consumer-groups.sh --bootstrap-server $BS --describe --group <group>
```

The scripts need Java. If they fail with `java: not found` or a `JAVA_HOME` error, run them from a login shell of the TigerGraph OS user, or point `JAVA_HOME` at the JDK that ships with TigerGraph:

```
find $APP -type f -path '*/bin/java' 2>/dev/null | head -1   # .../bin/java → JAVA_HOME is two levels up
```

If the cluster has internal Kafka TLS enabled, these commands also need the client security settings; ask the user rather than guessing.
