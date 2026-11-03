# Bundled services: Kafka, ETCD, ZooKeeper

<!-- OUTLINE -->

- TigerGraph runs its own Kafka, ETCD and ZooKeeper. Day-to-day work stays in `gadmin` and `gsql`; use these tools only to look, never to change state.
- Locating the binaries under `<AppRoot>` and the ports via `gadmin config list entry | grep -i port`.
- They bind to the node IP, not loopback: use `$(gmyip)`.
- Kafka: `JAVA_HOME` must be set in non-login shells; read-only commands (`kafka-topics.sh --list`, `kafka-consumer-groups.sh --describe`).
- ETCD and ZooKeeper: keep to listing keys; do not dump configuration (it can contain credentials).
- Any repair or reset of these services: contact TigerGraph Support.
