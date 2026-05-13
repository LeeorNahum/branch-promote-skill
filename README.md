# owner-branch-promote-skill

`owner-branch-promote` helps promote a collaborator's branch into `main` through the repository owner's Git identity.

Use it for private web repos where Vercel or GitHub account limits block collaborator-authored deployment commits, but the owner can deploy normally.

The skill confirms the source branch, verifies the incoming changes, runs normal checks, and creates a normal owner-authored target-branch commit whose tree matches the source branch.
