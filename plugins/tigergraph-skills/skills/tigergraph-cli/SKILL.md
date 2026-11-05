---
name: tigergraph-cli
description: Operate and troubleshoot a self-managed TigerGraph cluster from the shell — `gadmin` (service status, start/stop/restart, configuration, logs, backup, cluster resize), `gsql` (schema, loading jobs, queries, users) and the cluster scripts (`grun`, `gssh`, `gcollect`, ...). Load it before running commands on a TigerGraph node for any task: a health check, a service that is down, queries or REST calls that hang, fail or time out, a loading job that failed or loaded fewer rows than expected, following one request through the logs, changing a setting, taking a backup, wiping or reloading a graph's data. Also use it when the user asks how to run one of these commands, whether a command (including a GSQL statement) is safe or right for the job, or pastes their output, even if they don't name the command. Does NOT cover how the services fit together — use tigergraph-cluster. Does NOT cover writing GSQL query logic — use the GSQL docs or the tigergraph MCP plugin.
compatibility: TigerGraph 3.10 or later, self-managed. Needs a shell on a cluster node as the TigerGraph OS user; not applicable to TigerGraph Savanna.
license: Apache-2.0
metadata:
  author: TigerGraph
  version: "0.1.0"
---

# TigerGraph CLI

A map of the commands a user runs on a TigerGraph node: which tool fits the job, where its details live, and the traps worth knowing. The installed version's help and the docs hold the details.

## How to work

- **Doing a task on the cluster** (diagnose a problem, change a setting, take a backup): look before you change anything (`gadmin status -v`, logs, `gadmin config get`), make the smallest change that does the job, then verify it took effect (`gadmin status`, `config get`, `backup list`). Report what you ran, what you found and what changed in a few lines.
- **Cover every node.** Logs, disk and processes are per node, and requests reach whichever node's NGINX/RESTPP the client called. List the nodes with `gadmin config get System.HostList` (one line with every node and IP; `gssh` output is long and easy to cut off), then check all of them (`grun all '<cmd>'`), or search every node's logs at once with `gcollect`. A finding from the local node alone describes part of the cluster.
- **A service is Down:** before starting it, find out whether it crashed or was stopped (`references/gadmin.md`, "status"); that decides whether it will happen again. If the cluster has one replica (`gadmin status -v` shows only `#1` instances), nothing else serves that partition; the lasting fix is HA, which needs a replication factor of at least 2 and more machines (tigergraph-cluster).
- **Reproduce what the user sees, the way they see it.** If an app's REST call fails, make the same REST call (`curl` to the NGINX port, `$(gadmin config get Nginx.Port)`), not a `gsql` equivalent: the error the app gets is the one to explain.
- **"Don't change anything" or "don't run it yet"** rules out the change, not read-only checks. Look at the cluster (which graphs exist, current values, recent logs) so the answer fits it.
- **Answering a question:** lead with the command to run next, or the cause and fix. Put every command or query the user needs in the reply itself; files you wrote along the way aren't visible to them. Add background only where it changes what the user should do, keep warnings to the ones that apply, and point to the docs page for the rest.

## Where answers come from

The cluster in front of you and the docs for the user's version are the sources of truth. This skill says where to look and warns about traps they don't make obvious; its flag and entry names are reminders, so check them before you run them or hand them to the user.

- **Commands and flags:** `gadmin <subcommand> -h` works at every level and shows the flags, whether there is a `-y`, examples and caveats. `gsql -h`, `gsql help basic` / `help query` and `gcollect -h` do the same; the other cluster scripts print usage when run with no arguments, and `ls $(gadmin config get System.AppRoot)/cmd/` lists them.
- **Settings:** `gadmin config list entry` prints every entry with a short description and an example value. Search it with context, `gadmin config list entry | grep -i -A2 <keyword>`, then read this cluster's value with `gadmin config get <Entry>`. When an entry name in the docs or here differs from the live list, the live list is right. Read paths the same way (`System.AppRoot`, `System.DataRoot`, `System.LogRoot`) instead of assuming them.
- **Procedures and behaviour** (install, upgrade, backup semantics, HA, security): fetch the docs page for the user's version ("Official documentation" below) rather than recalling it.
- **Another version:** help and config output describe the version installed here. When the user's cluster runs a different release, use that release's docs and point out where it differs.
- **Which release added something:** the docs are versioned, so check whether the feature's page exists in each release's docs (a page that returns 404 for 3.10 and 200 for 4.1 arrived in 4.1). A quick fetch settles most "is this supported on our version" questions; do that before reporting it as unverified.

