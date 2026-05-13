# owner-branch-promote-skill

`owner-branch-promote` helps promote or repair a collaborator's branch into `main`, `dev`, or another deployment-sensitive target branch through the repository owner's Git identity.

Use it for private web repos where Vercel or GitHub account limits block collaborator-authored deployment commits, but the owner can deploy normally.

The skill confirms the source and target branches, verifies the incoming changes, runs normal checks, and creates a normal owner-authored target-branch commit whose tree matches the source branch. It can also repair a blocked same-branch preview by making the branch tip owner-authored without changing the final file tree.

Current version: `1.1.0`.
