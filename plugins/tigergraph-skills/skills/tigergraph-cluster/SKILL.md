---
name: tigergraph-cluster
description: How TigerGraph works inside — its services (GPE, GSE, GSQL, RESTPP, NGINX, GUI, CTRL, EXE, IFM, ADMIN, DICT, ETCD, ZooKeeper, Kafka, KAFKACONN, KAFKASTRM-LL, and the file and Kafka loaders), the partition × replica cluster model, high availability and failover, data distribution and skew, and how schema changes, data loading and queries flow through a cluster. Use this skill whenever the user asks how TigerGraph works, plans cluster size or HA, sees a service name in `gadmin status` or a log and wants to know what it does, asks why an operation is blocked when a node is down, or needs to know which service to look at for a problem. Does NOT cover running commands — use tigergraph-cli.
compatibility: TigerGraph 3.10 or later. Written for self-managed clusters; on TigerGraph Savanna the same concepts apply but the cluster is managed for you.
license: Apache-2.0
metadata:
  author: TigerGraph
  version: "0.1.0"
---

# TigerGraph Cluster

A high-level picture of a TigerGraph cluster, enough to reason about sizing, availability and where a problem lives. Details are in `references/`. Items tagged `[4.1+]`, `[3.10 only]` or `[before 4.1]` apply only to those releases; when it matters, ask the user for their version (`gadmin version` on a node).

## How to respond

Lead with the answer (the sizing, the cause, the service to look at), then only the background that explains it, and point to the docs page for the rest. When diagnosing a live cluster, use the service roles and start order here to decide where to look, then confirm with real output (`gadmin status -v`, the service's logs) before naming a cause; the commands are in the tigergraph-cli skill.

## What TigerGraph is

A distributed native parallel graph database. The core engine is written in C++ and has two parts:

- **GPE** (Graph Processing Engine) holds the graph and executes queries and updates.
- **GSE** (Graph Storage Engine) runs the ID service (IDS), which translates between the IDs users see and internal IDs.

Around the engine:

- **Entry points:** NGINX (HTTP gateway and TLS) in front of RESTPP (REST API and query runtime), GSQL (schema, loading jobs, query compilation; serves the `gsql` client) and GUI (GraphStudio and Admin Portal).
- **Cluster management:** `gadmin` works through CTRL (the controller), EXE (the executor, one per node) and IFM (the informant: health and metrics). ADMIN is a per-node management service.
- **Metadata and coordination:** DICT (the dictionary: catalog and system metadata), ETCD (configuration and coordination) and ZooKeeper (distributed coordination).
- **Messaging and loading:** an internal Kafka for message passing between services; KAFKACONN runs Kafka Connect for `DATA_SOURCE` loading; KAFKASTRM-LL aggregates loading statistics for loading-status queries.

## Partitions and replicas

A cluster is a grid of **P partitions × R replicas** with `cluster size = P × R`.

- You choose R (the replication factor) at install; P is the number of machines divided by R, and leftover machines stay unused.
- R = 1 means no high availability. HA needs R ≥ 2 and at least 3 machines.
- All replicas are equal and serve both reads and writes: a write goes to every replica and completes only when all acknowledge; a read can be served by any replica.

## How work flows

- **Schema change or query install:** client → GSQL → catalog, applied to every replica together, so a down node blocks it.
- **Loading:** local files through the file loader, `DATA_SOURCE` sources (S3, Kafka, warehouses) through KAFKACONN, or the Spark connector through RESTPP; the data lands in GSE (IDs) and GPE (graph).
- **Query:** client → NGINX → RESTPP → GSE translates IDs and GPE runs the query in parallel across partitions → RESTPP → client.

## Where to look next

| Question | Reference |
|---|---|
| What does this service in `gadmin status` do? What should start before it? | `references/services.md` |
| How many machines for HA? What happens when a node fails? Why is data skewed? | `references/clustering.md` |
| Which services handle loading, queries, schema changes? Which logs to read? | `references/data-flows.md` |
| What happens behind `gadmin` commands, config changes and cluster resizing? | `references/cluster-management.md` |

For the commands themselves, use the tigergraph-cli skill.

## Official documentation

The TigerGraph Server docs are the source of truth. Pages follow `https://www.tigergraph.com/docs/tigergraph-server/<version>/<page>`, where `<version>` is the major.minor release, such as `3.10`, `4.1`, `4.2` or `4.3`; there is no `current` or `latest` alias. Fetch the page for the user's version when you need detail beyond this skill. A release whose docs aren't published yet returns 404: use the closest earlier version and say so.

| Topic | Page |
|---|---|
| Architecture overview | `intro/internal-architecture` |
| Continuous availability, replication | `intro/continuous-availability-overview` |
| HA cluster sizing, bucket skew | `cluster-and-ha-management/ha-cluster` |
| What is limited when a node fails | `cluster-and-ha-management/ha-overview` |
| Region-aware clusters | `cluster-and-ha-management/region-aware` |
| Glossary of services | `reference/glossary` |
