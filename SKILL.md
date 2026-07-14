---
name: "branch-promote"
description: "Inspect branch state and safely promote code between deployment branches. Use when moving changes between development, staging, and production branches, handling branch drift or divergence, or chaining promotion stages."
metadata:
  author: "Leeor Nahum"
  version: "2.3.0"
---

# Branch Promote

Promote code between deployment branches with full situational awareness. Branches play roles (development, staging, production). Identify those roles from the repo rather than assuming fixed names.

## Inspect First

Before anything else, read the full branch state:

- Fetch all remotes and list branches with their tips, authors, and ahead/behind counts relative to each other
- Identify the branch hierarchy the repo uses and the role each branch plays
- Flag anything unusual: a commit landed directly on a production or staging branch, a staging branch is behind development, branches have diverged unexpectedly, parallel work exists on sibling branches, or a branch tip is authored by a collaborator in a way that may block deployment

Report what you find before asking for confirmation. Give the user a clear picture so they can make an informed call.

## Confirm

Use the structured question tool when available. Confirm the source branch, target branch, and how to handle any unusual state you spotted.

If the user indicates they trust your judgment or gives enough context to proceed, skip detailed questions and move forward with well-reasoned defaults. State your assumptions briefly.

## Verify

Run the repo's available checks (lint, typecheck, build) to understand the state of the code before promoting. If checks fail, report and stop. Do not promote unless the user explicitly says to proceed anyway.

Local checks validate the source against locally generated types. They cannot detect a deployed backend whose functions or schema have drifted from the committed code. Confirm the live deployment's contract too (for example, its deployed function signatures or schema), so a passing local build never masks a stale or unpushed deployment.

## Runtime Promotion

A branch promotion includes every durable runtime update required for that target stage to work, not just Git.

Before promoting, identify required non-Git state for the target stage: backend function deployments, database schema changes, generated backend clients, provider config, environment variables, storage buckets, queues, webhooks, scheduled jobs, and deployment platform settings.

If a required runtime update is safe, stage-appropriate, and already authorized by the user's prompt, perform it as part of the promotion. If it affects production, secrets, billing, DNS, data deletion, or irreversible migrations, require explicit approval unless the user already explicitly authorized that exact class of action in the promotion request.

The promotion is not complete until Git, required runtime state, and target-stage checks are all updated or explicitly reported as deferred.

## Promote

Choose the cleanest promotion strategy given the actual state. Prefer fast-forward when branches are aligned. When diverged, explain the options clearly and let the user decide. When there are conflicts, surface what is conflicting rather than resolving silently.

If the user requests the commit to be authored by their configured Git identity (for example, so a deployment service recognizes them as the owner), apply the source tree to the target branch as a normal commit without force-pushing.

## Own The Deployment Through Terminal Status

When the repository defines separate staging and production deployment branches, a promotion is not complete when Git accepts the push. Remain active until every required check and deployment for the exact pushed commit reaches a terminal state.

- Wait for all required staging checks, hosted surfaces, and runtime deployments to succeed before promoting that commit to production
- Match provider results to the exact commit and stage instead of treating an older successful deployment as evidence for the new push
- If a check or deployment fails, inspect its real logs, diagnose the cause, make any in-scope repair already authorized by the promotion request, rerun local validation, and restart from the first affected stage
- Do not leave a known failure for the user to discover through a provider notification
- If repair needs new authority, secrets, billing or DNS changes, irreversible data work, or another scope expansion, stop and report the exact blocker instead of guessing
- After promoting to production, wait again for every required check and deployment, then smoke-test the canonical live origins or health endpoints
- Keep the user informed during long builds, but do not hand back while required deployment state is still pending

Do not invent this staged monitoring loop for a repository with only one main branch or for a repository where branch pushes do not trigger meaningful deployments.

## After

Verify the push succeeded and the target tip matches what was intended. Report any parity concerns across other branches, including any drift or lag introduced by the promotion.
