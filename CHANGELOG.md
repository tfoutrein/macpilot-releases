# Changelog

What each macpilot release brings, newest first.

<!-- Maintainers: each "## X.Y.Z — YYYY-MM-DD" section becomes the notes of its GitHub release
     (release.sh reads them, and refuses a version without its section); this file is also
     copied to the public release repository. First line: a one-sentence summary (shown in the
     public README). Then ### Highlights, ### Changed (what an existing user may notice),
     ### Fixed, ### Known limits as needed. User-facing only: no internal issue numbers, no paths. -->

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
