---
name: fleet
description: Run a list of fixes and features in parallel sessions, one per group, and merge each result into the current branch. Manual only.
disable-model-invocation: true
---
Items: $ARGUMENTS

You are the main session. You group the items, prepare the codebase, start one session per group, merge each result, and update `.agents/CONTEXT.md`.
Do not implement the items yourself. This `/fleet` call is the user's approval for the local merges below.

## 1. Group

Read `.agents/CONTEXT.md` and the code that each item touches.
Put items in the same group when they touch the same files or when one item depends on another. Keep sub-items with their parent.
Two groups must not edit the same file. Prefer fewer groups: each session costs a full context.
Show a table: slug, items, files. Then continue. Do not wait for approval.

## 2. Setup

Run `git status`. If the tree has changes, ask the user to commit or stash them. Sessions start from the last commit, and a merge needs a clean tree.
Read `.agents/fleet/SETUP.md`. If it does not exist, write it. It tells how to make each session compile and test as little as possible:
- `Prepare`: the commands to run once in this checkout before the launch, and what each one makes. Examples: install the dependencies, build the frontend, fill a build cache.
- `Worktree`: the commands that a session runs first in its worktree to reuse these outputs. Write the path of this checkout as `$MAIN`. Link a directory only if no tool writes in it during a build or a test. Look for caches inside it, such as `node_modules/.vite`. Copy the other directories with `cp -r --reflink=auto`.
- `Checks`: one line per area: the paths, and the fastest command that checks a change there.
- `Shared resources`: the ports, databases, containers and fixed temporary paths that the checks use. Give each session its own value from its group number `$N`. This checkout uses `$N` = 0.
- `Main only`: the slow or global checks, such as end-to-end tests and the full build. Sessions skip them. You run them in step 5.
Use the `Commands` section of `.agents/CONTEXT.md` as a start. When you create the file, run each `Prepare` and `Checks` command once. Keep only the commands that work. Max 40 lines.
Run the `Prepare` commands. If a command fails or is out of date, fix `SETUP.md` and tell the user.

## 3. Launch

Call `ListAgents`. Its first line gives the name and the ref of this session.
For each group, call `start_session` with `initiation: "user_asked"`, `context: "fresh"`, `use_worktree: true`, `branch_name: "fleet-<slug>"` and `source_branch: <current branch>`. Use this brief:

````
Items: <the items of the group, verbatim>
Files in scope: <files>. Do not edit the files of the other groups: <files>.
Read `<absolute path of this checkout>/.agents/CONTEXT.md`. Do not create or edit a file in `.agents/`.
First, run: <the Worktree commands>. Use these values: <the Shared resources>. $MAIN and $N are replaced.
Implement the items. Check them with: <the Checks commands of the group areas>. Fix the failures. Do not run other tests or a full build.
If an item is not clear, ask the user in this session. Do not guess.
Commit all changes on your branch. Do not push. Do not merge.
Then call `ListAgents`, find the session with ref `<ref>`, and send it this report with `SendMessage`. End your turn with the same report:
```
FLEET DONE <slug>
branch: <your branch>
checks: <commands>: pass|fail
context: <new facts for CONTEXT.md: commands, areas, conventions> | none
```
````

If a group changes a dependency manifest, replace the `Worktree` commands in its brief with a local install. The shared outputs must not change for the other sessions.
If `start_session` is not available, start one `developer` agent per group with the Agent tool and `isolation: "worktree"`. Give the same brief without the `ListAgents` step. Its final result is the report.

## 4. Merge

Do this for each `FLEET DONE` report as soon as it arrives. Do not wait for the other groups.
A report is not proof. Make sure that `git log --oneline HEAD..<branch>` shows commits.
Run `git merge --no-ff -m "<message>" <branch>` into the current branch. Write the message in the commit convention of the repo, such as `chore: merge <slug>`. Do not push.
If the merge has conflicts that are not obvious to fix, run `git merge --abort`. Tell the session to rebase on `<current branch>` and to send the report again.
Run the `Checks` commands of the group. If they fail, send the failure to the session of this branch and wait for its new report.
Do not run `Prepare` again while a session runs: the sessions use its outputs. If the group changed a dependency manifest, do its `Checks` in step 5.
Add the `context` facts to `.agents/CONTEXT.md`. Keep its sections and its limit of 100 lines. Commit it only if git tracks it.

## 5. Finish

When all groups are merged and no session runs, run `Prepare` again if a group changed a dependency manifest. Then run the `Checks` that step 4 did not do, and the `Main only` checks once.
If a check fails, send the failure to the session of the area that fails.
Show a table: slug, branch, merge status, checks.
Then tell the user that `/cleet` deletes the sessions, their worktrees and their merged branches.
