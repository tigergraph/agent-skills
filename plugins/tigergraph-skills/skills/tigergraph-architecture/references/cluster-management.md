# Cluster management

<!-- OUTLINE -->

- What happens behind a `gadmin` command: gadmin → CTRL (cluster-wide decisions) → EXE on each node (runs the action); IFM reports health and metrics.
- Configuration: staged, applied cluster-wide; some changes hot-reload, others need a restart.
- Lifecycle: start/stop follow the service dependency order.
- Topology changes (expand, shrink) and backup/restore are coordinated cluster-wide.
