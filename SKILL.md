---
name: "branch-promote"
description: "Use when moving changes between development, staging, and production deployment branches, checking branch state before a promotion, handling branch drift or divergence, chaining promotion stages, or removing branches whose work has already landed. Inspects branch state and promotes code between deployment branches safely."
metadata:
  author: "Leeor Nahum"
  version: "2.6.0"
---

# Branch Promote

Promote code between deployment branches with full situational awareness. Branches play roles (development, staging, production). Identify those roles from the repo rather than assuming fixed names.

## Inspect First

Before anything else, read the full branch state:

- Fetch all remotes and list branches with their tips, authors, and ahead/behind counts relative to each other
- Identify the branch hierarchy the repo uses and the role each branch plays
- Flag anything unusual: a commit landed directly on a production or staging branch, a staging branch is behind development, branches have diverged unexpectedly, parallel work exists on sibling branches, or a branch tip is authored by a collaborator in a way that may block deployment
- List every branch that has no role, and say for each whether its tip is already contained in the development branch or still ahead of it

Report what you find before asking for confirmation, as one table with a row per branch, so the user can make an informed call:

| Branch | Role | Ahead/Behind | Action |
| --- | --- | --- | --- |
| <branch> | <development, staging, production, or temporary> | <counts against the next role branch> | <proposed step, and any unusual state flagged> |

## Confirm

Use the structured question tool when available. Confirm the source branch, target branch, and how to handle any unusual state you spotted.

If the user indicates they trust your judgment or gives enough context to proceed, skip detailed questions and move forward with well-reasoned defaults. State your assumptions briefly.

## Verify

Run the repo's available checks (lint, typecheck, build) to understand the state of the code before promoting. If checks fail here, before anything is pushed, report and stop. Do not promote unless the user explicitly says to proceed anyway. The repair loop in Own The Deployment Through Terminal Status applies only to a commit already pushed to a stage.

Local checks validate the source against locally generated types. They cannot detect a deployed backend whose functions or schema have drifted from the committed code. Confirm the live deployment's contract too (for example, its deployed function signatures or schema), so a passing local build never masks a stale or unpushed deployment.

## Runtime Promotion

A branch promotion includes every durable runtime update required for that target stage to work, not just Git.

Before promoting, identify required non-Git state for the target stage: backend function deployments, database schema changes, generated backend clients, provider config, environment variables, storage buckets, queues, webhooks, scheduled jobs, and deployment platform settings.

If a required runtime update is safe, stage-appropriate, and already authorized by the user's prompt, perform it as part of the promotion. If it affects production, secrets, billing, DNS, data deletion, or irreversible migrations, require explicit approval unless the user already explicitly authorized that exact class of action in the promotion request.

The promotion is not complete until Git, required runtime state, and target-stage checks are all updated or explicitly reported as deferred.

## Promote

Choose the cleanest promotion strategy given the actual state. Prefer fast-forward when branches are aligned. When diverged, explain the options clearly and let the user decide. When there are conflicts, surface what is conflicting rather than resolving silently.

If the user requests the commit to be authored by their configured Git identity, apply the source tree to the target branch as a normal commit without force-pushing.

## Own The Deployment Through Terminal Status

When the repository defines separate staging and production deployment branches, a promotion is not complete when Git accepts the push. Remain active until every required check and deployment for the exact pushed commit reaches a terminal state.

- Wait for all required staging checks, hosted surfaces, and runtime deployments to succeed before promoting that commit to production
- Match provider results to the exact commit and stage instead of treating an older successful deployment as evidence for the new push
- If a check or deployment fails, inspect its real logs, diagnose the cause, make any in-scope repair already authorized by the promotion request, rerun local validation, and restart from the first affected stage
- If repair needs an action Runtime Promotion reserves for explicit approval, or any other scope expansion, stop and report the exact blocker instead of guessing
- After promoting to production, wait again for every required check and deployment, then smoke-test the canonical live origins or health endpoints
- Keep the user informed during long builds, but do not hand back while required deployment state is still pending

Do not invent this staged monitoring loop for a repository with only one main branch or for a repository where branch pushes do not trigger meaningful deployments.

## Temporary Branches

Only the role branches persist. Every other branch exists for one piece of work and is temporary unless the repo's agent instructions declare it otherwise, so removing it is part of the promotion that lands its work.

- Delete a landed temporary branch locally and on the remote in the same pass, and prune remote-tracking references
- Confirm before deleting a branch whose tip is ahead of every role branch, has an open pull request, or belongs to someone else. Work in flight is not clutter
- Where the hosting service can delete a branch automatically when its pull request merges, turn that on once per repository

## After

Verify the push succeeded and the target tip matches what was intended. End with the Inspect table updated to the final state: the Action column records what was promoted and deleted, any parity concern across other branches, including drift or lag the promotion introduced, and that no temporary branch was left behind.

When the promotion touched a deployed stage, follow it with a second table, one row per check, deployment, runtime update, and smoke test each stage required:

| Stage | Check or deployment | Result |
| --- | --- | --- |
| <staging, production, or another stage> | <the check, deployment, runtime update, or smoke test, for the exact pushed commit> | <its terminal status, or deferred and why> |
