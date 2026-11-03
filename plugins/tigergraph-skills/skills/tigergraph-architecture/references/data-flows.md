# Data flows

<!-- OUTLINE -->

- Schema change and query install: gsql → NGINX → GSQL → catalog → engine (install only); synchronous across replicas, a down node blocks it.
- Loading: three entry paths (local files, data sources such as S3 / Kafka / warehouses, Spark connector) converge through Kafka into GPE → GSE; loading status comes from the loading-progress service.
- Real-time upserts: client → NGINX → RESTPP → GPE → GSE.
- Queries: installed vs interpreted; distributed across partitions; pinning reads to a replica for read-after-write (verify the header is documented publicly).
- Operation → services table.