Do the lookup yourself and give the user the concrete command, value or step it produced; "`-h` shows the syntax" or "see the docs" is not an answer. Say briefly what you checked ("`gadmin backup create -h` on this 4.4 cluster", "the 4.3 upgrade page") and mark as unverified only what you couldn't check.

## Before you start

- These tools live on cluster nodes, on the `PATH` of the OS user that installed TigerGraph (often `tigergraph`). Agent shells are often non-login shells that miss that `PATH`. Only if `gadmin` isn't found, try `bash -lc 'gadmin version'`; if that works, run TigerGraph commands through `bash -lc`, otherwise call them directly.
- If you can't run commands on a node yourself (you're on the user's laptop, or the cluster is remote), write the commands for the user to run and tell them what to look for in the output.
- Check the version (`gadmin version`, or ask the user) before relying on anything tagged `[4.1+]`, `[4.3+]` or `[before 4.1]`; untagged items work on 3.10 and later.
- TigerGraph Savanna (the managed cloud service) gives no node shell. Savanna users pause, resume and manage workspaces in the Savanna console or its REST API, run GSQL in the console's GSQL Editor, and can connect agents through the `tigergraph` MCP plugin.

## Running commands as an agent

Several commands wait for input, which blocks a non-interactive shell:

- `gadmin` subcommands that interrupt or delete (`stop`, `restart`, `config apply`, `backup remove`, ...) ask "(y/N)"; add `-y` only once the user has agreed to the action (see Safety). Not every subcommand prompts or accepts `-y`; its `-h` says.
- `gsql` with no arguments opens an interactive shell; pass a statement (`gsql 'ls'`) or a script file instead.
- `gsql -u <user>`, `CREATE USER` and `gadmin config entry` / `group` prompt for values. Ask the user to run these themselves rather than handling their passwords.

## The command families

| Family | Entry point | Covers | Reference |
|---|---|---|---|
| Cluster control | `gadmin` | status, start/stop/restart, config, logs, metrics, backup, cluster resize, license, connectors, cross-region replication | `references/gadmin.md` |
| Graph language | `gsql` | schema (DDL), loading jobs, queries (interpreted and installed), users and roles | `references/gsql.md` |
| Cluster scripts | `grun`, `grun_p`, `gscp`, `gfetch`, `gssh`, `gcollect`, `ghostname`, `gmyip`, `gstatusgraph` | run a command or copy files across nodes, cluster topology, cross-node log search | `references/cluster-scripts.md` |
| Bundled services | `kafka-*.sh` and friends | read-only diagnostics of the Kafka, ETCD and ZooKeeper that TigerGraph runs internally | `references/bundled-services.md` |
| Install and lifecycle | `install.sh`, `gadmin cluster`, `gadmin config` | install prerequisites and `install_conf.json`, upgrade, changing a node's IP, replacing a lost node, firewall ports, HTTPS, query memory limits | `references/lifecycle.md` |

Read the reference for the family you need; each is short.

## Targeting services and nodes

`gadmin` commands that take services (`status`, `start`, `stop`, `restart`, `run`) accept service IDs:

```
<Service>[_<partition>][#<replica>]    e.g. GPE_1#2 = GPE, partition 1, replica 2
```

Drop trailing fields to widen the match (`GPE_1` = every replica of partition 1, `gpe` = all GPEs). Only GPE and GSE have both fields; other services carry one instance number (`RESTPP#1`, `EXE_1`).

