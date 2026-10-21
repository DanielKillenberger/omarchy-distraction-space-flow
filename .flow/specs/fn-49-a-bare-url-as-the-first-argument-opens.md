# A bare URL as the first argument opens it

## Problem
Issue #44. `omarchy-launch-browser` (and so `$BROWSER`) runs only the first token of the default browser's `Exec=` line, so a link arrives as `distractions https://...`. `main()` inserts `open` only for no arguments or a leading flag, so argparse rejects the URL as an unknown command, and the launcher discards stderr: `gcloud auth login` and anything else using `$BROWSER` opens nothing.

## Requirements
- **R1:** `distractions <target>`, where target has a URL scheme (`https:`, `http:`, `file:`, `about:`, and any other `scheme:`) or names an existing local path, behaves exactly as `distractions open <target>`, trailing arguments included.
- **R2:** Every existing command, `-h`/`--help`, the bare binary, and a leading browser flag behave as today.
- **R3:** A first argument that is neither a command nor a URL nor an existing path still fails with argparse's error, so a mistyped command is reported, not sent to the browser.
- **R4:** Tests cover `distractions https://...` and `distractions file:///...` without `open`; `manifest.json` carries an unreleased version.

## Decision
A bare list entry or catalog name (`distractions youtube`) is not taken as a target: callers that drop `open` pass URLs or files, and treating any unknown word as a target would turn command typos into browser launches.
