---
satisfies: [R1, R2, R3, R4, R5]
---
# fn-46-a-local-flood-cannot-stop-the-feedback.1 Bound live feedback handlers and survive a failed thread start

## Description
TBD

## Acceptance
- [ ] TBD

## Done summary
`_accept_loop` claims a slot from a shared counter (MAX_CONNS = MAX_SPLICES + 64) before starting a handler; `_serve` releases it. Over the cap the connection is closed and logged at most once a minute; a RuntimeError from Thread.start() releases the slot, closes the connection and the loop continues. Two tests in tests/test_feedback.py, both failing on the old code; docs/reference.md names the bound; manifest 3.4.3.

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: f4cac2e, e813df1
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: https://github.com/DanielKillenberger/omarchy-distraction-space/pull/41