# Signal launches its real desktop entry

## Problem

The catalog names Signal's system desktop entry `signal-desktop`, but Arch's `signal-desktop` package installs `signal.desktop` with `Icon=signal-desktop`. `open Signal` cannot run the entry and `setup` shadows nothing. The fix lived only as a local commit in the owner's installed plugin since 2026-09-20, which also blocked `omarchy plugin update` from fast-forwarding.

## Acceptance

- R1: Signal's catalog entry carries `desktop: "signal"`; `open Signal` runs `signal.desktop`, and `setup` shadows it.
- R2: An optional catalog field `icon` names the launcher icon when it differs from the desktop id; Signal's launcher keeps `Icon=signal-desktop`.
- R3: An entry without `icon` renders as before; an older `expansion.json` without `icon` still reads.
