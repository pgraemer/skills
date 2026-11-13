# skills

My skills for AI coding tools. This repository is a
[Spellbook](https://github.com/graemer-org/spellbook) registry: subscribe to
it, and the skills are copied to where Claude Code, GitHub Copilot, Cursor,
Codex and Gemini CLI read them.

## Skills

| Skill | What it does |
|---|---|
| `grill-me`, `grilling` | Interviews you about a plan or design until every branch of the decision tree is resolved. Imported from [mattpocock/skills](https://github.com/mattpocock/skills), see `registry.toml`. |
| [`friction-driven-development`](misc/friction-driven-development/SKILL.md) | Lets someone without inside knowledge try a real task through one surface of a product, and fixes what gets in their way: each friction point is verified, fixed with the smallest change and confirmed by another attempt. |

## Subscribe

```sh
spellbook init
spellbook registry add pgraemer https://github.com/pgraemer/skills.git
```

Only one of them:

```sh
spellbook registry add pgraemer https://github.com/pgraemer/skills.git --skill grill-me
```

In a project instead of the home directory, so the skills are committed
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
groups on purpose: without them, every skill here is offered to every
subscriber. To update an imported skill, move its `ref` to a newer commit.
