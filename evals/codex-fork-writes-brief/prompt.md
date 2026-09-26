---
max_turns: 12
timeout_seconds: 300
allowed_tools: [Read, Glob, Grep, Bash, Write, Skill]
tags: [core, brief]
description: A git repo, ORCA_TERMINAL_HANDLE unset, target agent Codex. Tests the brief path.
expected_outcome: One file at <root>/_agent/fork/<slug>.md with Task, Context, Known (Verified and Believed), Boundaries and Source; `_agent` is in .git/info/exclude; the printed launch command is `codex '<prompt>'` pointing at the absolute brief path; there is no `--resume` in it.
---

Fork this into Codex: have it write the missing unit tests for the date parser while we keep going on the UI.
