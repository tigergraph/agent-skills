# Services

The services `gadmin status` lists and what each does.

## At a glance

| Service | Role |
|---|---|
| GPE | Graph Processing Engine: holds the graph, runs queries and updates |
| GSE | Graph Storage Engine: runs the ID service that translates user IDs and internal IDs |
| RESTPP | REST API: built-in endpoints, installed-query endpoints, loading requests |
| GSQL | GSQL server: schema, loading jobs, query compilation; serves the `gsql` client |
| GUI | Backend for GraphStudio and Admin Portal |
| NGINX | HTTP gateway and TLS termination |
| CTRL | Controller: the cluster-management service `gadmin` talks to |
| EXE | Executor: carries out management actions on its node |
| IFM | Informant: health and metrics |
| ADMIN | Per-node management service |
| DICT | Dictionary: catalog and system metadata |
| ETCD | Configuration and coordination store |
| ZK | ZooKeeper: distributed coordination |
| KAFKA | Internal message passing between services |
| KAFKACONN | Kafka Connect runtime for `DATA_SOURCE` loading (S3, Kafka, warehouses) |
| KAFKASTRM-LL | Aggregates loading statistics for loading-status queries |
| TS3, TS3SERV `[before 4.1]` | Monitoring backend for Admin Portal; removed in 4.1 |

ZooKeeper and ETCD need more than half of their nodes up, which is why HA needs at least 3 machines. Run `gssh` with no arguments to see where each service runs on a given cluster.

## Reading instance IDs

`gadmin status -v` and `ghostname` show instance IDs. For GPE and GSE, `GPE_2#1` is partition 2, replica 1. Other services show one number that identifies the instance, such as `RESTPP#1` or `EXE_1`; don't read it as a partition.

Loading jobs run in the file loader and the Kafka loader rather than as separate `gadmin` services; their logs appear under those names.

## Start order

`gadmin start` and `gadmin restart` bring services up in this order:

```
ZK → ETCD → DICT → KAFKA → ADMIN → GSE → NGINX → GPE → RESTPP → KAFKASTRM-LL → KAFKACONN → GSQL → GUI
```

When a service won't start, check the services before it first, starting from ZK, ETCD and Kafka at the bottom. A problem low in the order (for example ETCD) shows up as failures in everything above it.
