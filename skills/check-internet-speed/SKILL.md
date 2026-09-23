---
name: "check-internet-speed"
description: "Run an internet speed test on Ilia's Mac and report download/upload/ping/jitter/packet loss — use when he asks to check internet speed, run a speed test, or troubleshoot connection quality."
---

# Check internet speed

Goal: get a real speed-test reading from Ilia's actual network path (not a sandboxed VM path) and report it in chat: download, upload, ping, jitter, packet loss, and server used.

## Preferred path: Ookla CLI via device_bash

1. `mcp__remote-devices__device_bash`: check `command -v speedtest`.
2. If present, run `speedtest --accept-license --accept-gdpr -f json` and parse:
   - download/upload: bytes/s → Mbit/s (`value * 8 / 1e6`)
   - ping (latency, ms), jitter (ms), packetLoss (%)
   - server name/location
3. Report the numbers immediately. Skip step below.

Only use this path if you're confident `device_bash`'s network reflects the host's real connection (bridged, not an isolated/NAT'd VM with different egress) — if unsure, prefer the native-app path below since that's what actually measures the Mac's real network.

## Fallback: drive the native Speedtest.app in the background

Use this when the CLI isn't installed/available, or when accuracy on the real host network matters more than speed.

1. If not already granted, call `mcp__remote-devices__computer_resolve_access` with `apps: ["Speedtest"]`, then pass the returned entries verbatim to `computer_request_access` and wait for approval.
2. `computer_open_application` / `computer_app_focus` to bring Speedtest.app up (don't take full screen control — use the background `computer_app_*` tools so Ilia isn't interrupted).
3. Locate the Start button (labeled "Начать" in Russian locale, "Go"/"Start" in English) via `computer_app_ax_find` and click it with `computer_app_click`.
4. `computer_wait` ~30-40s for the test to run through ping → download → upload.
5. Read the result numbers via `computer_app_ax_find` (preferred) or `computer_app_screenshot`: download (Mbit/s), upload (Mbit/s), ping (ms), jitter (ms), packet loss (%), and the server name/location shown.

## Reporting

Give Ilia the numbers directly and concisely (he prefers dense, casual, no fluff): download/upload in Mbit/s, ping/jitter in ms, packet loss %, and which server was used. No extra narration about how the test was run unless something failed.