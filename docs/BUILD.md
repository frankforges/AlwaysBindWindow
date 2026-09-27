# Build, run, and test

## Toolchain

- Rust stable with the MSVC target (`x86_64-pc-windows-msvc`). Verified with rustc/cargo **1.95.0**; upstream says 1.75+.
- MSVC Build Tools + Windows SDK, as required by the MSVC Rust target.
- `build.rs` embeds a DPI-awareness manifest through `winresource`. If that step fails, the build prints a warning and continues without the manifest.

## Commands

Run all commands from the repo root.

| What | Command |
|------|---------|
| Debug build | `cargo build` |
| Release build | `cargo build --release` → `target\release\always-bind-window.exe` |
| Run with logs | `cargo run` (the debug build keeps a console; set `RUST_LOG=debug` for more) |
| Already running? | `Get-Process always-bind-window` |
| Stop | Tray icon → Quit, or `Stop-Process -Name always-bind-window` |

**Running it from an agent session:** launch it in the background and send the log to a file, so the agent can read it after the user tests:

```powershell
cargo run *> <scratch-dir>\abw.log
```

### Log lines worth knowing

| Line | Meaning |
|------|---------|
| `Lasso selected N windows` / `Bound N windows: <exe>` | Result of a lasso bind |
| `FG sync: N windows` | A grouped window came to the front. N = group size |
| `Min sync: N` / `Restore sync: N` | Minimize or restore passed to N siblings |
| *(nothing)* | An ungrouped window was focused. That case is silent |

## Manual test: same-app windows stay independent

This is the regression test for DECISIONS D1. Run it after any change to binding or activation.

1. Start a debug build with logs.
2. Open at least three windows of one app: W1, W2, W3.
3. Press `Ctrl+Alt+G` and lasso **only W1 and W2**. Expect `Bound 2 windows`.
4. Click W3. Expect: only W3 comes to the front, and no `FG sync` line appears.
5. Click W1. Expect: W1 and W2 come to the front together, and `FG sync: 2 windows` appears.
6. Drag W1. Expect: W2 follows, and W3 stays put.
7. Click W3, then W1 again. Expect: still `FG sync: 2 windows`. If the count grows, the bug is back.

## Manual test: move sync only on real moves

This is the regression test for DECISIONS D2. Move sync writes nothing to the log, so judge it by eye.

1. Group two windows, W1 and W2. At least one should have draggable content, such as a chart.
2. Drag inside W1 (pan the chart). Expect: W2 doesn't move.
3. Drag W1 by its title bar. Expect: W2 follows and keeps its relative position.
4. Resize W1 from its left or top edge. Expect: only W1 changes; W2 doesn't move.

## Installed copy

The copy in daily use lives at `C:\Program Files\AlwaysBindWindow\always-bind-window.exe`, and Auto Start points at that path. To update it, from an elevated shell:

1. `cargo build --release`
2. Quit the running copy (tray → Quit, or `Stop-Process -Name always-bind-window`). Its groups are lost.
3. Copy `target\release\always-bind-window.exe` over the installed file.
4. Relaunch it unelevated, the way Auto Start does: `Start-Process explorer.exe "C:\Program Files\AlwaysBindWindow\always-bind-window.exe"`

## Releases

Pushing a `vX.Y.Z-frank` tag on `main` publishes a public GitHub Release automatically. `.github/workflows/release.yml` builds Windows x64 and ARM64 on GitHub's runners and attaches both `.exe` files.

To cut a release:

1. Make sure `main` is committed, pushed, and tested (the manual tests above).
2. `git tag -a vX.Y.Z-frank -m "<one-line summary>"`
3. `git push origin vX.Y.Z-frank`
4. Watch the build: `gh run watch -R frankforges/AlwaysBindWindow`

**Versioning:** bump the last number (Z) for bug fixes and the middle one (Y) for new features. `Cargo.toml`'s version stays at upstream's value.

**Always-latest download link:** `https://github.com/frankforges/AlwaysBindWindow/releases/latest/download/AlwaysBindWindow-windows-x64.exe`

**Fork divergence:** our `release.yml` is trimmed to Windows only and has its own release text. When merging `upstream/main`, keep our version of this file if it conflicts.
