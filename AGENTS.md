# AGENTS.md

Rules for editing the **branch-promote** skill. User-facing guidance lives in `SKILL.md`. `README.md` is the human skim layer.

## File roles

| File | Role |
| --- | --- |
| `SKILL.md` | Inspect, confirm, verify, runtime promotion, promote, deployment ownership through terminal status, temporary branch cleanup, after steps, and the branch report table |
| `README.md` | Short human summary |

## Editing

- Bump `metadata.version` by the release-versioning skill's rules for skills.
- Quote every frontmatter string value. Keys stay unquoted.
- No em dashes, and no semicolons used to join what should be separate sentences. Use commas, periods, parentheses, or "to".
- Capitalized bullets and parallel list voice.
- Keep the skill IDE and stack agnostic. Branch roles are identified from the repo, never assumed from fixed names.

## Before finishing

- `metadata.version` bumped as the release-versioning skill requires.
- `README.md` matches the actual file layout.
