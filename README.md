# Skills

Personal agent skills, project-agnostic: each project's `AGENTS.md` supplies its docs, issue conventions and verify commands.

## Workflow

`/planning` -> `/implement-plan <issue>`

- `planning`: explores the repo, interviews you with `AskUserQuestion`, publishes one GitHub issue with a task checklist. No code.
- `implement-plan`: builds the issue in one session, test-first, verifies, reviews the diff once.

## Contents

| Path | What it is |
|---|---|
| `skills/ask-sonner/` | Guide to Sonner, the React toast library — install and wire up the Toaster, pick the right toast() call, promi |
| `skills/break-ui/` | Try to break a piece of UI by feeding it worst-case data — long names, unbreakable emails, one-letter names, m |
| `skills/code-review/` | Review the changes since a fixed point (commit, branch, tag, or merge-base) along two axes: Standards (does th |
| `skills/domain-modeling/` | Build and sharpen a project's domain model. Use when discussing codebase terminology, writing or editing a GLO |
| `skills/frontend-design/` | Guidance for distinctive, intentional visual design when building new UI or reshaping an existing one. Helps w |
| `skills/implement-plan/` | Implement a GitHub plan issue (from planning) in one session, test-first, then verify and review once. |
| `skills/mobile-native/` | Make a web app feel native on a phone — the small CSS and meta-tag fixes that separate "a website in a browser |
| `skills/pick-ui-library/` | Pick the right library for a given frontend task from a curated, opinionated list — numbers, OTP inputs, chart |
| `skills/planning/` | Interview the user about a feature or change, then publish one GitHub issue with a task checklist. Planning on |
| `skills/prototype/` | Build multiple genuinely different versions of a UI piece you describe, rendered behind a visual picker so you |
| `skills/ui-design/` | Use when the users asks for a ui/frontend change or generation. |
| `skills/web-design-guidelines/` | Review UI code for Web Interface Guidelines compliance. Use when asked to "review my UI", "check accessibility |

## Install

Copy into `~/.agents/skills/` and symlink into `~/.claude/skills/`:

```sh
cp -r skills/<name> ~/.agents/skills/ && ln -sfn ../../.agents/skills/<name> ~/.claude/skills/<name>
```

Domain-modeling and code-review are copied from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT) at abeeaf9.
