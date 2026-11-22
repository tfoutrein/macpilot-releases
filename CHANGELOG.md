# Changelog

What each macpilot release brings, newest first.

<!-- Maintainers: each "## X.Y.Z — YYYY-MM-DD" section becomes the notes of its GitHub release
     (release.sh reads them, and refuses a version without its section); this file is also
     copied to the public release repository. First line: a one-sentence summary (shown in the
     public README). Then ### Highlights, ### Changed (what an existing user may notice),
     ### Fixed, ### Known limits as needed. User-facing only: no internal issue numbers, no paths. -->

## 1.1.0 — 2026-11-22

Companion mode (preview): ask in plain words in a floating panel and watch an agent drive your Mac, step by step.

### Highlights
- **Companion mode**: `macpilot companion` opens a floating panel. You write (or dictate) a
  request; an agent, Claude Code or Codex, drives the Mac with macpilot's tools and nothing else
  (no shell, no file writes, no web outside the apps it drives). Every step appears in an activity
  feed, in French, with its outcome; each turn ends with « Qu’est-ce qu’on fait maintenant ? » and
  the conversation keeps its context. One request can span several apps (read a page in Safari,
  write in TextEdit).
- **You see what it does**: before each action, a halo frames the element about to be touched and
  a ghost pointer travels to it; your own cursor never moves. An inner glow frames the window of
  the app being driven, and the app comes to the front so you watch it happen.
- **You keep control**: ⌥Space shows or hides the panel; Stop (or Escape) ends the turn at once; a
  message written during a turn reaches the agent at its next step; sending, deleting, paying,
  publishing and anything in a terminal wait for your Oui in the panel. The companion pauses as
  soon as you use your mouse or keyboard (only your own hardware input counts) and resumes on
  « Reprendre ». The panel itself is yours: the agent cannot click it.
- **Two brains**: `--brain claude|codex` and `--model`, or pick both in the panel between two
  turns. A new model keeps the conversation; a new brain starts a new one.
- **Clear errors**: when the agent fails (not logged in, usage limit, no network, unknown model),
  the panel says why and what to do in one sentence, and the next request goes on. A locked screen
  or a missing permission is said too, with `macpilot doctor` when it helps. Every turn is kept in
  a daily journal, secrets masked.

### Changed
- Two new commands: `companion`, and `confirm` (used by the agent inside the companion to ask you
  Oui/Non). Outside the companion, every other command behaves exactly as in 1.0.0.
- MCP: inside the companion only, the server also offers a `confirm` tool, and an action may wait
  up to 10 minutes for your answer.

### Known limits
- Preview: the panel and the feed are in French.
- The companion needs Claude Code or Codex installed and logged in on this Mac.
- Codex cannot read a message during its turn: your correction reaches it when its next action is
  refused, as with Claude Code.
- Automatic pause is off in background mode (`--background`), which tests and agents use.

## 1.0.0 — 2026-11-21

Install with Homebrew, keep secrets out of logs, and know which app holds the permissions.

### Highlights
- **Homebrew**: `brew tap tfoutrein/macpilot && brew trust tfoutrein/macpilot && brew install
  macpilot` (recent Homebrew asks to trust third-party taps). Each release also publishes
  `checksums.txt` (SHA-256 of every archive).
- **Secrets stay out of logs**: `type`, `paste` and `set-value` read their text from standard
  input with `--stdin`, so it never appears in the process list or the shell history. A `run`
  step marked `"sensitive": true` (and any `password`, `token` or `secret` key) is masked as
  `•••` in results, reports and the MCP log. Password fields read back as `•••`.
- **The guide ships inside the binary**: `macpilot skills list` and `macpilot skills get
  [--json]` print the agent guide matching this exact version; MCP clients get it with the
  read-only `skills_get` tool.
- **Permissions, explained**: macOS grants Accessibility and Screen Recording to the app that
  launched macpilot (Terminal, iTerm, VS Code, an MCP host…), not to macpilot itself. `doctor`
  now names that app (`permission_holder`) and gives a `hint` saying what to enable;
  `setup-permissions --guide` opens the right System Settings panes.

### Changed
- `setup-permissions` reads Screen Recording again in a fresh process, so a grant given while it
  runs is reported (macOS caches the answer per process).

### Fixed
- `doctor` named macpilot itself instead of the host app when that app had been updated on disk
  while it was running.

## 0.9.0 — 2026-11-21

Wait for events, read overlays, and know exactly what an action did.

### Highlights
- **Wait for an event without knowing its title**: `wait --event window-opened`,
  `window-closed`, `surface-appeared`, `surface-dismissed`, `app-launched`, `app-terminated` or
  `focus-changed`. The result lists what changed (`changes`: title, type, pid…). In a `run`
  scenario, a `wait` step right after an action takes its baseline before that action, so a
  window opened by the click is caught. From the command line, start the wait before the
  trigger.
- **Overlays**: `snapshot --surface menu|sheet|popover|alert` reads only that overlay (alerts
  report their message); `not_found` when none is open.
- **Detailed action results**: every action now returns
  - `steps`: what it actually did, step by step — label, mechanism (`ax` accessibility call,
    `pid_event` keys posted to the process, `hid` system events), result, and whether its
    effect was verified; a fallback appears as its own step;
  - `post_state`: the target read back after the action (`value`, `checked`, `selected`,
    `expanded`, `focused`, `enabled`, or `exists: false`); password fields are masked;
  - `surfaces`: the menus, sheets, popovers and alerts open after the action (~10-25 ms).
- **More wait conditions**: `wait --on … --predicate actionable` (the same checks as `click-at`, without
  acting), `wait --text … --count N` (exactly N matching lines, on a complete read).

