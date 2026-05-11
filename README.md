# Margosha Agent Documentation

> Public operator documentation for **Маргоша** — a Hermes Agent instance running for Stanislav on a macOS Mac mini host.

This repository describes how this agent is organized, what capabilities are enabled, and the target architecture for browser control, voice, memory, LLM Wiki, and vector search.

## What this is

Маргоша is a long-running Hermes Agent setup with:

- Telegram gateway for day-to-day interaction.
- macOS host automation and keep-awake service.
- Browser automation through Hermes browser tools plus a Chrome DevTools Protocol endpoint.
- Voice replies through ElevenLabs with fallback to free/local TTS providers.
- Persistent memory and skills.
- Planned knowledge infrastructure combining an LLM Wiki and vector databases.

## Current status

- Browser interaction: enabled and tested against `https://example.com`.
- Chrome CDP endpoint: `http://127.0.0.1:9222` configured in Hermes.
- Voice: ElevenLabs female voice configured; fallback chain configured.
- macOS availability: user-level `caffeinate -dimsu` LaunchAgent keeps the host awake.
- Documentation: this repo.

## Documents

- [Agent profile](docs/agent-profile.md)
- [Browser integration](docs/browser-integration.md)
- [Voice and fallback TTS](docs/voice-and-tts.md)
- [macOS always-on operations](docs/macos-always-on.md)
- [LLM Wiki + vector database architecture](docs/llm-wiki-vector-architecture.md)
- [Operations checklist](docs/operations-checklist.md)

## Security notes

This repository intentionally does **not** contain API keys, bot tokens, credentials, private chat IDs, or raw `.env`/`config.yaml` dumps. Secrets live in the local Hermes environment and should stay out of Git.

Generated: 2026-05-11
