---
satisfies: [R1, R2, R3, R4, R5, R6, R7, R8, R9]
---
# fn-37-setup-installs-the-hyprland.1 Implement Setup installs the Hyprland configuration

## Description
TBD

## Acceptance
Every R-ID in the parent spec's ## Acceptance Criteria is satisfied; judge this task against the spec's criteria directly.

## Done summary
Setup now owns one Lua file in the user's Hyprland config (`~/.config/hypr/distraction-space.lua`, a header plus the three shipped snippets in order) and one marked optional-require line in `hyprland.lua`, recorded as path and digest in `hypr.json` under the state directory; a matching rerun changes nothing, remove deletes the line and the file or moves an edited file into `hypr-backup/` and prints where, pasted 3.x snippets and an unrecognised `hyprland.lua` are reported and skipped while the rest of setup runs, and a write is followed by `hyprctl reload` with a reload-or-re-login report when Hyprland is unreachable. README, docs/internals.md, and the marketplace submission notes describe the new install, upgrade, and remove paths; manifest is 3.2.0.

Range: 4415c0f..bf5a3f9 (5448de2, 7d11472, bf5a3f9 are the conductor's review-fix commits; each carries the findings it closes in its message). `8792d7f` (the bridged child) carries the implementation, tests, docs, and version bump; `f6b06a0` (worker) rewrites the first comment line of hypr/bindings.lua and hypr/autostart.lua, which still read as paste instructions inside the file setup writes. Tests for every enumerated case live in tests/test_setup.py: fresh write, rerun no-op, snippet-change rewrite, remove with a matching, an edited, and a missing file or line, each of the three pasted-snippet paths (subTest), each of the four unrecognised hyprland.lua cases (subTest), the failed-write byte-identical guarantee, the reload call and its unreachable fallback, and Lua evaluation of the written file against fake Omarchy helpers and of the marked line with the module absent.

baseline: green (PATH=/usr/bin:$PATH python3 -m unittest discover -s tests, 491 tests OK; gate check exit 1: receipts dir resolves outside the repository, so the full command ran)
verify: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests, 502 tests OK (skipped=1), gate classify FULL (ds/setup.py)

stage: impl-review - ran (model: claude-opus-5 via host, read-only Explore subagents; round 1 fan-out NEEDS_WORK -> 5448de2, round 2 SHIP with P3s -> 7d11472, round 3 fan-out NEEDS_WORK -> bf5a3f9, round 4 re-review SHIP; two P3 follow-ups deferred and named in the evidence)
stage: implement - ran (model: cursor-grok-4.6-high-fast via cursor-agent -p --force --trust; delegated: 0)

Notes for the conductor: the bridged child ran `flowctl done` on its own despite the brief; the worker reset and re-started the task, so it is `in_progress` again, but the task markdown in the flow repository still holds the child's Done summary text (a guard refused restoring the file from git; the conductor's `flowctl done` rewrites that section). The child's commit `8792d7f` carries the required `Co-Authored-By: Claude Fable 5.1` line followed by a `Co-authored-by: Cursor` trailer cursor-agent appended; history was not rewritten. Reviewer-facing surface: tests/test_setup.py `_install` now filters stderr lines starting with `hyprland:` the same way it filters the wireplumber and notification-hold reports, because the sandbox has no stock hyprland.lua, so every pre-existing setup test takes the reported skip path; the hypr tests capture the report by calling `setup.sync_hypr()` directly. Follow-up not built: `remove` overwrites an earlier `hypr-backup/distraction-space.lua` when a second edited file is moved aside.

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: 8792d7f65f657bfa7a9632924b7dd46003749e51, f6b06a037aced5d627ac2329ca3b797478ee005b, 5448de2e1ecac4285ebbd12ab3d7b69faf17e458, 7d11472cc473566328360b3959f15b15634c062c, bf5a3f9463a9f18a02c842bf3fb19ce6940b500b
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests (baseline: 491 tests OK; verify: 502 tests OK, skipped=1), PATH=/usr/bin:$PATH python3 -m unittest tests.test_setup -k hypr -k Hypr (7 tests OK), PATH=/usr/bin:$PATH python3 -m unittest tests.test_setup -k hypr -k Hypr -k status_reflects -k first_write -k symlinked -k trailing_newline -k mode_of (13 tests OK, after the round-1 fixes), PATH=/usr/bin:$PATH python3 -m unittest discover -s tests (OK, skipped=1, after the round-1 fixes), PATH=/usr/bin:$PATH python3 -m unittest tests.test_setup -k hypr -k Hypr -k status_reflects -k first_write -k symlinked -k trailing_newline -k mode_of (16 tests OK, after the round-2 fixes), PATH=/usr/bin:$PATH python3 -m unittest tests.test_status (27 tests OK, after the round-2 fixes), PATH=/usr/bin:$PATH python3 -m unittest discover -s tests (511 tests OK, skipped=1, after the round-2 fixes), PATH=/usr/bin:$PATH python3 -m unittest tests.test_setup -k hypr -k Hypr -k status_reflects -k first_write -k symlinked -k trailing_newline -k mode_of -k newline_flag -k owned_file -k pasted_report (23 tests OK, after the round-3 fixes), PATH=/usr/bin:$PATH python3 -m unittest tests.test_status (27 tests OK, after the round-3 fixes), PATH=/usr/bin:$PATH python3 -m unittest discover -s tests (517 tests OK, skipped=1, after the round-3 fixes)
- PRs: