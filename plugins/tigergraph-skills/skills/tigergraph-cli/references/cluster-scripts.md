# Cluster scripts

<!-- OUTLINE -->

- Where they live (`<AppRoot>/cmd/`, on `PATH` for the TigerGraph OS user).
- Local info: `ghostname`, `gmyip`, `gstatusgraph`.
- Fan-out: `grun` (ordered) vs `grun_p` (parallel); target syntax shared with `gadmin run`.
- File transfer: `gscp` (push), `gfetch` (pull).
- `gssh`: no args prints cluster topology; with a host, ssh using the cluster's keys.
- `gcollect`: collect then grep with `-D`; `-r <request_id>` only on collect; component name aliases.
- Support bundle: the public command for collecting logs for TigerGraph Support (verify which one the docs recommend).
- Discover the rest: `ls $(gadmin config get System.AppRoot)/cmd/`.
