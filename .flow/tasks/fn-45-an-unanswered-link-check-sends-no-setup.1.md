# fn-45-an-unanswered-link-check-sends-no-setup.1 The displaced notice waits for an answered link check

## Description
TBD

## Acceptance
- [ ] TBD

## Done summary
`check_links` takes `answered` (the reconcile result's `links_ok`) and sends the setup notice only for an answered displacement; the state still reads `displaced` for an unanswered check. `test_an_unanswered_link_check_sends_no_setup_notice` covers R1 and R2; `docs/internals.md` says the notice waits for an answer (R3). The version bump to 3.4.3 shipped with fn-46 (#41), so this branch carries none after the rebase; both fixes release together as v3.4.3.

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: 268d91d
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: https://github.com/DanielKillenberger/omarchy-distraction-space/pull/42