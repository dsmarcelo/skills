# Issue tracker: GitHub

> Adapted from `setup-matt-pocock-skills/issue-tracker-github.md` in [mattpocock/skills](https://github.com/mattpocock/skills) (MIT).

Plans live as GitHub issues: one **parent** issue for the plan, one **task sub-issue** per task. Use the `gh` CLI for everything. `gh` infers the repo from `git remote -v` when run inside a clone.

## Basics

- **Create**: `gh issue create --title "..." --body-file <file> --label "..."`. Write long bodies to a temp file outside the repo rather than quoting them inline.
- **Read**: `gh issue view <n> --comments`, or `gh issue view <n> --json number,title,body,labels,comments`.
- **Comment**: `gh issue comment <n> --body-file <file>`.
- **Comment and keep its id**: `gh api --method POST repos/<owner>/<repo>/issues/<n>/comments -F body=@<file> --jq .id`.
- **Edit a comment by id**: `gh api --method PATCH repos/<owner>/<repo>/issues/comments/<id> -F body=@<file>`. Do not use `gh issue comment --edit-last`: every agent posts as the same GitHub user, so "last" may be someone else's comment.
- **Labels**: `gh issue edit <n> --add-label "..." --remove-label "..."`.

## Sub-issues

- **Create as a sub-issue**: `gh issue create --parent <parent> ...` (gh 2.94+), or afterwards `gh issue edit <parent> --add-sub-issue <child>`.
- **Older gh**: `gh api --method POST repos/<owner>/<repo>/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>`.
- **Without sub-issues**: put `Part of #<parent>` at the top of the child body.
- **List a parent's sub-issues**: `gh api repos/<owner>/<repo>/issues/<parent>/sub_issues --jq '.[] | {number, title, state, labels: [.labels[].name]}'`.

## Blocking edges

GitHub's native issue dependencies are the visible form.

- **Database id** (not the `#number`): `gh api repos/<owner>/<repo>/issues/<n> --jq .id`.
- **Add an edge** (child is blocked by blocker): `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`.
- **List a task's blockers**: `gh api repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by --jq '.[].number'`.
- **Without dependencies**: use the task template's `Blocked by` section.

Task issues stay open until the pull request merges, so GitHub's "open blockers" count does not tell you whether a blocker is done. A blocker is done when it has the `agent:passed` label.

## Labels

Create any missing labels once per repo (`--force` updates a label that already exists):

```
gh label create ready-for-agent   --color 0E8A16 --description "Fully specified, ready for an agent" --force
gh label create agent:plan        --color 5319E7 --description "Plan parent issue" --force
gh label create agent:in-progress --color FBCA04 --description "Implementer working" --force
gh label create agent:in-review   --color 1D76DB --description "Reviewer checking" --force
gh label create agent:needs-fix   --color D93F0B --description "Review asked for fixes" --force
gh label create agent:passed      --color 0E8A16 --description "Review passed, merged to integration branch" --force
gh label create agent:blocked     --color B60205 --description "Needs a human" --force
```

A task issue has exactly one state label at a time: remove the old label in the same `gh issue edit` call that adds the new one.

## Comment markers

Comments start with a hidden marker so agents can find them:

| Marker | Posted by | On |
|---|---|---|
| `<!-- scout-notes -->` | planner | parent |
| `<!-- orchestrator-status -->` | orchestrator (edited in place) | parent |
| `<!-- implementer-report -->` | implementer | task |
| `<!-- review-verdict -->` | reviewer | task (parent for the final review) |

The latest comment with a given marker is the current one.
