# Roadmap

Candidates, not commitments. The current release (v0.1.1-frank) already covers the main use case. Pick items up only when there's a real need.

## Ideas

### R1 — Opt-in "rejoin" for reopened windows

Bring back the behavior removed in DECISIONS D1, but so that it can't cause the D1 bug again.

- **Not a global tray toggle.** Turned on globally, it *is* the D1 bug: every window of a grouped exe could join.
- A safer shape: a per-group option. When a group member closes, the group keeps an empty slot for that exe (and possibly its title pattern or window class). Only a *new* window that matches the slot can fill it, and only once.
- Open questions: what counts as "the same window" for an app like NinjaTrader, where many windows share one exe and class? Titles may be the only signal. And where does the switch live: tray menu, or a small per-group dialog?
- Needs a proper design pass before any code.

## Known issues (upstream behavior, not yet addressed)

- **K1 — the lasso over-selects.** Any window with a visible part inside the rectangle is picked (`overlay.rs`, `rects_intersect` + visibility sampling). This picked up an unintended window during the v0.1.0 test. Possible fixes: require a minimum overlap, or show the picked windows before confirming.
- **K2 — `sync_move` / `sync_minimize` in `settings.json` are not wired up.** Every group always syncs both.
- **K3 — groups are not saved.** They are lost when the tool quits. Saving them runs into the same "which window is which" question as R1, because window handles don't survive restarts.
- **K4 — `Cargo.lock` is gitignored,** so builds aren't reproducible.
- **K5 — `release.yml` uses actions that still target Node.js 20** (`checkout@v4`, `upload-artifact@v4`, `download-artifact@v4`, `action-gh-release@v2`). GitHub currently forces them onto Node 24 with a deprecation warning. Bump them to their Node 24 versions before GitHub stops doing that.
