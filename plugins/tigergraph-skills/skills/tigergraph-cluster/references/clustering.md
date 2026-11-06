# Clustering and high availability

## The grid

```
               Replica 1   Replica 2   ...   Replica R
Partition 1      P1·R1       P1·R2             P1·RR
Partition 2      P2·R1       P2·R2             P2·RR
  ...
Partition P      PP·R1       PP·R2             PP·RR
```

- One copy of the graph is split across P partitions; the cluster keeps R copies.
- `cluster size = P × R`. You set R (replication factor) at install, default 1. P = machines ÷ R, rounded down; leftover machines are unused (7 machines with R = 2 gives P = 3 and one idle machine).

## HA sizing

- HA needs R ≥ 2 and a cluster of at least 3 machines.
- Smallest HA cluster: 3 machines as 1 partition × 3 replicas. Smallest HA cluster with partitioning: 2 × 2 (4 machines).
- ZooKeeper and ETCD need more than half of their nodes up, independent of R. That is why HA needs 3 machines even when R = 2.

## Replication

- All replicas are equal and serve both reads and writes.
- A write goes to all replicas and completes only when all acknowledge, so write latency follows the slowest replica.
- A read can be served by any replica and sees every write that completed before it. Query load spreads across replicas, so read throughput grows with R.
- To pin a REST query to one replica, send the HTTP header `GSQL-REPLICA: <n>` with n in 1..R.

## When a node fails

- Requests are routed away from the failed node. Queries running on it can fail while the cluster adjusts (typically up to about 30 seconds), so clients should retry on transient errors.
- Until the node is recovered, replaced or removed, these are blocked or limited:
  - schema changes;
  - installing new queries (interpreted and already-installed queries still run);
  - connector-based loading (S3, Kafka and other data sources) into the affected partition;
  - database export;
  - backup `[3.10 only]`.
- If the node can't be recovered, removing it from the cluster restores full operation; that partition then runs with one fewer replica.

## Data distribution and skew

Inside each copy of the data, vertices hash into buckets, and buckets are spread round-robin over the P partitions. The number of buckets is `2^GPE.NumberOfHashBucketInBit`; the default 5 gives 32 buckets.

When the bucket count doesn't divide evenly by P, some partitions get an extra bucket and hold more data. Skew is worst when `buckets mod P` is near P/2 and smallest when it is near 0 or near P. For example, P = 8 divides 32 exactly; P = 9 leaves a remainder of 5 out of 9, which skews noticeably.

Pick the lowest bucket count that leaves the remainder near 0 or near P: more buckets also mean more resource contention. Don't go below the default. Set it before loading data. On a cluster that already holds data it is not a simple config change; ask TigerGraph Support first.

## Changing the layout

- **More machines:** expand the cluster (`--ha` also sets a new R, which is how a 2-node R = 1 cluster becomes HA); data is redistributed.
- **Different P × R on the same machines** (for example 6 × 2 to 4 × 3): repartition in place.
- Both need a backup first and no loading jobs or queries running, and both take the cluster down for a few minutes (the docs' examples: about 4 minutes for 100 GB per copy, 9 minutes for 500 GB). There is no data reload.
- The commands are in the tigergraph-cli skill (`references/gadmin.md`, "cluster").

## Topologies

- **Single node** (1 × 1): development and small workloads, no HA.
- **Distributed without HA** (P > 1, R = 1): suits workloads whose data can be reloaded after a failure.
- **HA** (R ≥ 2): the normal production setup.
- **Region-aware HA** `[4.1+]`: replicas placed across availability zones or regions so the cluster survives a zone or region outage.
- **Cross-region replication (CRR)**: a separate disaster-recovery cluster kept in sync asynchronously.
