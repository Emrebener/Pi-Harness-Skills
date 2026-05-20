<img width="999" height="533" alt="1_2NjPwcujUP4RBuF1XJ39tw" src="https://github.com/user-attachments/assets/7f32cec4-f413-42f1-8433-50b269e3683a" />

# Pi-Harness-Skills

Two skills for AI coding agents that can't run MCP servers, like
[Pi](https://pi.dev). Both are self-contained CLI tools designed to be
dropped into a skills directory and invoked by an agent as ordinary shell
commands.

- [`browser-tools`](browser-tools/): Chromium browser automation and
  debugging over the Chrome DevTools Protocol. An MCP-free alternative to
  Playwright MCP and Chrome DevTools MCP.
- [`web-search`](web-search/): Web search via a self-hosted
  [SearXNG](https://docs.searxng.org/) instance. An MCP-free alternative to
  search MCP servers.

Both work in any harness that can run a Node.js script (Pi, Claude Code,
Codex, your own loop). They were built primarily for Pi, which doesn't
support MCP at all, but nothing about them is Pi-specific.

---

## browser-tools

23 small CLI scripts wrapping a bundled puppeteer. Connects to Chromium on
`:9222` over CDP. No MCP server, no always-on daemon, no system browser
required.

### What it covers

| Capability | Tool |
|---|---|
| Launch / quit Chromium | `browser-start.js`, `browser-sessions.js` |
| Navigate, tabs | `browser-nav.js`, `browser-tabs.js` |
| Accessibility snapshot with `@eN` refs | `browser-snapshot.js` |
| Click, type, hover, select, drag, scroll, key | `browser-click.js`, `browser-type.js`, `browser-hover.js`, `browser-select.js`, `browser-drag.js`, `browser-scroll.js`, `browser-key.js` |
| Upload files | `browser-upload.js` |
| Handle native dialogs (`alert`/`confirm`/`prompt`) | `browser-dialog.js` |
| Wait for conditions (visible, gone, text, idle, navigation) | `browser-wait.js` |
| Screenshot, readable page content, cookies | `browser-screenshot.js`, `browser-content.js`, `browser-cookies.js` |
| Console & network capture, performance trace | `browser-monitor.js`, `browser-console.js`, `browser-network.js`, `browser-trace.js` |
| Run arbitrary JavaScript in the page | `browser-eval.js` |

Full command reference and usage patterns live in
[`browser-tools/SKILL.md`](browser-tools/SKILL.md).

### Highlight features

**Accessibility snapshot with stable refs.** `browser-snapshot.js` returns a
compact accessibility tree where every interactive element gets a stable
`[ref=eN]` id. Every interaction tool accepts `@eN` in place of a CSS
selector. The recommended loop is *snapshot → act on refs → re-snapshot*.
The agent doesn't have to guess selectors.

```
$ browser-snapshot.js
button "Sign Up" [ref=e5]
textbox "Email" [ref=e3]

$ browser-type.js @e3 "user@example.com"
$ browser-click.js @e5
```

**Named multi-sessions.** Every tool operates on a named session. Each
session is its own browser instance with its own port, profile, and logs.
Independent tasks run browsers in parallel without colliding. Use
`--session NAME` per call, or export `BROWSER_SESSION=NAME` once and forget
about it.

**Actionability waits.** Click, type, hover, select, and friends wait for
the target element to be visible, enabled, and stable before acting. No
flaky clicks on a button still mid-animation.

**Console & network capture as a background daemon.** CDP only delivers
live events, no replay. Start `browser-monitor.js` before reproducing an
issue, then read `browser-console.js --errors` and `browser-network.js
--failed` to see exactly what blew up. Per-session log files.

**Native dialog handling without races.** `browser-dialog.js` writes a
marker file the moment its handler is attached, so the bash caller can
arm the handler, wait for the marker, *then* trigger the dialog. No
flaky `sleep` guesses.

**Smart drag-and-drop.** `browser-drag.js` detects whether the source uses
native HTML5 drag (`draggable="true"`) or expects mouse-gesture drag, and
uses the right mechanism for each.

**Bundled Chromium, cross-platform.** `npm install` downloads a private
Chromium build (~150 MB, one time). No system browser needed. Works on
Linux, macOS, and Windows. A `package.json` engines guard refuses install
on Node versions with the known puppeteer extraction bug.

### Why a skill instead of an MCP server

| | Playwright MCP / Chrome DevTools MCP | `browser-tools` skill |
|---|---|---|
| **Context cost when idle** | 20+ tool definitions in every conversation | Zero. Loads only when triggered. |
| **Runs in Pi** | No (Pi doesn't support MCP) | Yes |
| **Transparency** | Black-box server protocol | 23 readable scripts you can edit |
| **Cross-browser engines** | Yes (Playwright) | Chromium only |
| **Trace viewer / codegen** | Yes (Playwright) | No |
| **Network mocking** | Full | None |

The trade is real. Playwright MCP is the right tool for QA engineers
authoring full test suites. `browser-tools` is the right tool for an agent
that needs to drive a browser as part of a task, in a harness that won't
or can't load an MCP server.

### Quickstart

```bash
cd browser-tools
npm install                       # ~150 MB Chromium download, one time
./browser-start.js                # launches Chromium on :9222
./browser-nav.js https://example.com
./browser-snapshot.js
./browser-screenshot.js
```

Then point your agent's skills directory at this folder and the agent can
call the scripts directly. Node 20, 22, or 24 LTS. Not Node 26 (known
puppeteer extraction bug; the install will refuse).

---

## web-search

A single dependency-free CLI that queries a self-hosted SearXNG instance's
JSON API and returns ranked results.

```bash
./search.js "claude code skills"
./search.js "claude code" --count 5 --time week
./search.js "openai release notes" --category news --json
```

Filters by count, category (general / news / images / videos), time range
(day / week / month / year), language, and safe-search level. Output is
plain text by default or JSON with `--json`. SearXNG endpoint is set via
the `SEARXNG_URL` environment variable or a local `config.json`.

Full options and setup in [`web-search/SKILL.md`](web-search/SKILL.md).

You'll need access to a running SearXNG instance with JSON format enabled.
The [SearXNG docs](https://docs.searxng.org/admin/installation.html) cover
the install.

---

## A note on Pi

[Pi](https://pi.dev) is an AI coding harness by Mario Zechner that
deliberately doesn't support MCP. The argument: MCP tools cost context
tokens whether or not you ever invoke them, and skills are the better
primitive for agent capabilities. These two skills exist because that
argument convinced me, and I still needed a browser and a search.

The `browser-tools` skill started as a fork of
[badlogic/pi-skills/browser-tools](https://github.com/badlogic/pi-skills),
Mario's own first cut at this idea, and grew into the version you see
here.

---

## License

MIT. Use them, fork them, send patches.
