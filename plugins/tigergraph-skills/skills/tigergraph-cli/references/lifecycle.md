# Install, upgrade and change the cluster

These are documented procedures whose details change between releases. For each one, fetch the docs page for the user's version (`https://www.tigergraph.com/docs/tigergraph-server/<version>/<page>`, version like `4.3`; an unpublished version returns 404, so use the closest earlier one and say so) and follow it. The notes below say where to look on the cluster and what is easy to miss. If you can't fetch the page, say that your steps come from memory and should be checked against it.

## Install (`installation/hw-and-sw-requirements`, `installation/bare-metal-install`, `installation/post-install-check`)

- The requirements page lists what every machine needs before the installer runs (OS user and SSH access, open ports, time sync, disk, required utilities). Give the user that checklist rather than a partial one.
- The non-interactive install fills in `install_conf.json`, which ships in the installer package with every key; the install page explains each one and the flag for non-interactive mode.
- Sizing is the decision the installer can't make for the user. Partitions = machines / replication factor, rounded down, and machines left over are unused; HA needs a replication factor of at least 2 on at least 3 machines. So 3 machines for HA means replication factor 3 (1 partition x 3 replicas): factor 2 would leave the third machine idle. More in tigergraph-cluster, "Partitions and replicas".
- After installing, run the checks on the post-install page.

## Upgrade (`installation/upgrade`)

- Upgrades run the new version's installer, not `gadmin`; the page lists the supported upgrade paths and flags.
- There is no downgrade: take a full backup first, because it is the way back. The services stop during the upgrade, so it needs a maintenance window with no loading or queries.
- Rolling (online) upgrades exist only for some clusters and version jumps; check the page before promising no downtime.

## Change a node's IP or hostname (`installation/change-ip-or-hostname`)

The documented procedure re-initialises ETCD and the cluster (`gadmin init ...`), so it needs a maintenance window and a backup first. Give the page's steps for the user's version in order, then verify with `gadmin status -v` and a `gssh`/`grun` to every node, since the nodes must reach each other on the new addresses. One trap: the last step uses `gadmin init cluster --init-etcd`, a hidden flag that `gadmin init cluster -h` doesn't list; it is correct as documented.

## A node is lost (`cluster-and-ha-management/remove-failed-node`, `cluster-and-ha-management/replace-a-node`)

- `gadmin cluster remove -h` explains removing a failed node; it needs a replication factor above 1. Removal doesn't bring the lost replicas back; adding a replacement machine with `gadmin cluster expand` does. Expansion moves data, needs a backup first and no loading or queries, and takes the cluster offline for a few minutes (tigergraph-cluster, "Changing the layout").
- If the node's disk survived and the cluster was installed with hostnames, the replace-a-node page moves the disk to a new machine instead. Check that page's conditions before choosing it.

## Ports (`reference/ports`)

- Read this cluster's ports from the live config (`gadmin config list entry | grep -i -A2 port`, then `get`); they are configurable, so don't assume defaults when writing firewall rules.
- The ports page says which ports clients need and which are internal. Clients need far fewer than the services listen on: most listening ports (RESTPP, GSQL, Kafka, ETCD, ...) are only for traffic between nodes, and from 4.1 clients reach REST, GSQL and the GUI through the NGINX port.

## HTTPS on the NGINX port (`security/encrypting-connections`)

The `Nginx.SSL.*` entries (`gadmin config list entry | grep -A2 Nginx.SSL`) hold the switch, certificate and key. After applying, restart the services the page names. Clients then switch to `https://` on the same port and must trust the certificate. Client-certificate checks (mTLS) exist only in newer releases; check the page for the user's version.

## Query memory limits (`system-management/memory-management`)

- Search the live config for the per-query limit and read its description: `gadmin config list entry | grep -i -A2 memlimit`. Several entries have "memory" in the name but limit other things, so check what each one caps before recommending it.
- The page also covers a per-request override (an HTTP header) and machine-wide free-memory thresholds. The thresholds are options inside `GPE.BasicConfig.Env`, not entries of their own, so append them to that value (`references/gadmin.md`, "config").
