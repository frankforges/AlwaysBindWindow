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

## Releases

Our versions are git tags `vX.Y.Z-frank` on `main` (see `AGENTS.md` → Git workflow). Pushing a `v*` tag triggers `.github/workflows/release.yml` only if workflows are enabled on the fork. When it runs, it builds Windows/macOS/Linux binaries and publishes a public GitHub Release. Otherwise, build locally with `cargo build --release`.
