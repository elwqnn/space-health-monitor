---
name: cleet
description: Clean up after `/fleet`. Delete the fleet sessions, their stale worktrees and their merged branches. Manual only.
disable-model-invocation: true
---
Clean up the fleet of this repo. Keep `.agents/fleet/`: the next fleet uses `SETUP.md` again.
Never delete work that is not merged. If git refuses an operation, keep the item and tell the user why.

## 1. Sessions

Call `list_sessions` with `include_archived: true`. Select the sessions of this repo whose branch contains `fleet-`.
Keep a session if it is working, or if `git merge-base --is-ancestor refs/heads/<branch> HEAD` fails: its work is not merged.
Delete the other sessions with one `delete_session` call. Do not set `force_worktree_cleanup`.
If `list_sessions` is not available, go to step 2.

## 2. Worktrees and branches

Run `git worktree prune`.
For each worktree in `git worktree list` whose path or branch contains `fleet-`, run `git worktree remove <path>`. Skip this checkout and the worktrees of kept sessions. Do not use `--force`.
For each branch in `git branch --list '*fleet-*'`, run `git branch -d <branch>`. Skip the branches of kept sessions. Do not use `-D`.

## 3. Report

Show a table: item, type (session, worktree, branch), status (deleted, kept), reason.
