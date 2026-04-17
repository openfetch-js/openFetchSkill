# openFetch skill (Claude Code plugin)

An **[Agent Skills](https://agentskills.io/specification)** bundle so coding agents reliably use **[`@hamdymohamedak/openfetch`](https://www.npmjs.com/package/@hamdymohamedak/openfetch)** — the fetch-based HTTP client with interceptors, middleware, retries, caching, and optional fluent APIs — instead of guessing axios-style APIs.

**Targets library version:** `v0.2.9` (see [`skills/openfetch/SKILL.md`](./skills/openfetch/SKILL.md)).

## Why install this

- **Accurate defaults** — e.g. retry only on safe methods unless configured otherwise; `fetch`-only transport.
- **Up-to-date surface** — plugins (`retry`, `timeout`, `debug`, …), sugar/fluent client, `rawResponse`, cache keying, idempotency helpers.
- **Portable format** — same `SKILL.md` layout used by registries and tools such as **[skills.sh](https://skills.sh)**.

## Install (Claude Code)

```bash
claude plugin marketplace add openfetch-js/OpenFetch
claude plugin install openfetch@openfetch-js
```

If you use a fork, substitute the GitHub owner in both commands (for example `your-org/OpenFetch`).

## What’s in the package

| Path | Purpose |
|------|--------|
| [`.claude-plugin/plugin.json`](./.claude-plugin/plugin.json) | Plugin manifest for Claude Code |
| [`skills/openfetch/SKILL.md`](./skills/openfetch/SKILL.md) | Primary skill (YAML frontmatter + instructions) |
| [`skills/openfetch/references/quick-reference.md`](./skills/openfetch/references/quick-reference.md) | Compact API and config reference |

```text
openfetchskill/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── openfetch/
│       ├── SKILL.md
│       └── references/
│           └── quick-reference.md
└── README.md
```

## Learn the library

- **Documentation:** [openfetch-js.github.io/openfetch-docs](https://openfetch-js.github.io/openfetch-docs/)
- **Source & issues:** [github.com/openfetch-js/OpenFetch](https://github.com/openfetch-js/OpenFetch)
- **Skill spec:** [agentskills.io/specification](https://agentskills.io/specification)
