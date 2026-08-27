# branch-promote

Agent skill for promoting code between deployment branches.

Gives the agent orientation to inspect branch state, identify branch roles, detect drift and divergence, verify before promoting, handle conflicts or unusual branch topology, own staged deployments through terminal status, and remove temporary branches once their work has landed.

## Install

```bash
git submodule add https://github.com/LeeorNahum/branch-promote-skill.git .agents/skills/branch-promote
```

## Files

- `SKILL.md` - skill guidance loaded by the agent
- `AGENTS.md` - maintenance contract for editing this skill
