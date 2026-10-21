# Roles, models and T3 Code templates

This file defines the model profiles and is the one place to change a model, an effort level, or how an agent is started. Ask which profile to use; default to GPT when the user does not specify one. A user may choose another available model for configurable roles, subject to the fixed planner, scout, and reviewer roles below. Always check `orchestrator_capabilities` and use its exact model and option ids.

## Roles

| Role | Skill | Default model | Default effort | Claude profile | Runs as | Started by | Writes |
|---|---|---|---|---|---|---|---|
| Planner | `plan-work` | gpt-6.1-sol | medium | Always gpt-6.1-sol (fixed) | main thread (you start it) | you | GitHub parent issue and task sub-issues |
| Scout / codebase exploration / file search / summarization | `scout` | gpt-6-luna | medium | gpt-6-luna (fixed) | child task | planner | nothing (notes come back as its result) |
| Orchestrator | `orchestrate-plan` | gpt-6.1-sol | medium | claude-opus-5-5, low | new top-level thread in its own worktree | planner | labels, status comment, merges on the integration branch |
| Builder / implementer / test author | `implement-task` | gpt-6.1-sol | low | claude-sonnet-5-5, medium | child task, own worktree | orchestrator | commits on `task/<n>-r<round>`, report comment |
| Tests and code reviewer | `review-task` | gpt-6-luna | high | Always gpt-6-luna (fixed) | child task, own worktree | orchestrator | verdict comment |

The planner is always gpt-6.1-sol and the reviewer is always gpt-6-luna, regardless of profile or user preference. The default GPT profile uses gpt-6.1-sol for the orchestrator and builder. The Claude profile changes only those two roles to Opus 5.5 (low) and Sonnet 5.5 (medium). The reviewer remains GPT-6 Luna (high), so reviews stay on the GPT family even when Claude builds the change. The builder writes and runs tests; the reviewer independently runs verification and reviews the code.

At the start of planning, ask whether to use the default GPT profile, the Claude profile, or custom available models for the configurable orchestrator and builder roles. The planner, scout, and reviewer stay fixed as listed above. Record the selected profile and resolved model/effort for each role in the parent issue's Decisions Log and `agent-workflow` block, and include them in the orchestrator launch message. Later stages must follow that recorded configuration. If a requested model is unavailable, explain which role cannot be started and offer the supported profile options; do not silently substitute.

## How T3 Code starts agents

All of this goes through the `t3-code` MCP server that T3 Code injects into every thread (`mcp__t3-code__*`). Facts the skills rely on, from T3 Code's orchestrator MCP docs and schemas:

- **`orchestrator_capabilities`** lists each provider instance, its models, and each model's options (effort and so on). Call it first to get the exact `providerInstanceId`, model ids and option ids. Use the values it returns over the ones below if they differ.
- **`delegate_task`** creates a child thread. The child receives **only the task prompt**, never the parent's conversation, so every prompt must be self-contained: here that means issue numbers, SHAs and branch names, since the content lives in GitHub. Model and effort go in `target`. `isolation: "worktree"` gives the child its own checkout of the parent's **current committed** HEAD; uncommitted changes are not copied.
- **Async children** wake the parent with a notification when they finish. End the turn instead of polling; use `task_status` only when a result is needed in the middle of a turn.
- **A finished child cannot be continued.** Every review or fix round is a new `delegate_task` call with its own `clientRequestId`. Do not send follow-up work to an old child with `t3_thread_send`.
- **`t3_thread_launch`** creates a new top-level thread bound to a workspace. Model and effort go in `modelSelection`, and `title` is required. Launching requires the user to have asked for a new thread.
- **Modes never widen.** A child or launched thread cannot have a broader runtime or interaction mode than its caller. Start the planner thread in the default interaction mode (not plan mode) and the runtime mode you want the whole run to have.

### Effort option ids

| Provider | Option id | Value used here |
|---|---|---|
| Claude | `effort` | `low` / `medium` |
| Codex | `reasoningEffort` | `low` / `medium` / `high` |

