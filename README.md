<h1 align="center">Fork Session Agent Skill</h1>

<p align="center">
    <img src="https://img.shields.io/badge/parallel-fork%20a%20session-2d7ff9.svg" alt="Fork a session" />
    <img src="https://img.shields.io/badge/Claude%20Code-native%20fork-7c5cf6.svg" alt="Claude Code" />
    <img src="https://img.shields.io/badge/Orca-optional-6cd4dc.svg" alt="Orca optional" />
    <img src="https://img.shields.io/badge/license-MIT-lightgrey.svg" alt="MIT License" />
    <a href="https://agentskills.io/home">
        <img src="https://img.shields.io/badge/Agent%20Skills-Compatible-purple.svg" alt="Agent Skills Compatible" />
    </a>
</p>

An agent skill that **forks the current session** into a second, independent session. The fork runs in parallel in its own git worktree, and the current session keeps working.

A handoff ends a session and continues the same work. A fork ends nothing: it starts a second agent from what this one knows and gives it one task, such as trying the other approach, writing the tests, or researching a question.

It uses the [Agent Skills](https://agentskills.io/home) format. Claude Code is the primary target, and the fork can be any agent.


## What It Does

- **Claude to Claude: a native fork.** It runs `claude --resume <session-id> --fork-session '<prompt>'`. The fork gets the **whole conversation** under a new session id, and the parent session does not change.
- **To any other agent: a fork brief.** Codex, Grok, Gemini, OpenCode and Cursor cannot read a Claude conversation. The skill writes a short brief to `<repo-root>/_agent/fork/<slug>.md` (Task, Context, Known, Boundaries, Source) and starts that agent on it.
- **Its own worktree by default**, on branch `fork/<slug>` from the current `HEAD`, so two agents never edit the same checkout. Use `--here` for read-only forks.
- **Spawns through [Orca](https://github.com/stablyai/orca)** when the session runs inside an Orca worktree: `orca worktree create`, then `orca terminal create --command`. Without Orca, it prints the commands to paste (`git worktree add ...` and the launch line).
- **Keeps `_agent/` out of git** with `.git/info/exclude`. No tracked file changes.


## Installing

```bash
npx skills add https://github.com/n0an/fork-session-skill --skill fork-session
```

**Claude Code:**

```bash
/plugin install n0an/fork-session-skill
```

**Gemini:**

```bash
gemini extensions install https://github.com/n0an/fork-session-skill.git --consent
```

The `/fork` command already exists in Claude Code, so this skill uses the name `fork-session`.


## Using It

> /fork-session "try replacing the JSON cache with SQLite and benchmark it"

> /fork-session codex "write the missing unit tests for the date parser"

> /fork-session grok --here "review the diff on this branch and list risks"

In natural language:

> Fork this into Codex and have it do the migration script

> Spin off a parallel agent to try the other approach

| Agent      | Launch                                              |
| ---------- | --------------------------------------------------- |
| `claude`   | `claude --resume <id> --fork-session '<prompt>'`    |
| `codex`    | `codex '<prompt>'`                                  |
| `grok`     | `grok '<prompt>'`                                   |
| `gemini`   | `gemini -i '<prompt>'`                              |
| `opencode` | `opencode --prompt '<prompt>'`                      |
| `cursor`   | `cursor-agent '<prompt>'`                           |

For any other agent, the skill asks you for its launch command.


## What It Touches

- With a new worktree: one git worktree and one branch, `fork/<slug>`. With Orca, Orca creates and manages the worktree.
- For a non-Claude fork: one file in `<repo-root>/_agent/fork/`, plus one line, `_agent`, in `.git/info/exclude` if that line is missing.
- Nothing else in the repo changes.


## Pairs With

- [handoff-skill](https://github.com/n0an/handoff-skill): the fork brief links an existing handoff step instead of repeating it. The `_agent/` folder is shared.


## License

Available under the [MIT License](LICENSE).
