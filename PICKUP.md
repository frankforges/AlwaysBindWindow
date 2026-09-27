# Pickup

## Current state (2026-09-27)

- **v0.1.0-frank** is tagged on `main` of `frankforges/AlwaysBindWindow` and in daily use. It fixes same-app windows being locked together (DECISIONS D1).
- The user tested it manually with NinjaTrader windows: the group stayed at exactly the lassoed windows over ~7 minutes of switching.
- Upstream PR XR-stb/AlwaysBindWindow#4 (branch `fix/independent-same-app-windows`) is open and waiting for the maintainer.
- Release binary: not built. The user currently runs the debug build through `cargo run`.

## Next steps

- Watch PR #4 for review comments. If it's merged, `git merge upstream/main` into `main`. It should apply cleanly, since the same commit is already on `main`.
- Candidates, when there's a need: ROADMAP R1 (opt-in rejoin), K1 (lasso over-selection).
