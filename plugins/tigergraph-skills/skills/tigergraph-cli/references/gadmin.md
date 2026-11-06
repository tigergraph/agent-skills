# gadmin

The cluster-control CLI. `gadmin -h` lists the subcommands of the installed version and `gadmin <subcommand> -h` their flags and examples; this page covers how to use them well and what the help doesn't say.

## Subcommands

| Subcommand | Purpose |
|---|---|
| `status` | Running state of services |
| `start`, `stop`, `restart` | Service lifecycle, in start order |
| `config` | Read and change configuration (staged, see below) |
| `log` | List the log files of a service |
| `metric` | CPU, memory, disk and network use per service |
| `run` | Run a shell command on the hosts of a service |
| `backup` | Create, list, verify, restore and clean up cluster backups |
| `cluster` | Expand, shrink or repartition the cluster; remove a failed node |
| `connector`, `crr` | Data-source connectors; cross-region replication |
| `license` | Show, request and install the license |
| `version` | Show the installed version |
| `init`, `reset` | Initialise or wipe services and their data |

## status

```
gadmin status                # every service, one line each
gadmin status -v             # per instance (the _partition#replica suffixes)
gadmin status gpe gse        # only these services
```

A service that is not `Online` is the first thing to explain. `gadmin log <service>` tells you where to look next.

Before starting a `Down` service again, find out whether it crashed or was stopped. A crash leaves errors near the end of that service's own log (`gadmin log <service>`). A deliberate stop shows a clean shutdown there, and the executor log on that node (`$(gadmin config get System.LogRoot)/executor/`) records the stop request and the host it came from. The answer decides whether it will happen again.

## config: staged changes

`set` only stages a change; `apply` makes it live, so a typo can be caught with `diff` first and dropped with `discard`.

To find the entry for a setting, search the live list, which describes every entry: `gadmin config list entry | grep -i -A2 <keyword>` (try a few keywords: `timeout`, `mem`, `log`, `port`, `ssl`, `backup`). `gadmin config list group` shows the groups the entries belong to. Then read the current value with `gadmin config get <Entry>`.

Plain `apply` writes the change but restarts nothing; it ends with "Please restart services to make it effective immediately". Restart the services the entry belongs to (the prefix usually names the service: `GPE.*` → GPE), or use `apply --restart-deps` `[4.1+]`, which restarts exactly those services. Either way the restart interrupts running queries and loading jobs, so on a busy cluster agree a time with the user first.

Some settings aren't entries of their own but options packed into one entry's value (for example a service's `BasicConfig.Env`). When the docs say to add an option there, `get` the current value and append to it rather than replacing it.

Don't dump the whole configuration to read one value: it includes passwords and keys. Read single entries with `get`, and not the ones that hold secrets (`SKILL.md`, "Where answers come from").

## start, stop, restart

Pass service names or instance IDs (`gpe`, `GPE_1#2`, `all`). Prefer restarting only the affected services over `all`, which interrupts every running query and loading job. `-h` lists options such as waiting until the service is online.

## run, log, metric

`gadmin run <service|all> "<cmd>"` runs a shell command on the hosts of a service (`grun` in `cluster-scripts.md` does the same over plain ssh). `gadmin log <service>` lists the log files on each node; `gadmin metric -h` lists the resource views.

## backup

- Backups go to the storage the `System.Backup.*` entries configure (a local path or a cloud bucket). If neither is enabled, `backup create` has nowhere to write, so check those entries before planning backups.
- `gadmin backup list` shows what exists. If it's empty there is nothing to restore; say so rather than guessing at a tag.
- `gadmin backup create -h` and `restore -h` list the options of the installed version (incremental backups, keeping users, staging path, ...). How incremental backups behave has changed between releases, so read the backup docs for the user's version.
- A restore takes the database offline while it runs, rolls back every graph to the backup and can't be cancelled once started. To roll back a single graph, use single-graph export and import (docs: `backup-and-restore/single-graph-import-export`).
- `restore`, `remove` and `retain` are destructive (see `SKILL.md`, "Safety").

## license

`gadmin license status` shows the edition, limits and EndTime. Renewing goes through TigerGraph: generate a seed file (`gadmin license seed -h`), send it to TigerGraph as the license docs page (`installation/license`) describes, install the key you get back (`gadmin license set -h`), apply the config, and restart at an agreed time so it takes effect. Check `license status` afterwards. Without a valid license, database operations stop, so renew before EndTime.

## cluster

`gadmin cluster <expand|shrink|remove> -h` has examples for adding nodes, changing the replication factor and removing a failed node. All of them move data or change the layout: take a backup first, stop loading jobs and queries, and plan for downtime. Background on what each layout means is in the tigergraph-cluster skill.
