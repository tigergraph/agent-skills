# Cluster scripts

Small utilities in `$(gadmin config get System.AppRoot)/cmd/`, on the `PATH` of the TigerGraph OS user. They make cluster-wide work easy without going through `gadmin`.

## Where am I, what runs here

```
ghostname        # this node's name and the service instances on it, e.g. "m1 GPE_1#1 GSE_1#1 ..."
gmyip            # this node's private IP
gssh             # no arguments: every node's name and IP, and which services run where
gstatusgraph     # per-node graph size and vertex/edge counts
```

`gssh` with no arguments answers "is this service on one node or all of them?" before you start searching logs. `gstatusgraph` is the quickest check that data is present on every node.

## Run a command on many nodes: grun, grun_p

```
grun all "uptime"            # one node after another, output in order
grun_p all "df -h"           # all nodes in parallel, faster on large clusters
grun m1,m2 "free -g"         # explicit node list
```

Targets are `all`, a node list (`m1,m2`), or a lowercase service name (`gpe` runs on every node that hosts GPE). Instance IDs such as `GPE_1#2` are not accepted; use `gadmin run` for those.

## Copy files: gscp, gfetch

```
gscp all /tmp/script.sh /tmp/        # push a local file to every node
gfetch all /path/to/file.log /tmp/logs/   # pull a file from every node
```

## Connect to another node: gssh

```
gssh m2              # interactive ssh to node m2 using the cluster's keys
gssh m2 'uptime'     # one command
```

## Search logs across nodes: gcollect

Use `gcollect` when a question spans several services, a time window, or one request ID. It knows where every service writes its logs. `gcollect -h` lists the options, the component names `-c` accepts and worked examples; read it first. Its three commands:

- `show`: the requests in a time window (`gcollect -t 3600s show`);
- `collect`: gather matching log lines from every node into `./output` (or `-o <dir>`), optionally around one request (`-r`, with `-b`/`-d` seconds before and after);
- `grep`: search what `collect` gathered; `-D` prints the matches.

What the help doesn't make obvious:

- `-r` takes the middle of a request ID. A REST response carries the full ID in its `GSQL-Task-Key` header (and `requestid` fields), e.g. `17170433.RESTPP_2_1.1793702088230.N`; pass `RESTPP_2_1.1793702088230`, because gcollect rejects the full form as an illegal request_id. The full ID is what to grep for in the RESTPP and GPE logs.
- `-t` needs a unit (`300s`, `2h`); a bare number fails with "missing unit in duration", even though some versions of the docs show it that way.
- Component names are lowercase and some differ from the `gadmin` service names (RESTPP is `rest`); use the names `-h` lists.
- The output directory is not replaced if it exists, so use a fresh `-o` per collection.

## Collect diagnostics for TigerGraph Support

`gsupport_collect.sh` (in `cmd/` on recent releases; otherwise see the troubleshooting guide) gathers what Support needs. Run it on one node as `gsupport_collect.sh -t 86400s` (pass the unit: the script hands `-t` to `gcollect`, and its built-in default has none), then attach the resulting archive to the support ticket. It uses `gcollect`, `gssh` and `gstatusgraph` to gather recent logs and cluster state.

## Find the rest

`ls $(gadmin config get System.AppRoot)/cmd/` lists every script. Most print usage when run with no arguments.
