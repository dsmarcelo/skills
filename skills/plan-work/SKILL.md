---
name: plan-work
description: Start here for any new feature, change or fix that should go through the scout, plan, orchestrate, review workflow. Interviews the user until there is a shared understanding, publishes the plan as a GitHub parent issue with one sub-issue per task, then launches the orchestrator in a new T3 Code thread. This thread is the planner and never writes code.
disable-model-invocation: true
---

# Plan work

This thread is the **planner**, the first stage of a four-role workflow:

1. **Planner** (this thread, gpt-6.1-sol, medium): interviews the user, writes the plan to GitHub.
2. **Scout** (gpt-6-luna, medium): child task this thread delegates to look up facts in the code.
3. **Orchestrator** (Opus 5.5, medium, a new top-level thread): executes the plan with implementers (Sonnet 5.5) and reviewers (gpt-6.1-sol).
4. **Reviewer**: checks every task against the issue you write.

Everything after this thread starts from a clean context and reads only GitHub. The issues you publish are the whole contract: implementers build against them, the reviewer judges against them, the orchestrator schedules from them. If something was decided in this conversation and is not in an issue, it is lost.

Read `docs/agents/roles.md` (models, option ids, tool templates) and `docs/agents/issue-tracker.md` (gh commands) before starting. Never write or edit code in this thread.

## Phase 0: Preflight

Check these facts yourself before the first question, and stop with a clear message if one fails:

- `gh auth status` succeeds and `git remote -v` points at the GitHub repo.
- The workflow labels exist. Create any that are missing (commands in `issue-tracker.md`).
- Call `orchestrator_capabilities` on the `t3-code` MCP server. Confirm the four models in `roles.md` are available and note each provider instance id and effort option id. If gpt-6.1-sol cannot run child tasks, stop: the reviewer must be a different model family from the implementer, and there is no fallback.
- This thread runs in the default interaction mode, not plan mode. A thread launched from here cannot have broader runtime or interaction modes than this one, so the orchestrator would be unable to edit code. If you are in plan mode, ask the user to switch before going further.

## Phase 1: Start the scout early

As soon as you know the goal, delegate a scout (template in `roles.md`) with `mode: "async"`, so it explores while you interview. Do not wait for it. Questions that need its facts wait; every other question goes out now.

Delegate a further scout whenever a later question needs a fact you do not have. Keep each scout's question narrow.

## Phase 2: Grill until there is a shared understanding

> Adapted from `grilling` in [mattpocock/skills](https://github.com/mattpocock/skills) (MIT). The method is unchanged; the T3 Code and planning specifics are added.

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Word each question so "yes" accepts your recommended answer.

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a scout to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the scout to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.

### Branches the tree must reach

These are not a questionnaire to read out in order. They are the branches a plan cannot do without, so check the tree has visited each before you call the frontier empty:

- **Problem and outcome**: who has the problem, and what is observably different when it is solved.
- **Behavior**: the user stories, including error, empty and edge cases.
- **Non-goals**: what looks related but is out of scope.
- **Shape**: which modules change, their interfaces, any schema, API or config changes, and which existing patterns to follow.
- **Test seams**: the public boundaries where tests go. Prefer existing seams, the highest one possible, and as few as possible. The implementers cannot ask the user, so the seams are agreed here.
- **Verification**: the exact commands that prove it works (from the scout), plus anything that needs a manual or browser check.
- **Risk**: migrations, irreversible steps, shared systems, performance and security concerns.
- **Base branch**: what the work starts from, and the integration branch name.

## Phase 3: Agree the task breakdown

Once the design is settled, draft the tasks and put them to the user before publishing.

Break the work into **tracer bullet** tasks (from mattpocock/skills `to-tickets`):

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests): vertical, NOT a horizontal slice of one layer.
- A completed slice is demoable or verifiable on its own.
- Each slice is sized to fit in a single fresh context window.
- Any prefactoring is done first ("make the change easy, then make the easy change").
- A wide mechanical refactor (one change that breaks many call sites at once) is the exception: sequence it as expand, migrate in batches, contract.

