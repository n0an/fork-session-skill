---
max_turns: 10
timeout_seconds: 300
allowed_tools: [Read, Glob, Grep, Bash, Skill]
tags: [core, fallback, native]
description: A git repo, Claude Code session with a scratchpad path, ORCA_TERMINAL_HANDLE unset. Tests the native fork and the manual fallback.
expected_outcome: No brief file is written, no `orca` or interactive agent command is run, and the reply prints `git worktree add -b fork/<slug> ...` plus `cd <fork path> && claude --resume <SESSION-ID> --fork-session '<prompt>'`, where the prompt names the new path and branch.
---

/fork-session "try replacing the JSON cache with SQLite and benchmark it"
