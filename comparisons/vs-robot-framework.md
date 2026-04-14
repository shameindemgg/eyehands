---
title: "eyehands vs Robot Framework: BDD test suites vs AI agent control"
description: "Robot Framework is a keyword-driven test automation framework. eyehands is an HTTP server built for AI agent desktop control. Different tools for different jobs."
tags: robot-framework, testing, windows, automation, claude, ai
canonical_url: https://eyehands.fireal.dev/vs/robot-framework
published: false
---

# eyehands vs Robot Framework

**Short answer:** Robot Framework is a keyword-driven, BDD-style test automation framework with dozens of libraries (SeleniumLibrary, AppiumLibrary, RPA.Browser, ImageHorizonLibrary, WhiteLibrary for WinForms). It's aimed at QA teams writing test suites in human-readable DSL. eyehands is a local HTTP server aimed at AI agents that need to click a button in a Windows app *right now*, in a loop, with OCR and UIA.

They're not really competitors — Robot Framework is a test framework, eyehands is an agent tool. But they show up in the same searches, so here's an honest comparison.

## TL;DR table

| | Robot Framework | eyehands |
|---|---|---|
| Shape | Keyword-driven test framework | HTTP REST API |
| Primary audience | QA engineers writing test suites | AI agents + devs controlling Windows |
| DSL | `.robot` files (custom keyword syntax) | JSON over HTTP |
| Test runner / reporter | ✅ (`robot` CLI, XML/HTML reports) | ❌ |
| Parallel execution | ✅ (Pabot) | ❌ |
| Web automation | ✅ (SeleniumLibrary, RPA.Browser) | ❌ |
| Mobile automation | ✅ (AppiumLibrary) | ❌ |
| Windows UIA | Partial (WhiteLibrary, abandoned) | ✅ first-class |
| OCR | Optional (RPA.recognition) | ✅ `/find` built-in + cached |
| Pointer-lock apps | Library-dependent | ✅ `SendInput` |
| AI agent integration | ❌ (batch runner model) | ✅ (REST + SKILL.md) |
| CI integration | ✅ mature | Via curl / HTTP |
| License | Apache-2.0 | Proprietary ($49 one-time, 14-day trial) |

## Where Robot Framework wins

**Test framework ergonomics.** Robot Framework gives you structured test suites with setup/teardown, data-driven tests, parallel execution, XML/HTML reports, and CI integration. If you're running 500 Windows desktop tests nightly and emailing a report to the QA lead, Robot Framework is built for that. eyehands has none of this — it's just a server.

**Library ecosystem.** Robot Framework has libraries for Selenium (web), Appium (mobile), SAP, database, SSH, image matching, OCR, network, REST, and dozens more. If your test has to start a Docker container, send an email, verify a database row, and click a button in a desktop app, Robot Framework can do all of it in one `.robot` file.

**Keyword-driven DSL.** `.robot` files read like natural language:

```robot
*** Test Cases ***
Valid Login
    Open Application    ${APP_PATH}
    Input Text          username_field    admin
    Input Text          password_field    secret
    Click Button        login_button
    Page Should Contain    Welcome
```

For a QA engineer who isn't a full-time developer, this is dramatically easier to write and read than pytest or Selenium Python. eyehands has no DSL — you write JSON bodies in whatever language you're calling from.

**Mature CI integration.** Robot Framework plugs into Jenkins, GitLab CI, Azure DevOps, GitHub Actions with standard XML reports. eyehands works over HTTP from any runner but doesn't emit structured test reports.

## Where eyehands wins

**AI agent shape.** Robot Framework is a batch runner — you write `.robot` files and run them. It's not designed for an agent that takes a tool call, reads the result, and loops. eyehands is exactly that — a long-running HTTP server that an agent can hit repeatedly while it's reasoning about what to do next.

**First-class Windows UI Automation.** Robot Framework's Windows desktop story is fragmented — WhiteLibrary is abandoned, FlaUILibrary exists but is less widely used, and most teams end up writing their own UIA helpers. eyehands wraps UIA at `/ui/find`, `/ui/click_element`, and `/ui/tree` as first-class endpoints.

**Frame-cached OCR.** eyehands' `/find` is OCR with memoization per frame hash. Robot Framework's OCR libraries re-OCR on every call, which adds up in a long test.

**Pointer-lock compatibility.** eyehands works with games, Parsec, remote desktop, and full-screen browsers via `SendInput`. Robot Framework's behavior here depends on which library you're using, and most don't go through Raw Input.

**Lower ceremony.** For "I want Claude to click OK in this dialog", an HTTP call is lighter-weight than writing a `.robot` test case, setting up a suite, and running `robot`.

**Packaged Claude Code skill.** eyehands ships with `SKILL.md` that teaches Claude how to use it efficiently. Robot Framework has no equivalent for AI agents.

## Where they overlap

Both can drive Windows GUI applications. Both can take screenshots and find things on screen. That's roughly where the overlap ends.

## When to use Robot Framework

- You're building a **test automation suite** for QA with structured reports
- You need **web, mobile, database, and API** testing in the same runner
- You want a **keyword-driven DSL** readable by non-developers
- You have **CI/CD pipelines** that expect standard test reports (XML, HTML)
- You need **parallel test execution** and data-driven tests

## When to use eyehands

- You're wiring up **Claude Code, Cursor, or a local LLM** to a Windows desktop
- You need **first-class UI Automation** without wiring up abandoned libraries
- You need **pointer-lock compatibility** for games, Parsec, or remote desktop
- You want **frame-cached OCR** that doesn't re-run on every call
- You want a **long-running server** that agents can hit in a loop

## Can you use both?

Yes, and this is a legitimate pattern. A Robot Framework suite can use HTTP calls (via the `RequestsLibrary`) to hit eyehands for the Windows UI pieces it can't do well, and use SeleniumLibrary / AppiumLibrary / database libs for the rest. You get Robot Framework's test structure and reporting plus eyehands' agent-optimized UIA + OCR + input.

```robot
*** Settings ***
Library    RequestsLibrary

*** Test Cases ***
Click Save Button Via eyehands
    ${response}=    POST    http://127.0.0.1:7331/ui/click_element
    ...             json={"name": "Save"}
    ...             headers={"Authorization": "Bearer ${TOKEN}"}
    Should Be Equal    ${response.json()}[ok]    ${True}
```

## Install

```bash
# Robot Framework
pip install robotframework

# eyehands
pip install eyehands
eyehands
```

## Links

- **eyehands repo:** https://github.com/shameindemgg/eyehands
- **Robot Framework:** https://robotframework.org/

---

*Robot Framework is an excellent tool — it's just solving a different problem. If you're a QA team writing test suites, stick with Robot Framework and call eyehands from it when you need agent-optimized Windows desktop control.*
