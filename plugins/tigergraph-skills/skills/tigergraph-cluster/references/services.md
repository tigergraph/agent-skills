# Services

The services `gadmin status` lists, what each does, and how many instances run by default.

## At a glance

| Service | Role | Default instances |
|---|---|---|
| GPE | Graph Processing Engine: holds the graph, runs queries and updates | one per partition × replica |
| GSE | Graph Storage Engine: runs the ID service that translates user IDs and internal IDs | one per partition × replica |
| RESTPP | REST API: built-in endpoints, installed-query endpoints, loading requests | every node |
| GSQL | GSQL server: schema, loading jobs, query compilation; serves the `gsql` client | up to 5 nodes; one leader (`gsql --leader`) |
| GUI | Backend for GraphStudio and Admin Portal | up to 3 nodes |
| NGINX | HTTP gateway and TLS termination | every node |
| CTRL | Controller: the cluster-management service `gadmin` talks to | up to 3 nodes |
| EXE | Executor: carries out management actions on its node | every node |
| IFM | Informant: health and metrics | up to 3 nodes |
| ADMIN | Per-node management service | every node |
| DICT | Dictionary: catalog and system metadata | odd number of nodes, at most 5 |
| ETCD | Configuration and coordination store | odd number of nodes, at most 5 |
| ZK | ZooKeeper: distributed coordination | odd number of nodes, at most 5 |
| KAFKA | Internal message passing between services | every node |
| KAFKACONN | Kafka Connect runtime for `DATA_SOURCE` loading (S3, Kafka, warehouses) | every node |
| KAFKASTRM-LL | Aggregates loading statistics for loading-status queries | every node |
| TS3, TS3SERV `[before 4.1]` | Monitoring backend for Admin Portal; removed in 4.1 | |

The counts are installer defaults. DICT, ETCD and ZK use the largest odd number of nodes up to 5, so a 2-node cluster has one of each and a 4-node cluster three; that is why HA needs at least 3 machines. Run `gssh` with no arguments to see where each service runs on a given cluster.

## Reading instance IDs

`gadmin status -v` and `ghostname` show instance IDs. For GPE and GSE, `GPE_2#1` is partition 2, replica 1. Other services show one number that identifies the instance, such as `RESTPP#1` or `EXE_1`; don't read it as a partition.

Loading jobs run in the file loader and the Kafka loader rather than as separate `gadmin` services; their logs appear under those names.

## Start order

`gadmin start` and `gadmin restart` bring services up in this order:

```
ZK → ETCD → DICT → KAFKA → ADMIN → GSE → NGINX → GPE → RESTPP → KAFKASTRM-LL → KAFKACONN → GSQL → GUI
```

When a service won't start, check the services before it first, starting from ZK, ETCD and Kafka at the bottom. A problem low in the order (for example ETCD) shows up as failures in everything above it.
