---
name: context
description: Generate `.agents/CONTEXT.md`, the map of the codebase. Use when `.agents/CONTEXT.md` does not exist.
---
If `.agents/CONTEXT.md` exists, do not overwrite it. Tell me and stop.

Explore the codebase before you write. Read the manifests, the build and test config, the entry points and the existing docs.
Write `.agents/CONTEXT.md` with these sections:
- Purpose: what the project does, in 2 or 3 sentences.
- Stack: languages, frameworks, main dependencies and their versions.
- Commands: the exact commands to install, build, test, run and lint.
- Map: one line per top-level directory or area. Link each area guide that exists.
- Entry points: the files where execution starts.
- Conventions: rules that the code follows but does not state.
Write only facts that you verified in the code. Write a map, not a copy of the code. Max 100 lines.
