# Agent model configuration

Edit this table to select the model and effort for each role. Resolve the exact provider, model and option IDs using `orchestrator_capabilities` before launching; if a configured model or effort is unavailable, report it and do not silently substitute. The planner runs in the current thread and must be started with its configured model and effort. An explicit user override takes precedence and is recorded in the plan.

| Role | Model | Effort |
|---|---|---|
| Planner | gpt-6.1-sol | medium |
| Scout | claude-haiku-5-5 | medium |
| Orchestrator | claude-opus-5-5 | low |
| Builder | claude-sonnet-5-5 | medium |
| Reviewer | gpt-6.1-terra | medium |

Record the resolved settings for all roles in the parent issue so an in-progress plan remains consistent if this file changes later. Scout and orchestrator use the configuration resolved by the planner; subsequent builders and reviewers use the settings recorded in the plan.
