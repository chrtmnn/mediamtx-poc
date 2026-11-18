# AGENTS.md

Guidance for AI coding agents (Claude Code, Copilot, Cursor, Codex, etc.) working in this repository.

## What this is

Proof of concept: receive the PS5's Twitch broadcast on a PC in the home network instead of Twitch, show it in a
window that can be shared in Discord, and record it. No application code, only a Compose file and a MediaMTX
config. Runs under Docker Desktop on the Windows desktop PC.

## Layout

| Path | Description |
| --- | --- |
| `compose.yaml` | One service, `mediamtx`, publishing RTMP on 1935. |
| `mediamtx/mediamtx.yml` | Server config: RTMP only, auth, relay `app/<key>` → `ps5`, recording. Mounted as a directory at `/config`. |
| `.env.template` | Optional overrides, all commented out. |
| `recordings/` | Default recording target. Gitignored. |

## Conventions

- Repository language is English; LF line endings, enforced by `.gitattributes`.
- Pin the image to an exact version (`<version>-ffmpeg`, the relay needs ffmpeg). Never `latest`.
- Never bind-mount a single git-managed file; mount its directory.
- The DNS redirect is not part of this repo: it is one line in the existing Pi-hole (see README).
- The stream key is the real Twitch key. Keep it out of file names, commits and screenshots. The raw
  `app/<key>` path is deliberately not readable from the LAN.
- **Branch names:** New branches get a type prefix (`feat/`, `fix/`, `chore/`, `docs/`, …) followed by a short
  kebab-case name that describes the content, e.g. `fix/relay-restart`. No generic or random names such as
  `claude/<random>`. A short suffix appended by a tool (an agent session id) is fine.

## Testing without a PS5

Publish a test stream the way the PS5 does and read it back:

```bash
ffmpeg -re -f lavfi -i testsrc2=size=1920x1080:rate=60 -f lavfi -i sine \
  -c:v libx264 -preset ultrafast -pix_fmt yuv420p -c:a aac -f flv rtmp://127.0.0.1/app/live_test
ffprobe rtmp://127.0.0.1/ps5
```
