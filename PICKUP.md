# Pickup

## Current state (2026-09-27)

- `main` carries two fixes on top of upstream:
  - D1: only lassoed windows bind. Tagged `v0.1.0-frank`.
  - D2: move sync follows the window, not the cursor. Tagged `v0.1.1-frank` (the current version).
- Both were tested manually by the user with NinjaTrader windows (see the two manual tests in `docs/BUILD.md`), and both passed.
- The installed copy at `C:\Program Files\AlwaysBindWindow\always-bind-window.exe` is built from `main` with both fixes. Auto Start points there. To update it, follow `docs/BUILD.md` → Installed copy.
- Upstream PRs are open and waiting for the maintainer: XR-stb/AlwaysBindWindow#4 (D1) and #5 (D2).

## Next steps

- Watch PRs #4 and #5 for review comments. If they are merged, `git merge upstream/main` into `main`.
- Candidates, when there's a need: ROADMAP R1 (opt-in rejoin), K1 (lasso over-selection).
