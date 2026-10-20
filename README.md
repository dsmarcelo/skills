# Plan, orchestrate, review (for T3 Code)

A multi-agent coding workflow for [T3 Code](https://github.com/pingdotgg/t3code), using models from OpenAI and Anthropic. The thread you start is the **planner**: it interviews you until you share an understanding of the work, publishes the plan as GitHub issues, and launches the **orchestrator** in a new thread. The orchestrator then delegates each task to an implementer and gates it with a reviewer on a different model family.

```
you ── $plan-work ──> PLANNER thread (gpt-6.1-sol, medium)
                        │  delegate_task ──> SCOUT (gpt-6-luna, medium): facts, file:line notes
                        │  grilling rounds with you until the design tree is resolved
                        │  task breakdown, approved by you
                        │  GitHub: parent issue + task sub-issues + blocking edges
                        │  t3_thread_launch (after your yes)
                        v
                      ORCHESTRATOR thread (claude-opus-5-5, effort medium, own worktree)
                        │  for each ready task (parallel only if Touches don't overlap, max 3):
                        │    delegate_task ──> IMPLEMENTER (claude-sonnet-5-5, medium, own worktree)
                        │    delegate_task ──> REVIEWER   (gpt-6.1-sol, medium, own worktree)
                        │         PASS: merge into work/<parent>-<slug>
                        │         FIX:  new implementer round from its last SHA (max 3)
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
| `skills/implement-task/` | Implementer: one task, test-first, own worktree, reports a SHA. |
| `skills/review-task/` | Reviewer: runs Verify, checks every criterion with evidence, posts PASS / FIX / BLOCKED. |
| `docs/agents/roles.md` | Models, effort option ids, and the exact `delegate_task` / `t3_thread_launch` payloads. Edit this file to change a model. |
| `docs/agents/issue-tracker.md` | `gh` commands for issues, sub-issues, blocking edges, labels, comment markers. |

## Install

In the target repo:

```sh
mkdir -p .agents/skills docs
cp -r skills/* .agents/skills/
cp -r docs/agents docs/
ln -s ../.agents/skills .claude/skills   # Claude reads .claude/skills; Codex reads .agents/skills
```

If `.claude/skills` already exists, copy or link `orchestrate-plan` and `implement-task` into it instead: those two run on Claude. `plan-work`, `scout` and `review-task` run on Codex. In T3 Code, run **Restart agent session** after adding skills, then type `$` in the composer to confirm each one is listed for its provider.

Requirements:

- T3 Code with V2 orchestration (the `t3-code` MCP tools), and both the Codex and Claude providers signed in with the four models available.
- `gh` signed in on the machine running the T3 Code server (2.94+ for `--parent`; older versions use the API fallback in `issue-tracker.md`).
- A clean working tree on the base branch.

## Use

1. In T3 Code, start a new thread on **gpt-6.1-sol, medium**, in the **default** interaction mode (not plan mode: the orchestrator can never have broader modes than the planner) and the runtime mode you want the whole run to use.
2. Type `$plan-work` followed by what you want built.
3. Answer the grilling rounds. Each question comes with a recommended answer; "yes" accepts it.
4. Approve the task breakdown. The planner publishes the issues and shows you the links.
5. Say yes to launch the orchestrator. It opens as its own thread on Opus 5.5 at medium effort, and reports there and on the parent issue.
6. When it asks, approve the push and the pull request.

## Notes

- The orchestrator's effort is fixed at **medium** in the launch payload. If you ever start it by hand, pick medium in the model picker; never low.
- Delegated children get only their prompt, so all shared context lives in GitHub issues and comments, not in local files. `.work/` folders from the earlier draft are gone.
- Task issues stay open with `agent:passed` until the PR merges, so nothing is closed for work that has not shipped.
- These are drafts. Run one small, real task through the whole loop first and tighten the wording where the agents stumble.

## Credits

The grilling method in `plan-work`, the vertical-slice rules, the issue templates and the GitHub tracker conventions are adapted from [mattpocock/skills](https://github.com/mattpocock/skills), MIT License, Copyright (c) 2026 Matt Pocock.
