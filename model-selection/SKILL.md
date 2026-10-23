---
name: model-selection
description: Decide which Claude model (Haiku, Sonnet or Opus) and which effort level should handle a task, and route the work to the matching subagent. Use this skill at the start of every non-trivial task, before delegating work to a subagent, and whenever the user asks which model to use, "should this be Sonnet or Opus", or wants to save cost or get deeper reasoning. Haiku handles quick lookups and simple summaries; Sonnet handles coding, configuration and hands-on mechanical work; Opus handles documentation, reasoning, definitions, systems analysis, software architecture and security identification.
---

# Model Selection

Pick the model that fits the kind of work, not the size of the request.

- **Haiku** is the fastest and cheapest. Use it when the answer is easy to find and easy to check.
- **Sonnet** is fast and affordable. Use it for execution.
- **Opus** is slower and more expensive. Use it when the quality of the thinking is the deliverable.

## Routing rules

### Use Haiku for quick, low-stakes work

- Finding files, symbols, usages or config values in the codebase
- Short summaries of a file, a log or a command output
- Simple classification or listing (e.g. "which files import X?", "list the TODOs")
- Any read-only lookup where a wrong answer is cheap and easy to spot

Do not use Haiku for anything that edits files or makes a decision.

### Use Sonnet for execution work

- Writing, editing or refactoring code from a clear spec
- Configuration: settings files, CI/CD pipelines, Dockerfiles, IaC, environment setup
- Mechanical tasks: renames, migrations, formatting, dependency bumps, boilerplate, scaffolding
- Running commands, scripts, builds and tests, and fixing straightforward failures

### Use Opus for thinking work

- Documentation: READMEs, ADRs, guides, specs, technical writing
- Reasoning and planning: breaking down ambiguous problems, weighing options, deciding an approach
- Definitions: requirements, domain models, API contracts, acceptance criteria
- Systems analysis: understanding how an existing system behaves, root-cause analysis of hard bugs
- Software architecture: system design, service boundaries, trade-offs, scalability
- Security identification: threat modeling, vulnerability review, auth and data-exposure analysis

### Tie-breakers

- **Mixed tasks:** split them. Opus defines or designs, Sonnet implements. Example: "design and build an auth module" becomes an Opus design step followed by a Sonnet implementation step. In the main session, `opusplan` does this split automatically (see below).
- **Unclear scope:** if a coding task turns out to need design decisions, stop and route the decision to Opus before continuing.
- **Risk:** when a mistake would be costly or hard to undo (production data, security, public APIs), prefer Opus even for work that looks mechanical.
- **Explicit user choice:** if the user names a model or effort level, use it. These rules are defaults, not overrides.

## Effort level

The model sets how capable the reasoning is; the effort level sets how much reasoning is spent. Choose both.

| Work                                                  | Model  | Effort   |
| ----------------------------------------------------- | ------ | -------- |
| Lookups, short summaries                              | Haiku  | default  |
| Mechanical edits, renames, formatting, dependency bumps | Sonnet | `low`    |
| Feature implementation, configuration, fixing tests   | Sonnet | `medium` |
| Documentation, definitions, planning                  | Opus   | `high`   |
| Architecture, hard root-cause analysis                | Opus   | `high`   |
| Security review, threat modeling                      | Opus   | `xhigh`  |

Raise the effort one step when the task is ambiguous or high-risk; lower it one step when the task is small and well-defined. Available levels depend on the model.

## How to apply the choice

### 1. Classify

Classify the task with the rules above and state the choice in one short line, for example: `Model: Sonnet, effort medium (configuration work)`.

### 2. Delegate to the matching subagent

This skill ships three subagents in `agents/`, each with its model and effort fixed in the frontmatter:

| Subagent         | Model  | Effort   | Use for                                    |
| ---------------- | ------ | -------- | ------------------------------------------ |
| `quick-explorer` | haiku  | default  | Read-only lookups and short summaries      |
| `implementer`    | sonnet | `medium` | Coding, configuration, mechanical work     |
| `architect`      | opus   | `high`   | Docs, definitions, analysis, architecture, security |

Delegate with the `Agent` tool using `subagent_type` set to the subagent name. If the classification calls for a different effort than the subagent's default (e.g. `xhigh` for a security review, `low` for a rename), pass `effort` on the `Agent` call only when the user asked for that level; otherwise use the subagent as is.

If the subagents are not installed, fall back to `subagent_type: "general-purpose"` with the `model` parameter set to `"haiku"`, `"sonnet"` or `"opus"`, and tell the user how to install them (see **Installation**).

### 3. Main session

You cannot switch your own model. If the current model does not match the task, tell the user and suggest the command. Do not block on it; continue if the user does not want to switch.

- For a task that is mostly execution: `/model sonnet`
- For a task that is mostly thinking: `/model opus`
- For a task that needs a plan first and then implementation: `/model opusplan`. It uses Opus in plan mode and switches to Sonnet automatically when execution starts. Switching models keeps the conversation, so Sonnet sees the plan Opus wrote.
- To change the reasoning budget: `/effort <level>`

Prefer suggesting `opusplan` over manual switching whenever the work has a clear plan-then-build shape.

### 4. Long tasks

Reclassify at each phase change (design → implementation → review) and delegate each phase to the matching subagent.

## Quick reference

| Task                                          | Route                      |
| --------------------------------------------- | -------------------------- |
| Find where a function is used                 | `quick-explorer` (Haiku)   |
| Summarize a log or a file                     | `quick-explorer` (Haiku)   |
| Implement a feature from a clear spec         | `implementer` (Sonnet)     |
| Edit `settings.json`, CI, Dockerfile          | `implementer` (Sonnet)     |
| Rename, migrate, bump dependencies            | `implementer` (Sonnet)     |
| Write a README or ADR                         | `architect` (Opus)         |
| Design a system or service boundaries         | `architect` (Opus)         |
| Define requirements or a domain model         | `architect` (Opus)         |
| Investigate a hard bug's root cause           | `architect` (Opus)         |
| Security review or threat model               | `architect` (Opus, `xhigh`) |
| Plan and build a feature in the main session  | suggest `/model opusplan`  |

## Installation

The subagents only load from a Claude Code agents directory. Copy them once:

```bash
# for every project (user scope)
cp agents/*.md ~/.claude/agents/

# or for one project only
cp agents/*.md <project>/.claude/agents/
```

Restart the Claude Code session after copying; subagents are loaded at startup.
