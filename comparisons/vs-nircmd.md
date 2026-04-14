---
title: "eyehands vs NirCmd: quick actions vs full automation for AI agents"
description: "NirCmd is a single-purpose CLI utility for Windows actions. eyehands is a full HTTP server with screen capture, input, OCR, and UIA for AI agents."
tags: nircmd, windows, cli, automation, claude, ai
canonical_url: https://eyehands.fireal.dev/vs/nircmd
published: false
---

# eyehands vs NirCmd

**Short answer:** NirCmd is Nir Sofer's venerable Windows command-line utility for "do one specific thing" actions: set volume, eject CD, dial modem, send mouse click at XY. It's a single `.exe` with hundreds of subcommands. eyehands is a full HTTP server that gives AI agents a persistent set of eyes and hands on a Windows desktop.

These are different shapes of tool. NirCmd is great for shell scripts and batch files that need a one-liner Windows action. eyehands is for agents and applications that need to *observe* and *respond* to what's on the screen, in a loop.

## TL;DR table

| | NirCmd | eyehands |
|---|---|---|
| Interface | CLI (single `.exe`) | HTTP REST API |
| Install | Download + place in PATH | `pip install eyehands` |
| Persistent state | ❌ (every call is fresh) | ✅ (frame buffer, OCR cache) |
| Screen capture | `savescreenshot` | `/screenshot`, `/latest`, `/view` |
| OCR | ❌ | ✅ `/find` |
| UI Automation | ❌ | ✅ `/ui/*` |
| Pointer-lock apps | ❌ (mouse_event) | ✅ (SendInput) |
| Mouse / keyboard | ✅ (basic) | ✅ (full SendInput) |
| Audio / volume / CD | ✅ | ❌ |
| Shutdown / registry / clipboard | ✅ | ❌ |
| AI agent integration | ❌ | ✅ (REST, SKILL.md) |
| Speed | Launches per call | Long-running server |
| License | Freeware | Proprietary ($29 one-time, 14-day trial) |

## Where NirCmd wins

**Breadth of one-off actions.** NirCmd has hundreds of subcommands: set system volume, empty recycle bin, change display resolution, call `RegJumpToKey`, drop the clipboard, dial a modem. eyehands focuses on screen + mouse + keyboard + OCR + UIA and doesn't try to cover the rest of the Win32 surface area.

**Zero-dependency shell integration.** NirCmd is one `.exe`. You drop it in `C:\Windows` and it works in any batch file, PowerShell script, or AHK script. eyehands is a running process with a port.

**No server to manage.** NirCmd is "fire and forget". eyehands needs you to start the server, wait for it to boot, and read the bearer token.

## Where eyehands wins

**Observation, not just action.** NirCmd is write-only — it sends clicks and keys but doesn't read the screen. eyehands has a persistent frame buffer updating at 20 fps in the background so every `/find` runs against a fresh cached image, and the OCR result is memoized per frame hash. For "wait for this element to appear" patterns, NirCmd can't help.

**OCR + UIA.** eyehands' `/find` (EasyOCR) and `/ui/find` (Windows UI Automation) return *screen coordinates* for elements identified by text or name. NirCmd has no equivalent — you'd have to screenshot, run OCR externally, and pipe coordinates back in.

**Pointer-lock compatibility.** NirCmd's mouse and key injection goes through the legacy `mouse_event` / `keybd_event` APIs, which silently fail inside pointer-lock applications like games and Parsec. eyehands goes through `SendInput` and the Raw Input pipeline, so it works in those apps.

**Long-running, shared state.** Multiple Claude Code sessions, multiple scripts, and multiple terminals can all hit the same eyehands server. The frame buffer and OCR cache are shared, so the second call is faster than the first. NirCmd starts fresh on every invocation.

**AI agent shape.** Agents like Claude Code need to call a tool, read the result, decide the next action, and loop. A long-running HTTP server is the right shape for that. NirCmd's "launch-per-action" model means re-paying the process startup cost every time.

## Where they overlap

Both can click at an XY coordinate and send a keystroke. That's roughly where the overlap ends. NirCmd's mouse-click support is limited to immediate-mode single actions; it has no concept of "hold Shift while clicking" or "type this unicode string".

## When to use NirCmd

- Quick one-liners in batch files, PowerShell scripts, or AHK scripts
- Windows system actions (volume, display, clipboard, shutdown) outside eyehands' scope
- You don't need to *read* the screen, only to act
- You want a single `.exe` in PATH with no runtime dependencies

## When to use eyehands

- Connecting Claude Code or any AI agent to a Windows desktop
- You need OCR-based element detection or UIA tree walking
- You need pointer-lock compatibility (games, Parsec)
- You need a long-running process with shared frame/OCR cache
- You want a REST API instead of a CLI

## Can you use both?

Yes. NirCmd handles the "set system volume" side and eyehands handles the "click the OK button in this dialog" side. They don't conflict — different tools for different jobs.

## Install

```bash
# NirCmd
# Download from https://www.nirsoft.net/utils/nircmd.html

# eyehands
pip install eyehands
eyehands
```

## Links

- **eyehands repo:** https://github.com/shameindemgg/eyehands
- **NirCmd:** https://www.nirsoft.net/utils/nircmd.html

---

*This comparison focuses on the automation subset of NirCmd's enormous command set. NirCmd does many things eyehands doesn't try to — if your use case is system-level actions rather than screen automation, NirCmd is likely the right tool.*
