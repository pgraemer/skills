# skills

My skills and instructions for AI coding tools. This repository is a
[Spellbook](https://github.com/graemer-org/spellbook) registry: subscribe to
it, and they are copied to where Claude Code, GitHub Copilot, Cursor, Codex
and Gemini CLI read them.

## Skills

| Skill | What it does |
|---|---|
| `grill-me`, `grilling` | Interviews you about a plan or design until every branch of the decision tree is resolved. Imported from [mattpocock/skills](https://github.com/mattpocock/skills), see `registry.toml`. |
| `security-audit` | Security guidance and vulnerability review for codebases, APIs, services, CLI tools and libraries, up to a full audit with a report. Imported from [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill), see `registry.toml`. |
| `agent-browser` | Lets the agent drive a browser through the `agent-browser` CLI: open pages, fill forms, click, take screenshots, test web apps. Needs the CLI (`npm i -g agent-browser && agent-browser install`). Imported from [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser), see `registry.toml`. |
| [`friction-driven-development`](misc/friction-driven-development/SKILL.md) | Lets someone without inside knowledge try a real task through one surface of a product, and fixes what gets in their way: each friction point is verified, fixed with the smallest change and confirmed by another attempt. |

## Instructions

Standing guidance that always applies. Each is a Markdown file in
`instructions/`.

| Instruction | What it covers |
|---|---|
| [`coding`](instructions/coding.md) | The agent owns the code it writes, plus comments, TypeScript, testing and verification. |
| [`writing`](instructions/writing.md) | Prose style, commits, pull requests and tickets. |

## Subscribe

```sh
spellbook init
spellbook registry add pgraemer https://github.com/pgraemer/skills.git
```

Only some of it:

```sh
spellbook registry add pgraemer https://github.com/pgraemer/skills.git --skill grill-me
spellbook registry add pgraemer https://github.com/pgraemer/skills.git --instruction coding --instruction writing
```

In a project instead of the home directory, so the files are committed
with the repository, add `--project` to both commands.

`spellbook sync` fetches newer versions later.

## Adding a skill

A skill is a directory with a `SKILL.md` whose frontmatter has a `name` and
a `description`. Spellbook finds it anywhere in the repository, so the
folder it lives in (`misc/` today) is only for order. The `name` must be
unique in the repository.

To propose a skill from a project where it was written:

```sh
spellbook push .claude/skills/my-skill --to pgraemer
```

`registry.toml` holds the imports from other repositories. It defines no
groups on purpose: without them, everything here is offered to every
subscriber.

Every import is pinned to a commit. Once a day the
[Imports workflow](.github/workflows/imports.yml) runs
`spellbook registry imports --update` and opens a pull request when an
imported repository changed one of the skills taken from it. To look by
hand:

```sh
spellbook registry imports            # which pins are behind
spellbook registry imports --update   # move them in registry.toml
```
