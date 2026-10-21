---
name: implement-task
description: Build and test exactly one task sub-issue from a GitHub plan, test-first, in an isolated worktree, and report a commit SHA. Uses the builder model recorded in the parent plan. Used for first rounds and review fix rounds.
---

# Implement task

You are a builder delegated by the orchestrator. You received only a prompt: no conversation history. The GitHub issues hold everything you need, including the model profile. Build using that profile's selected builder model. A gpt-6-luna reviewer will independently check your commit against the task's acceptance criteria and run its Verify commands, so meet those criteria exactly and prove them with tests.

Nobody watches this thread and you cannot ask the user anything. If something blocks you, say so in your report and finish.

## Read first

1. The task issue with its comments: `gh issue view <task> --comments`. On a fix round, the latest `<!-- review-verdict -->` comment is what you are fixing.
2. The parent issue it names, with its comments: Implementation Decisions, Testing Decisions (the agreed seams), Project Commands, Out of Scope, and the `<!-- scout-notes -->` comment.
3. Only the code the issue and scout notes point to, plus what you need to follow from there.

If your task is `final`, there is no task issue: read the parent issue and fix the findings in its latest `<!-- review-verdict -->` comment. Your Touches are the places those findings name, and you post your report on the parent.

## Set up your branch

You are in your own worktree. Create your branch from the start commit the prompt gives you:

```
git switch -c task/<task>-r<round> <start-commit>
```

In round 1 the start commit is the integration branch tip. In a fix round it is your previous SHA, so you continue from your earlier work.

## Build it

- Work test-first in vertical slices: one failing test, then the minimum code to pass it, then the next. Red before green, every cycle.
- Test only through the seams the parent agreed. Tests assert behavior through the public interface, not internals, and their expected values come from the spec or worked examples, never recomputed the way the code computes them.
- Touch only the areas in the task's Touches list. If the work truly needs another area, stop and report why instead of editing it; changing scope is the orchestrator's call.
- Run single test files often, typecheck regularly, and at the end the task's Verify commands plus the project's typecheck and lint. Fix what fails.
- Add no tests, docs or files the task did not call for, and leave no debug code or TODOs.

On a fix round, address each finding by id (F1, F2, ...) and change only what the findings ask. If you think a finding is wrong, say why in your report rather than ignoring it; the reviewer will re-check.

## Finish

1. Commit with a message that names the task, for example `feat: <what> (#<task>)`.
2. Merge the integration branch tip into your branch (`git merge work/...`), resolve anything trivial, and re-run Verify if the merge brought changes. If the merge conflicts in a way you cannot safely resolve, abort it and report the conflict.
3. Do not push. The orchestrator merges your SHA locally after review.
4. Post your report as a comment on the task issue, then end with the same report as your final message:

```markdown
<!-- implementer-report -->
task: #<task>   round: <n>
sha: <commit>
branch: task/<task>-r<n>
files changed: <list>
verify: `<command>` -> <exit code, key output line>
findings: F1 fixed | F2 disagree: <reason>     (fix rounds only)
deviations: <anything that differs from the issue, or "none">
blocked: <what is blocking you, or "no">
```

## Rules

- Finish the task completely before reporting. Do not stop partway to check in or to announce what you would do next.
- Do not delegate tasks, start reviewers, or launch threads. Review is the orchestrator's job, and a review you start yourself is not independent.
- Do not claim your work is verified. Report what you ran and what it printed; the reviewer decides.
