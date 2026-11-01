# OKForest skills

[Agent Skills](https://agentskills.io) for [OKForest](https://www.okforest.ai):
writing and changing [Open Knowledge Format](https://github.com/GoogleCloudPlatform/open-knowledge-format)
(OKF) knowledge trees with your AI assistant.

| Skill | What it teaches |
| ----- | --------------- |
| [`okforest`](skills/okforest/SKILL.md) | Writing OKF concepts, links, and `index.md` and `log.md` files; OKForest's settings, type schemas, diagrams, and checks; and asking for or proposing changes through OKForest's MCP tools, which wait for a tree's editors. |

## Install

**Claude Code:**

```text
/plugin marketplace add okforest/skills
/plugin install okforest@okforest
```

**Other agents** (Cursor, Codex, and more), with the
[skills](https://skills.sh) CLI:

```sh
npx skills add okforest/skills
```

**Claude apps:** zip the [`skills/okforest`](skills/okforest) folder and
upload it as a skill in Claude's settings.

To change trees through OKForest, also connect its MCP server,
`https://mcp.okforest.ai/mcp`: see
[okforest.ai/docs/mcp](https://www.okforest.ai/docs/mcp/). The skill also
helps without it, when you edit a tree in its repository and check it with
`npx okforest validate .`.

## License

Open source under [Apache-2.0](LICENSE) · © Preneur LLC · OKForest™ is a
trademark of Preneur LLC ([trademark policy](https://www.okforest.ai/trademarks/)).
