# An unanswered link check sends no setup notice

## Conversation Evidence

> user, 2026-10-07: "have to run distractions setup apparently. Got a notification"; then, on the diagnosis, "ok pls fix"

On 2026-10-07 the session stalled from about 11:37 to 11:52 UTC: `omarchy-shell` answered "not responding" and `hyprctl` timed out, both in the plugin log. The listener's link check gives `xdg-settings get default-web-browser` 5 seconds; a timeout returns unanswered, `_observe_links` read that as `displaced`, and `check_links` sent "Links no longer open in the space / Another browser is the default. Run: distractions setup". The next check read the plugin's handler as the default again, and setup had nothing to fix.

## Goal & Context

The setup notice claims another browser took the default. It is sent only when `xdg-settings` answered with another id. [user]

## Acceptance Criteria

- **R1:** An unanswered link check (failure, timeout, or exception) sends no notice; a later answered displacement still sends its one notice. [inferred]
- **R2:** The state keeps reading `displaced` for an unanswered check, as setup already does. [inferred]
- **R3:** `docs/internals.md` says the notice needs an answer; `manifest.json` moves past the last tag. [inferred]

## Boundaries

- Not changing what `state.json` or `distractions status` report for an unanswered check. [inferred]
