---
name: tigergraph-architecture
description: How TigerGraph works inside — its services (GPE, GSE, GSQL, RESTPP, NGINX, GUI, and the cluster-management and messaging services), the partition × replica cluster model, high availability, and how schema changes, loading and queries flow through a cluster. Load when the user asks how TigerGraph works, plans cluster size or HA, reads service names in `gadmin status`, or needs to know which service is involved in a problem. Does NOT cover running commands — use tigergraph-cli.
compatibility: TigerGraph 3.10 or later. Written for self-managed clusters; on TigerGraph Savanna the same concepts apply but the cluster is managed for you.
license: Apache-2.0
metadata:
  author: TigerGraph
  version: "0.1.0"
---

# TigerGraph Architecture

<!-- OUTLINE: section plan, not final text -->

## What TigerGraph is
- Distributed native parallel graph database.
- Core engine: GSE (ID service, maps user IDs to internal IDs) and GPE (stores attributes, runs queries and updates).
- Around it: gateways (NGINX, RESTPP, GSQL, GUI), cluster management (CTRL, EXE, IFM, ADMIN), state (ETCD, DICT, ZooKeeper), messaging (Kafka and its loading services).

## Partitions and replicas
- Cluster = P partitions × R replicas; you set R, P follows from cluster size.
- Replicas are active-active: writes go to all replicas synchronously, reads from any one.
- HA needs R ≥ 2 and at least 3 machines.

## How work flows
- Schema change, loading, and query, one line each, with a pointer to `references/data-flows.md`.

## Common questions → where to look
- "What is this service in `gadmin status`?" → `references/services.md`
- "How many machines for HA?" / "Why is data skewed?" → `references/clustering.md`
- "Which service handles loading / my query?" → `references/data-flows.md`
- "What happens when I run a gadmin command?" → `references/cluster-management.md`

## Acronyms
- HA, AZ, IDS; service acronyms are expanded in `references/services.md`.