The cluster scripts `grun`, `grun_p`, `gscp` and `gfetch` take `all`, a lowercase service name (`gpe`), or a node list (`m1,m2`), not instance IDs. `gssh` takes a node name.

Node names such as `m1` and `m2` are TigerGraph's aliases, not hostnames, so a plain `ssh m2` usually fails. Reach other nodes with the cluster scripts (`grun m2 '<cmd>'`, `gssh m2 '<cmd>'`), or ssh to the IP that `gssh` lists. The remote shell may not have TigerGraph's tools on its `PATH` ("gadmin: command not found"); call them by full path, `$(gadmin config get System.AppRoot)/cmd/<tool>`, which is the same on every node.

## Which command for which job

- **Health check:** `gadmin status -v`, then what services don't show: disk on every node (`grun all 'df -h'`), clocks in sync across nodes (the cluster needs NTP or similar; compare `grun all date` and check a time service is running), and `gadmin license status`.
- **Look, change nothing:** `gadmin status -v`, `gadmin config get`, `gadmin log`, `gadmin metric`, `ghostname`, `gssh` with no arguments, `gstatusgraph`.
- **Lifecycle and configuration:** `gadmin start | stop | restart`; `gadmin config set` then `gadmin config apply` (changes are staged until applied; restart the affected services afterwards, or use `apply --restart-deps` `[4.1+]`).
- **Schema, data, queries, users:** `gsql`. Inventory, counts and schema-change rules are in `references/gsql.md`: check whether a type is global or local first, and expect a schema change to disable loading jobs and uninstall queries that use what it drops or alters.
- **Resource use:** `gadmin metric` per service. On containers and shared hosts, `free` and `nproc` show the host, not the node's limits, so compare against the container's limits before concluding anything.
- **Loading from Kafka, S3 or other sources:** these run as data sources and loading jobs (`gsql 'SHOW DATA_SOURCE *'`, `gadmin connector list`). The KAFKA, KAFKACONN and KAFKASTRM-LL services in `gadmin status` are TigerGraph's internal plumbing, present whether or not any Kafka source is configured.
- **Did a load work, what was rejected:** each loading run writes a summary with rejected lines by reason; `gadmin log` shows the loader log directories (`references/gsql.md`, "Loading results").
- **What is running now, stop one query:** the REST built-in endpoints list running queries and abort one by request ID (docs: `API/built-in-endpoints`); prefer that to restarting GPE.
- **Who can log in, what they can do, whether REST needs a token:** `gsql 'SHOW USER'`, roles granted per graph, and `gadmin config get RESTPP.Factory.EnableAuth` (docs: `user-access/` pages).
- **Same shell command on many nodes:** `grun` (ordered output) or `grun_p` (parallel).
- **Copy files between nodes:** `gscp` (push), `gfetch` (pull).
- **Search logs across nodes or follow one request:** `gcollect`.
- **Install, upgrade, change a node's IP, replace a lost node, firewall ports, HTTPS, query memory limits:** `references/lifecycle.md` names the docs page for each and what is easy to miss.
- **License status or renewal:** `gadmin license status` shows EndTime. Once it passes, database operations stop, so renew before then: a seed file goes to TigerGraph, the new key is installed with `gadmin license set`, then the config is applied and services restarted (`references/gadmin.md`, "license", and the docs page `installation/license`).
- **Collect diagnostics for TigerGraph Support:** the support script described in `references/cluster-scripts.md`.

## Safety

These commands delete data, change the cluster layout, or interrupt work:

