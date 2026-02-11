# codex-session-titles

External terminal-title updater for Codex sessions.

How it works:

- Launch Codex via `bin/codex-title`.
- Wrapper injects a `notify` hook (`-c notify=[...]`).
- On each completed turn, the hook receives Codex JSON context.
- Hook calls `codex exec -m gpt-5.1-codex-mini` to generate a short title.
- Hook writes OSC title escape sequences to the parent TTY.
- Runs once per thread within each `codex-title` session.

No Codex app code changes required.

## Requirements

- `codex` CLI on `PATH`
- `jq` on `PATH`

## Install

```bash
cd /path/to/codex-session-titles
chmod +x bin/codex-title bin/codex-title-notify
```

Optional alias in `~/.zshrc`:

```bash
alias ct='/path/to/codex-session-titles/bin/codex-title'
```

Reload shell:

```bash
source ~/.zshrc
```

## Usage

```bash
./bin/codex-title
# or with prompt
./bin/codex-title "fix flaky test in auth middleware"
```

## Notes

- Generated title is capped to 44 chars (plus `Codex:` prefix).
- Fallback behavior: if `codex exec` returns no title, the hook uses a sanitized/truncated copy of `input-messages[0]`.
- Last generated title is also persisted at:
  - `~/.codex/tmp/codex-session-title.txt`
- Last generator stderr is persisted at:
  - `~/.codex/tmp/codex-session-title.last-generator-stderr.log`
- Last hook error is persisted at:
  - `~/.codex/tmp/codex-session-title.last-error.log`
- The hook has a recursion guard (`CODEX_TITLE_GENERATOR_ACTIVE=1`) and disables nested notify (`-c 'notify=[]'`).
- The wrapper sets `CODEX_TITLE_RUN_ID`; the hook writes per-thread marker files under `~/.codex/tmp/`.
- Direct hook execution without the wrapper is unsupported.

## Quick test (without opening Codex UI)

```bash
payload='{"type":"agent-turn-complete","thread-id":"t1","turn-id":"u1","cwd":"/path/to/project","input-messages":["Investigate flaky CI test around retries"],"last-assistant-message":"Identified race in retry backoff"}'
CODEX_TITLE_RUN_ID=test CODEX_TITLE_TTY=/dev/null ./bin/codex-title-notify "$payload"
cat "$HOME/.codex/tmp/codex-session-title.txt"
```
