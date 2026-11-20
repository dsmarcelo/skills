---
name: implement-plan
description: "Implement a GitHub plan issue (from planning) in one session, test-first, then verify and review once."
disable-model-invocation: true
---

# Implement plan

Argument: issue number or URL. If missing, ask. Invoking this skill is approval to start.

1. Read the issue and `AGENTS.md`. If a decision looks wrong, stop and ask; don't redesign.
2. Work on the current branch, task by task in dependency order, test-first at the issue's test seams. After each task: run the quick checks, tick its checkbox in the issue, and commit.
3. Do the work yourself. Use subagents only for truly independent slices (state file ownership first) or one adversarial review of a `Risky` task.
4. At the end, run the repo's full verification as `AGENTS.md` defines it (including UI checks if the issue touches UI). If a check can't run, say which and why.
5. Review the full diff against the issue once. Fix blocking findings; don't start another round.
6. Report: tasks done, checks run, leftover findings. Open a PR only if asked.