These ids come from T3 Code's own adapter tests. `orchestrator_capabilities` is the authority if they change. GPT profile effort defaults are medium for planner, scout, and orchestrator, low for builder, and high for reviewer. Claude profile effort defaults are low for Opus and medium for Sonnet.

## Templates

Replace `<...>`. Keep prompts to pointers; do not paste issue content into them.

### Scout (planner -> `delegate_task`; fixed model)

```json
{
  "task": "Use the scout skill.\nQuestion: <one narrow question>\nInclude project commands: <yes|no>",
  "title": "Scout: <topic>",
  "target": {
    "providerInstanceId": "<codex instance id>",
    "model": "gpt-6-luna",
    "options": [{ "id": "reasoningEffort", "value": "medium" }]
  },
  "role": "research",
  "mode": "async",
  "clientRequestId": "scout-<slug>-<n>"
}
```

### Orchestrator (planner -> `t3_thread_launch`; selected model and effort)

```json
{
  "title": "Orchestrate: <plan title>",
  "message": "Use the orchestrate-plan skill.\nPlan: #<parent>\nBase branch: <base>\nIntegration branch: work/<parent>-<slug>\nModel profile: <GPT|Claude|custom>\nOrchestrator: <model>, <effort>\nBuilder: <model>, <effort>\nReviewer: gpt-6-luna, high",
  "modelSelection": {
    "instanceId": "<selected provider instance id>",
    "model": "<selected orchestrator model>",
    "options": [{ "id": "<selected effort option id>", "value": "<selected effort>" }]
  },
  "workspaceStrategy": {
    "type": "worktree",
    "baseRef": "<base>",
    "branch": "work/<parent>-<slug>",
    "startFromOrigin": false
  }
}
```

### Builder, round 1 (orchestrator -> `delegate_task`; selected model and effort)

```json
{
  "task": "Use the implement-task skill.\nTask: #<task>\nRound: 1\nStart commit: <integration tip sha>\nIntegration branch: work/<parent>-<slug>",
  "title": "Implement #<task> r1",
  "target": {
    "providerInstanceId": "<selected provider instance id>",
    "model": "<selected builder model>",
    "options": [{ "id": "<selected effort option id>", "value": "<selected effort>" }]
  },
  "role": "implementation",
  "isolation": "worktree",
  "mode": "async",
  "clientRequestId": "impl-<task>-r1"
}
```

### Implementer, fix round

Same as round 1, with:

```
Use the implement-task skill.
Task: #<task>
Round: <n>
Start commit: <previous implementer sha>
Integration branch: work/<parent>-<slug>
Fix the findings in the latest review-verdict comment on #<task>.
```

and `clientRequestId: "impl-<task>-r<n>"`. For a merge conflict, replace the last line with `Your commit <sha> conflicts with the integration branch tip <tip sha>. Merge the tip and resolve.` For the final fix round, use `Task: final (parent #<parent>)` and the integration tip as the start commit.

### Reviewer (orchestrator -> `delegate_task`)

```json
{
  "task": "Use the review-task skill.\nTask: #<task>\nRound: <n>\nSHA to review: <implementer sha>\nIntegration branch: work/<parent>-<slug>",
  "title": "Review #<task> r<n>",
  "target": {
    "providerInstanceId": "<GPT instance id>",
    "model": "gpt-6-luna",
    "options": [{ "id": "reasoningEffort", "value": "high" }]
  },
  "role": "review",
  "isolation": "worktree",
  "mode": "async",
  "clientRequestId": "review-<task>-r<n>"
}
```

For the final review use `Task: final (parent #<parent>)`, `Range: <base>..work/<parent>-<slug>` instead of the SHA, and `clientRequestId: "review-final-r<n>"`.

## Branches

- Integration branch: `work/<parent>-<slug>`, in the orchestrator's worktree.
- Implementer branches: `task/<task>-r<round>`, in each child's worktree. They share the repository's object store, so the orchestrator can merge any reported SHA without pushing.
- Nothing is pushed until the human says yes at the end.
