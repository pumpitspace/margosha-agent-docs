# Voice and TTS

## Goal

Маргоша should provide natural voice replies on request, while preserving quota and falling back automatically when paid services fail.

## Primary provider

- Provider: ElevenLabs.
- Voice: female voice `Sarah`.
- Model: `eleven_multilingual_v2`.
- Telegram voice compatibility: verified with OGG/Opus output.

Secrets are stored locally in the Hermes environment file and are not committed to Git.

## Fallback chain

Configured fallback order:

1. `edge` — free Microsoft Edge TTS.
2. `neutts` — local model if installed.
3. `kittentts` — local model if installed.
4. `piper` — local model if installed.

Config shape:

```yaml
tts:
  provider: elevenlabs
  fallback_providers:
    - edge
    - neutts
    - kittentts
    - piper
```

## Behavior

If ElevenLabs fails because of invalid key, quota exhaustion, billing issue, network error, or SDK exception, Hermes attempts the fallback providers in order.

The fallback path was tested by intentionally running TTS with an invalid ElevenLabs key. The system automatically switched to `edge` and produced a Telegram-compatible voice file.

## Voice policy

- Use voice only when the user asks for voice or sends a voice-first interaction that expects voice.
- Keep spoken content concise.
- Do not speak URLs, commands, tokens, long configs, or exact stack traces; leave those in text.
- Russian voice should be female/natural.

## Recommended improvement

Add quota-aware preflight for ElevenLabs:

1. Query `/v1/user/subscription` before long voice generation.
2. If remaining characters are below a threshold, skip ElevenLabs and use `edge`.
3. Store recent quota status in a short-lived cache to avoid unnecessary API calls.
