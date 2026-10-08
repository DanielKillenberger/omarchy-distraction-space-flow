# fn-45-an-unanswered-link-check-sends-no-setup.1 The displaced notice waits for an answered link check

## Description
TBD

## Acceptance
- [ ] TBD

## Done summary
check_links sends the displaced notice only when xdg-settings answered with another id; an unanswered check still reads displaced in the state. Regression test test_an_unanswered_link_check_sends_no_setup_notice failed before the change and passes after, and also proves a later real displacement still gets its one notice. docs/internals.md updated; 3.4.3. Review skipped: a one-condition change to when a notice shows, with no state, security or data-layout change.

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: 446dcca
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: