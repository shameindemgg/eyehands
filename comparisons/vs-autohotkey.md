---
title: "eyehands vs AutoHotkey: scripting vs AI-driven Windows automation"
description: "AutoHotkey is a 20-year-old Windows scripting language. eyehands is an HTTP server built for AI agents. When to use each."
tags: autohotkey, ahk, windows, automation, claude, ai
canonical_url: https://eyehands.fireal.dev/vs/autohotkey
published: false
---

# eyehands vs AutoHotkey

**Short answer:** AutoHotkey (AHK) is a 20-year-old Windows scripting language with its own syntax, enormous community, and unmatched hotkey/remapping capabilities. eyehands is a local HTTP server that gives AI agents (Claude Code, Cursor, local LLMs) REST access to Windows input and screen capture.

If you want to remap Caps Lock to Escape, build a window switcher, or write a standalone macro that runs in the background, AutoHotkey is still the right tool. If you want Claude Code to click a button in a legacy WinForms app and report back what the screen said, eyehands was built for that.

## TL;DR table

| | AutoHotkey v2 | eyehands |
|---|---|---|
| Interface | Custom scripting language | HTTP REST API |
| Install | Installer `.exe` + script files | `pip install eyehands` |
| Language | AHK v2 DSL | Any language (HTTP) |
| Hotkey / remapping | ✅ best-in-class | ❌ not the use case |
| Persistent background scripts | ✅ | ❌ (server is foreground) |
| Pointer-lock apps (games, Parsec) | Partial (SendInput mode) | ✅ (default) |
| Per-Monitor DPI v2 | Manual (`DllCall`) | ✅ automatic |
| UI Automation / accessibility | Partial (via COM) | ✅ first-class |
| OCR | ❌ (third-party libraries) | ✅ built-in (`/find`) |
| AI agent integration | ❌ | ✅ (REST, SKILL.md) |
| Multi-monitor abs coords | Manual | ✅ `VIRTUALDESK` |
| License | GPL | Proprietary ($49 one-time, 14-day trial) |

## Where AutoHotkey wins

**Hotkey remapping and global keybinds.** Nothing beats AHK for "remap Caps to Esc", "Win+Space opens my launcher", or "Ctrl+Shift+V pastes without formatting". eyehands doesn't do this — it's a server, not a keyboard hook.

**Native window manipulation DSL.** `WinActivate`, `WinMove`, `WinClose`, `ControlClick` — AHK has first-class syntax for these. eyehands has `/ui/click_element` via UIA but no equivalent to AHK's `Send`-plus-window-context pattern.

**Mature community.** AHK has 20+ years of Stack Overflow answers, forums, and snippet libraries. You can find an AHK script for nearly any Windows automation problem.

**Standalone .exe compilation.** AHK scripts can be compiled to single `.exe` files that run without any runtime install. eyehands needs Python.

## Where eyehands wins

**It's an HTTP server, not a language.** If you want Claude Code to automate a Windows app, you don't want to teach it AHK's DSL — you want it to send a JSON body to a REST endpoint. eyehands is the right shape for agent tool use.

**OCR is built in and frame-cached.** AHK has no native OCR; you'd wire up Tesseract or EasyOCR yourself. eyehands ships with `/find?text=...` that returns screen coordinates and caches OCR results per frame hash — the second search on an unchanged screen is instant.

**UI Automation without the COM dance.** AHK can call UIA through COM but the syntax is verbose and error-prone. eyehands wraps it behind `/ui/find`, `/ui/click_element`, and `/ui/tree` — three GET/POST calls that serialize the UIA tree to JSON.

**Packaged Claude Code skill.** eyehands ships a `SKILL.md` that teaches Claude Code when to prefer UIA over OCR over screenshots. There's nothing equivalent for AHK; you'd have to write the agent prompting yourself.

**Capture speed.** AHK's `ImagePutFile` and screenshot routines hit ~10 fps with GDI. eyehands auto-selects BetterCam (DXGI, ~120 fps) or DXcam (~39 fps).

**No new language to learn.** AHK v2 is a lot better than v1, but it's still a DSL. eyehands is "curl a URL" — every developer already knows how to do that.

## Where they overlap

Both tools can:
- Move the mouse and click
- Type keys and text
- Take screenshots
- Detect the active window
- Work with SendInput-compatible pointer-lock applications

If you're comfortable with AHK and don't need AI agent integration, there's no reason to switch. eyehands isn't trying to replace AHK for macro-writing — it's trying to give Claude Code a set of hands.

## When to use AutoHotkey

- Global hotkey remapping, launchers, window switchers
- Persistent background macros that run when you log in
- Standalone `.exe` tools that run on machines without Python
- Complex window manipulation DSL with `WinWait`, `WinActivate`, `ControlClick`
- When you're already an AHK expert and the task doesn't involve an AI agent

## When to use eyehands

- Giving Claude Code / Cursor / a local LLM the ability to operate a Windows desktop
- OCR-based element detection that doesn't require wiring up Tesseract
- UI Automation tree walking without writing COM glue
- Any-language REST access to Windows input (bash, Node, Go, Rust)
- Pointer-lock-safe automation of games, Parsec, remote desktop
- Multi-monitor / high-DPI setups where you want physical-pixel coords

## Can you use both?

Absolutely. A common pattern: AHK handles global hotkeys and launcher keys, and eyehands handles anything an AI agent needs to do during a Claude Code session. They don't conflict — AHK's keyboard hook runs in user mode, and eyehands' SendInput calls go through the Raw Input pipeline. Both can coexist on the same machine without interference.

## Install

```bash
# AutoHotkey v2
# Download from https://www.autohotkey.com/

# eyehands
pip install eyehands
eyehands
```

## Links

- **eyehands repo:** https://github.com/shameindemgg/eyehands
- **AutoHotkey:** https://www.autohotkey.com/

---

*If this comparison is wrong about AutoHotkey, [open an issue](https://github.com/shameindemgg/eyehands/issues).*
