# Decisions

Newest last. Each entry records what was decided, why, and what it costs, so nobody undoes it by accident.

## D1 — Bind by exact window only, no auto-bind by process (2026-09-27, v0.1.0-frank)

**Context.** When upstream created a group, it also stored a process-name matcher. On every foreground change, a window with no siblings went through `GroupManager::try_auto_bind`. That caused two bugs:

1. Any other window of the same exe joined the group the first time it was focused. Lassoing two of an app's windows eventually locked *all* of its windows together (upstream issue #3).
2. A bound window whose siblings had all closed also had no siblings, so it was re-matched and could move into a *different* group with the same exe.

**Decision.** On Windows, the foreground handler (`win_event_callback`, event `0x0003`) no longer auto-binds. A group contains exactly the windows that were lassoed. Commit `6278a56`, offered upstream as PR #4.

**Cost.** Upstream had an undocumented side effect: a grouped app that was closed and reopened in the same session rejoined its group when focused. That behavior is gone; a reopened app has to be lassoed again.

**Rejected alternative.** Keep auto-bind, but only fill a slot that a closed window of the same exe left empty. Rejected for v0.1.0: it is more code, and it still surprises the user, because *any* window of that exe can fill the slot, not necessarily the one that closed. Kept as a candidate in ROADMAP R1.

**Deliberately left alone.** `create_group_from_hwnds` still stores matchers, and `try_auto_bind` still exists, because the Linux backend uses both. They are dead code in Windows builds.

## D2 — Move sync follows the window, not the cursor (2026-09-27)

**Context.** `move_sync_loop` treated any left-button drag on a grouped foreground window as a window move and shifted the siblings by the cursor delta. Panning a NinjaTrader chart (a drag *inside* the window) moved every other window in the group. Resizing from an edge did the same.

**Decision.** During a drag, each poll compares the foreground window's rect with the previous poll's rect. The change counts only when the window moved and its size stayed the same, and the siblings move by the total of those changes. Commit `ff511c6`, upstream PR #5.

**Rejected alternative.** Sync only between `EVENT_SYSTEM_MOVESIZESTART`/`END`. That is the formal Windows signal for "a window is being moved", but apps that draw their own title bar (NinjaTrader/WPF) don't reliably go through the system move loop. If they don't, move sync would stop working entirely. It is also more code.

**Known limits.** Moves without the left button held (Win+Arrow snapping, keyboard moves) are not synced; that is the same as upstream. When a maximized window is dragged off the top of the screen, its size changes at the moment it restores, so that single step is skipped. The rest of the drag syncs normally.
