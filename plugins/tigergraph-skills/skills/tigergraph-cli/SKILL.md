---
name: tigergraph-cli
description: TigerGraph's command-line tools on a cluster node — `gadmin` (service status, start/stop, configuration, logs, backup), `gsql` (schema, loading jobs, queries, users), and the cluster scripts (`grun`, `gscp`, `gssh`, `gcollect`, ...). Load when the user wants to operate or inspect a self-managed TigerGraph cluster from the shell, or asks about any of these commands. Does NOT cover how the services fit together — use tigergraph-architecture. Does NOT cover writing GSQL queries.
compatibility: TigerGraph 3.10 or later, self-managed. Needs shell access to a cluster node as the TigerGraph OS user; not applicable to TigerGraph Savanna.
license: Apache-2.0
metadata:
  author: TigerGraph
  version: "0.1.0"
---

# TigerGraph CLI

<!-- OUTLINE: section plan, not final text -->

## Before you start
- Confirm the agent is on a cluster node as the TigerGraph OS user; otherwise stop and say so.
- Detect the version with `gadmin version`; tagged features below need the stated release.
- Savanna users have no node shell: point them to the Savanna console or the `tigergraph` MCP plugin.

## The three command families
- Table: family | entry point | covers | reference file.
  - Cluster control: `gadmin` → `references/gadmin.md`
  - Graph language: `gsql` → `references/gsql.md`
  - Cluster scripts: `grun`, `gscp`, `gfetch`, `gssh`, `gcollect`, ... → `references/cluster-scripts.md`
- One line on bundled services (Kafka, ETCD, ZooKeeper): read-only diagnostics only → `references/bundled-services.md`.

## Learn any command by asking it
- `--help` first; `gadmin <cmd> --help` recursive; scripts print usage with no args.

## Find paths and settings with `gadmin config`
- `System.AppRoot`, `System.DataRoot`, `System.LogRoot`; list-then-get for entry names. Never hardcode version directories.

## Service-ID syntax
- `<Service>[_<partition>][#<replica>]`, e.g. `GPE_1#2`; dropping fields broadens the match; `all` and host lists.

## Which command for which job
- Inspect only: `gadmin status -v`, `gadmin config get`, `gadmin log`, `gadmin metric`.
- Lifecycle and config: `gadmin start|stop|restart`, `gadmin config set` + `apply`.
- Schema, data, queries, users: `gsql`.
- Cluster-wide shell and files: `grun`/`grun_p`, `gscp`/`gfetch`.
- Logs across nodes: `gcollect`.

## Safety
- Destructive commands (`gadmin reset`, `gadmin init`, `gadmin cluster shrink`, backup restore, `gsql --reset`, `DROP ALL`): show the command and ask the user before running.
- Anything that needs repairing internal services: stop and recommend TigerGraph Support.
