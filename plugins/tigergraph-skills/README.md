# tigergraph-skills

Skills that teach an AI agent how TigerGraph works and how to operate it from the command line. The skills are plain knowledge and need no connection to a database, so they work in any agent that loads skills. Pair them with the `tigergraph` MCP plugin when the agent should also query or change a live graph.

| Skill | Use it for |
|---|---|
| `tigergraph-cli` | `gadmin`, `gsql` and the cluster scripts on a TigerGraph node |
| `tigergraph-architecture` | Services, partitions and replicas, and how queries and loading flow through a cluster |

## Install

```
claude plugin marketplace add tigergraph/agent-skills
claude plugin install tigergraph-skills@tigergraph
```

## License

[Apache 2.0](../../LICENSE)
