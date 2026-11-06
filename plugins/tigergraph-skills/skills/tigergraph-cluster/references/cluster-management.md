# Cluster management

What sits behind `gadmin`. For the commands, use the tigergraph-cli skill.

## Who does what

- **CTRL** (controller) receives every `gadmin` request and decides what happens across the cluster.
- **EXE** (executor, one per node) carries out the actions on its node.
- **IFM** (informant) collects health and metrics, which `gadmin status` and `gadmin metric` report.

When a `gadmin` command fails, CTRL's and EXE's logs are the place to look.

## Lifecycle

`gadmin start`, `stop` and `restart` follow the start order in `services.md`, so restarting a service low in the order (ETCD, Kafka) affects everything above it.

## Configuration

Configuration is stored in ETCD and changed in two steps: stage the change, then apply it cluster-wide. Most changes take effect only after the affected services restart, which `gadmin config apply --restart-deps` `[4.1+]` does for you.

## Topology changes and backup

- **Expand, shrink, remove and repartition** re-plan the partition × replica grid and move data in place. They need a downtime window with no loading jobs or queries running, and a backup beforehand. See `clustering.md`, "Changing the layout".
- **Backup and restore** are coordinated across all nodes to produce a consistent snapshot of the whole cluster.
