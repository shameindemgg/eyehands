# eyehands

**50x faster desktop automation for AI agents on Windows.**

eyehands is a local HTTP server that gives any AI agent eyes and hands on Windows. Screen capture in 57ms. Mouse and keyboard via SendInput. OCR text search. Windows UI Automation. All through a simple REST API on localhost.

Claude Computer Use takes 2-4 seconds per screenshot because every action round-trips through the cloud. eyehands stays local — your agent gets its next frame before the LLM finishes generating its next token.

| Operation | eyehands | Claude Computer Use |
|---|---|---|
| Screenshot | 57ms | 2-4s |
| 5-action batch | 67ms | 3-5s |
| Screenshot resolution | 1920x1080 | 1456x816 |
| Typing | Native SendInput | Clipboard paste |
| OCR text search | Built-in | Not available |
| UI Automation | Built-in | Not available |

## Quick Start

1. Install from [PyPI](https://pypi.org/project/eyehands/): `pip install eyehands`
2. Run `eyehands`
3. The server starts on `http://localhost:7331`

14-day free trial, all features included. After that, $29 one-time at [portal.fireal.dev](https://portal.fireal.dev).

Take a screenshot:

```bash
curl http://localhost:7331/screenshot --output screen.jpg
```

Click at coordinates:

```bash
curl -X POST http://localhost:7331/click_at -d "{\"x\":500,\"y\":300}"
```

## Works With Everything

eyehands is agent-agnostic. Anything that can make HTTP calls can drive it.

**Python SDK:**
```bash
pip install eyehands-sdk
```
```python
from eyehands import EyehandsClient
c = EyehandsClient()
c.screenshot("screen.jpg")
c.click_at(500, 300)
```

**MCP server** (Claude Code, Cursor, etc.):
```bash
pip install "eyehands-sdk[mcp]"
eyehands-mcp
```

**OpenAI function calling:**
```python
from eyehands.openai_tools import tools, dispatch
# Pass `tools` to your OpenAI chat call, then dispatch the response
```

**Raw HTTP / any language:**
```bash
curl -X POST http://localhost:7331/type_text -d "{\"text\":\"hello\"}"
curl -X POST http://localhost:7331/batch -d "{\"actions\":[...]}"
```

## What It Does

### Eyes
Screen capture at ~20fps into a rolling buffer. On-demand screenshots in 57ms. Capture backend auto-selects the fastest available: BetterCam (DXGI, ~120fps) > DXcam (~39fps) > mss (GDI fallback). Multi-monitor, window-specific capture, and region crops all work. Coordinates are physical screen pixels with Per-Monitor DPI v2 awareness.

### Hands
Mouse and keyboard through SendInput via ctypes. Goes through the Raw Input pipeline, so pointer-lock apps (games, Parsec, full-screen browsers) see the input correctly. Most Python automation libraries use the deprecated `keybd_event`/`mouse_event` APIs that pointer-lock apps silently ignore.

### Understanding
OCR text search via EasyOCR with frame-level caching. Find text on screen and get its coordinates. Windows UI Automation tree walking to find and interact with native UI elements by name and type. Your agent can click "Save" by name instead of guessing at pixel coordinates.

### Speed
Batch up to 100 actions in a single HTTP call. Frame-hash ETag polling for zero-token screen watching. Cached OCR returns instantly on unchanged frames. Everything stays on localhost.

## API Reference

All POST endpoints accept JSON. All coordinates are physical screen pixels. Bearer token auth on all endpoints (token printed at startup, saved to `.eyehands-token`).

### Screen

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/ping` | Health check, version, license status |
| GET | `/cursor_pos` | Current cursor position `{x, y}` |
| GET | `/screenshot` | On-demand JPEG capture (full screen or region) |
| GET | `/screenshot_b64` | Screenshot as base64 JSON |
| GET | `/latest` | Latest frame from background buffer (JPEG) |
| GET | `/latest_b64` | Latest buffered frame as base64 JSON with metadata |
| GET | `/view` | Live HTML page with auto-refresh and click-through overlay |
| GET | `/find` | OCR text search -- returns screen coordinates of matched text |

### Mouse

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/move` | Relative mouse move `{dx, dy}` |
| POST | `/move_absolute` | Absolute mouse move `{x, y}` |
| POST | `/click` | Click `{button: "left"\|"right"\|"middle"}` |
| POST | `/double_click` | Double click |
| POST | `/mousedown` | Press and hold mouse button |
| POST | `/mouseup` | Release mouse button |
| POST | `/scroll` | Scroll wheel `{dy}` (positive = up) |
| POST | `/smooth_move` | Smooth relative mouse move `{dx, dy, steps, delay_ms}` |
| POST | `/click_at` | Move to coordinates + click `{x, y}` |
| POST | `/click_and_wait` | Click + wait for screen content to change |

### Keyboard

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/key` | Keypress with optional modifiers `{vk, modifiers}` |
| POST | `/type_text` | Type unicode text `{text}` |
| POST | `/type_and_enter` | Type text + press Enter `{text}` |

### Automation

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/batch` | Execute up to 100 actions sequentially |
| POST | `/ui/click_element` | Find a UI element by name/type and click it |
| GET | `/ui/windows` | List all top-level windows |
| GET | `/ui/find` | Search UI Automation elements by name, type, depth |
| GET | `/ui/at` | Get UI element at screen coordinates |
| GET | `/ui/tree` | Get nested UI Automation element tree |

## Installation

### From PyPI

```bash
pip install eyehands
eyehands
```

Requires Python 3.10+ on Windows 10/11.

### Python SDK (for building agents)

```bash
pip install eyehands-sdk              # HTTP client only
pip install "eyehands-sdk[mcp]"       # + MCP server for Claude/Cursor
```

The SDK is a separate package that talks to the server over HTTP. You need the server running too.

## Why Not Claude Computer Use?

Claude Computer Use works out of the box and requires zero setup. Use it when convenience matters more than speed, when you need cross-platform (Mac + Windows), or when you want the built-in security features.

Use eyehands when speed matters. When your agent needs 50 screenshots in the time Computer Use takes one. When you need OCR search or UI Automation. When you're automating games or pointer-lock apps. When you want screen data to stay on your machine. When you're building with OpenAI, local models, or any non-Claude agent.

## Comparisons

- [eyehands vs Claude Computer Use](comparisons/vs-claude-computer-use.md)
- [eyehands vs PyAutoGUI](comparisons/vs-pyautogui.md)
- [eyehands vs AutoHotkey](comparisons/vs-autohotkey.md)
- [eyehands vs AutoIt](comparisons/vs-autoit.md)
- [eyehands vs SikuliX](comparisons/vs-sikulix.md)
- [eyehands vs nut.js](comparisons/vs-nutjs.md)
- [eyehands vs Robot Framework](comparisons/vs-robot-framework.md)
- [eyehands vs NirCmd](comparisons/vs-nircmd.md)
- [eyehands vs just screenshotting](comparisons/vs-screenshots-only.md)

## FAQ

**Does it work on macOS or Linux?**
No. eyehands uses SendInput, Windows UI Automation, and Per-Monitor DPI v2. All Windows-specific. For cross-platform, look at Claude Computer Use or PyAutoGUI.

**Do I need Claude to use eyehands?**
No. It's a plain HTTP server. The SDK ships with a Claude Code skill, an MCP server, and OpenAI adapters, but the server doesn't care who's calling.

**Does it work with games?**
Yes. SendInput goes through the Raw Input pipeline. Games and pointer-lock apps see eyehands input the same way they see real hardware input.

**Does my screen data get sent anywhere?**
No. Capture, OCR, and UI Automation all run locally. The only outbound traffic is license validation at startup and update checks (cached 6 hours).

**Why a local HTTP server instead of a library?**
Works from any language, multiple agents can share one server, and EasyOCR's model load is paid once instead of on every call.

## Pricing

14-day free trial with all features. Then **$29, one-time** at [portal.fireal.dev](https://portal.fireal.dev). No subscription. Per-machine activation.

## Support

- **Issues:** [github.com/shameindemgg/eyehands/issues](https://github.com/shameindemgg/eyehands/issues)
- **Store / License:** [portal.fireal.dev](https://portal.fireal.dev)
- **Email:** fireal6353@gmail.com
