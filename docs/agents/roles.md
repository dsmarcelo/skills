# Roles, models and T3 Code templates

Model and effort selections for every role live in [`models.md`](models.md), the single configuration file. Read it before launching agents, resolve IDs against capabilities, and record the resolved settings for every role in the parent issue. Use the recorded settings for the lifetime of the plan. Explicit user overrides take precedence; unavailable selections must be reported without substitution.

| Role | Skill | Runs as | Started by |
|---|---|---|---|
| Planner | `plan-work` | main thread | user |
| Scout | `scout` | child task | planner |
| Orchestrator | `orchestrate-plan` | top-level thread in its own worktree | planner |
| Builder | `implement-task` | child task in its own worktree | orchestrator |
| Reviewer | `review-task` | child task in its own worktree | orchestrator |

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

These ids come from T3 Code's own adapter tests. `orchestrator_capabilities` is the authority if they change. Use the configured effort in `models.md`.

## Templates

Replace `<...>`. Keep prompts to pointers; do not paste issue content into them.

### Scout (planner -> `delegate_task`; configured model)

```json
{
  "task": "Use the scout skill.\nQuestion: <one narrow question>\nInclude project commands: <yes|no>",
  "title": "Scout: <topic>",
  "target": {
    "providerInstanceId": "<scout provider instance id>",
    "model": "<configured scout model>",
    "options": [{ "id": "<scout effort option id>", "value": "<scout effort>" }]
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
  "message": "Use the orchestrate-plan skill.\nPlan: #<parent>\nBase branch: <base>\nIntegration branch: work/<parent>-<slug>\nOrchestrator: <model>, <effort>\nBuilder: <model>, <effort>\nReviewer: <model>, <effort>",
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
    "model": "<configured reviewer model>",
    "options": [{ "id": "<reviewer effort option id>", "value": "<reviewer effort>" }]
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
