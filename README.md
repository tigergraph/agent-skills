# TigerGraph Agent Skills

Official TigerGraph plugins and skills for AI coding agents such as Claude Code, Codex, Cursor, GitHub Copilot and Grok. This repository hosts the `tigergraph` plugin marketplace.

| Plugin | What it gives the agent |
|---|---|
| `tigergraph` | The [TigerGraph MCP server](https://github.com/tigergraph/tigergraph-mcp): connect to a TigerGraph deployment, explore the schema, run GSQL and Cypher, load data, and search vectors. |
| `tigergraph-skills` | TigerGraph skills for operating a cluster: the command-line tools, and how a cluster is built and behaves. |

## Install

### Claude Code

```
claude plugin marketplace add tigergraph/agent-skills
claude plugin install tigergraph-skills@tigergraph
claude plugin install tigergraph@tigergraph
```

### Other agents

Codex, GitHub Copilot CLI, VS Code and Grok read the same marketplace file (Cursor support is best-effort). Add `tigergraph/agent-skills` as a marketplace in your agent, then install the plugins you need.

To install only the skills with the [skills CLI](https://skills.sh):

```
npx skills add tigergraph/agent-skills
```

## Skills

| Skill | Use it for |
|---|---|
| `tigergraph-cli` | `gadmin`, `gsql` and the cluster scripts on a TigerGraph node |
| `tigergraph-cluster` | Services, partitions and replicas, and how queries and loading flow through a cluster |

## Version compatibility

Skills track the TigerGraph releases that are currently supported. Each skill states its baseline in the `compatibility` field, and features that need a newer release are tagged inline, for example `[4.1+]`.

## Contributing

See [AGENTS.md](AGENTS.md) for the skill authoring rules.

## License

[Apache 2.0](LICENSE)
