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
TBD

## Evidence
- Commits:
- Tests:
- PRs:
