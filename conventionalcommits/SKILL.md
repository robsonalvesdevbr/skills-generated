---
name: conventionalcommits
description: Write and create git commits that follow the Conventional Commits 1.0.0 specification. Use this skill EVERY time the user asks to commit, make a commit, "commit this", "commit the changes", write a commit message, amend a commit message, or any request that ends in running `git commit`, even when Conventional Commits is not mentioned. Before committing it asks which language to write the message in and asks the user to confirm the commit author (default taken from Claude Code settings) or provide a new one.
---

# Conventional Commits

Every commit made under this skill MUST follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/). The full specification is in `references/index.md`; read it when a rule below is unclear or an edge case comes up.

## Workflow

Follow these steps in order. Do not run `git commit` before steps 1 and 2 are answered.

### 1. Ask for the language

Ask which language the commit message should be written in, using `AskUserQuestion`:

- Header: `Language`
- Options: `English`, `Portuguese (Brazil)`, `Spanish` (the user can type another one via "Other")

The chosen language applies to the **description** and the **body** only. These parts always stay in English, because tools parse them:

- the `type` (`feat`, `fix`, `docs`, ...)
- footer tokens (`BREAKING CHANGE`, `Refs`, `Reviewed-by`, `Author`, ...)

The `scope` is a codebase noun (module, package, directory name). Keep it as it appears in the code.

### 2. Confirm the author

Read the default author from Claude Code settings, checking these files in order and using the first match:

1. `.claude/settings.local.json` (project)
2. `.claude/settings.json` (project)
3. `~/.claude/settings.json` (user)

The value lives at `attribution.commit`, for example `"Author: Robson Alves <robsonalves.us@gmail.com>"`. Strip the `Author: ` prefix to get `Name <email>`. If none of the files has it, fall back to `git config user.name` and `git config user.email`.

Ask with `AskUserQuestion`:

- Header: `Author`
- Question: `Commit as <Name <email>>?`
- Options: `Use <Name <email>>` (Recommended), `Use a different author`

If the user picks a different author (or types one via "Other"), ask for it in the format `Name <email>` and check it matches that format before going on.

### 3. Inspect the changes

Run `git status` and `git diff --staged` (and `git diff` if nothing is staged) to understand what changed. If nothing is staged, ask the user what to stage rather than running `git add -A` on your own.

If the changes cover more than one unrelated purpose, suggest splitting them into separate commits (spec FAQ: "Go back and make multiple commits whenever possible").

### 4. Write the message

Structure:

```
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

Pick the type:

| Type | Use for | SemVer |
|------|---------|--------|
| `feat` | A new feature | MINOR |
| `fix` | A bug fix | PATCH |
| `docs` | Documentation only | none |
| `style` | Formatting, whitespace; no code behavior change | none |
| `refactor` | Code change that neither fixes a bug nor adds a feature | none |
| `perf` | Performance improvement | none |
| `test` | Adding or fixing tests | none |
| `build` | Build system or external dependencies | none |
| `ci` | CI configuration and scripts | none |
| `chore` | Maintenance that doesn't touch src or tests | none |
| `revert` | Reverts earlier commits (add `Refs: <sha>, <sha>` footer) | none |

Any type with a breaking change maps to MAJOR.

Rules from the spec:

- `feat` MUST be used for new features and `fix` MUST be used for bug fixes.
- The prefix is the type, an optional `(scope)`, an optional `!`, then a REQUIRED `: ` (colon and space).
- The scope is a noun for a section of the codebase in parentheses, e.g. `fix(parser):`.
- The description comes right after `: `. It is a short summary of the change.
- The body is optional, free-form, can have several paragraphs, and MUST start one blank line after the description.
- Footers go one blank line after the body. Each footer is a token, then `: ` or ` #`, then a value (git trailer format). Tokens use `-` instead of spaces (`Reviewed-by`, `Acked-by`). The only exception is `BREAKING CHANGE`.
- A breaking change MUST be marked with `!` right before the `:`, or with a `BREAKING CHANGE: <description>` footer, or both. `BREAKING CHANGE` MUST be uppercase. `BREAKING-CHANGE` is a synonym.
- If `!` is used without the footer, the description must describe the breaking change.

House style (on top of the spec):

- Use lowercase types.
- Write the description in the imperative mood in the chosen language ("add", "adiciona", "agrega"), with no trailing period. Keep the header line at 72 characters or less.
- Wrap the body at 72 characters. Explain what changed and why, not how.

### 5. Add the author and commit

Add the confirmed author as the last footer, in the same form as the settings value:

```
Author: Name <email>
```

If the user picked a different author, this line replaces the default one. Keep any other attribution trailers the session requires (for example a `Claude-Session:` link) below it.

Also set git's author field to the confirmed author with `--author="Name <email>"`.

Show the final message to the user, then commit with a heredoc so the formatting is kept:

```bash
git commit --author="Name <email>" -F - <<'EOF'
feat(auth): add OAuth2 login with Google

Users can now sign in with their Google account. The existing
email/password flow is unchanged.

Refs: #42
Author: Name <email>
EOF
```

Run `git log -1` afterwards to confirm the commit landed with the right message and author.

## Examples

Description with a breaking change footer:

```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

Scope and `!`:

```
feat(api)!: send an email to the customer when a product is shipped
```

No body:

```
docs: correct spelling of CHANGELOG
```

Portuguese (Brazil) description and body; type and footer tokens stay in English:

```
fix(parser): corrige leitura de arrays com espaços múltiplos

O parser ignorava elementos quando havia mais de um espaço entre
eles. Agora os espaços são normalizados antes da divisão.

Refs: #123
Author: Robson Alves <robsonalves.us@gmail.com>
```

Revert:

```
revert: let us never again speak of the noodle incident

Refs: 676104e, a215868
```
