# Browser Integration

## Goal

Маргоша should be able to interact with browsers for web QA, login-assisted workflows, screenshots, page inspection, and browser-based research.

## Current setup

Two layers are used:

1. **Hermes browser tool**
   - Verified by navigating to `https://example.com`.
   - Returned page title: `Example Domain`.
   - Returned accessibility snapshot with the page heading and link.

2. **Chrome DevTools Protocol (CDP)**
   - Local Chrome launched with remote debugging.
   - CDP URL configured in Hermes:

```text
http://127.0.0.1:9222
```

Hermes config key:

```yaml
browser:
  cdp_url: http://127.0.0.1:9222
```

## Current launch command

```bash
mkdir -p ~/.hermes/browser-cdp-profile
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=$HOME/.hermes/browser-cdp-profile \
  --no-first-run \
  --no-default-browser-check \
  about:blank
```

## Verification

Check CDP:

```bash
curl -fsS http://127.0.0.1:9222/json/version
```

Expected: JSON with `Browser`, `Protocol-Version`, and `webSocketDebuggerUrl`.

Check Hermes browser tool:

- Navigate to a simple page.
- Confirm title and accessibility snapshot are returned.

## Persistent service

CDP Chrome is promoted to a user LaunchAgent so browser automation survives agent restarts:

- Plist: `~/Library/LaunchAgents/com.hermes.chrome-cdp.plist`
- Label: `com.hermes.chrome-cdp`
- Program: Google Chrome binary
- Arguments: remote debugging port, dedicated profile, no-first-run
- KeepAlive: true
- Logs: `~/.hermes/logs/chrome-cdp.log` and `~/.hermes/logs/chrome-cdp.error.log`

Service check:

```bash
launchctl print gui/$(id -u)/com.hermes.chrome-cdp
curl -fsS http://127.0.0.1:9222/json/version
```
