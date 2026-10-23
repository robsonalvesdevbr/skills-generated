# skills-generated

A collection of [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills) for Claude Code. Each skill is a folder with a `SKILL.md` (instructions and metadata) and, when needed, supporting files such as references and subagents.

## Catalog

| Skill | What it does | When it triggers |
| --- | --- | --- |
| [`conventionalcommits`](conventionalcommits/SKILL.md) | Creates commits that follow the [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification. Asks which language to write the message in and confirms the author before committing. | Whenever you ask to commit, write a commit message, or amend one. |
| [`model-selection`](model-selection/SKILL.md) | Picks the right Claude model (Haiku, Sonnet or Opus) and effort level for a task, and routes the work to the matching subagent. | At the start of non-trivial tasks, before delegating to a subagent, or when you ask which model to use. |

### conventionalcommits

- Workflow: ask the language, confirm the author, then build and create the commit.
- The chosen language applies to the description and body. The `type` and footer tokens (`BREAKING CHANGE`, `Refs`, ...) always stay in English.
- The full specification lives in [`conventionalcommits/references/index.md`](conventionalcommits/references/index.md).

### model-selection

Routing rules:

| Model | Type of work |
| --- | --- |
| Haiku | Quick lookups and short summaries, read-only |
| Sonnet | Code, configuration, mechanical tasks, builds and tests |
| Opus | Documentation, planning, architecture, systems analysis and security |

Subagents shipped in [`model-selection/agents/`](model-selection/agents):

| Agent | Model | Use |
| --- | --- | --- |
| [`quick-explorer`](model-selection/agents/quick-explorer.md) | Haiku | Find files, symbols and config values; short summaries (read-only) |
| [`implementer`](model-selection/agents/implementer.md) | Sonnet | Implement from a clear spec, refactor, run builds and tests |
| [`architect`](model-selection/agents/architect.md) | Opus | READMEs, ADRs, architecture, root-cause analysis, threat modeling |

## Installation

### Using `npx skills`

Install a single skill straight from GitHub:

```sh
npx skills add https://github.com/robsonalvesdevbr/skills-generated --skill conventionalcommits
npx skills add https://github.com/robsonalvesdevbr/skills-generated --skill model-selection
```

### Manual

Clone the repository, then copy (or symlink) the skills you want.

```sh
git clone git@github.com:robsonalvesdevbr/skills-generated.git
cd skills-generated

# Personal skills, available in every project
mkdir -p ~/.claude/skills
ln -s "$PWD/conventionalcommits" ~/.claude/skills/conventionalcommits
ln -s "$PWD/model-selection" ~/.claude/skills/model-selection

# Subagents used by the model-selection skill
mkdir -p ~/.claude/agents
ln -s "$PWD"/model-selection/agents/*.md ~/.claude/agents/
```

To use them in a single project only, replace `~/.claude/` with `.claude/` at that project's root.

## Layout

```text
.
├── conventionalcommits/
│   ├── SKILL.md
│   └── references/index.md
└── model-selection/
    ├── SKILL.md
    └── agents/
        ├── architect.md
        ├── implementer.md
        └── quick-explorer.md
```

## Adding a skill

1. Create a folder named after the skill, with a `SKILL.md` containing `name` and `description` in the frontmatter.
2. Put supporting material in subfolders (`references/`, `agents/`, ...).
3. Add the skill to the catalog above and an entry to [`CHANGELOG.md`](CHANGELOG.md).
4. Commit following Conventional Commits.

## Changelog

Changes are tracked in [`CHANGELOG.md`](CHANGELOG.md), following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## License

Released under the [MIT License](LICENSE).
