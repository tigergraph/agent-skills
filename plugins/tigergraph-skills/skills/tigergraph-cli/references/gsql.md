# gsql

The client for the GSQL language. It talks to the GSQL service, so `gadmin status gsql` must show it online.

## Ways to run it

```
gsql                          # interactive shell
gsql 'ls'                     # one statement, then exit
gsql my_script.gsql           # run a file of statements
gsql -g <graph> '...'         # set the working graph for this call
gsql -u <user> '...'          # log in as <user>; prompts for the password (let the user run it)
```

`gsql -v` prints the version. `gsql --reset` clears the whole catalog and graph store (see `SKILL.md`, "Safety").

## Inside the shell

```
ls                       # vertex and edge types, graphs, queries, loading jobs
USE GRAPH <name>         # switch graph (USE GLOBAL for global types)
SHOW VERTEX <Type>       # one type's definition; also SHOW EDGE, SHOW GRAPH
help basic               # schema and loading syntax summary
help query               # query language summary
```

`help basic` and `help query` show the exact syntax on the installed version, which is safer than recalling it from memory.

## Minimal end-to-end example

```
CREATE VERTEX Person (PRIMARY_ID id STRING, name STRING)
CREATE UNDIRECTED EDGE Knows (FROM Person, TO Person)
CREATE GRAPH social (Person, Knows)

USE GRAPH social
CREATE LOADING JOB load_people FOR GRAPH social {
  DEFINE FILENAME f1 = "/path/to/people.csv";
  LOAD f1 TO VERTEX Person VALUES ($0, $1) USING SEPARATOR=",", HEADER="true";
}
RUN LOADING JOB load_people

CREATE QUERY hello(VERTEX<Person> p) FOR GRAPH social { PRINT p; }
INSTALL QUERY hello
RUN QUERY hello("p1")
```

`INSTALL QUERY` compiles the query for production use and can take a while. `INTERPRET QUERY` runs it without installing, which is quicker to iterate on. `DROP ALL` removes every type, graph, job and query, so treat it like `--reset`.

## What's on the cluster

- `gsql ls` with no graph lists the global vertex and edge types and the graphs; `gsql -g <graph> ls` lists that graph's types, loading jobs and queries, and marks the installed ones. Statements such as `SHOW QUERY *` need a graph (`-g`).
- Vertex and edge counts: the REST built-in `stat_vertex_number` / `stat_edge_number` (`curl -X POST http://$(gmyip):$(gadmin config get Nginx.Port)/restpp/builtins/<graph> -d '{"function":"stat_vertex_number","type":"*"}'`) returns counts per type (its options are on the `API/built-in-endpoints` docs page). `gstatusgraph` gives approximate per-node sizes.

## Schema changes

Read `gsql ls` first to see whether a type is global (listed without a graph; shared by every graph that uses it) or local to one graph. That decides the command and the blast radius:

- Global types change through a `GLOBAL SCHEMA_CHANGE JOB` (after `USE GLOBAL`), local ones through a graph's `SCHEMA_CHANGE JOB`. A global change affects every graph that uses the type.
- An attribute's type can't be changed in place. Dropping and re-adding it loses the stored values; adding a new attribute, backfilling it and switching readers over keeps them.
- After a change, queries and loading jobs that refer to dropped types or attributes become invalid: queries are uninstalled and loading jobs disabled (older releases dropped them), and they must be recreated. GSQL lists what it will disable when the job runs.
- A `SCHEMA_CHANGE JOB` stays in the catalog after it runs (`DROP JOB` removes it), so its presence doesn't mean it is pending. Check whether its changes are already in the schema (`ls`) and look for the run in the GSQL log (`gadmin log gsql`).
- `DROP GRAPH` removes the graph, its jobs and queries and its local types; global types it used, and their data, stay.

The docs advise backing up first and avoiding high-traffic periods; check `ddl-and-loading/modifying-a-graph-schema` in the GSQL reference for the user's version (`SKILL.md`, "Official documentation").

## Loading results

`RUN LOADING JOB` reports per file how many lines loaded and how many were rejected. Afterwards, each run leaves a directory under the loader's log directory (`gadmin log` lists it, e.g. `RESTPP-LOADER` for file loading) named after the graph, job and node, with a `summary` file that counts rejected lines by reason (not enough tokens, invalid attributes, ...) and gives sample line numbers. A load that "didn't add rows" usually means rejected lines, or valid lines whose IDs already existed: loading upserts, so existing IDs update instead of adding vertices. Compare the vertex count before and after, and the file's IDs with existing ones.

## Users and roles

```
CREATE USER                          # prompts for name and password; the user should run it
GRANT ROLE <role> ON GRAPH <g> TO <user>
SHOW USER
CREATE SECRET <alias>                # secret for requesting REST API tokens
```

## Cluster helpers

- `gsql --leader` prints which node is the current GSQL leader. Its logs are the ones to read when a GSQL command fails.

## When gsql can't connect

`Connection refused` usually means the GSQL service is down: check `gadmin status gsql` and start it with `gadmin start gsql`. To use a GSQL client on another machine, the remote-client docs page (`gsql-shell/using-a-remote-gsql-client`) covers pointing it at the server (its `--ip` option or a `gsql_server_ip_config` file) and the client version to download.
