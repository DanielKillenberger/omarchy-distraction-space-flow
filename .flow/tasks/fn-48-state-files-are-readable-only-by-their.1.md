---
satisfies: [R1, R2, R3, R4, R5]
---
# fn-48-state-files-are-readable-only-by-their.1 Owner-only state dir and appended files

## Description
state.state_dir chmods 0700; a state helper opens append files 0600 (fchmod when wider); hold, hypr, lock, net, summary log writers and held.jsonl + its trim use it. Tests under umask 022.

## Acceptance
R1-R5 tested; full suite green.

## Done summary
`state.state_dir` creates the state directory 0700 and tightens an existing wider one; `state.open_private` opens a state file 0600 for append or rewrite and tightens one left wider. held.jsonl, its trim rewrite, and the log appenders in hypr, hold, summary, lock and net use it. A failed chmod/fchmod is ignored. JSON state already went through mkstemp (0600); a test pins it. tests/test_private.py: 8 tests under umask 022, all failing without the fix. manifest 3.4.4 in its own chore(release) commit.
Full suite: 571 ran, OK, 1 skipped.

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: 94ee34b, 0931439, 8fd8ab4
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: