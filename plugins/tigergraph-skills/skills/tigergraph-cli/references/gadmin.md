# gadmin

<!-- OUTLINE -->

- Source of truth: `gadmin --help`; the table below is a snapshot.
- Subcommand table: status, start/stop/restart, config, log, metric, run, cluster, backup, crr, connector, license, upgrade, support, version. Drop `init` and `reset` from the table; list them only under destructive.
- `status`: `-v`, filter by service, `-l` local only.
- `config`: staging model (`set` → `diff` → `apply` / `discard`); discovering entries; reading a saved config file.
- `start`/`stop`/`restart`: dependency-aware order; restart what `config apply` reports.
- `run`: fan-out through the cluster's own runner; compare with `grun`.
- `log`, `metric`, `support`.
- Destructive subcommands: read `--help` first and confirm with the user.
- Version tags to verify per subcommand (e.g. `crr`, `support healthcheck`).
