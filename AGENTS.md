# AGENTS.md — AlwaysBindWindow (frank fork)

Read this before any task in this repo. The parent workspace rules (`../AGENTS.md`) still apply; this file adds what is specific to this project.

## What this is

A fork of [XR-stb/AlwaysBindWindow](https://github.com/XR-stb/AlwaysBindWindow): a Rust tray app that binds several top-level windows so they activate, move, and minimize together. The user presses a hotkey and lasso-selects the windows to bind.

**Why the fork exists:** the maintainer uses it to group a *subset* of the windows of one multi-window app (a NinjaTrader 8 workspace). The other windows of that app must stay independent. Upstream bound windows by process name, which pulled every window of the app into the group (upstream issue #3). This fork binds only the exact windows selected.

**Fixes carried on `main`:**
- Bind only lassoed windows: `6278a56`, D1, upstream PR XR-stb/AlwaysBindWindow#4
- Move sync follows the window, not the cursor: `ff511c6`, D2, upstream PR #5

Each fix lives on its own branch cut from `upstream/main`, and that branch is merged into `main`.

## Scope

- **Windows 10/11 only.** The macOS/Linux code comes from upstream, is untouched, and is unverified by us. Do not edit `src/platform/macos.rs` or `src/platform/linux.rs` unless asked.
- `src/group.rs` is shared by all platforms. Linux still calls `try_auto_bind`, so a change there changes Linux behavior too.
- `README.md` is upstream's and stays unmodified on purpose, so upstream merges stay clean. Our docs live in the files below.

## Docs map

| File | Purpose |
|------|---------|
| `docs/BUILD.md` | Toolchain, build, run with logs, the manual test procedure, releases |
| `docs/DECISIONS.md` | Why things are the way they are. Read before changing binding behavior |
| `docs/ROADMAP.md` | Candidate improvements and known issues (not commitments) |
| `PICKUP.md` | Current state and the next step. Update after material work |

## Architecture (Windows)

The app runs three threads that share one `Arc<Mutex<GroupManager>>`:

- **Main / tray** — `tray::run_tray`: winit event loop, tray menu, global hotkeys. Lasso → `overlay::run_picker_overlay` → `GroupManager::create_group_from_hwnds`.
- **Monitor** — `platform::windows::start_monitor`: `SetWinEventHook` for foreground (`0x0003`), minimize start/end (`0x0016`/`0x0017`), and object destroy (`0x8001`). All handled in `win_event_callback`.
- **Move sync** — `move_sync_loop`: polls every 8 ms. While the left button is held on a grouped foreground window, it moves the siblings by however far that window itself moved without changing size. It never uses the cursor delta (see DECISIONS D2).

Group membership is `GroupManager.active_bindings: HashMap<hwnd, group_id>`, which is the only source of truth. `WindowGroup.window_matchers` is still filled in at lasso time but is **not used on Windows** (see DECISIONS D1).

**Loop prevention:** the app's own window actions fire the same hooks it listens to. Anything that activates, minimizes, or moves windows programmatically must set `SUPPRESSED` while it runs and refresh `LAST_FG_SYNC_MS` (150 ms debounce), like `bring_group_to_front` does. `MOVE_IN_PROGRESS` covers the move thread.

## Gotchas

- **Groups live in memory only.** Quitting or restarting the tool clears them. Window handles (HWNDs) change when an app restarts.
- **The lasso picks every window with any visible part inside the rectangle.** Partial overlap counts. Draw tight rectangles.
- **`sync_move` / `sync_minimize` in `settings.json` do nothing.** Nothing reads them, and every group uses the `WindowGroup::new` defaults (both `true`).
- **Shared machine state with upstream builds:** the settings file (`%APPDATA%\AlwaysBindWindow\settings.json`) and the auto-start value (`HKCU\Software\Microsoft\Windows\CurrentVersion\Run\AlwaysBindWindow`) are the same for every build. Toggling Auto Start in a dev build writes *that* exe's path into the Run key.
- **One instance at a time:** a second copy cannot register the global hotkeys. Check with `Get-Process always-bind-window` before launching.
- **Debug vs release:** a debug build has a console and logs (`env_logger`, default level `info`). A release build uses the GUI subsystem: no console, and logs are lost.
- **`Cargo.lock` is gitignored upstream,** so dependency versions can drift between builds.
- **The build shows ~14 warnings** (dead code, unused `BOOL` results). They are expected. Don't fix them as a side task.
- **Pushing a `v*` tag triggers `.github/workflows/release.yml`** if workflows are enabled on the fork. It builds all platforms and publishes a public GitHub Release.

## Git workflow

- **Remotes:** `origin` = `frankforges/AlwaysBindWindow` (ours), `upstream` = `XR-stb/AlwaysBindWindow`.
- **`main` is our version.** Sync with upstream: `git fetch upstream` then `git merge upstream/main`.
- **Changes meant for upstream:** branch from `upstream/main`, keep our docs out of the branch, and open the PR against XR-stb.
- **Our versions are tags `vX.Y.Z-frank`**, independent of `version` in `Cargo.toml`. That stays at upstream's value to avoid merge conflicts.
- No Claude co-author trailer in commits (workspace rule).

## Verifying changes

There are no automated tests, and window behavior needs a real desktop. For any change that touches binding, activation, moving, or minimizing: build, run with logs, and walk the user through the manual test in `docs/BUILD.md`. The user does the mouse actions; the agent reads the log.
