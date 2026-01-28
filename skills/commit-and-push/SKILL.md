---
  name: commit-and-push
  description: Add all changes, commit with an appropriate message, and push to remote
  disable-model-invocation: true
  allowed-tools: ["Bash", "Read", "Grep"]
---

# Purpose

Perform a complete git commit and push workflow.

## Instructions

1. Run `git status` to see what has changed
2. Run `git add .` to stage all changes
3. Analyze the staged changes to create a meaningful commit message
4. Run `git commit` with the generated message (or use $ARGUMENTS if provided)
5. Run `git push` to push to the remote repository

Follow the git commit best practices described in your Bash tool instructions.
