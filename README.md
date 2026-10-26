# .claude/tools — browser, web and desktop control for Claude Code

Three small scripts that give any Claude Code session (or any agent, or you) the
ability to **see and drive a screen, drive a real browser, and talk to the web** —
in any project, for any task. Nothing here is tied to a particular repo.

| Tool | What it does | Needs |
|---|---|---|
| **`web.py`** | HTTP: fetch pages as readable text, parse JSON, list links, download files | nothing (stdlib) |
| **`browser.py`** | A real browser you keep open across commands — click, fill, screenshot, run JS; logins persist | `playwright` |
| **`screen.py`** | The whole desktop — screenshot, click, type, keys, window focus. Drives *any* app or browser | `pillow`, Windows |
| `doctor.py` | Checks and installs the dependencies above | nothing (stdlib) |

---

## Install

**Already here?** Just check dependencies:

```bash
python .claude/tools/doctor.py            # report
python .claude/tools/doctor.py --install  # install whatever's missing
```

**Into another project:**

```powershell
.\install.ps1 -Target C:\code\my-project   # Windows
./install.sh ~/code/my-project             # macOS / Linux
```

**Globally, for every project on the machine:**

```powershell
.\install.ps1 -Global      # -> ~/.claude/tools
./install.sh --global
```

The installer copies the scripts and then runs `doctor.py --install`. Add
`-NoDeps` / `--no-deps` to skip that, `-Force` / `--force` to overwrite existing
copies.

Requirements: Python 3.8+. `web.py` runs anywhere with nothing installed.
`browser.py` works on Windows, macOS and Linux. `screen.py` is Windows-only.

---

## Which tool?

```
Need a page's content?            -> web.py get URL
Page needs JS or a login?         -> browser.py
Not a browser at all (an app,
a game, an installer, Firefox)?   -> screen.py
```

Start with `web.py` — it's instant and free. Reach for `browser.py` when the
site needs JavaScript, a session, or clicking. Reach for `screen.py` when
there's no DOM to talk to.

---

## `web.py` — HTTP without a browser

```bash
python web.py get   https://example.com [--max 20000]   # HTML stripped to text
python web.py raw   https://example.com                 # body exactly as sent
python web.py json  https://api.example.com/x [--path data.items.0.name]
python web.py links https://example.com [--filter blog]
python web.py head  https://example.com                 # status + headers
python web.py dl    https://example.com/file.zip [out]
python web.py post  https://api.x/y '{"a":1}' [--json|--form]
```

Flags on any command: `-H "Name: value"` (repeatable), `--ua "..."`,
`--timeout 30`, `--cookies cookies.json`, `--insecure`.

HTTP errors are printed with their body instead of raising, so you can read what
the server actually said.

---

## `browser.py` — a real browser that stays open

The browser launches once and **keeps running between commands**, so a login you
do once holds for everything after it.

```bash
python browser.py start [--headless]        # optional; any command auto-starts
python browser.py open https://example.com
python browser.py text [SELECTOR] [--max 20000]
python browser.py shot [out.png] [--full]
python browser.py click "button.submit"     # or "text=Sign in"
python browser.py fill  "#email" "me@example.com"
python browser.py press Enter
python browser.py wait  ".results" [--timeout 30000]
python browser.py eval  "document.title"
python browser.py links [--filter /docs/]
python browser.py tabs | tab 1 | newtab URL | closetab
python browser.py cookies --out cookies.json
python browser.py status | stop
```

Also: `html`, `select`, `upload`, `type`, `scroll`, `back`, `forward`,
`reload`, `pdf`.

### Logins and profiles

Profiles live in `~/.claude/browser-profiles/NAME` and are shared by every
project on the machine.

```bash
python browser.py --profile work open https://intranet.example.com
python browser.py --profile personal open https://example.com
```

Run without `--headless` and the window is visible: **sign in by hand once**,
and every later command — in this project or any other — is already signed in.
This is also the answer to a captcha or an MFA prompt: do that one step
yourself in the window, then hand control back.

### Any Chromium browser

```bash
python browser.py start --channel chrome    # your installed Chrome
python browser.py start --channel msedge    # Edge
CLAUDE_BROWSER_EXE="C:\...\brave.exe" python browser.py start   # Brave/Vivaldi/Opera
```

With no `--channel`, Playwright's bundled Chromium is used.

**Firefox and Safari have no CDP** — drive them with `screen.py` instead.

