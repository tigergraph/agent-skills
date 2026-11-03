# Clustering and high availability

<!-- OUTLINE -->

- P × R grid diagram; `cluster_size = P × R`; R configured, P derived, leftover machines unused.
- Smallest HA layouts: 3 machines (R=3, P=1), 4 machines (R=2, P=2).
- Coordination services (ETCD, ZooKeeper) are sized for quorum, independent of R.
- Replication: synchronous writes to all replicas; reads from any replica; no primary.
- Failover: requests rerouted; in-flight transactions on a failed node are aborted, clients retry.
- Degraded cluster: what still works, what is blocked (schema change, query install, loading into the affected partition, export).
- Hash buckets and partition skew: how bucket count and P interact; set before loading (verify the config entry is documented publicly).
- Deployment topologies: single node, distributed without HA, HA, region-aware.
