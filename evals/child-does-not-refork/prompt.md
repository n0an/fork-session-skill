---
max_turns: 6
timeout_seconds: 240
allowed_tools: [Read, Glob, Grep, Bash, Skill]
tags: [core, guard]
description: FORK_SESSION_CHILD=1 is set in the environment. Tests the re-entry guard.
expected_outcome: The skill's step 0 prints CHILD, no `claude --resume`, `orca` or brief write happens, and Claude answers the task directly.
---

/fork-session "list the three largest files in this folder"