**A browser that's *already* open can't be attached to** unless it was started
with a debug port. To automate the Chrome window you already have in front of
you, use `screen.py`. Use a second `--port` (e.g. `--port 9333`) to run a
toolkit browser alongside your everyday one.

---

## `screen.py` — the whole desktop (Windows)

```bash
python screen.py shot [out.png] [--region X,Y,W,H] [--full]
python screen.py click X Y [right|middle|double]
python screen.py drag X1 Y1 X2 Y2
python screen.py type "text"
python screen.py paste "long text"        # via clipboard: instant and exact
python screen.py key ctrl+l | enter | alt+tab | ctrl+shift+n
python screen.py scroll -3                # + up / - down
python screen.py focus "window title"
python screen.py windows                  # list visible windows
python screen.py pos                      # cursor position
```

**Coordinates come from the screenshot you just looked at.** `shot` downscales
to ≤1600px wide and records the scale and origin, so an X,Y read off the image
maps back to the real pixel — including on 4K displays and `--region` crops.
Take a shot, read the coordinates off it, click them.

`paste` beats `type` for anything long: it goes through the clipboard, so it's
instant and can't drop characters.

---

## Recipes

**Read a page that needs no browser**
```bash
python web.py get https://example.com/docs --max 5000
```

**Scrape something behind a login**
```bash
python browser.py start                       # visible window
python browser.py open https://site/login     # sign in by hand, once
python browser.py open https://site/reports
python browser.py text ".report-table"
```

**Hand a browser session to the fast HTTP tool**
```bash
python browser.py cookies --out c.json
python web.py get https://site/api/items --cookies c.json
```

**Fill and submit a form**
```bash
python browser.py open https://site/form
python browser.py fill "input[name=email]" "me@example.com"
python browser.py click "button[type=submit]"
python browser.py text body --max 500
```

**Drive an app that has no DOM** (Firefox, a game, an installer)
```bash
python screen.py focus "Mozilla Firefox"
python screen.py shot                  # look at it
python screen.py key ctrl+l
python screen.py type "example.com"
python screen.py key enter
python screen.py shot                  # confirm
```

**Pull a JSON API and dig into it**
```bash
python web.py json https://api.github.com/repos/python/cpython --path stargazers_count
```

---

## Tell Claude about it

Paste this into your project's `CLAUDE.md` so sessions reach for the toolkit on
their own:

```markdown
## Browser / web / desktop tools
`.claude/tools/` holds scripts for web and screen work. Check with
`python .claude/tools/doctor.py` (add `--install` to fix anything missing).
- `python .claude/tools/web.py get URL` — fetch a page as text; also `json`,
  `links`, `dl`, `post`. No dependencies; try this first.
- `python .claude/tools/browser.py open URL` — real browser that stays open
  between commands, so logins persist; `text`, `click`, `fill`, `shot`, `eval`.
- `python .claude/tools/screen.py shot` — screenshot the desktop, then
  `click X Y` / `type` / `key` using coordinates read off that screenshot.
Run any of them with no arguments for full usage.
```

---

## Troubleshooting

**`Missing playwright` / `Missing Pillow`** — `python doctor.py --install`.

**`Browser did not open a CDP port`** — something already uses the port. Pass
`--port 9333`, or `python browser.py status` to see what's running.

**`stop` says it can't stop the browser** — close the window, or kill the
process holding the port. `stop` closes the tabs, then kills the recorded pid,
then falls back to whoever is listening on the port.

**Clicks land in the wrong place** — take a fresh `screen.py shot` first.
Coordinates are relative to the most recent screenshot; if the window moved or
you cropped a region since, the mapping is stale.

**A page is empty or differs from the browser** — it's JavaScript-rendered.
Use `browser.py` rather than `web.py`.

**Unicode** — all tools force UTF-8 output, so CJK, emoji and accented text
print correctly even in a `cp1252` Windows console.

---

## Notes

- Sites with bot challenges will show a captcha. That's a stop sign, not a bug:
  run the browser visibly and clear it yourself, then carry on — the session
  persists.
- Profiles hold real cookies and logins. `~/.claude/browser-profiles/` is as
  sensitive as your browser profile; don't commit it or copy it around.
- `screen.py` controls the actual mouse and keyboard. Whatever is focused
  receives the input, so check with `shot` before typing.

MIT licensed — see `LICENSE`.
