# ElevenLabs audio pipeline — Deutsch Tracker

Two scripts. Run them in order, locally (not here), from a folder that
also has your `index.html`.

## 1. Extract the text

```
node 1-extract-text.js /path/to/index.html
```

Pulls every unique German string the app can attach audio to — vocab
terms, dialogue lines, and conjugation-card fronts (same detection
logic as the app's own quick-add feature) — and writes `manifest.json`:

```json
[
  { "text": "Guten Morgen", "hash": "a1b2c3...", "filename": "a1b2c3....mp3" },
  ...
]
```

Already run once against your uploaded `index.html`: **398 unique
strings, ~11,500 characters, ~$1.15 estimated cost** — comfortably
inside the $1.50–$2.50 range from before.

Re-run this any time the curriculum content changes; the manifest
just regenerates (hashes are stable for unchanged text, so old audio
files stay valid).

## 2. Generate the audio

```
ELEVENLABS_API_KEY=xxx node 2-generate-audio.js
```

Reads `manifest.json`, calls the ElevenLabs TTS API once per entry,
and writes `audio/<hash>.mp3`. Defaults to voice `HKUbFf0kZdG11lOEhRi2`
(pass a different voice ID as an argument if you want to try another).

- **Safe to re-run** — any file that already exists gets skipped, so
  if it dies partway (network blip, rate limit) just run it again.
- Failures are logged to `failures.json` with the offending text; a
  re-run retries only what's missing.
- Throttled to ~3 requests/sec; tune the `sleep(300)` call if your
  plan allows more.

**Never put the API key directly in a command you paste into chat or
commit to a script** — keep it as an environment variable in your own
shell, or in a local `.env` you `.gitignore`.

## What's next (not built yet)

Once `audio/*.mp3` exists locally, the remaining piece is wiring the
app itself: load `manifest.json` at runtime, add speaker-icon buttons
next to vocab/dialogue/flashcards that look up the matching file and
play it, falling back to the browser's `SpeechSynthesis` if a clip is
missing. Best done once there's real audio to test playback against —
bring the generated `audio/` folder (or just confirm it worked) back
and we'll wire that up next.
