---
satisfies: [R1, R2, R3]
---
# fn-50-signal-launches-its-real-desktop-entry.1 Signal catalog entry names signal.desktop, with an icon field

## Description
catalog.json Signal desktop signal plus icon signal-desktop; ds/catalog.py carries icon through expansion; ds/setup.py renders Icon from icon, then desktop, then the safe name; tests in test_catalog, test_launch, test_setup; docs/internals.md catalog shapes.

## Acceptance
- [ ] TBD

## Done summary
Signal now names signal.desktop and keeps the signal-desktop icon via a new optional catalog field icon. Upstreamed from the local commit in the installed plugin.

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: ec94286, a662ab5
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: