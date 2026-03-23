# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Project

All files are standalone HTML — no build step, no dependencies.

Serve locally (required for Safari microphone + CORS):
```
python3 -m http.server 8080
```
Then open `http://localhost:8080/jarvis.html` in Chrome or Safari.

Opening via `file://` will cause Safari to silently deny microphone access and may block Claude API calls in some browsers.

## Architecture

Both files are **single-file HTML apps** — all CSS and JS are inline. No frameworks, no modules, no build tooling. This is intentional; keep new features within the same single-file pattern.

### jarvis.html

**State machine** — a single `state` variable drives everything: `idle | listening | processing | speaking`. All visual changes flow from `hudWrapper.dataset.state` being set via `setState()`, which CSS `[data-state="..."]` selectors respond to automatically. Never update visuals directly; always go through `setState()`.

**DOM refs** — all elements are queried once at the top of the script block into `const` variables. Do not use `document.querySelector` inline elsewhere.

**TTS unlock pattern** — Safari requires `speechSynthesis.speak()` to be called within a user gesture. `unlockTTS()` fires a silent utterance on the first PTT press to satisfy this. A `setInterval` heartbeat calls `speechSynthesis.resume()` every 5s during speech to prevent Safari from silently pausing mid-utterance.

**Demo mode** — when no API key is present (or the API call fails), `askJarvis()` returns a canned response from `DEMO_RESPONSES[]` instead of calling Claude. Errors in `handleInput()` also fall back to demo responses rather than surfacing API error messages.

**Weather** — Nashville weather is fetched from `wttr.in/Nashville,TN?format=j1` at page load and injected into the Claude system prompt as `weatherContext`. JARVIS is instructed to only discuss Nashville weather.

**Claude API** — called directly from the browser using `fetch()`. Requires the header `'anthropic-dangerous-direct-browser-access': 'true'`. API key is stored in `localStorage` under `jarvis_api_key`.

## Git & GitHub

- Remote: `https://github.com/erdnal-ai/jarvis-ai-agent`
- Branch: `main`
- Commit and push after every meaningful change.
