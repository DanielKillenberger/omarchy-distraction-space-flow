---
satisfies: [R2]
---
# fn-34-github-actions-ci-runs-the-test-suite.2 Make the suite green on the GitHub runner

## Description
TBD

## Acceptance
The tests.yml job is green on ubuntu-latest for both matrix entries. Tests that need a Lua interpreter get it from a documented apt step; tests that need a user systemd bus or another desktop facility the runner lacks are skipped with the facility named, never weakened. The first observed green run on GitHub is the evidence.

## Done summary
Merged to main through DanielKillenberger/omarchy-distraction-space#28 on 2026-09-15; CI green on the PR head; the whole stack (#28, #24, #25, #27, #29) landed in that order and main was bumped to 3.1.0 at d26faef.
## Evidence
- Commits:
- Tests:
- PRs: