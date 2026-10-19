# Roles, models and T3 Code templates

This file is the one place to change a model, an effort level, or how an agent is started. The skills point here instead of hard-coding any of it.

## Roles

| Role | Skill | Model | Effort | Runs as | Started by | Writes |
|---|---|---|---|---|---|---|
| Planner | `plan-work` | gpt-6.1-sol | medium | main thread (you start it) | you | GitHub parent issue and task sub-issues |
| Scout | `scout` | gpt-6-luna | medium | child task | planner | nothing (notes come back as its result) |
| Orchestrator | `orchestrate-plan` | claude-opus-5-5 | **medium, never low** | new top-level thread in its own worktree | planner | labels, status comment, merges on the integration branch |
| Implementer | `implement-task` | claude-sonnet-5-5 | medium | child task, own worktree | orchestrator | commits on `task/<n>-r<round>`, report comment |
| Reviewer | `review-task` | gpt-6.1-sol | medium | child task, own worktree | orchestrator | verdict comment |

The reviewer must stay on a different model family from the implementer. That difference is what makes the review gate worth having.

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
| Claude | `effort` | `medium` |
| Codex | `reasoningEffort` | `medium` |

These ids come from T3 Code's own adapter tests. `orchestrator_capabilities` is the authority if they change.

## Templates

Replace `<...>`. Keep prompts to pointers; do not paste issue content into them.

### Scout (planner -> `delegate_task`)

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

### Orchestrator (planner -> `t3_thread_launch`)

```json
{
  "title": "Orchestrate: <plan title>",
  "message": "Use the orchestrate-plan skill.\nPlan: #<parent>\nBase branch: <base>\nIntegration branch: work/<parent>-<slug>",
  "modelSelection": {
    "instanceId": "<claude instance id>",
    "model": "claude-opus-5-5",
    "options": [{ "id": "effort", "value": "medium" }]
  },
  "workspaceStrategy": {
    "type": "worktree",
    "baseRef": "<base>",
    "branch": "work/<parent>-<slug>",
    "startFromOrigin": false
  }
}
```

### Implementer, round 1 (orchestrator -> `delegate_task`)

```json
{
  "task": "Use the implement-task skill.\nTask: #<task>\nRound: 1\nStart commit: <integration tip sha>\nIntegration branch: work/<parent>-<slug>",
  "title": "Implement #<task> r1",
  "target": {
    "providerInstanceId": "<claude instance id>",
    "model": "claude-sonnet-5-5",
    "options": [{ "id": "effort", "value": "medium" }]
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
    "providerInstanceId": "<codex instance id>",
    "model": "gpt-6.1-sol",
    "options": [{ "id": "reasoningEffort", "value": "medium" }]
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
