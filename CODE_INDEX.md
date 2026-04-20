# CODE_INDEX

Public-facing docs repo for eyehands. No source code lives here. The server itself (Python + Nuitka-built `.exe`) ships from a private repo and is distributed via [fireal.dev/eyehands/](https://fireal.dev/eyehands/) and GitHub Releases.

## Purpose

- Landing/overview README for the GitHub repo at `shameindemgg/eyehands`
- Comparison pages against other automation tools
- Issue tracker home (users file bugs here against the distributed binary)

## Structure

```
eyehands-public/
├── README.md           # main overview, endpoint table, pricing
├── LICENSE             # license text
├── CODE_INDEX.md       # this file
└── comparisons/        # head-to-head writeups vs other tools
    ├── vs-claude-computer-use.md
    ├── vs-pyautogui.md
    ├── vs-autohotkey.md
    ├── vs-autoit.md
    ├── vs-sikulix.md
    ├── vs-nutjs.md
    ├── vs-robot-framework.md
    ├── vs-nircmd.md
    └── vs-screenshots-only.md
```

## Key files

- `README.md` — pitch, install instructions, full endpoint table, pricing ($49 one-time, 7-day trial). Must stay in sync with the real server's endpoints and the price shown on fireal.dev.
- `comparisons/*.md` — each file is a standalone comparison with one competing tool. Linked from the bottom of README.

## What's NOT here

- Server source (Python, capture backends, OCR, UI Automation wrappers)
- SDK source (`eyehands-sdk` on PyPI)
- Browser extension source (shipped with the `.exe`)
- `openapi.json` — served dynamically by the running server at `GET /openapi.json`, not committed as a static file

## Update triggers

- Price change anywhere (portal, server code) — update README pricing section and trial length
- New endpoint or renamed endpoint on the server — update the endpoint table in README
- New comparison target — add a file in `comparisons/` and link it from README
