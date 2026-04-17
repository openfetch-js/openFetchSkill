# openfetchskill — Claude Code plugin bundle

This folder is the **published plugin** for **`@hamdymohamedak/openfetch` v0.2.9** (pin this line when you cut a release so it matches [`openFetch/package.json`](../openFetch/package.json)). Bump **`.claude-plugin/plugin.json` → `version`** and the **“Library version”** line in `skills/openfetch/SKILL.md` together with that release. Claude Code copies the whole directory into its plugin cache, so the skill and references must live **inside** this tree (not outside).

The **library** (types, runtime, npm metadata) lives in **[`openFetch/`](../openFetch/)** — [`package.json`](../openFetch/package.json), [`src/`](../openFetch/src/), [`README.md`](../openFetch/README.md), [`CHANGELOG.md`](../openFetch/CHANGELOG.md).

The bundled **`skills/openfetch/SKILL.md`** follows the [Agent Skills](https://agentskills.io/specification) `SKILL.md` shape (YAML frontmatter + Markdown). That format is what registries and tools such as **[skills.sh](https://skills.sh)** expect for portable agent skills.

## Layout

```text
openfetchskill/
├── .claude-plugin/
│   └── plugin.json          # plugin manifest
├── skills/
│   └── openfetch/
│       ├── SKILL.md         # agent skill (YAML frontmatter + body)
│       └── references/      # optional deep docs
│           └── quick-reference.md
└── README.md
```

## Install (Claude Code)

```bash
claude plugin marketplace add openfetch-js/OpenFetch
claude plugin install openfetch@openfetch-js
```

Use `hamdymohamedak/OpenFetch` if that is your fork.

## Authoring

Edit **`skills/openfetch/SKILL.md`** (and `references/`). Keep frontmatter **`name`** aligned with the skill folder name.

## Structure reference (template)

To generate a **new** skill with the same folder pattern, copy from **[`openFetch/examples/claude-skill/`](../openFetch/examples/claude-skill/README.md)** (see also [`openFetch/examples/README.md`](../openFetch/examples/README.md)).
