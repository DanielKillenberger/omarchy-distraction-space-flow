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
- In `ds/handoff.py` `load_rules(cfg)`: parse each person rule with the same checks as shipped ones; skip a malformed rule, unknown field, bad regex or refused scheme with one `launch._log` line naming its index/app; merge per Architecture (person first in file order, same app name replaces shipped, `off` drops shipped by name).
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
TBD

## Evidence
- Commits:
- Tests:
- PRs:
