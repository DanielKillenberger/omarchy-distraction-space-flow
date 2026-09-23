---
satisfies: [R1, R2, R3]
---
# fn-42-leaving-the-space-never-lands-on-a.1 Cycle skips special workspaces

## Description
TBD

## Acceptance
- [ ] TBD

## Done summary
cycle() now skips special: workspaces; regression test with the live ids fails before the change and passes after.
## Evidence
- Commits: 68e8936
- Tests: python3 -m unittest tests.test_hypr tests.test_enter, python3 -m unittest discover -s tests (556 run; 1 unrelated timing flake in test_net, passes alone)
- PRs: