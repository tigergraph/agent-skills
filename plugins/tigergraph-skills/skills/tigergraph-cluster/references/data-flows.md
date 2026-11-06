# Data flows

How each kind of work moves through the services. Use it to decide which service's logs to read.

## Schema changes and query install

```
gsql / GraphStudio ──▶ NGINX ──▶ GSQL ──▶ catalog (DICT)
                                   └────▶ GPE   (query install only)
```

Applied to every replica together; see `clustering.md` for what a down node blocks.

## Loading

```
local files      ──▶ file loader ──────────────┐
DATA_SOURCE      ──▶ KAFKACONN ──▶ loader ─────┼──▶ GSE (IDs) + GPE (graph)
Spark connector  ──▶ RESTPP ───────────────────┘
loading statistics ──▶ KAFKASTRM-LL ──▶ loading status
```

`DATA_SOURCE` sources include S3 and other object stores, external Kafka, and data warehouses. Local files on the server are the fastest path.

## Real-time updates

```
client ──▶ NGINX ──▶ RESTPP ──▶ GSE (IDs) + GPE (graph)
```

## Queries

```
client ──▶ NGINX ──▶ RESTPP ──▶ GSE translates IDs ──▶ GPE runs the query (all partitions in parallel)
                        ▲                                   │
                        └──────── result (IDs translated back by GSE) ◀┘
```

- **Installed queries** are compiled ahead of time; the production path.
- **Interpreted queries** run without installing; quicker to iterate on, slower to execute.

To follow one request, read the NGINX access log, then RESTPP, GPE and GSE logs for its request ID (`gcollect -r <request_id>` gathers them in one go).

## Which service does what

| Operation | Services involved |
|---|---|
| Schema parsing and validation | GSQL |
| Catalog storage | DICT |
| Query install | GSQL, GPE |
| File loading | file loader, GSE, GPE |
| Data-source loading | KAFKACONN, loader, GSE, GPE |
| Loading status | KAFKASTRM-LL |
| Query execution | RESTPP, GSE, GPE |
| Updates and upserts | RESTPP, GSE, GPE |
| TLS | NGINX |
| Web UI | NGINX, GUI |
