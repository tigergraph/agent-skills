# Services

<!-- OUTLINE -->

- One table of the services `gadmin status` shows: name | role | runs where (partitioned and replicated / one per partition / one per node / leader-elected).
- Loader sub-services as users see them in logs and status (file loader, Kafka loader).
- Reading `gadmin status -v` instance IDs (`_partition#replica`), with the caveat for non-partitioned services.
- ETCD vs ZooKeeper in one line each.
- Startup dependency graph between services (useful for "why didn't X start").
