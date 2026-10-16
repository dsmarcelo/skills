---
name: "build-feature"
description: "Guide a feature or meaningful code change from shared understanding through architecture and an approved, published implementation plan. Use when implementing a new feature, changing existing behavior, or making a non-trivial product or architectural change."
---

# Build Feature

Interview the user relentlessly until you reach a shared understanding. Map this as a design tree: every decision branches into the decisions that hang off it.

This skill covers design and planning only. Do not write implementation code. Implementation is done by a separate agent using the `implement-plan` skill, once the plan is approved and published.

## 1. Reach shared understanding

Explore the repository before asking questions that the codebase can answer.

Read, when present:

- existing architecture and developer documentation
- relevant ADRs
- domain/context documentation
- related implementation and tests
- package/framework documentation when current behavior matters

Finding facts is the agent's job. Do not ask the user for information that can be discovered from the repository, tools, or authoritative documentation.

Treat the feature as a decision tree.

Work through the unresolved frontier: ask only questions whose prerequisites are already settled. Group independent questions when useful; defer dependent questions until their prerequisites are answered.

For every meaningful decision:

- explain the decision briefly;
- give the recommended option;
- mention important alternatives or trade-offs when relevant;
- let the user decide.

Clarify at minimum, where applicable:

- intended user behavior;
- success criteria;
- edge cases and failure behavior;
- scope and explicit non-goals;
- domain terminology;
- UI/UX behavior;
- data model changes;
- API/contracts;
- state and synchronization;
- permissions/security;
- compatibility and migrations;
- observability;
- testing expectations;
- rollout or backwards compatibility.

Continue until no material branch remains silently assumed.

Summarize the resulting shared understanding.

Then ask the user explicitly whether you may continue to the architectural decision.

Stop until the user confirms.

## 2. Create the architectural decision record

Inspect existing ADR conventions first. Follow the repository's convention when one exists.

Otherwise store the ADR under:

`docs/adr/NNNN-<decision-slug>.md`

Scan existing ADRs and choose the next available number.

The ADR should record the durable decision, not the implementation procedure.

Keep it concise. Include:

```md
# <Decision title>

<Context: what requires a decision.>

<Decision: what architecture or approach was chosen and why.>

## Considered Options
- <option and relevant trade-off>
- <option and relevant trade-off>

## Consequences
- <important consequence>
- <important constraint>
```

Omit optional sections when they add no useful information.

Do not put transient implementation details, file lists, step-by-step instructions, or code snippets into the ADR unless they represent a durable architectural contract.

If the proposed feature conflicts with an existing ADR, surface that conflict before replacing or superseding it.

Write the ADR into the repository's `docs` architecture area.

Present the completed ADR to the user and ask:

**Is this architectural decision correct?**

If the user requests changes, revise it and ask again.

Do not continue until the user explicitly approves the ADR.

## 3. Write the implementation plan

After the ADR is approved, create the implementation plan.

Do not implement anything.

Explore enough of the repository to make the plan concrete and realistic.

Prefer existing architecture, modules, conventions, interfaces, and testing seams over introducing new ones.

The plan must describe the desired result rather than prescribing unnecessary low-level code.

The plan will be executed by a different agent that has none of this conversation's context, so it must be self-contained enough to act on together with the ADR and the repository.

Structure the work as a task graph, not merely a sequential checklist.

Each task should be a small, independently understandable vertical slice whenever possible.

For every task include:

```md
### <Task title>

Goal:
<observable result>

Work:
- <required change>
- <required change>

Verification:
- <test, build, typecheck, runtime check, etc.>

Blocked by:
- <task IDs or "none">
```

Prefer vertical slices that deliver working behavior across the required layers rather than separate "database", "API", and "UI" phases.

Use dependency edges only when a task genuinely cannot begin before another is complete.

Identify opportunities for TDD and the seams at which behavior should be tested.

The plan should include, where applicable:

- repository preparation or prefactoring;
- schema/data changes;
- domain/business logic;
- APIs/contracts;
- UI/client behavior;
- integrations;
- migrations;
- tests;
- documentation required by the feature;
- final integration verification.

Explicitly include:

### Completion Criteria

The feature is not complete until:

- every approved requirement is implemented;
- relevant tests pass;
- typechecking passes;
- lint/build checks required by the repository pass;
- no known requirement from the approved plan remains partial;
- final review finds no unresolved blocking issues.

Present the complete plan to the user.

Ask:

**Is this implementation plan correct?**

Revise until the user explicitly approves it.

## 4. Publish the approved plan

After approval, save the implementation plan to GitHub.

Publish in GitHub Issues, create one GitHub issue containing:

- feature/problem summary;
- link or path to the ADR (make sure the ADR is committed and reachable, so the implementing agent can read it);
- approved implementation plan;
- task IDs;
- blocking relationships;
- completion criteria.

Preserve the task graph and dependency information in the issue.

Use the project's existing labels, milestone, issue conventions, and parent/sub-issue relationships when available. Do not invent project-management conventions when the repository already defines them.

If GitHub cannot be used, stop and explain the blocker rather than silently choosing another tracker.

## 5. Hand off

After publishing the plan, give the user the GitHub issue reference and the ADR path.

Tell the user that implementation is a separate step: start a new coding agent with the `implement-plan` skill and pass it the issue reference.

Do not start implementing in this session. Your work ends here.