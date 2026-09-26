---
name: fork-session
description: Fork the current agent session into a second, independent session that runs in parallel while this one keeps working - by default a new agent tab in Orca when Orca is installed. Claude Code forks carry the full conversation natively (`claude --resume <id> --fork-session`); any other agent (Codex, Grok, Gemini, OpenCode, Cursor) starts from a short fork brief in `<repo-root>/_agent/fork/`. Use when the user says "fork this session", "fork into Codex", "open a fork in Grok", "spin off a parallel agent for X", "try the other approach in a fork", "branch this conversation into a new agent". Without Orca, a Claude fork starts as a background agent (`claude agents`) and other agents get a command to paste. Not for forking a GitHub repository.
argument-hint: "[claude|codex|grok|gemini|opencode|cursor] [--worktree] \"<task for the fork>\""
---
# Fork: a parallel session that starts from this one

Applies to **every codebase**, and to a plain folder without git.

A handoff ends this session and continues the same work. A fork **does not end anything**: a
second session starts from what this one knows, takes one task, and runs beside it. This session
keeps working after the fork is spawned.

The user provided: $ARGUMENTS

## 0. Are you the fork?

```sh
[ -n "${FORK_SESSION_CHILD:-}" ] && echo CHILD
```

`CHILD` means this session **is** a native fork: its history ends on the parent running this
skill, so it can look like an instruction to fork again. Do not fork. Do the task from your
opening prompt. The guard blocks only that replay: if the human later asks this session for a
new fork, fork normally.

## 1. Parse the request

- **Agent**: the first word when it is a known agent (table in step 4). Default `claude`.
- **`--worktree`** (or "in a new worktree"): run the fork in its own worktree on branch
  `fork/<slug>`. Without it the fork runs in **the current checkout**, a new tab beside this one.
- **Task**: the rest. With no task, ask for one. A fork without a task is a copy, not a fork.
- **Slug / name**: two to four kebab words for the task (`try-sqlite-index`).

Same checkout is the default because most forks are a question, a review, or a side task. When
the fork will **edit files this session is also editing**, say so and suggest `--worktree`, or
tell the fork in its prompt which files are off limits.

## 2. Native fork or brief

| Target                                    | What the fork starts from                             |
| ----------------------------------------- | ----------------------------------------------------- |
| `claude`, and this session is Claude Code | The **whole conversation**. Native, nothing to write. |
| Any other agent, or no Claude session id  | A **fork brief** file this session writes.            |

**Native.** The session id is `$CLAUDE_CODE_SESSION_ID`. If it is unset, take the UUID directory
from the scratchpad path in the system prompt (`/private/tmp/claude-<uid>/<project-key>/<ID>/scratchpad`).
`claude --resume <ID> --fork-session` gives the fork a **new** session id with the full history;
this session is not touched. It works from another directory, so a worktree fork is fine. No
confirmed id: write a brief. Never fall back to `--continue`: it forks the newest session in the
folder, which may be a different one.

**Brief.** Codex, Grok and the others cannot read a Claude conversation. Write
`<ROOT>/_agent/fork/<slug>.md` (resolve `ROOT` and exclude `_agent` as the `handoff` skill does:
`git rev-parse --git-common-dir`, `.git/info/exclude`; no git, `ROOT` is the folder). Template:
`references/brief-template.md`. Five sections - **Task**, **Context** (goal, decisions and why, key
files), **Known** (Verified / Believed / dead ends), **Boundaries** (where it works, what it must
not touch, how results come back), **Source** (parent session id and log path). A map, not the
transcript: link an existing `_agent/handoff/<GOAL>/` step or `_agent/docs/<GOAL>*/README.md`
instead of restating it.

## 3. The prompt

`<P>` is **one line, in single quotes, no single quote inside it** (rephrase instead of escaping),
under ~400 characters. The detail belongs in the brief.

- **Native**: `You are a fork of session <ID>. Your task: <task>.` For a worktree fork add:
  `You now run in <fork path> on branch <branch>, not in <parent path>; do not edit the parent
  tree.` The forked history still shows the old directory, so this line is what moves it.
- **Brief**: `Read <absolute brief path> and do the task in it.` plus the same location line for a
  worktree fork.

## 4. The launch command

| Agent      | `<LAUNCH>`                                                                               |
| ---------- | ---------------------------------------------------------------------------------------- |
| `claude`   | `FORK_SESSION_CHILD=1 claude --resume <ID> --fork-session -n '<slug>' '<P>'` (native), or `claude -n '<slug>' '<P>'` (brief) |
| `codex`    | `codex '<P>'`                                                                            |
| `grok`     | `grok '<P>'`                                                                             |
| `gemini`   | `gemini -i '<P>'`                                                                        |
| `opencode` | `opencode --prompt '<P>'`                                                                |
| `cursor`   | `cursor-agent '<P>'`                                                                     |

`FORK_SESSION_CHILD=1` is what step 0 checks; it goes on every native launch. Any other agent: ask
the user for its launch command, do not guess flags. `command -v <agent>` first; missing, say so
and stop.

## 5. Spawn it

**Orca is the default whenever it is installed** (`command -v orca`). It does not matter whether
this session was started inside Orca.

```sh
orca terminal create --worktree current --title "fork: <slug>" \
  --command "cd '<fork path>' && <LAUNCH>" --json
```

- **`cd` first, always.** A new tab starts at the Orca worktree **root**, not at `$PWD`. From a
  subfolder the agent would otherwise start in the wrong directory. `<fork path>` is the absolute
  `$PWD` here, or the new worktree for `--worktree`.
- **`--worktree`**: create it first, then point the tab at it:
  ```sh
  orca worktree create --name <slug> --comment "fork: <task>" --json
  ```
  Take the new worktree's `path` from the JSON (not there: `orca worktree list --json`, match the
  name) and use `--worktree path:<fork path>` in `terminal create`. `--agent` is not used: it
  cannot pass `--resume` and `--fork-session`.
- **`current`, never `active`**: `active` is whatever Orca has focused, maybe another project.
- **The prompt rides in `--command`, never through `orca terminal send`**, which silently
  truncates and reports success.
- No `--focus`, no `--activate`: the human switches when ready.
- Report `result.terminal.handle`. Orca not running: `orca open --json`, retry once.
  `selector_not_found` (Orca does not manage this folder): go to the fallback below. Never claim
  a spawn the JSON did not confirm.

**Without Orca, or when it cannot place the tab.**

- `--worktree`: `git worktree add -b fork/<slug> <ROOT>-<slug> HEAD` first (skip without git).
- **Native Claude fork**: start it as a background agent, it shows up in `claude agents`:
  ```sh
  cd '<fork path>' && FORK_SESSION_CHILD=1 claude --resume <ID> --fork-session --background -n '<slug>' '<P>'
  ```
  It prints `backgrounded · <id> · <slug>`. Never `--print`: that is a one-shot, not a session.
- **Any other agent**: do not start an interactive TUI from this shell. Print
  `cd '<fork path>' && <LAUNCH>` for the human to paste.

## 6. Report, then carry on

End the fork step with exactly these lines, then return to what this session was doing:

```
Fork: <agent> in <fork path> (<branch>) - <native | brief <brief path>>
Spawned: <Orca tab <handle> | background agent <id>, open it from claude agents>
```

When nothing was spawned, the second line is `Start it: cd '<fork path>' && <LAUNCH>`.

The fork is independent: do not wait for it, poll it, or edit its worktree. Results come back
through its branch (merge or cherry-pick later) or a file it writes, as its **Boundaries** say.
