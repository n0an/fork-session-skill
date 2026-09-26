---
name: fork-session
description: Fork the current agent session into a second, independent session that runs in parallel - in its own git worktree by default - while this session keeps working. Claude Code forks carry the full conversation natively (`claude --resume <id> --fork-session`); any other agent (Codex, Grok, Gemini, OpenCode, Cursor) starts from a short fork brief in `<repo-root>/_agent/fork/`. Use when the user says "fork this session", "fork into Codex", "open a fork in Grok", "spin off a parallel agent for X", "try the other approach in a fork", "branch this conversation into another worktree". Spawns the fork through Orca when this folder is an Orca worktree, and otherwise prints the commands that start it. Not for forking a GitHub repository.
argument-hint: "[claude|codex|grok|gemini|opencode|cursor] [--here] \"<task for the fork>\""
---
# Fork: a parallel session that starts from this one

Applies to **every codebase**, and to a plain folder without git.

A handoff ends this session and continues the same work. A fork **does not end anything**: a
second session starts from what this one knows, takes one task, and runs beside it. This session
keeps working after the fork is spawned.

The user provided: $ARGUMENTS

- **Agent**: the first word when it is a known agent (table below). Default `claude`.
- **`--here`**: run the fork in the current checkout instead of a new worktree.
- **Task**: the rest. With no task, ask for one. A fork without a task is a copy, not a fork.

## Two kinds of fork

| Target                                    | What the fork starts from                          |
| ----------------------------------------- | -------------------------------------------------- |
| `claude`, and this session is Claude Code | The **whole conversation**. Native, nothing to write. |
| Any other agent, or no Claude session id  | A **fork brief** file this session writes.          |

**Native (Claude to Claude).** The session id is the UUID directory in the scratchpad path from
the system prompt, `/private/tmp/claude-<uid>/<project-key>/<SESSION-ID>/scratchpad`. `ls` the log
`~/.claude/projects/<project-key>/<SESSION-ID>.jsonl` to confirm it exists. Then
`claude --resume <SESSION-ID> --fork-session '<prompt>'` gives the fork a **new** session id with
the full history; the parent session is not touched. It works from a different directory, so the
fork can run in a new worktree. Without a confirmed id, write a brief instead. Never invent an id.

**Brief (any other agent).** Codex, Grok and the others cannot read a Claude conversation. Write
`<ROOT>/_agent/fork/<slug>.md` (resolve `ROOT` and exclude `_agent` exactly as the `handoff` skill
does: `git rev-parse --git-common-dir`, `.git/info/exclude`). Short, five sections:

| Section           | Content                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| `## Task`         | The one thing the fork does, and when it is done.                                        |
| `## Context`      | The goal of the parent work, the decisions made, and **why**. Key files by path.           |
| `## Known`        | `### Verified` (what was run or observed) and `### Believed` (not checked). Also dead ends. |
| `## Boundaries`   | Where the fork works, what it must not touch (the parent's tree, its branch), how to hand results back. |
| `## Source`       | Parent session id and its log path, so the fork can `grep` the transcript for details.     |

Template: `references/brief-template.md`. The brief is a map, not the transcript. If a `_agent/handoff/<GOAL>/` step or a
`_agent/docs/<GOAL>*/README.md` exists, link it in **Context** instead of restating it.

## Where the fork runs

- **New worktree (default).** Two agents editing one checkout will collide. A fork gets its own
  branch, based on the current `HEAD` so it sees the parent's work.
- **`--here`**: only when the user asks, or the task is read-only (research, review, a
  question). Say in the prompt that the fork must not edit files the parent is changing.

Slug: two to four kebab words for the task (`try-sqlite-index`). Branch: `fork/<slug>`.

## Launch command per agent

`<P>` is the prompt: **one line, in single quotes, no single quote inside it** (rephrase instead
of escaping). Keep it under ~400 characters; the detail belongs in the brief.

| Agent      | Command                                            |
| ---------- | -------------------------------------------------- |
| `claude`   | `claude --resume <SESSION-ID> --fork-session '<P>'` (native), or `claude '<P>'` (brief) |
| `codex`    | `codex '<P>'`                                      |
| `grok`     | `grok '<P>'`                                       |
| `gemini`   | `gemini -i '<P>'`                                  |
| `opencode` | `opencode --prompt '<P>'`                          |
| `cursor`   | `cursor-agent '<P>'`                               |

Any other agent: ask the user for its launch command. Do not guess flags. Check the binary with
`command -v <agent>` first; when it is missing, say so and stop.

What `<P>` says:

- **Native**: `You are a fork of session <SESSION-ID>. You now run in <fork path> on branch
  <branch>, not in <parent path>; do not edit the parent tree. Your task: <task>.` The forked
  history still shows the old directory, so this line is what moves it.
- **Brief**: `Read <absolute brief path> and do the task in it. You run in <fork path> on branch
  <branch>.`

## Spawning

**With Orca.** Only when `ORCA_TERMINAL_HANDLE` is set. Two commands, so every agent gets the
same launch path (`--agent` cannot pass `--resume` and `--fork-session`):

```sh
orca worktree create --name <slug> --comment "fork of <parent branch>: <task>" --json
orca terminal create --worktree path:<fork path> --title "fork: <slug>" --command "<launch command>" --json
```

- Read `<fork path>` from the first call's JSON (the worktree object's `path`; if it is not
  there, `orca worktree list --json` and match the name). Then write the prompt; it names that path.
- `--here`: skip the first call and use `--worktree current`, never `active` (`active` is
  whatever Orca has focused).
- **The prompt rides in `--command`, never through `orca terminal send`**, which silently
  truncates and reports success.
- No `--focus`, no `--activate`: the human switches when ready.
- Report the handle from `result.terminal.handle`. If the first call fails with
  `selector_not_found` or "not managed", fall through to the manual path. Only if Orca itself is
  not running: `orca open --json` and retry once. Never claim a spawn the JSON did not confirm.

**Without Orca.** Do not start an interactive agent from this shell. Print the commands:

```sh
git worktree add -b fork/<slug> <ROOT>-<slug> HEAD     # skipped for --here and for a plain folder
cd <fork path> && <launch command>
```

## After the spawn

Return to what this session was doing. The fork is independent: do not wait for it, poll it, or
edit its worktree. Results come back through its branch (merge or cherry-pick later) or through a
file it writes, as its **Boundaries** say.

End the fork step with exactly these lines, then continue:

```
Fork: <agent> in <fork path> (branch <branch>) - <native | brief <brief path>>
Spawned: <terminal handle>
```

When nothing was spawned, the second line is instead `Start it: <the command block above>`.
