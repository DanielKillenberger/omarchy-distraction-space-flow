# The distraction profile stays in the browser that ran it

## Conversation Evidence

> user (turn 1): "why i lose all my profiles on the distraction space?"
> user (turn 2): "ok fix it"
> user (turn 4): "X is still logged out"
> user (turn 5): "what's the bug in the plugin? this is annoying.. can you import the cookies as much as possible"
> user (turn 6): "and fix it with pr yes"

## Goal & Context

With `browser` left at `auto`, the plugin picks the distraction browser from the default browser, or from the handler setup recorded when the plugin is the default. A default it cannot classify as Chromium-family by name falls back to `chromium`, silently. The owner's default became a link-router wrapper script (`workspace-browser.desktop`, which execs `google-chrome-stable`), so a profile Google Chrome had built started running in Chromium. Chrome and Chromium encrypt cookies under different keyring entries, so every login in the profile looked gone, and the cookies were dropped. Nothing in `status`, the log, or a notice said the browser had changed. [paraphrase]
<!-- Source: 20% user / 70% [paraphrase] / 10% [inferred] -->

## Architecture & Data Models

A new state record, `browser.json` (`{"binary": "<argv0>", "updated": "<iso>"}`, owner-only like every state file), names the binary that last launched the distraction profile. A successful launch into the space writes it. The `auto` pick consults it only when the default is not a browser the plugin knows. [inferred]

## Acceptance Criteria

- **R1:** With `browser: auto` and a default (direct, or the recorded previous handler) that is not a known Chromium-family id, `open` launches the profile in the binary that last ran it, while that binary still resolves on PATH. Errors: no record, a malformed record, or a binary no longer installed → the `chromium` fallback as before. [paraphrase]
- **R2:** A known default (any Chromium-family id, `chromium` included) still decides as before, and a configured `browser` argv still wins over everything. [inferred]
- **R3:** Every successful launch records its binary; a change of binary from the recorded one writes one log line naming both. Errors: a failed record write costs one log line and never fails the launch. [inferred]
- **R4:** Setup's predicted launcher window class (`StartupWMClass`) follows the same pick, so launcher entries match the windows `open` makes. [inferred]

## Boundaries

- No automatic cookie migration or recovery between browsers; the owner's cookies were restored by hand. [inferred]
- No classification of wrapper scripts by reading or running them. [inferred]

## Decision Context

Recording the profile's own browser fixes the failure for any unclassifiable default (wrappers, Firefox) without guessing what a script runs. Rejected: reading the wrapper's Exec target (scripts are arbitrary), and refusing to launch (worse than the old fallback for a fresh profile). An existing install with no record keeps the old fallback until its first launch records one. [inferred]
