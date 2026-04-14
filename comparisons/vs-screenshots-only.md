---
title: "eyehands vs just screenshotting with Claude: the token cost argument"
description: "Most 'Claude Code + desktop automation' workflows lean on screenshots. Here's why that's 4× more expensive than eyehands' OCR and UIA path, with real numbers."
tags: claude-code, token-cost, screenshots, automation, ocr, uia
canonical_url: https://eyehands.fireal.dev/vs/screenshots
published: false
---

# eyehands vs "just screenshot everything with Claude"

**Short answer:** You can absolutely wire Claude Code to control a Windows desktop by taking a screenshot, sending it to Claude, asking where the button is, and sending click coordinates back. It works. It's also 3–5× more expensive in tokens than using OCR or UI Automation, and it's less reliable because Claude is guessing coordinates from a vision model pass.

This isn't a comparison with another product — it's a comparison with the "naive" approach, which is what most people do when they first try this. The eyehands `SKILL.md` exists specifically to teach Claude *not* to do this, and my own bills dropped ~4× when I wrote that rule into the skill.

## The naive approach

Here's what "just screenshots" looks like in practice:

1. Claude takes a screenshot (1 image input)
2. Claude analyzes the pixels, identifies "the OK button is around (382, 237)" (part of the completion)
3. Claude sends a click at (382, 237)
4. Claude takes *another* screenshot to verify the click worked (1 more image input)
5. Claude analyzes the new pixels to confirm the dialog closed (more completion)

Per interaction: **2 screenshots in, 2 analysis passes, 1 click**. At current Claude pricing, a single 1920×1080 screenshot is ~1500 input tokens (varies by quality/resolution). Multiply that by the number of interactions in a session and you'll see four-figure token counts for a task a human could do in 15 seconds.

## The eyehands approach

Here's the same interaction with eyehands:

1. Claude calls `/find?text=OK` — returns `{"matches": [{"x": 382, "y": 237, "conf": 0.95}]}`
2. Claude calls `/click_at` with `{"x": 382, "y": 237}`
3. Claude calls `/click_and_wait` which tells it whether the screen changed (`{"changed": true}`)

Per interaction: **zero images, three HTTP calls, JSON responses**. Each call is ~50 tokens of JSON in + ~50 tokens out. The whole interaction is probably ~300 tokens versus ~3000+ for the screenshot-based approach.

## The real numbers from my own usage

After using Claude Code with eyehands for a month on a mix of Windows automation tasks (QA testing a WinForms app, automating a Godot game via Parsec, driving a legacy dashboard), my observed token cost dropped ~4× compared to a screenshot-only Claude Code session doing the same work.

These aren't synthetic benchmarks — they're real bills on real work. The savings come from three places:

1. **Not sending images unless absolutely necessary.** `/ui/click_element?name=OK` uses zero image tokens. `/find?text=OK` uses zero image tokens on the Claude side (the OCR runs locally on your machine). Only screenshots are image-rate.

2. **Caching.** eyehands' frame buffer updates at 20 fps in the background and the OCR layer caches results against the frame hash. The second `/find?text=OK` on an unchanged screen returns the same result without re-running EasyOCR. If Claude calls `/find` five times while reasoning about what to do next, only the first one pays the OCR cost.

3. **Frame-hash polling.** `/latest` returns an `X-Frame-Hash` header. Claude can poll with `If-None-Match: <hash>` and get a 304 when nothing has changed. For "watch a build run and ping me when it's done", this means Claude uses zero image tokens during the waiting state.

## The reliability argument

Screenshots aren't just expensive — they're less reliable.

**Vision models guess coordinates.** When Claude looks at a screenshot and says "the button is at (382, 237)", it's a best guess from a model that's good at this but not perfect. Off-by-10-pixel errors happen, especially on small buttons, dense UI, or high-DPI displays.

**OCR returns exact pixel centers.** `/find` runs EasyOCR and returns the centroid of the bounding box. No guessing — either the OCR found the text or it didn't, and if it did, the coordinates are accurate to the pixel.

**UI Automation returns *programmatic* positions.** `/ui/click_element` asks Windows directly: "where is the button named 'OK'?" Windows tells it the exact center of the control. No vision model, no OCR, no pixels — just the accessibility tree reporting what it already knows.

Reliability order, best to worst: **UIA > OCR > vision model on a screenshot**.

## When screenshots are actually right

Screenshots are the right tool when:

1. **The target has no accessibility tree and no readable text.** Canvas-rendered web apps, DirectX game HUDs, Flash-era software, bespoke UI toolkits. OCR won't find it. UIA won't enumerate it. Vision-model-on-screenshot is the only path.

2. **You need to verify *visual* state.** "Did the progress bar turn green?" is not answerable by UIA or OCR. It's a pixel question.

3. **You're debugging the agent itself.** When Claude is doing something unexpected, a screenshot is the fastest way to understand what state the screen is in.

eyehands' packaged `SKILL.md` codifies this as a rule: **screenshots are the last resort, not the first**. Try `/ui/click_element` first. If that fails (not a native Windows app), try `/find`. If OCR can't find it either, *then* take a screenshot.

## The bottom line

If your Claude Code desktop automation workflow is "screenshot -> vision model -> coordinates -> click", you're leaving money on the table. eyehands has a 14-day free trial with all features, then it's $29 one-time. No tiers, no feature gating. Typical payback on the $29 is one or two medium-sized automation sessions, based on my own bills.

## The code

```bash
# Naive approach
# ... take screenshot, send to Claude, parse response, extract coords ...

# eyehands approach
curl -s "http://127.0.0.1:7331/find?text=OK" \
  -H "Authorization: Bearer $(cat .eyehands-token)"
# {"ok": true, "matches": [{"x": 382, "y": 237, "conf": 0.95}]}

curl -s -X POST "http://127.0.0.1:7331/click_at" \
  -H "Authorization: Bearer $(cat .eyehands-token)" \
  -H "Content-Type: application/json" \
  -d '{"x": 382, "y": 237}'
# {"ok": true}
```

## Install

```bash
pip install eyehands
eyehands --install-skill   # bakes the Claude Code skill into ~/.claude/skills/
eyehands                   # starts the server on http://127.0.0.1:7331
```

The `--install-skill` step is the important one. It writes the SKILL.md with the token path baked in, and once Claude has the skill loaded it'll automatically prefer UIA > OCR > screenshots without you having to remind it.

## Links

- **eyehands repo:** https://github.com/shameindemgg/eyehands
- **eyehands docs:** https://eyehands.fireal.dev

---

*This post exists because "just screenshot it with Claude" is everyone's first instinct, and it's a great 5-minute hack that gets expensive fast. The SKILL.md is the thing that changed my own usage — it's worth installing even during the trial.*
