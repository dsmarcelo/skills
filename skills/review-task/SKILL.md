---
name: review-task
description: Independent, read-only review of an implementer's commit against its GitHub task sub-issue, or of the whole integration branch against the parent plan issue (task "final"). Runs the Verify commands, checks every acceptance criterion with evidence, and posts a PASS, FIX or BLOCKED verdict as an issue comment. Used by reviewer children the orchestrator delegates.
---

# Review task

You are the reviewer, delegated by the orchestrator. You run on a different model family from the implementer and start with no conversation history, on purpose: authors tend to rate their own work too generously, and a separate reviewer can afford to be skeptical. Treat the implementer's report as a claim to check, not as evidence.

Nobody watches this thread. Do not ask questions; put anything you need in the verdict and finish.

## Inputs (from your prompt)

- Task issue number, or `final` with the parent issue number.
- The SHA to review. For `final`, the base and the integration branch.
- The integration branch name and the round number.

## Process

1. **Read the contract.** `gh issue view <task> --comments`, then the parent issue it names with its comments: Testing Decisions, Project Commands, Out of Scope, Decisions Log, and the `<!-- scout-notes -->` comment. Earlier `<!-- review-verdict -->` and `<!-- implementer-report -->` comments on the task show prior rounds: check that each earlier finding is now fixed or credibly answered.
2. **Check out the work.** In your worktree run `git checkout --detach <sha>`. Diff it against where it started: `git diff <integration-branch>...<sha>`. List the changed files and compare them with the task's Touches.
3. **Run it.** Run the task's Verify commands yourself, then the project's typecheck and lint. Record each exit code and the key output line. A command that cannot run is a finding, not a pass. Do manual or browser checks the issue lists, if your tools allow; otherwise mark those criteria "cannot verify".
4. **Judge each acceptance criterion**: met, not met, or cannot verify, each with evidence (command output or `path:line`).
5. **Check the tests.** Each criterion should have a test that asserts behavior through an agreed seam and would fail without the change (reason from the code; do not modify the repo to try it). A test that mirrors the implementation, or recomputes its expected value the way the code does, does not count.
6. **Check scope.** Edits outside Touches, extras the issue did not ask for, debug code, TODOs, dead code, anything that touches an Out of Scope item.
7. **Correctness.** Report a bug only when you can state a concrete scenario where it fails. This keeps false alarms from sending implementers on pointless fix rounds.

For `final`: the criteria are the parent's User Stories and Decisions, every task issue passed, Out of Scope untouched, and the full test suite green. Review how the tasks fit together; do not re-review each task.

## Verdict

Post the verdict as a comment on the task issue (the parent issue for `final`), using this format:

```markdown
<!-- review-verdict -->
VERDICT: PASS | FIX | BLOCKED
task: #<n>   round: <n>   sha: <sha>
summary: <1-2 sentences>

## Criteria
| # | criterion | result | evidence |
|---|-----------|--------|----------|
| 1 | <text from the issue> | met / not met / cannot verify | <output line or path:line> |

## Verification run
- `<command>` -> exit <code>; <key output>

## Findings
### F1 [blocker|major|minor] <title>
- where: path:line
- problem: <what is wrong, and the scenario that exposes it>
- fix: <smallest change that resolves it>

## Prior findings
- r<n> F<k>: fixed | still open | answered: <reason accepted or rejected>

## Out-of-scope observations
- <optional; not a reason to fail the task>
```

Choose the verdict this way:

- `PASS`: every criterion met, Verify passes, no blocker or major finding. Minor findings are listed but do not fail the task; otherwise the loop never ends.
- `FIX`: something the implementer can correct. Give each finding an id so the implementer can answer it by id.
- `BLOCKED`: the issue is contradictory or incomplete, a human decision is needed, or the environment cannot run Verify. Say exactly what is needed.

End with a final message of only the first line of the verdict and the comment URL. The orchestrator reads the comment if it needs more.

## Rules

- Stay read-only on the code. Do not edit, format, stage, commit or push. Your only write is the verdict comment; commands that write only to build output or temp folders are fine.
- Judge against the issue and the repo's documented standards, not your own preferences. Taste goes under Out-of-scope observations.
- Be specific and brief. A finding without a location and a failing scenario is noise.
