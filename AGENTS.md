# Authoring guide

Rules for adding or changing skills in this repository. Everything here is public.

## Layout

```
.claude-plugin/marketplace.json        marketplace "tigergraph"
plugins/tigergraph-skills/
  .claude-plugin/plugin.json
  skills/<skill-name>/
    SKILL.md
    references/*.md                    loaded on demand from SKILL.md
```

The `tigergraph` plugin (MCP) lives in [tigergraph-mcp](https://github.com/tigergraph/tigergraph-mcp) and is referenced from the marketplace, not copied here. If you change its display fields there, update its entry in `marketplace.json` to match.

## Naming

- Skill names start with `tigergraph-`, in lowercase kebab-case, and match the folder name.
- Keep `SKILL.md` short: when to use the skill, the core facts, and an index of `references/`.

## Frontmatter

```yaml
---
name: tigergraph-example
description: What the skill does and when to load it. Does NOT cover X — use tigergraph-y.
compatibility: TigerGraph 3.10 or later, self-managed.
license: Apache-2.0
metadata:
  author: TigerGraph
  version: "0.1.0"
---
```

- `description` names the trigger situations and, with "Does NOT cover … — use …", hands off to sibling skills that overlap.
- `compatibility` states the oldest TigerGraph release the skill is written for and the deployment types it applies to.
- `metadata.version` is the skill's own version. Bump it, and the plugin `version` in `plugin.json`, when the content changes.

## Version compatibility

- Write for the oldest supported release in `compatibility`. Tag anything newer inline: `[4.1+]`, or `[3.10 only]` for behaviour that was removed.
- Tell the agent to detect the version before relying on a tagged feature (`gadmin version` on a node, or ask the user). When the version is unknown, use only untagged features.
- One `main` branch. Do not keep per-release branches or copies of a skill.

## Public content only

These skills ship to customers and to public registries. Before you open a PR, check that the change contains none of:

- source repository names, source paths, or internal class, constant or file names;
- internal URLs (issue tracker, wiki, internal docs or repositories) or customer names;
- commands that print secrets or full configuration dumps;
- repair or reset procedures for internal services. Point to TigerGraph Support instead.

When in doubt, keep to what [docs.tigergraph.com](https://docs.tigergraph.com) already says.

## Plugin rules

- No symlinks inside `plugins/`.
- Run `claude plugin validate ./` and `claude plugin validate ./plugins/tigergraph-skills --strict` before you push. CI runs the same checks.
