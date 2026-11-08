# macpilot

macpilot lets AI agents drive and verify macOS apps (native, Electron, Tauri) **in the
background**: it reads the interface as text, clicks and types through accessibility, waits,
runs scenarios, answers dialogs, reads logs, reads drawn text by OCR, compares captures and
records a window. The user's mouse, keyboard and frontmost app are left alone.

This repository only distributes the binaries. It contains no source code.

## Install

macOS 14 or later, on Apple silicon or Intel. The universal build covers both architectures.

```sh
V=0.6.0
cd "$(mktemp -d)"
curl -fsSLO "https://github.com/tfoutrein/macpilot-releases/releases/download/v$V/macpilot-$V-universal.tar.gz"
curl -fsSLO "https://github.com/tfoutrein/macpilot-releases/releases/download/v$V/macpilot-$V-universal.tar.gz.sha256"
shasum -a 256 -c "macpilot-$V-universal.tar.gz.sha256"     # must print: OK
tar -xzf "macpilot-$V-universal.tar.gz"
xattr -d com.apple.quarantine macpilot 2>/dev/null || true
mkdir -p ~/.local/bin && mv macpilot ~/.local/bin/          # ~/.local/bin must be on your PATH
macpilot --version
macpilot setup-permissions                                  # Accessibility + Screen Recording
macpilot doctor
```

The binary is signed ad hoc with the identifier `dev.claude.macpilot`, not with an Apple
Developer ID, so it is not notarized. You need to remove the quarantine attribute as shown
above.

Grant Accessibility and Screen Recording: `macpilot setup-permissions`. Ad hoc signature:
macOS ties the grants to this exact binary (cdhash), so re-run it after each update; when run
from a terminal, the grants go to the terminal app instead. `macpilot doctor` checks.

## Use it with an agent

- Claude Code MCP server: `claude mcp add -s user macpilot -- macpilot mcp`
- Agent cheat sheet: `macpilot help --agent`
- The `macpilot` skill is published on the Pulse by Astek DevHub.

## License

Free to use, binary only: see [LICENSE](LICENSE). No modification or redistribution; provided as is.
