# skills

My skills for AI coding tools. This repository is a
[Spellbook](https://github.com/graemer-org/spellbook) registry: subscribe to
it, and the skills are copied to where Claude Code, GitHub Copilot, Cursor,
Codex and Gemini CLI read them.

## Skills

| Skill | What it does |
|---|---|
| [`grill-me`](misc/grill-me/SKILL.md) | Interviews you about a plan or design, one question at a time, until every branch of the decision tree is resolved. |
| [`experiment-user`](misc/experiment-user/SKILL.md) | Runs a usability experiment: a simulated user tries a task through one channel of a product, friction is fixed during the run, and a friction report comes out. |

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

There is no `registry.toml` on purpose: without one, every skill here is
offered to every subscriber. Groups are only worth it once different
people should get different sets.
