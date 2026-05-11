# Operations Checklist

## Daily checks

```bash
hermes gateway status
hermes doctor
launchctl print gui/$(id -u)/com.hermes.keepawake
curl -fsS http://127.0.0.1:9222/json/version
```

## Browser recovery

If CDP is down:

```bash
mkdir -p ~/.hermes/browser-cdp-profile
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=$HOME/.hermes/browser-cdp-profile \
  --no-first-run \
  --no-default-browser-check \
  about:blank
```

Then verify:

```bash
curl -fsS http://127.0.0.1:9222/json/version
```

## Voice recovery

If ElevenLabs fails:

1. Confirm the API key is present locally, without printing it.
2. Test subscription endpoint.
3. If quota/key is bad, rely on fallback providers.
4. Verify TTS from Hermes venv.

```bash
cd ~/.hermes/hermes-agent
venv/bin/python - <<'PY'
from pathlib import Path
from tools.tts_tool import text_to_speech_tool
out = str(Path.home()/'.hermes/audio_cache/tts_check.ogg')
print(text_to_speech_tool('Проверка голоса.', output_path=out))
PY
```

## Gateway recovery

```bash
hermes gateway restart
sleep 3
hermes gateway status
```

## Keep-awake recovery

```bash
launchctl kickstart -k gui/$(id -u)/com.hermes.keepawake
launchctl print gui/$(id -u)/com.hermes.keepawake
```

## Documentation update rule

When the live setup changes materially, update this repository and push a new commit.
