# AGENTS.md

Rules for editing the **branch-promote** skill. User-facing guidance lives in `SKILL.md`. `README.md` is the human skim layer.

## File roles

| File | Role |
| --- | --- |
| `SKILL.md` | Inspect, confirm, verify, runtime promotion, promote, temporary branch cleanup, and after steps |
| `README.md` | Short human summary |

## Editing

- Bump `metadata.version` with semver in the same change whenever behavior changes: patch for wording, minor for new guidance or a new inspection step, major for a changed promotion strategy or scope.
- Quote every frontmatter string value. Keys stay unquoted.
- No em dashes, and no semicolons used to join what should be separate sentences. Use commas, periods, parentheses, or "to".
- Capitalized bullets and parallel list voice.
- Keep the skill IDE and stack agnostic. Branch roles are identified from the repo, never assumed from fixed names.

## Before finishing

- `metadata.version` bumped if and only if behavior changed.
- `README.md` matches the actual file layout.
