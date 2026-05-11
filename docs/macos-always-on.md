# macOS Always-On Operations

## Goal

Keep the Mac mini awake and keep Hermes reachable through Telegram.

## Current status

A user LaunchAgent runs `caffeinate` continuously:

```bash
/usr/bin/caffeinate -dimsu
```

LaunchAgent path:

```text
~/Library/LaunchAgents/com.hermes.keepawake.plist
```

This prevents idle sleep while the user session is active.

## Screen lock / screensaver

The current user settings disable screensaver locking:

```text
askForPassword = 0
askForPasswordDelay = 0
idleTime = 0
```

## Autologin

Autologin is configured for the local user `minibot`, and FileVault is off. This means macOS can auto-login after reboot.

## Limitations

Some power settings require administrator privileges:

```bash
sudo pmset -a sleep 0 displaysleep 0 disksleep 0
```

Without sudo/root, the user-level `caffeinate` LaunchAgent is the practical persistent solution.

## Verification commands

```bash
launchctl print gui/$(id -u)/com.hermes.keepawake
pmset -g custom
defaults read com.apple.screensaver
defaults -currentHost read com.apple.screensaver
hermes gateway status
```

## Recommended hardening

1. If admin access is available, set system-level `pmset` values.
2. Add a lightweight cron/watchdog that checks:
   - Hermes gateway is running.
   - CDP endpoint responds.
   - `com.hermes.keepawake` is loaded.
3. If any check fails, restart the relevant LaunchAgent and send a Telegram alert.
