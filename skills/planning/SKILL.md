---
name: planning
description: "Interview the user about a feature or change, then publish one GitHub issue with a task checklist. Planning only; no code."
disable-model-invocation: true
---

# Planning

Planning only. Never write implementation code.

1. **Explore first.** Read the code and whatever docs `AGENTS.md` points to. Look facts up yourself; never ask the user what the repo can answer.
2. **Interview with `AskUserQuestion`.** Ask open decisions in rounds of up to 4, recommended option first. Only ask what changes the plan: behavior and edge cases, non-goals, data, permissions, UI, risk, test seams. When nothing material is assumed, summarize in 5 lines and get a yes.
3. **Publish one issue** with `gh`, following the repo's issue conventions in `AGENTS.md`:
   - Problem and outcome, in 2-3 sentences.
   - Decisions, one line each.
   - Out of scope.
   - Tasks: a checklist of vertical slices, each with an observable acceptance criterion and how to verify it. Note real dependencies. Mark `Risky` on schema, billing, auth or permission work.
   - Test seams: where tests go (existing seams first).
4. Print the issue URL and stop. The issue is the whole contract; anything not in it is lost.

Keep it short: no file paths or code unless a snippet encodes a decision.
