---
name: owner-branch-promote
description: Reapply a collaborator's branch onto main, dev, or another deployment-sensitive target branch through the repository owner's Git identity. Use when the user explicitly mentions this skill or asks to take changes from a specified branch, verify them, and land them on a target branch so Vercel/GitHub sees the owner as the branch tip author.
disable-model-invocation: true
metadata:
  author: Leeor Nahum
  version: "1.1.0"
---

# Owner Branch Promote

Move a collaborator's branch to a deployment target without letting the collaborator-authored commit be the branch tip that Vercel deploys.

This skill is for private repos where Vercel or GitHub account limits make collaborator commits fail preview or production deployment even though the code is acceptable. It can promote to `main`, repair `dev` previews, or update any explicitly confirmed target branch.

## Non-Negotiables

- Always confirm the source branch with the user before running promotion commands.
- Default target branch is `main`, but confirm if the user did not explicitly name a target branch.
- Never force-push, reset hard, amend, rebase, or rewrite remote history.
- Do not proceed with a dirty working tree unless the user explicitly approves how to handle it.
- Do not bypass hooks or checks unless the user explicitly asks.
- The final pushed target tree must match the confirmed source branch tree.
- The final target branch tip must be authored by the repo owner's configured local Git identity.
- If the target branch itself is blocked because the collaborator authored the latest commit, reapply that same tree onto the target branch with an owner-authored no-force commit.

## Confirmation

Before promotion, ask:

```text
Confirm promotion:
- Source branch: <branch>
- Target branch: <branch, usually main>
- Owner identity: <git user.name> <<git user.email>>

Proceed?
```

Prefer the structured question tool when available.

## Workflow

1. Inspect state:
   - `git status --short --branch`
   - `git fetch origin --prune`
   - `git branch -vv --all`
   - `git log --format='%h %an <%ae> %s' -8`
   - `git config --get user.name` and `git config --get user.email`

2. Confirm with the user:
   - source branch
   - target branch
   - owner identity that should author the production commit

3. Sync and compare:
   - ensure the working tree is clean
   - update the source branch from remote
   - update the target branch from remote
   - show commits in `origin/<target>..origin/<source>`
   - show `git diff --stat origin/<target>...origin/<source>`
   - inspect changed files enough to catch obvious production risks

4. Verify:
   - run the repo's normal checks before promotion
   - for TypeScript web repos, prefer `pnpm lint`, `pnpm typecheck`, and `pnpm build` when available
   - if scripts differ, read package manifests and choose the closest lint/type/build checks
   - stop and report failures; do not promote failing code

5. Promote or repair with owner-authored tip:
   - switch to target branch and pull `--ff-only`
   - record `base=<current target HEAD>`
   - record `source=<origin/source commit>`
   - temporarily set the target tree to the source tree using Git tree/index operations
   - commit once with an owner-authored message such as `Apply <source> updates from owner account.`
   - push target branch normally

   If source and target are the same branch, use the target's current remote tree as `source` and create an owner-authored reapply commit on top of that same branch. This repairs blocked preview deployments without changing the final file tree.

6. Verify after push:
   - `git status --short --branch`
   - `git ls-remote origin refs/heads/<target> refs/heads/<source>`
   - `git diff --quiet origin/<source> origin/<target>` should report no tree diff
   - show the latest target commit author

## Preferred Promotion Command Shape

Use the repo's shell safely, adapting syntax for the user's shell:

```text
git switch <target>
git pull --ff-only origin <target>
git read-tree --reset -u origin/<source>
git commit -m "Apply <source> updates from owner account."
git push origin <target>
```

Only use `git read-tree --reset -u` after a clean-tree check, source/target confirmation, and successful verification. It changes the local worktree and index to exactly match the source tree, then creates a normal non-force commit on the target branch.

For same-branch repair, create a temporary inverse/reapply pair or equivalent normal commit sequence so the final target tree is unchanged but the latest target commit is owner-authored. Never amend or force-push unless the user explicitly asks and understands the remote-history impact.

## Final Response

Report:

- source and target branches
- verification commands and pass/fail result
- pushed commit hash and author
- whether target and source trees match
- any residual deployment or branch-protection risk
