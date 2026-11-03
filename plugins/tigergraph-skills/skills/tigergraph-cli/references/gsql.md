# gsql

<!-- OUTLINE -->

- Needs the GSQL service up: `gadmin status gsql`.
- Modes: REPL, one-line statement, script file, `-g <graph>`; common flags (`-u`, `-p`, `--version`); `--reset` is destructive.
- REPL basics: `ls`, `USE GRAPH`, `USE GLOBAL`, `SHOW VERTEX|EDGE|GRAPH`, `help basic`, `help query`.
- Minimal end-to-end example: create vertex and edge types, a graph, a loading job, run it, then create, install and run a query; interpreted vs installed.
- Users and roles: `CREATE USER`, `GRANT ROLE`, `CREATE SECRET`.
- Connection refused: check and start GSQL; remote client via `gsql_server_ip_config`.
- Pointer: writing GSQL queries is out of scope for this skill.
