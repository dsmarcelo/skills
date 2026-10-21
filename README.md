# Plan, orchestrate, review (for T3 Code)

A multi-agent coding workflow for [T3 Code](https://github.com/pingdotgg/t3code), using models from OpenAI and Anthropic. The planner interviews you, publishes the plan as GitHub issues, and launches the orchestrator in a new thread. Choose the default GPT profile, the Claude profile, or available custom orchestrator and builder models. The planner is always gpt-6.1-sol and the reviewer is always gpt-6-luna.

```
you ── $plan-work ──> PLANNER thread (gpt-6.1-sol, medium; fixed)
                        │  delegate_task ──> SCOUT (gpt-6-luna, medium; fixed): code search and notes
                        │  grilling rounds with you until the design tree is resolved
                        │  task breakdown, approved by you
                        │  GitHub: parent issue + task sub-issues + blocking edges
                        │  t3_thread_launch (after your yes)
                        v
                      ORCHESTRATOR thread (GPT default: gpt-6.1-sol, medium; Claude: Opus 5.5, low)
                        │  for each ready task (parallel only if Touches don't overlap, max 3):
                        │    delegate_task ──> BUILDER (GPT default: gpt-6.1-sol, low; Claude: Sonnet 5.5, medium)
                        │    delegate_task ──> REVIEWER (gpt-6-luna, high; fixed): tests and code review
                        │         PASS: merge into work/<parent>-<slug>
                        │         FIX:  new builder round from its last SHA (max 3)
                        │  final review of the whole branch vs the parent issue
                        v
                      asks you before pushing and opening the PR (Closes #parent, #tasks)
```

## Contents

| Path | What it is |
|---|---|
| `skills/plan-work/` | Entry point. Planner: grilling interview, task breakdown, GitHub issues, launches the orchestrator. User-invoked only. |
| `skills/scout/` | Read-only code reconnaissance; returns notes the planner posts on the parent issue. |
| `skills/orchestrate-plan/` | Orchestrator: schedules tasks, tracks state with labels, merges passing work, final review. |
| `skills/implement-task/` | Builder: one task, test-first, own worktree, reports a SHA. |
| `skills/review-task/` | Reviewer: runs Verify, checks every criterion with evidence, posts PASS / FIX / BLOCKED. |
| `docs/agents/roles.md` | Model profiles, effort option ids, and the exact `delegate_task` / `t3_thread_launch` payloads. |
| `docs/agents/issue-tracker.md` | `gh` commands for issues, sub-issues, blocking edges, labels, comment markers. |

## Install

In the target repo:

```sh
mkdir -p .agents/skills docs
cp -r skills/* .agents/skills/
cp -r docs/agents docs/
ln -s ../.agents/skills .claude/skills   # Claude reads .claude/skills; Codex reads .agents/skills
```

If `.claude/skills` already exists, copy or link `orchestrate-plan` and `implement-task` into it as well, so the Claude profile can use them. In T3 Code, run **Restart agent session** after adding skills, then type `$` in the composer to confirm each one is listed for its provider.

Requirements:

- T3 Code with V2 orchestration (the `t3-code` MCP tools), and the providers signed in for the selected profile (GPT and Claude profile models as needed).
- `gh` signed in on the machine running the T3 Code server (2.94+ for `--parent`; older versions use the API fallback in `issue-tracker.md`).
- A clean working tree on the base branch.

## Use

1. In T3 Code, start a new thread on **gpt-6.1-sol, medium** (the planner is fixed), in the **default** interaction mode (not plan mode: the orchestrator can never have broader modes than the planner) and the runtime mode you want the whole run to use.
2. Type `$plan-work` followed by what you want built.
3. Answer the grilling rounds. Each question comes with a recommended answer; "yes" accepts it.
4. Approve the task breakdown. The planner publishes the issues and shows you the links.
5. Say yes to launch the orchestrator. It uses the model and effort recorded in the parent plan: gpt-6.1-sol medium by default, or Opus 5.5 low in the Claude profile. Builders use gpt-6.1-sol low by default, or Sonnet 5.5 medium in the Claude profile. Reviewers always use gpt-6-luna high.
6. When it asks, approve the push and the pull request.

## Notes

- You may select available custom models for the orchestrator and builder. The planner stays on gpt-6.1-sol and the reviewer stays on gpt-6-luna. The parent issue records the resolved model and effort for each role.
- Delegated children get only their prompt, so all shared context lives in GitHub issues and comments, not in local files. `.work/` folders from the earlier draft are gone.
- Task issues stay open with `agent:passed` until the PR merges, so nothing is closed for work that has not shipped.
- These are drafts. Run one small, real task through the whole loop first and tighten the wording where the agents stumble.

## Credits

The grilling method in `plan-work`, the vertical-slice rules, the issue templates and the GitHub tracker conventions are adapted from [mattpocock/skills](https://github.com/mattpocock/skills), MIT License, Copyright (c) 2026 Matt Pocock.
