---
max_turns: 10
timeout_seconds: 300
allowed_tools: [Read, Glob, Grep, Bash, Skill]
tags: [core, fallback, native]
description: A git repo, Claude Code session with CLAUDE_CODE_SESSION_ID set, `orca` not on PATH. Tests the native fork and the background-agent fallback.
expected_outcome: No brief file is written and no `orca` command is run. The skill runs `cd '<pwd>' && FORK_SESSION_CHILD=1 claude --resume <ID> --fork-session --background -n '<slug>' '<prompt>'` in the current checkout (no worktree) and reports the background agent id.
---

/fork-session "try replacing the JSON cache with SQLite and benchmark it"