- `gadmin reset`, `gadmin init`, `gsql --reset`, `DROP ALL`, and `CLEAR GRAPH STORE` (deletes the data of every graph, not only the one in use; to empty one graph, delete its vertex types' data instead);
- deleting data (REST delete endpoints, `DELETE` in queries): deleting a vertex also deletes its edges, so emptying one vertex type can empty edge types the user didn't mention;
- schema changes, including adding an attribute: they can disable loading jobs (for good) and uninstall queries that use the type (`references/gsql.md`, "Schema changes");
- `gadmin cluster expand`, `shrink` and `remove`;
- `gadmin backup restore` (offline while it runs, rolls back every graph, can't be cancelled once started; to roll back one graph, export/import that graph instead), `remove` and `retain`;
- `gadmin upgrade`;
- restarts: `gadmin restart all`, and `gadmin config apply --restart-deps`.

Before running one of these, tell the user what it will do and what it can't undo, and get their go-ahead. A request covers only what the user can see they asked for: "delete the products" is a go-ahead to delete products, not the orders that go with them. If the action reaches further (edges, other graphs, disabled jobs, restarts of services others use), describe that first and wait for confirmation, even when the user said "go ahead". If you've been told not to change the cluster, give the commands instead of running them. When you only write commands for the user to run, mark these as destructive instead. Some documented procedures use these commands on purpose (changing a node's IP re-initialises ETCD and the cluster; upgrades go through the installer). For those, give the documented steps for the user's version, with the downtime and a backup first, rather than improvising a shorter path or only sending the user to Support.

Passwords on the command line (`gsql -p ...`) end up in shell history and process lists. Prefer letting `gsql -u <user>` prompt; for scripted access, use the REST API with a token.

Kafka, ETCD and ZooKeeper hold cluster state, and a wrong repair can leave the cluster unrecoverable. When a problem looks like it needs one of them repaired or reinitialised, explain that risk and recommend a TigerGraph Support ticket as the default. If the user still wants to proceed, or a `gadmin` message itself names the recovery command, help them do it carefully: take a backup first, run only that command, and check `gadmin status` afterwards.

## Official documentation

The TigerGraph Server docs are the source of truth. Pages follow `https://www.tigergraph.com/docs/tigergraph-server/<version>/<page>`, where `<version>` is the major.minor release, such as `3.10`, `4.1`, `4.2` or `4.3`; there is no `current` or `latest` alias. Fetch the page for the user's version when you need detail beyond this skill: with a web-fetch tool if you have one, otherwise `curl -sL <url> | sed 's/<[^>]*>//g'` from a shell gives readable text. A release whose docs aren't published yet returns 404: use the closest earlier version and say so. Prefer the installed `--help` when the two disagree.

| Topic | Page |
|---|---|
| `gadmin` commands | `system-management/management-commands` |
| Configuration entries | `reference/configuration-parameters` |
| Cluster scripts (`grun`, `gscp`, `gssh`, ...) | `cluster-and-ha-management/cluster-commands` |
| `gcollect` | `troubleshooting/gcollect` |
| Troubleshooting and the support script | `troubleshooting/troubleshooting-guide` |
| Remote `gsql` client | `gsql-shell/using-a-remote-gsql-client` |
| Backup and restore | `backup-and-restore/` |
| Expand, shrink, repartition | `cluster-and-ha-management/expand-a-cluster`, `shrink-a-cluster`, `repartition-a-cluster` |
| Install, upgrade, post-install checks | `installation/bare-metal-install`, `installation/upgrade`, `installation/post-install-check` |
| Change IP or hostname | `installation/change-ip-or-hostname` |
| Replace or remove a failed node | `cluster-and-ha-management/replace-a-node`, `remove-failed-node` |
| Ports | `reference/ports` |
| HTTPS / SSL | `security/encrypting-connections` |
| Memory limits | `system-management/memory-management` |
| REST built-in endpoints (running queries, abort, statistics) | `API/built-in-endpoints` |
| Users, roles, REST authentication | `user-access/user-management`, `user-access/enabling-user-authentication` |
| License | `installation/license` |
| Log files and rotation | `troubleshooting/log-files` |
| Audit logs | `troubleshooting/audit-log` |

The GSQL language reference (schema changes, loading jobs, query syntax) has its own versioned site: `https://www.tigergraph.com/docs/gsql-ref/<version>/<page>`, for example `ddl-and-loading/modifying-a-graph-schema`.

For TigerGraph Savanna, see `https://www.tigergraph.com/docs/savanna/main/overview/`.