Give each task its **blocking edges**: the tasks that must finish before it can start. Give each a **Touches** list of the modules or directories it will edit. The orchestrator runs two tasks in parallel only when their Touches do not overlap, so if two tasks must edit the same area, add an edge between them.

Describe each task by what and why. Specify how only where a real constraint exists, such as an interface another task depends on or a pattern to imitate. Prescriptive steps tend to be wrong in small ways, and a faithful implementer carries the error forward.

Show the breakdown as a numbered list: title, blocked by, touches, and what it delivers end to end. Then ask, as a grilling round:

- Does the granularity feel right (too coarse, too fine)?
- Is every blocking edge real, and is any missing?
- Should any task be merged or split?

Iterate until the user approves the breakdown.

## Phase 4: Publish the plan to GitHub

Use the templates below and the commands in `issue-tracker.md`.

1. Create the **parent issue** (label `agent:plan`).
2. Post the scout's notes as a comment on the parent, starting with the marker `<!-- scout-notes -->`. Implementers and reviewers read it; do not leave facts only in this conversation.
3. Create one **task sub-issue** per task, blockers first, so every edge can point at a real issue number. Attach each as a sub-issue of the parent, add native blocking edges, and apply `ready-for-agent`.
4. Self-check, then fix anything that fails:
   - Every acceptance criterion is observable (a command result, a visible behavior), not a quality word like "clean" or "robust".
   - Every Verify command is one the scout confirmed exists in this repo.
   - No dependency cycles, and tasks whose Touches overlap are ordered by an edge.
   - The tasks together cover the goal and touch no non-goal.
5. Show the user the parent issue link and the list of sub-issues.

### Parent issue template

Title: `Plan: <title>`

```markdown
<!-- agent-workflow
slug: <kebab-case>
base: <base branch>
integration: work/<parent-number>-<slug>
-->

## Problem Statement
<from the user's perspective>

## Solution
<from the user's perspective>

## User Stories
1. As an <actor>, I want <feature>, so that <benefit>
<a long list covering every behavior agreed in the interview, including edge cases>

## Implementation Decisions
- <modules built or modified, their interfaces, schema and API changes, patterns to follow>

## Testing Decisions
- Seams: <the agreed public boundaries tests go through>
- Good test here: <behavior through the public interface, not implementation details>
- Prior art: <similar tests in the codebase>

## Project Commands
- test: `...`
- typecheck: `...`
- lint: `...`
- build: `...`

## Out of Scope
- <non-goals>

## Decisions Log
- <each decision settled in the interview, one line, so later agents know it was a choice and not an accident>

## Tasks
<filled in after the sub-issues exist: - [ ] #<n> <title>>
```

Avoid file paths and code snippets in the parent: they go stale. The exception is a snippet that encodes a decision more precisely than prose (a type shape, a schema, a state machine).

### Task sub-issue template

Title: `<short descriptive name>`

```markdown
## Parent
#<parent number>

## What to build
<the end-to-end behavior this task makes work, from the user's perspective, and why>

## Acceptance criteria
- [ ] <observable criterion>
- [ ] <observable criterion>

## Verify
- `<exact command>`
- <manual or browser check, if any, with the steps>

## Test seams
- <which of the parent's seams this task tests through>

## Touches
- <modules or directories this task may edit>

## Blocked by
- #<n>, or "None (can start immediately)". Omit when native blocking edges were set.
```

## Phase 5: Launch the orchestrator

Ask the user whether to launch the orchestrator now. Launching starts a new top-level thread, which T3 Code only allows when the user has asked for it, so wait for a clear yes.

On yes, call `t3_thread_launch` with the orchestrator template in `roles.md`: Opus 5.5 with effort set to `medium` (never `low`), a new worktree based on the plan's base branch, and the integration branch name from the parent issue. Then give the user the new thread as `[title](t3-thread://v1/<threadId>)` and stop. Do not monitor the orchestrator from here; it reports in its own thread and on the parent issue.

If the user says no, tell them how to start it later: a new thread on Opus 5.5 at medium effort, in a worktree on the integration branch, with the message `$orchestrate-plan #<parent number>`.
