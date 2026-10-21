---
satisfies: [R1, R2, R3, R4]
---
# fn-51-the-distraction-profile-stays-in-the.1 Record and reuse the profile's browser for an unclassifiable default

## Description
ds/state.py browser.json record; ds/launch.py pick_browser reuse and _remember_browser after a successful web launch; ds/setup.py _class_prefix follows the record; tests in test_launch and test_setup; docs/reference.md and docs/internals.md.

## Acceptance
- [ ] TBD

## Done summary
open records the binary of every successful launch in browser.json and reuses it for a default it cannot classify (wrapper script, Firefox) while it is on PATH; a binary change is one log line. Setup class prediction follows, with omarchy-open-chrome mapped to chrome. Codex review: NEEDS_WORK (2 P2), then SHIP.

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: dbdf3f5, 15a53c4, f425c90
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: