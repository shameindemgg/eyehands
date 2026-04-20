# eyehands

AI agents on Windows are slow because every screenshot round-trips through the cloud. Claude Computer Use takes 2-4 seconds just to see the screen. eyehands runs locally and does it in 57ms.

Download the Windows .zip from [fireal.dev/eyehands/](https://fireal.dev/eyehands/) or [GitHub Releases](https://github.com/shameindemgg/eyehands/releases), extract, and run:

```
eyehands.exe
```

That's the whole setup. HTTP server on `localhost:7331`, works from any language:

```bash
curl http://localhost:7331/screenshot --output screen.jpg
curl -X POST http://localhost:7331/click_at -d "{\"x\":500,\"y\":300}"
```

Or use the Python SDK (`pip install eyehands-sdk`):

```python
from eyehands import EyehandsClient
c = EyehandsClient()
c.screenshot("screen.jpg")       # 57ms
c.click_at(500, 300)             # instant
c.type_text("hello world")      # native SendInput, not clipboard paste
```

| | eyehands | Claude Computer Use |
|---|---|---|
| Screenshot | 57ms | 2-4s |
| 5-action batch | 67ms | 3-5s |
| Resolution | 1920x1080 | 1456x816 |
| OCR text search | Optional* | No |
| UI Automation | Yes | No |
| Games / pointer-lock | Yes | No |

\* OCR (`/find`) requires ~2GB of EasyOCR/PyTorch dependencies, not bundled in the standalone .exe. Install from source or add easyocr to system Python to enable. All other endpoints work out of the box.

7-day free trial, then $49 one-time at [fireal.dev](https://fireal.dev). No subscription.

## How it works

A background thread captures the screen at ~20fps into a rolling buffer. The capture backend picks the fastest option at startup: BetterCam (DXGI, ~120fps) if available, then DXcam (~39fps), then mss (GDI fallback). Mouse and keyboard go through `SendInput` via ctypes, which hits the Raw Input pipeline. This matters because most Python automation tools use the deprecated `keybd_event`/`mouse_event` APIs, and pointer-lock apps (games, Parsec, full-screen browsers) silently ignore those.

There's also OCR via EasyOCR (results cached per-frame, so repeated lookups on the same screen are free) and Windows UI Automation for finding elements by name and type. Your agent can click "Save" instead of guessing pixel coordinates.

Everything stays on localhost. Screen data doesn't leave your machine unless you send it to a cloud LLM yourself. The only outbound traffic is license validation at startup and an update check cached for 6 hours.

## Integrations

Works with any agent or framework that can make HTTP calls.

```bash
pip install eyehands-sdk              # Python client
pip install "eyehands-sdk[mcp]"       # MCP server for Claude Code / Cursor
```

OpenAI function calling:
```python
from eyehands.openai_tools import tools, dispatch
```

Or skip the SDK and hit the endpoints directly from Go, Rust, Node, whatever.

## Browser extension

eyehands ships with a browser extension (`ext/` folder) that captures your screen at full resolution through the browser. Install it once and any agent gets the same screenshot quality that Claude gets with Claude in Chrome.

Works on Chrome, Edge, Opera (via CDP), and Firefox (via captureVisibleTab). With the server running, open `http://localhost:7331/ext/setup` for guided installation.

When connected, agents use `GET /ext/screenshot` instead of `/screenshot`. If the extension isn't running, it falls back to native capture automatically.

## Endpoints

All coordinates are physical screen pixels (Per-Monitor DPI v2). POST endpoints take JSON. Auth via bearer token (printed at startup, saved to `.eyehands-token`).

| Method | Endpoint | What it does |
|--------|----------|-------------|
| GET | `/ping` | Health check, version, license, extension status |
| GET | `/cursor_pos` | Current cursor `{x, y}` |
| GET | `/screenshot` | JPEG capture, full screen or region |
| GET | `/screenshot_b64` | Same thing, base64 JSON |
| GET | `/latest` | Latest frame from the background buffer |
| GET | `/latest_b64` | Latest frame, base64 JSON + metadata |
| GET | `/view` | Live HTML page with auto-refresh overlay |
| GET | `/find` | OCR text search, returns coordinates |
| GET | `/ext/screenshot` | Full-res screenshot via browser extension (falls back to native) |
| GET | `/ext/screenshot_b64` | Same thing, base64 JSON |
| GET | `/ext/setup` | Browser extension install page (no auth) |
| GET | `/ext/status` | Extension connection status |
| POST | `/move` | Relative mouse move `{dx, dy}` |
| POST | `/move_absolute` | Absolute move `{x, y}` |
| POST | `/click` | Click `{button: "left"\|"right"\|"middle"}` |
| POST | `/double_click` | Double click |
| POST | `/mousedown` | Hold button |
| POST | `/mouseup` | Release button |
| POST | `/scroll` | Scroll wheel `{dy}` |
| POST | `/smooth_move` | Smooth relative move for drags `{dx, dy, steps, delay_ms}` |
| POST | `/click_at` | Move + click in one call `{x, y}` |
| POST | `/click_and_wait` | Click, then wait for the screen to change |
| POST | `/key` | Keypress with modifiers `{vk, modifiers}` |
| POST | `/type_text` | Type unicode text `{text}` |
| POST | `/type_and_enter` | Type + press Enter |
| POST | `/batch` | Up to 100 actions in one call |
| POST | `/ui/click_element` | Find a UI element and click it |
| GET | `/ui/windows` | List top-level windows |
| GET | `/ui/find` | Search elements by name, type, depth |
| GET | `/ui/at` | Element at screen coordinates |
| GET | `/ui/tree` | Nested element tree |
| GET | `/license` | License status |
| POST | `/activate` | Activate license key |
| POST | `/update` | Self-update to latest version |
| GET | `/openapi.json` | OpenAPI spec |

## Limitations

Windows only. eyehands uses SendInput, Windows UI Automation, and Per-Monitor DPI v2, all of which are Win32-specific. No plans for Mac or Linux. If you need cross-platform, Claude Computer Use or PyAutoGUI are your options.

First OCR call is slow (~8 seconds) because EasyOCR loads its model. After that, results are cached per-frame and return instantly when the screen hasn't changed.

The server is single-instance on one machine. Multiple agents can share the same server, but you can't run two copies.

## Computer Use is fine, actually

Claude Computer Use works and requires zero setup. It's the right call when you want cross-platform support, teach mode, or the built-in security features (app masking, tiered permissions). Don't switch to eyehands just because it exists.

Switch when the 2-4 second screenshot latency is killing your agent's throughput. When you need OCR or UI Automation. When you're automating something with pointer-lock. When you're building with OpenAI or local models. When you care about screen data staying local. Some people use both: Computer Use for teach mode, eyehands for the fast loops.

[Detailed comparison](comparisons/vs-claude-computer-use.md)

## More comparisons

[vs PyAutoGUI](comparisons/vs-pyautogui.md) · [vs AutoHotkey](comparisons/vs-autohotkey.md) · [vs AutoIt](comparisons/vs-autoit.md) · [vs SikuliX](comparisons/vs-sikulix.md) · [vs nut.js](comparisons/vs-nutjs.md) · [vs Robot Framework](comparisons/vs-robot-framework.md) · [vs NirCmd](comparisons/vs-nircmd.md) · [vs just screenshotting](comparisons/vs-screenshots-only.md)

## Pricing

7-day trial, all features. Then **$49 one-time** at [fireal.dev](https://fireal.dev). Per-machine activation, no subscription.

[Issues](https://github.com/shameindemgg/eyehands/issues) · [fireal.dev](https://fireal.dev) · fireal6353@gmail.com
