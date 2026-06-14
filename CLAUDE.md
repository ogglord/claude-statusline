# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

npm package (`@kamranahmedse/claude-statusline`) that installs a bash statusline for Claude Code CLI. Two files are the entire codebase:

- `bin/install.js` — Node.js installer/uninstaller; copies `statusline.sh` to `~/.claude/` and patches `~/.claude/settings.json`
- `bin/statusline.sh` — the statusline script itself; receives Claude Code session JSON on stdin, outputs formatted ANSI status

## Install / Test

No build step. No tests. Test manually:

```bash
# Install locally (uses the local statusline.sh)
node bin/install.js

# Uninstall
node bin/install.js --uninstall

# Smoke-test the statusline with mock input
echo '{"model":{"display_name":"Claude Opus"},"context_window":{"context_window_size":200000,"current_usage":{"input_tokens":50000,"cache_creation_input_tokens":0,"cache_read_input_tokens":0}},"cwd":"/tmp","session":{"start_time":"2025-01-01T00:00:00Z"}}' | bash bin/statusline.sh
```

## Statusline Architecture

`statusline.sh` reads a JSON blob from stdin (Claude Code passes session state here) and prints two sections:

1. **Line 1**: `Model │ Context% │ dir (branch*) │ ⏱ duration │ effort`
2. **Lines 2–3**: Rate limit bars — `current` (5-hour window) and `weekly` (7-day), plus `extra` if extra usage is enabled

**Rate limit data sources** (in priority order):
1. Stdin JSON — `rate_limits.five_hour` / `rate_limits.seven_day` (newer Claude Code versions pass this directly)
2. API fallback — `GET https://api.anthropic.com/api/oauth/usage` with cached response at `/tmp/claude/statusline-usage-cache.json` (60s TTL)

**Token auth chain** (API fallback):
1. `$CLAUDE_CODE_OAUTH_TOKEN` env var
2. macOS Keychain via `security find-generic-password`
3. `~/.claude/.credentials.json`
4. Linux `secret-tool`

**Cross-platform date handling**: `date -j -r` (macOS BSD) with `date -d @epoch` (GNU/Linux) fallbacks throughout `format_epoch_time` and `iso_to_epoch`.

## Publishing

Bump `version` in `package.json`, then:

```bash
npm publish --access public
```
