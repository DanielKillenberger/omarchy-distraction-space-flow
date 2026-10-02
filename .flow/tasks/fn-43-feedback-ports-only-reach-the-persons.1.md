---
satisfies: [R1, R2, R3, R4, R5]
---
# fn-43-feedback-ports-only-reach-the-persons.1 Input chain gates the feedback ports to the person cgroup

## Description
TBD

## Acceptance
- [ ] TBD

## Done summary
Input chain in distractions-nft resets a new connection to 28080/28443 unless the receiving socket is in user.slice/user-<uid>.slice; check ds verifies it; the live namespace test covers IPv4/IPv6 reached vs reset; docs updated, manifest 3.4.1.

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: e335d19
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests, DS_LIVE_NFT_TEST=1 PATH=/usr/bin:$PATH python3 -m unittest tests.test_nft
- PRs: