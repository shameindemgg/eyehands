---
title: "eyehands vs PyAutoGUI: which should you use for Windows automation?"
description: "A candid comparison of eyehands and PyAutoGUI for desktop automation on Windows — covering pointer-lock compatibility, DPI awareness, UI Automation support, and AI agent integration."
tags: pyautogui, windows, automation, claude, ai
canonical_url: https://eyehands.fireal.dev/vs/pyautogui
published: false
---

# eyehands vs PyAutoGUI

**Short answer:** PyAutoGUI is a battle-tested Python library for scripts that *you* write and run. eyehands is a local HTTP server designed for AI agents (Claude Code, Cursor, local LLMs) to control a Windows desktop via REST. They solve overlapping problems but optimize for very different callers.

If you're writing a one-off Python script to fill in a form, PyAutoGUI is probably what you want — it's one `import` and a pip install. If you're giving Claude Code the ability to click a button in Notepad, or operating a native Windows app from a chat session, eyehands was built for that.

## TL;DR table

| | PyAutoGUI | eyehands |
|---|---|---|
| Interface | Python library | Local HTTP server (REST API) |
| Language | Python only | Any language that can make HTTP calls |
| Windows input API | `mouse_event` / `keybd_event` (deprecated) | `SendInput` (Raw Input pipeline) |
| Pointer-lock apps (games, Parsec) | ❌ doesn't work | ✅ works |
| Per-Monitor DPI v2 | ❌ partial | ✅ full |
| Multi-monitor absolute coords | Manual | `VIRTUALDESK` normalized |
| OCR | ❌ (pytesseract extension) | ✅ built-in (`/find` via EasyOCR) |
| UI Automation (UIA tree walking) | ❌ | ✅ (`/ui/*` endpoints) |
| Frame buffer / screen caching | ❌ | ✅ (20 fps background capture) |
| AI agent skill file | ❌ | ✅ (`SKILL.md` for Claude Code) |
| Capture backend | PIL `ImageGrab` | DXGI (BetterCam/DXcam) + mss fallback |
| Capture speed | ~5 fps | up to ~120 fps |
| Auth | N/A (in-process) | Bearer token on all endpoints |
| License | BSD | Proprietary ($49 one-time, 14-day trial) |

## Where PyAutoGUI wins

**You already have a Python script.** PyAutoGUI is one `pip install` and one import. No server, no port, no token. For a 50-line script that renames files in Explorer, spinning up a separate HTTP process is overkill.

**Cross-platform.** PyAutoGUI works on macOS and Linux too. eyehands is Windows-only.

**Enormous community.** 10 years of Stack Overflow answers, a wide ecosystem of tutorials, and integration with other Python automation libs.

**Fail-safe.** PyAutoGUI has a "move to top-left to abort" safety mechanism baked in. eyehands has bearer-token auth and Host-header validation but no mouse-position panic button.

## Where eyehands wins

**Pointer-lock compatibility.** Games, Parsec remote desktop, full-screen browsers, and any app that uses Raw Input will silently ignore PyAutoGUI's mouse events. They never see them — `mouse_event` injects into a legacy message queue that pointer-lock consumers don't subscribe to. eyehands uses `SendInput`, which goes through the same Raw Input pipeline that real hardware does. If you've been trying to automate a Godot game or drive a Parsec session from a script, this is the reason it hasn't been working.

**Per-Monitor DPI v2.** Plug a 4K laptop into a 1080p external and run a PyAutoGUI script — you'll get mis-aligned clicks because Windows silently scales coordinates behind the process. eyehands sets `SetProcessDpiAwarenessContext(Per-Monitor v2)` at startup so every coordinate is a physical screen pixel, and survives a mid-session DPI change (like unplugging a monitor).

**UI Automation.** Native Windows apps (Notepad, File Explorer, Settings, most WinForms/WPF tools) expose an accessibility tree. eyehands has `/ui/find` and `/ui/click_element` that let you click a button by name — no pixel math, no OCR, no screenshots. PyAutoGUI has no equivalent; you'd have to pip install `uiautomation` separately and wire the two together.

**Frame-cached OCR.** `/find?text=OK` runs EasyOCR against the most recent cached frame and returns pixel coordinates. The second `/find` on an unchanged screen is instant (the result is cached against `frame_hash()`). PyAutoGUI can locate *images* on the screen via pixel matching, but not text. For text, you'd need pytesseract, and neither is cached across calls.

**AI agent ergonomics.** The headline use case for eyehands is Claude Code or another LLM agent that needs to "click the OK button" in a loop. An HTTP server is the natural shape here: the server boots once, multiple agents can hit it, the bearer token is saved to disk so Claude can read it with one line of shell, and there's a packaged `SKILL.md` that teaches Claude to prefer UIA > OCR > pixels (which cuts token cost ~4× on desktop tasks).

**Capture speed.** PyAutoGUI uses PIL's `ImageGrab` under the hood, which is ~5 fps on most hardware. eyehands auto-selects BetterCam (DXGI, ~120 fps) or DXcam (~39 fps) with an mss GDI fallback. If you're watching for a UI change, this matters a lot.

## When to use PyAutoGUI

- You're writing a **standalone Python script**, not wiring up an AI agent
- You need **cross-platform** (macOS + Linux + Windows)
- The app you're automating isn't pointer-lock-aware and you're on a single monitor at 100% scale
- You want the smallest possible install footprint

## When to use eyehands

- You're giving **Claude Code, Cursor, or a local LLM** the ability to operate a Windows desktop
- You need to automate a **game, a Parsec session**, or any app that uses pointer-lock
- You're on a **multi-monitor / high-DPI** setup and tired of coordinate math
- You want **UI Automation** without separately wiring `uiautomation`
- You want **OCR that's cached across calls** without re-loading EasyOCR on every invocation
- You want any language (bash, Node, Go, Rust) to drive Windows input, not just Python

## Can you use both?

Yes. eyehands' server and PyAutoGUI's in-process API coexist fine — they're both just wrappers over Win32 input. Some users run PyAutoGUI in their own Python automation scripts and reach for eyehands when Claude Code needs to take the wheel. The `SendInput`-based path in eyehands is a strict superset of what PyAutoGUI can do on Windows, though, so if you're standardizing on one, eyehands is the more capable choice for Windows-specific work.

## Install

```bash
# PyAutoGUI
pip install pyautogui

# eyehands
pip install eyehands
eyehands                  # starts HTTP server on http://127.0.0.1:7331
```

## Links

- **eyehands repo:** https://github.com/shameindemgg/eyehands
- **eyehands docs:** https://eyehands.fireal.dev
- **PyAutoGUI docs:** https://pyautogui.readthedocs.io/

---

*This comparison is maintained by eyehands. If anything here is wrong about PyAutoGUI, [open an issue](https://github.com/shameindemgg/eyehands/issues) and we'll fix it.*
