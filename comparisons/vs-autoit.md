---
title: "eyehands vs AutoIt: two different eras of Windows automation"
description: "AutoIt is a classic Windows automation scripting language. eyehands is a REST API built for AI agents. Which fits which problem."
tags: autoit, windows, automation, claude, ai
canonical_url: https://eyehands.fireal.dev/vs/autoit
published: false
---

# eyehands vs AutoIt

**Short answer:** AutoIt is the grandfather of Windows scripting — BASIC-like syntax, tight integration with native controls, standalone `.exe` compilation, and 25 years of community scripts. eyehands is a modern HTTP server that gives AI agents REST access to the same set of Windows primitives, plus OCR and UI Automation out of the box.

If you're maintaining an existing AutoIt script, or you want a compiled `.exe` that runs anywhere without Python, AutoIt is still a reasonable choice. If you're wiring Claude Code or a local LLM into Windows automation, eyehands is shaped for that job.

## TL;DR table

| | AutoIt v3 | eyehands |
|---|---|---|
| Interface | Custom BASIC-like DSL | HTTP REST API |
| Install | Installer `.exe` | `pip install eyehands` |
| Language | AutoIt v3 | Any HTTP client |
| `ControlSend` / `ControlClick` | ✅ first-class | via UIA |
| Pointer-lock compatibility | ✅ (SendInput mode) | ✅ (default) |
| Per-Monitor DPI v2 | Manual | ✅ automatic |
| UI Automation | Via external UDFs | ✅ first-class |
| OCR | ❌ | ✅ `/find` |
| AI agent integration | ❌ | ✅ (REST + SKILL.md) |
| Standalone `.exe` | ✅ (Aut2Exe) | ❌ (Python) |
| Last major release | 2018 (3.3.14.5) | Active |
| License | Freeware (closed source) | Proprietary ($49 one-time, 14-day trial) |

## Where AutoIt still wins

**`ControlSend` and `ControlClick`.** AutoIt's control-based automation (finding a control by class + ID + text and sending keys to it) is legendary. For automating installer wizards and legacy dialogs, it's still unmatched. eyehands has `/ui/click_element` via UIA which covers a lot of the same ground but with different trade-offs.

**Standalone compilation.** Aut2Exe bundles your script into a `.exe` that runs on any Windows machine without anything pre-installed. eyehands needs Python 3.10+.

**Window matching.** AutoIt's `WinWait`, `WinWaitActive`, `WinWaitClose` are elegant for step-by-step wizard automation.

**Tiny install footprint.** AutoIt's runtime is a few MB. A Python install with the eyehands dependencies is larger.

## Where eyehands wins

**HTTP, not a DSL.** eyehands is called with `curl` or any language's HTTP client. AutoIt needs you to write scripts in its own language. For AI agent tool use, REST is the native shape.

**Active development.** AutoIt's last major release was in 2018. eyehands is actively developed — 1.5.0 was a security hardening pass, 1.6.0 added free-tier telemetry, and there's a Claude Code skill that ships with the package.

**OCR built in.** AutoIt has no OCR; you'd wire up a third-party UDF or call Tesseract out of process. eyehands has `/find?text=...` backed by EasyOCR with frame-level caching.

**UI Automation without UDFs.** AutoIt's UIA support comes through community UDFs (`UIAWrappers.au3`). eyehands exposes UIA directly at `/ui/find`, `/ui/click_element`, and `/ui/tree` — no external modules, no COM dance.

**Modern security.** eyehands has bearer-token auth on every endpoint (except `/ping`), Host-header validation against DNS rebinding, body-size caps, step caps on `/smooth_move` and `/ui/*`. AutoIt's automation model is "whatever your script does runs as you", with no auth boundary.

**Multi-monitor / high-DPI.** eyehands sets Per-Monitor DPI v2 at process startup and normalizes absolute coordinates to the full virtual desktop. AutoIt requires manual `DllCall` for DPI awareness and hand-rolled coord math across monitors.

**Token cost on AI agents.** The packaged `SKILL.md` teaches Claude Code to prefer UIA > OCR > screenshots, which cuts token usage ~4× compared to "just screenshot everything". AutoIt has no equivalent — you'd be writing the agent prompting yourself.

## Where they overlap

Both can:
- Move and click the mouse
- Type keys and unicode text
- Automate native Windows controls
- Work with SendInput-based pointer-lock applications
- Enumerate running windows and processes

## When to use AutoIt

- Maintaining an existing AutoIt codebase
- You need a compiled `.exe` with no runtime dependencies
- Classic installer / wizard automation with `ControlSend` / `ControlClick`
- You don't need AI agent integration and you know AutoIt already

## When to use eyehands

- Connecting Claude Code, Cursor, or local LLMs to a Windows desktop
- OCR-based element detection without third-party libraries
- UI Automation tree walking as a first-class feature
- Any-language REST interface (bash, Node, Go, Rust) to Windows input
- Modern security boundary (bearer tokens, Host-header validation)
- Active maintenance and a clear product roadmap

## Install

```bash
# AutoIt v3
# Download from https://www.autoitscript.com/site/autoit/downloads/

# eyehands
pip install eyehands
eyehands
```

## Links

- **eyehands repo:** https://github.com/shameindemgg/eyehands
- **AutoIt:** https://www.autoitscript.com/

---

*[Open an issue](https://github.com/shameindemgg/eyehands/issues) if anything here misrepresents AutoIt.*
