---
title: "eyehands vs SikuliX: image recognition vs text-and-accessibility for AI agents"
description: "SikuliX is image-based automation for any OS. eyehands is text-and-accessibility-based automation built for Windows and AI agents. When to use each."
tags: sikulix, sikuli, windows, automation, ocr, claude, ai
canonical_url: https://eyehands.fireal.dev/vs/sikulix
published: false
---

# eyehands vs SikuliX

**Short answer:** SikuliX automates the screen with computer vision — you take a screenshot of a button and SikuliX finds it by template matching. eyehands automates with OCR text search and the Windows UI Automation tree — you tell Claude "click OK" and it finds the button by name or text, not by pixel template.

Both solve "click this thing on screen" but from very different angles. SikuliX is great when your target has no accessibility metadata (a video game, a Flash-era app, a canvas-heavy web app). eyehands is great when the target *does* have accessibility metadata, or when you want text search instead of image matching — which covers the vast majority of native Windows apps.

## TL;DR table

| | SikuliX | eyehands |
|---|---|---|
| Automation primitive | Image template matching | OCR text + UI Automation tree |
| OS support | Windows / macOS / Linux | Windows only |
| Language | SikuliX IDE (Jython) or Java/Python via API | HTTP REST API (any language) |
| Template maintenance | You save `.png` files per target | ❌ (text / element name lookup) |
| OCR | Optional (Tesseract) | Built-in (EasyOCR) + frame-cached |
| UI Automation | ❌ | ✅ first-class |
| Pointer-lock apps | Partial | ✅ (SendInput) |
| Multi-monitor | Yes | ✅ `VIRTUALDESK` |
| AI agent integration | ❌ (designed for scripts) | ✅ (REST + SKILL.md) |
| License | MIT | Proprietary ($29 one-time, 14-day trial) |

## Where SikuliX wins

**OS coverage.** SikuliX runs on Windows, macOS, and Linux with the same API. eyehands is Windows-only.

**Image-based matching for opaque UIs.** If the app you're automating has no accessibility tree and no readable text — a Flash-era swf, a fully canvas-rendered web app, a bespoke Godot game HUD, a DirectX overlay — SikuliX's image template matching is the right hammer. eyehands can also OCR these, but if the element is an icon with no text, you're stuck taking a screenshot and sending it to a vision model.

**IDE workflow.** SikuliX has a graphical IDE where you capture regions by drawing rectangles on the screen. If your workflow is "point and click to build a script", this is more ergonomic than writing code.

**No dependency on text or accessibility.** SikuliX doesn't care whether a button has text, a label, or a UIA name. It just matches pixels. For reverse-engineering an opaque app, this is the cleanest approach.

## Where eyehands wins

**No template files to maintain.** The hidden cost of SikuliX is that your `.png` templates break on every UI change, every theme change, every DPI change, every OS version bump. If the button's gradient shifts by 3%, your script stops finding it. eyehands' `/find?text=OK` doesn't care about the visual — it finds "OK" wherever OCR sees the text.

**UI Automation tree walking.** Native Windows apps (Notepad, File Explorer, Settings, WinForms, WPF, most Electron apps) expose their control tree through Windows UI Automation. eyehands walks that tree with `/ui/find` and clicks by element name. No images, no OCR, no template matching — just the accessibility name. This is faster, more reliable, and survives theme/DPI/UI changes.

**Frame-cached OCR.** eyehands runs OCR against a cached frame buffer updated at 20 fps in the background. The second `/find?text=OK` on an unchanged screen is instant. SikuliX re-captures and re-matches on every call.

**Pointer-lock compatibility.** Games, Parsec, remote desktop, full-screen browsers — eyehands goes through the Raw Input pipeline via `SendInput`, so pointer-lock apps see synthetic mouse events. SikuliX's Java-based input path is less reliable in these environments.

**AI agent shape.** eyehands is an HTTP server with a packaged Claude Code skill that teaches Claude to prefer UIA > OCR > screenshots. SikuliX is a Jython IDE with a Java API. If you're wiring up Claude Code, one of these is the natural shape and the other isn't.

**No JVM.** SikuliX bundles a JRE. eyehands is Python, which most developers already have.

## Where they overlap

Both can take screenshots, click at coordinates, and type text. Both can find things on screen — SikuliX by image matching, eyehands by text/UIA.

## When to use SikuliX

- You're automating an app with **no accessibility tree and no readable text** (an old video game, a custom Direct2D canvas, a bespoke UI toolkit)
- You need **cross-platform** (macOS + Linux + Windows)
- You prefer a **GUI script builder** over writing code
- You're comfortable maintaining `.png` templates and the pain that comes with them

## When to use eyehands

- You're automating **native Windows apps** (Notepad, File Explorer, Settings, WinForms, WPF, most Electron apps) — UIA will work
- You want **text-based element lookup** that doesn't break on theme changes
- You're wiring up **Claude Code, Cursor, or a local LLM**
- You need **pointer-lock compatibility** for games or Parsec
- You want a **long-running server** with a frame buffer and cached OCR

## Can you use both?

Possible but niche. SikuliX and eyehands operate at different layers — SikuliX is a script runtime, eyehands is an HTTP server. You could have a SikuliX script call eyehands' `/find` via HTTP if you wanted OCR without wiring up Tesseract, but at that point you might as well just use eyehands directly.

## Install

```bash
# SikuliX
# Download JAR from https://sikulix.github.io/

# eyehands
pip install eyehands
eyehands
```

## Links

- **eyehands repo:** https://github.com/shameindemgg/eyehands
- **SikuliX:** https://sikulix.github.io/

---

*Fair disclosure: eyehands' author (me, Fireal) built eyehands because SikuliX's template-matching approach was too fragile for the AI agent use case I was targeting. If your targets are opaque UIs, SikuliX is still the right choice.*
