---
satisfies: [R4]
---
# fn-47-links-a-native-app-can-open-hand-off-to.2 Person-defined rules in config: add, replace, turn off

Touches: ds/config.py, ds/handoff.py, tests/test_config.py, tests/test_handoff.py

## Description
Let the person add rules, replace a shipped rule by app name, and turn shipped rules off, per Architecture's order.

**Size:** M
**Files:** `ds/config.py`, `ds/handoff.py`, `tests/test_config.py`, `tests/test_handoff.py`

### Approach
- Add `handoff: {"rules": [], "off": []}` to `DEFAULTS` (`ds/config.py:20-36`) and validate only the outer shape in `validate` (`ds/config.py:140-175`, `_need`): `rules` a list of objects, `off` a list of strings. `config set handoff.rules '<json>'` then works via the existing `_parse_value` path.
- In `ds/handoff.py` `load_rules(cfg)`: parse each person rule with the same checks as shipped ones; skip a malformed rule, unknown field, bad regex or refused scheme and return it in `skipped` (index/app and reason); `open_target` writes the one log line, so the engine keeps no dependency on `ds/launch.py`; merge per Architecture (person first in file order, same app name replaces shipped, `off` drops shipped by name).
- `_read_cfg()` falls back to DEFAULTS when the whole config fails validation; accept that (outer-shape validation keeps `config set` honest) and do not add a second reader.

### Investigation targets
**Required:** `ds/config.py:20-36,140-201,438-523`
**Optional:** `tests/test_config.py:100-160` (`test_every_schema_key_roundtrips`)

### Gotchas
- `test_every_schema_key_roundtrips` will pick up the new keys; give them values it can round-trip.
## Acceptance
- [ ] A person rule maps a link no shipped rule knows; a same-name person rule replaces the shipped one; a name in `off` stops the shipped rule (R4)
- [ ] Person rules win over shipped ones by order (R4)
- [ ] `config set handoff.rules` refuses a non-list and `handoff.off` a non-list of strings (R4)
- [ ] A malformed rule, an unknown field, a bad pattern and each refused scheme are skipped with one log line while the other rules still hand off (R4)
- [ ] Focused tests pass: `PATH=/usr/bin:$PATH python3 -m unittest tests.test_handoff tests.test_config`
## Done summary
Config gains `handoff: {"rules": [], "off": []}`; `validate` checks only the outer shape (a list of objects, a list of strings), so `config set` refuses a wrong type while a rule that does not parse is skipped at open time. `handoff.load_rules(cfg)` puts the person's rules first in file order, then each shipped rule not named in `off` and not replaced by a person rule of the same app name (case-insensitive; a person rule that fails to parse still claims its name, so the log explains why neither runs). Skipped rules come back as reasons; `open` logs one line each.
Tests: PersonRuleTests in tests/test_handoff.py (add, replace, off, order, skipped reasons, wrong-shaped section), GOOD/BAD values in tests/test_config.py, and an end-to-end `config set` + `open` test in tests/test_launch.py.
stage: impl-review - skipped(policy: deferred to one spec-level review after .4, the tasks share one feature surface)

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: 8327637ea7542b32aa6452ce4a7395c66f2d9414
- Tests: PATH=/usr/bin:$PATH python3 -m unittest tests.test_handoff tests.test_config tests.test_launch
- PRs: