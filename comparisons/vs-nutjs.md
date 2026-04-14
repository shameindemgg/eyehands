---
title: "eyehands vs nut.js: Node.js automation vs language-agnostic REST"
description: "nut.js is a Node.js library for cross-platform desktop automation. eyehands is a Windows-only HTTP server built for AI agents. When to use each."
tags: nutjs, nut-js, node, javascript, windows, automation, claude, ai
canonical_url: https://eyehands.fireal.dev/vs/nutjs
published: false
---

# eyehands vs nut.js

**Short answer:** nut.js is a cross-platform Node.js library — `npm install @nut-tree/nut-js` and you have mouse, keyboard, screen, and image matching in a JavaScript/TypeScript script. eyehands is a Windows-only HTTP server that gives AI agents REST access to the same kind of primitives, plus OCR and Windows UI Automation.

If you're building a Node.js automation script and you need cross-platform, nut.js is the right tool. If you're wiring Claude Code or a local LLM into Windows desktop control, eyehands is shaped for that — and ships with a Claude Code skill file out of the box.

## TL;DR table

| | nut.js | eyehands |
|---|---|---|
| Interface | Node.js library | HTTP REST API |
| Install | `npm install @nut-tree/nut-js` | `pip install eyehands` |
| Language | JavaScript / TypeScript | Any (HTTP) |
| OS support | Windows / macOS / Linux | Windows only |
| Pointer-lock apps | Partial | ✅ `SendInput` |
| Per-Monitor DPI v2 | Partial | ✅ automatic |
| Image matching | ✅ | ❌ (use OCR or UIA) |
| OCR | Optional (Tesseract) | ✅ built-in (EasyOCR) + cached |
| Windows UI Automation | ❌ | ✅ first-class |
| Capture speed | ~20-30 fps | up to ~120 fps (DXGI) |
| AI agent integration | ❌ | ✅ (REST + SKILL.md) |
| License | Apache-2.0 (commercial license available) | Proprietary ($29 one-time, 14-day trial) |

## Where nut.js wins

**Cross-platform.** Works on Windows, macOS, and Linux with the same API. eyehands is Windows-only.

**Node.js-native.** If your existing automation is a Node script or a Playwright suite, adding nut.js is one `npm install`. You don't need to run a separate server process.

**TypeScript types.** nut.js has first-class TypeScript definitions. eyehands' clients are whatever your language's HTTP client looks like.

**Image-based matching.** nut.js has `screen.find(imageResource(...))` for template-matching an image on the screen. Useful for opaque UIs with no accessibility tree. eyehands deliberately doesn't do image matching — it bets on OCR + UIA instead.

**In-process.** No separate server, no port to manage, no bearer token to wire up. If you're running a standalone Node script, this is simpler than an HTTP call.

## Where eyehands wins

**HTTP API, not a library.** If you're wiring Claude Code, Cursor, or a local LLM into desktop automation, a REST API is the natural shape. Agents make HTTP calls and read JSON — that's already their interface to every tool. eyehands slots in natively. nut.js requires you to spawn a Node subprocess from the agent and read stdout, which is awkward.

**Windows UI Automation.** eyehands has first-class UIA support (`/ui/find`, `/ui/click_element`, `/ui/tree`). nut.js has no UIA story — you'd need to wire up a Node binding for UIAutomation.dll yourself, and there isn't a mature one.

**Frame-cached OCR.** eyehands runs EasyOCR against a background frame buffer updated at 20 fps. The second `/find?text=...` on an unchanged screen is instant. nut.js's optional Tesseract integration re-OCRs on every call.

**Pointer-lock compatibility.** eyehands uses `SendInput` directly via ctypes, so games, Parsec, remote desktop, and full-screen browsers see synthetic mouse events. nut.js's platform-specific input paths are less reliable in these environments.

**Capture speed.** eyehands auto-selects BetterCam (DXGI, ~120 fps) or DXcam (~39 fps) with an mss fallback. nut.js's screen capture is ~20-30 fps on Windows.

**AI agent skill file.** eyehands ships with `SKILL.md` — a Claude Code skill that teaches Claude to prefer UIA > OCR > screenshots, cutting token cost ~4× on desktop automation tasks. nut.js has no equivalent.

**Any language.** eyehands is called from bash, Python, Go, Rust, Node, PowerShell — anything with an HTTP client. nut.js is Node-only.

## Where they overlap

Both can:
- Move the mouse and click
- Type keys and text
- Take screenshots
- Find elements on screen (nut.js by image, eyehands by text/UIA)
- Work with multi-monitor setups

## When to use nut.js

- You're building a **Node.js / TypeScript** automation tool
- You need **cross-platform** (macOS + Linux + Windows)
- You need **image template matching** (for opaque UIs)
- Your caller is a standalone script, not an AI agent
- You want **in-process** automation with no server to manage

## When to use eyehands

- You're wiring up **Claude Code, Cursor, or a local LLM** to control Windows
- You want **Windows UI Automation** tree walking out of the box
- You need **frame-cached OCR** that doesn't re-run on every call
- You need **pointer-lock compatibility** (games, Parsec, remote desktop)
- You want **any language** to drive Windows input (bash, Python, Go, Rust, Node)
- You want a **packaged Claude Code skill** that teaches the agent how to use it efficiently

## Can you use both?

Yes. A Node.js backend could call eyehands over HTTP for the Windows-specific features (UIA, high-fps DXGI capture, pointer-lock input) and use nut.js for cross-platform pieces. But if you're Windows-only, eyehands alone covers the full surface with fewer moving parts.

## Install

```bash
# nut.js
npm install @nut-tree/nut-js

# eyehands
pip install eyehands
eyehands
```

## Links

- **eyehands repo:** https://github.com/shameindemgg/eyehands
- **nut.js:** https://nutjs.dev/

---

*[Open an issue](https://github.com/shameindemgg/eyehands/issues) if any of this misrepresents nut.js.*
