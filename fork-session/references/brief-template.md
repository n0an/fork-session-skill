# Fork brief template

Copy the block below into `<repo-root>/_agent/fork/<slug>.md`. Angle-bracket text is a hint:
replace it, do not leave it in. Every section stays, even when it is one line.

---

# Fork: <task in a few words>

*Written <YYYY-MM-DD> for <agent>. Parent session `<SESSION-ID>` on branch `<parent branch>`, `<parent path>`*

## Task

<The one thing to do, and how the fork knows it is done: the test that passes, the file that exists.>

## Context

<The goal of the parent work in one to three sentences. The decisions already made, and why.
The key files by path. Link `_agent/handoff/<GOAL>/` or `_agent/docs/<GOAL>*/README.md` when they exist.>

## Known

### Verified

- <fact> - <what was run or observed>

### Believed

- <claim> - <why it was not checked>

Dead ends: <approaches already tried that failed, and why - so the fork does not repeat them>

## Boundaries

- Work only in `<fork path>` on branch `<branch>`.
- Do not edit `<parent path>` or push to `<parent branch>`.
- Hand back by: <committing on the fork branch | writing `_agent/fork/<slug>-result.md` | ...>

## Source

Parent transcript: `~/.claude/projects/<project-key>/<SESSION-ID>.jsonl` (JSON lines; `grep` it for
details the brief leaves out).
