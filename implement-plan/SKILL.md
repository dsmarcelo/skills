---
name: "implement-plan"
description: "Pick up an approved implementation plan (GitHub issue + ADR) and implement it through subagents, verify, review, and finish. Use after build-feature has produced an approved plan."
---

# Implement Plan

You are a fresh coding agent picking up a plan that was already designed and approved. The understanding, ADR, and implementation plan were settled in a previous session (see the `build-feature` skill). Your job is to execute that plan faithfully, verify it, and review it.

Do not re-open design decisions. If the approved plan or ADR turns out to be wrong, stop the affected work and return the decision to the user.

The user invoking this skill is the approval to start implementing. Do not ask again whether to begin.

## 1. Locate the plan

The input is a reference to the GitHub issue containing the approved plan (issue URL or number). If none was given, ask the user for it. Do not guess or search for a plan on your own.

Read:

- the GitHub issue: feature summary, task IDs, blocking relationships, completion criteria;
- the ADR linked from the issue;
- repository instructions (CLAUDE.md, AGENTS.md, contributing docs) and relevant developer documentation.

If the issue has no plan, no task graph, or no ADR reference, stop and tell the user what is missing. Do not reconstruct a plan from memory or invent one.

Check whether the plan already has completed tasks (checked boxes, closed sub-issues, merged branches, existing commits) so you resume from the correct point instead of redoing work.

## 2. Prepare implementation

1. Check the current branch.
2. Do not implement directly on `main` or `master`.
3. Create or use a dedicated integration/feature branch (reuse the existing one if the work is being resumed).
4. Determine the task graph's current ready frontier: every incomplete task whose blockers are all complete.

The main agent acts as orchestrator and decides whether one or multiple implementation subagents are appropriate.

Use multiple subagents when:

- two or more ready tasks are genuinely independent;
- they can work without modifying the same tightly coupled area;
- parallel work is unlikely to create unnecessary merge conflicts;
- each task has enough context to be executed independently.

Use a single implementation subagent when:

- only one task is ready;
- tasks are tightly coupled;
- parallelism would create more coordination than value;
- the feature is small enough that concurrency gives little benefit.

Never create subagents merely to increase agent count.

## 3. Choose implementation reasoning level

Implementation agents use **Claude Sonnet 5.5** (in Claude Code, pass `model: "sonnet"` to the Agent tool). Set reasoning effort per task where the harness allows it; otherwise use the harness default.

Use **medium effort** by default for:

- well-specified features;
- local changes;
- straightforward business logic;
- isolated UI work;
- normal CRUD/integration work;
- tasks with clear existing patterns.

Use **high effort** when the task involves meaningful complexity such as:

- cross-cutting architecture;
- difficult state management;
- concurrency;
- migrations;
- security-sensitive behavior;
- unfamiliar code;
- subtle compatibility requirements;
- large dependency graphs;
- difficult debugging;
- multiple interacting systems.

The main agent chooses the level per task based on actual complexity. Do not use high effort automatically.

## 4. Implement through subagents

Treat the approved plan as a task graph.

For each task selected for parallel implementation:

1. create an isolated worktree (in Claude Code, `isolation: "worktree"` on the Agent tool);
2. create a dedicated branch based on the current integration branch;
3. give the implementation subagent pointers to:
   - the GitHub plan;
   - the ADR;
   - relevant repository instructions;
   - the task ID;
   - relevant previous commits or research;
4. avoid duplicating large context already available through those pointers;
5. have the subagent implement only its assigned task.

Implementation agents should use TDD where practical at the pre-agreed testing seams.

Prefer: **red → green → refactor**

Each implementation subagent must:

- stay within its assigned scope;
- follow existing repository conventions;
- avoid unrelated refactors;
- run focused tests regularly;
- run typechecking regularly when applicable;
- verify the observable behavior described by its task;
- commit its completed work;
- report blockers rather than silently changing the approved architecture.

A subagent must not redefine the approved requirements or ADR.

If implementation reveals that the approved architecture or plan is wrong, stop that affected branch of work and return the decision to the user instead of silently deviating.

## 5. Merge the frontier

When an implementation subagent finishes:

1. verify its required checks;
2. merge the latest integration branch into its task branch if necessary;
3. use a merger/review agent when useful to integrate the task into the integration branch;
4. resolve integration problems without weakening requirements or tests;
5. mark that task complete (update the GitHub issue according to the repository's conventions).

Recalculate the ready frontier after each merge.

Start newly-unblocked independent tasks as capacity becomes available.

Continue until the entire task graph is complete.

## 6. Verify the complete feature

After all planned tasks are integrated, verify the feature as a whole.

Run the repository's real verification commands, including applicable:

- unit tests;
- integration tests;
- end-to-end tests;
- typechecking;
- lint;
- production build;
- schema/migration checks.

Run focused tests throughout implementation and the full relevant suite at the end.

A syntax check alone is not sufficient verification.

Compare the finished implementation against:

1. the approved requirements (as recorded in the issue and ADR);
2. the approved ADR;
3. the approved GitHub implementation plan, including its Completion Criteria.

Do not declare the feature finished while a known requirement is missing, partial, or unverified.

## 7. Final review

Run final review using separate concerns. Where subagents are available, run these independently:

### Standards review

Check whether the diff follows:

- repository instructions;
- documented coding standards;
- existing conventions;
- architectural constraints;
- relevant ADRs.

### Spec review

Check whether the diff:

- implements every approved requirement;
- satisfies the GitHub plan;
- respects the ADR;
- introduces unintended behavior or scope creep;
- leaves any task incomplete.

Keep these review contexts separate so conformity does not hide specification mistakes and specification correctness does not hide engineering-quality problems.

Fix blocking findings with an implementation subagent, then rerun the relevant verification.

## 8. Finish

Before declaring completion:

- confirm every plan task is complete;
- run final required checks;
- ensure the integration branch is clean;
- update the GitHub issue according to repository conventions;
- create or update the pull request when appropriate.

Report briefly:

- what was implemented;
- the ADR path;
- the GitHub plan reference;
- verification performed;
- review result;
- any remaining limitations.

Do not claim completion for anything that was not actually verified.