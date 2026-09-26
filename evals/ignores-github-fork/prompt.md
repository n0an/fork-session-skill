---
max_turns: 6
timeout_seconds: 240
allowed_tools: [Read, Glob, Grep, Skill]
tags: [negative]
description: A request about forking a repository on GitHub, not an agent session.
expected_outcome: Claude explains how to fork the repository with `gh repo fork` and never invokes the skill.
---

How do I fork the swift-format repo on GitHub and keep my fork in sync with upstream?