### Changed
- `run` copies `steps`, `post_state` and `surfaces` into each step of its result. `did` is kept.

### Fixed
- `click-xy --polite` refused before the gesture (target window not on top) reported
  `clicked: true`: it now says `clicked: false`, `ok: false`, `delivery: not_delivered`.

## 0.8.0 — 2026-11-21

Electron apps and very large accessibility trees.

### Highlights
- **Electron and Chromium apps at launch.** `launch` recognises them (`engine: electron|chromium`)
  and starts them with `--force-renderer-accessibility`, so the web content is in `snapshot` at
  once (`web_ax: forced`, `--no-web-ax` to skip). It no longer passes them Cocoa options, which
  made VS Code quit.
- **Isolated instances only.** A single-instance app (VS Code, Slack, Discord…) started on a
  profile already in use would hand over to the user's own app: `launch` refuses it
  (`refused`, `details.kind: would_reuse_user_instance`). Give it its own profile with
  `--arg=--user-data-dir=<empty folder>` (VS Code: also `--arg=--extensions-dir=<folder>`), or
  pass `--shared-profile`.
- **The frontmost app stays yours.** Electron activates itself when its window shows, even
  launched in the background: `launch` gives the front back to the app that had it
  (`front_restored`, `front_steals`).
- **Web accessibility checked, not assumed.** On an Electron app launched another way, `snapshot`
  turns web accessibility on (`AXManualAccessibility`, then `AXEnhancedUserInterface`) and reports
  `web_ax_enabled: true` only when the web content actually appeared (`web_ax_via`,
  `web_ax_tried`); otherwise `false` with a hint (relaunch with `launch`, or use `ocr`).
- **Large trees, bounded.** `snapshot` reads within one time budget (`--timeout`, default
  3000 ms) and says when it is incomplete (`complete: false`, `… (N more) [deadline]` markers,
  header hint).
- **Skeleton, then a region.** `snapshot --skeleton` shows the layout with child counts and a
  sample label; `snapshot --root eN` expands one region with the same refs. `-i` /
  `--interactive` keeps controls and their ancestors, `--compact` folds empty wrappers. Measured
  on a 600-item Electron page: skeleton in ~150 ms.
- **Keyboard layouts with a command layer.** Under cmd, a letter follows the layout's command
  layer: on "Dvorak – QWERTY ⌘", `key cmd+x` presses the QWERTY x key, as a person would.

### Changed
- A busy app is no longer mistaken for a missing element: element lookups and `wait` answer
  `timeout` with `details.kind: app_unresponsive` (`retryable: true`) instead of `not_found`.
- On a tree too large to read in the budget, a target found in the part read is used
  (`lookup_complete: false`); a target not found there gives `snapshot_incomplete` rather than
  `not_found`.
- `key --on <element>` focuses the element (and makes its window main) before the key, like
  `type --on`.
- `key` on a background Chromium/Electron app adds `background_note` (see Known limits).
- The key guard compares with the command layer too and, when macOS switches the input source
  per document, with every enabled layout.

### Known limits
- In the background, Electron apps get app-menu shortcuts, `click-at` (tree rows expand) and
  `set-value`, but not page shortcuts that need the focus (VS Code's keybindings): use `menu`,
  act on the element, or `--polite`.

## 0.7.0 — 2026-11-21

Reliable actions: every result says whether it reached the app.

### Highlights
- **Delivery on every action and error**: `delivery` is `not_delivered`,
  `delivery_uncertain`, `delivered_unverified` or `delivered_verified`, with `retry`
  (`safe` only when nothing was delivered), `retryable` and a stable `details.kind`. `run`
  never replays a step that may have acted; MCP reports a timeout as `delivery_uncertain`.
- **Effects checked**: after a press or `set-value`, macpilot reads back selection, expansion,
  focus, value or menu. Fallbacks (open, select by container, confirm) only for rows, cells,
  list items and disclosure triangles; a button is never pressed twice. `set-value` reports
  `normalized: true` when the app reformats the value, `no_effect` when it ignores it.
- **Actionability**: physical gestures wait for 3 stable samples; `--polite` checks occlusion on
  5 points and names the blocking element; disabled or non-editable targets fail at once
  (`retryable: false`).
- **Guard rails**: session-ending combos (`cmd+q`, log out, lock screen, Force Quit, empty Trash),
  also with extra modifiers, other layouts or as menu items, are refused unless `--force`.
  Session processes (Finder, Dock, WindowServer, loginwindow…) can never be quit, even with
  `--any`. `quit` re-checks the process identity before every signal.

## 0.6.0 — 2026-11-08

First public release.

### Highlights
- **Reliability**: auto-wait on every action that targets an element; `wait` command; stable
  error codes with a `hint`; readable, layout-aware keys (`key cmd+s`); menus by shortcut
  (`menu --shortcut cmd+n`), independent of the app's language.
- **Autonomy**: `launch` an isolated instance, `quit` only what macpilot launched, `logs`
  (stdout, unified log, crash reports), `dialogs` / `dialog` / `file-dialog`, `paste` without
  touching the user's clipboard.
- **Agents**: MCP server (`macpilot mcp`), agent cheat sheet (`macpilot help --agent`), run
  reports (`run --report out.md --junit out.xml`), a lock so that one agent at a time uses the
  real mouse, `audit` for accessibility issues.
- **Apps with little accessibility**: `ocr`, `find-text`, `click-text`; visual comparison
  (`shot-window --compare`); web content of Electron apps in `snapshot`; window video
  (`record start|stop`).
- Universal binary (arm64 + x86_64), ad-hoc signed with the identifier `dev.claude.macpilot`.
