---
satisfies: [R7]
---
# fn-47-links-a-native-app-can-open-hand-off-to.4 Document the handoff and release 3.5.0

Touches: docs/reference.md, docs/internals.md, manifest.json

## Description
Docs and version for the whole spec.

**Size:** S
**Files:** `docs/reference.md`, `docs/internals.md`, `manifest.json`

### Approach
- `docs/reference.md`: a paragraph after 'Links open in the space' (~line 96) on the handoff, the order, the `--app` exception and handler fallbacks; the `handoff` key in the config table (~124-155) with an example person rule; the shipped app list.
- `docs/internals.md`: the `open` resolution order (~line 29), the shipped data file in 'Catalog shapes' (~167), the `xdg-mime` fake in 'Tests' (~206).
- `manifest.json` version 3.5.0 in its own `chore(release)` commit (pr-format step 4).

### Investigation targets
**Required:** `docs/reference.md:23-27,96,124-163`, `docs/internals.md:29-37,167-206`

## Acceptance
- [ ] Reference and internals describe the behaviour, order, rule format with a person-rule example, `--app` exception and the test fake (R7)
- [ ] `manifest.json` is 3.5.0 in a `chore(release)` commit (R7)
- [ ] Full suite passes once: `PATH=/usr/bin:$PATH python3 -m unittest discover -s tests`

## Done summary
`docs/reference.md`: a "Links an app can open go to the app" paragraph (behaviour, shipped apps, Discord left out, usable-handler rule, order, `--app` exception, privacy of the log), `handoff.rules` and `handoff.off` rows in the config table, the rule format with a person-rule example (verified to parse and match), and the `open` command row. `docs/internals.md`: the hook point and engine/launch split in the `open` resolution order, why Steam uses `steam://store`, `handoff.json` under catalog shapes, `xdg-mime` in the test fakes. `manifest.json` 3.5.0 in its own `chore(release)` commit.
Full suite: 598 ran, OK, 1 skipped.
stage: impl-review - skipped(policy: deferred to one spec-level review after .4, the tasks share one feature surface)

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: 326c87ea47c02f82f722fc3b6021a8aee794047d, 718b2eacef22f75ba22dfd006ada4d17df4e0ba4
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: