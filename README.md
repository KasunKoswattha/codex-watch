# codex-watch

A live, readable view of Codex jobs that run in the background.

When you run [Codex](https://github.com/openai/codex) with `codex exec --json`, nothing shows on screen. Every step goes into a log as raw JSON, one long line per event. That's great for scripts and other agents (Claude Code handing work to Codex, for example), but hard to follow as a person.

`codex-watch` follows that log and prints what Codex is doing as it happens:

```
● Codex chat 01a1a0e5-6c26-71a3-bfb0-e76815d6e196
● working

I'll tighten the weather routing, then run the full test suite before pushing.

▶ git status --short --branch
▶ dotnet test tests/Api.Tests --no-restore
  ✗ exit 1
    Failed WeatherRangeTests.MixedQuestion [12 ms]
    Total tests: 412, Failed: 1

The full run found one bad test assertion. I corrected it and I'm rerunning the suite.

● done
```

- **Bold:** Codex's own progress messages
- **Dim:** each command it runs
- **Red:** failed commands, with their last few lines of output
- **Yellow:** files it changed, when Codex reports them as file changes (edits made through shell commands show as commands)
- **Done / failed:** when the turn ends

It only reads the log. It never talks to Codex or changes how it runs.

## Install

Needs `bash` and [`jq`](https://jqlang.org/download/). Works on macOS and Linux.

```bash
git clone https://github.com/KasunKoswattha/codex-watch.git
ln -s "$PWD/codex-watch/codex-watch" ~/.local/bin/codex-watch
```

Any folder on your `PATH` works in place of `~/.local/bin`.

## Usage

Pipe Codex straight into it:

```bash
codex exec --json "fix the failing tests" | codex-watch -
```

Follow a log another process is writing:

```bash
codex exec --json "fix the failing tests" > /tmp/codex-tests.jsonl &
codex-watch /tmp/codex-tests.jsonl
```

Follow whichever job is newest, and switch to the next one when it starts:

```bash
CODEX_WATCH_GLOB="/tmp/codex-*.jsonl" codex-watch
```

Print a finished job once and exit:

```bash
codex-watch --once /tmp/codex-tests.jsonl
```

### Options

| | |
|---|---|
| `codex-watch` | Follow the newest log matching `CODEX_WATCH_GLOB` |
| `codex-watch <file>` | Follow one log |
| `codex-watch -` | Read events from stdin |
| `codex-watch --once [file]` | Print a log once and exit (newest if no file) |
| `codex-watch --help` | Show usage |

| Environment variable | Default | |
|---|---|---|
| `CODEX_WATCH_GLOB` | `/tmp/claude-<uid>/*/*/scratchpad/codex-*.jsonl` | Where to look for logs. Several globs can be separated by spaces. |
| `CODEX_WATCH_INTERVAL` | `3` | Seconds between checks for a newer log |

## With Claude Code

The default glob matches Claude Code's session scratchpads. If you have Claude hand work to Codex, ask it to write the JSON log there with a `codex-` prefix:

```bash
codex exec --json -o "$SCRATCHPAD/codex-task-final.md" "..." > "$SCRATCHPAD/codex-task.jsonl"
```

Then keep `codex-watch` open in another terminal. Every Codex job any Claude session starts shows up there, and Claude still gets its own result when Codex finishes.

## With cmux

In a [cmux](https://github.com/manaflow-ai/cmux) workspace, split a pane (⌘D or ⌘⇧D) and run `codex-watch` in it. You get the agents in one pane and Codex's live work beside them.

## Limits

- View only. You can't send messages to Codex from it.
- Long commands show only their first line. A script passed with a heredoc appears as `python3 - <<'PY'`.
- The event names come from Codex's `--json` output, which may change between Codex versions. Unknown events are skipped, not shown as errors.

## License

[MIT](LICENSE). Not affiliated with OpenAI or Anthropic.
