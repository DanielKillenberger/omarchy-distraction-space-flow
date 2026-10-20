---
satisfies: [R1, R2, R3, R5]
---
# fn-47-links-a-native-app-can-open-hand-off-to.1 Handoff core: rule format, Spotify rule, handler lookup, hook before forward

Touches: ds/handoff.py, handoff.json, ds/launch.py, tests/test_handoff.py, tests/test_launch.py

## Description
Build the rule engine and wire it into `open`, proven end to end with the shipped Spotify rule. Person rules (config) come in .2, the other shipped apps in .3.

**Size:** M
**Files:** `ds/handoff.py` (new), `handoff.json` (new, repo root beside `catalog.json`), `ds/launch.py`, `tests/test_handoff.py` (new), `tests/test_launch.py`

### Approach
- `handoff.json`: list of rule objects `{app, scheme, hosts, shapes: [{path, query?, uri}]}`; `path` is a regex fully matched against the raw (undecoded) path, captures are named groups; `query` maps a param name to `{pattern, required}`; `uri` is a template with `{name}` placeholders and `{url}` for the whole URL. Load like `ds/catalog.py:15` `load_catalog`.
- `ds/handoff.py`: `load_rules(cfg) -> (rules, skipped)` (shipped only for now; keep a seam for .2); `ds/handoff.py` never logs or imports `ds/launch.py`; `open_target` logs `skipped`, `match(url, rules) -> (rule, uri) | None` implementing the Edge Cases contract (host exact after lowercasing and trailing-dot strip, default port only, no userinfo, captures re-checked against the default safe set: no `%`, `/`, whitespace, control; `{url}` gate), and `scheme_ok(scheme)` with the refused list.
- Handler lookup: new helper modelled on `_default_browser_id` (`ds/launch.py:458-469`, `net.run_command`, timeout 10, env without BROWSER) running `xdg-mime query default x-scheme-handler/<scheme>`; empty/failed → None. Usable per Architecture: `desktop_file` exists, id != `HANDLER_ID` (`ds/launch.py:32`) and != `state.read_entries()['previous_handler']`, and `parse_exec` + `expand_fields` (`ds/launch.py:351`, `408`) place the URI via a field code (do not rely on the append-when-no-code fallback).
- Hook: in `open_target` (`ds/launch.py:859`) on `target.kind == "forward"`, only for http(s) URLs and `not app`: try handoff first; on success log `handoff <app> via <desktop id>` (never the URI or link: Zoom's carries `pwd`) and return 0; `_detached` returning False (not spawned, or non-zero exit inside its settle window) falls through to `forward` with no extra log line, since `_detached` (`ds/launch.py:542-572`) already logs the failure. Flags present → reuse the existing 'not applied' log wording.
- Spotify shapes: optional `intl-[a-z]{2}(-[a-z]{2})?/` prefix; types track|album|playlist|artist|episode|show; id `[A-Za-z0-9]+`; `si` ignored.

### Investigation targets
**Required:** `ds/launch.py:228-262` (resolve_target), `ds/launch.py:773-867` (forward, split_args, open_target), `ds/launch.py:408-469` (expand_fields, exec_argv, _default_browser_id), `ds/catalog.py:15`
**Optional:** `tests/test_launch.py:27-43,159-210,276-330` (env keys, LOGGER, setUp, forward tests)

### Gotchas
- Memory: an early `strip()` of an as-given URL lost data; match against the URL exactly as `open` received it after `_url_host` passed.
- Tests need a new `xdg-mime` fake via `box.fake_bin` and its env key in `_ENV_KEYS`; the handler fake must be on PATH (`_detached` checks `which`). Sandbox leaves XDG_DATA_HOME unset unless `apply_env` runs: keep fakes inside the sandbox.
- Exit codes stay as documented: a failed handoff falls through to forward.
## Acceptance
- [ ] Spotify links of every type, with and without `intl-xx/` and `?si=`, launch the fake `spotify:` handler with `spotify:<type>:<id>` and no browser (R1)
- [ ] Each unusable-handler case (no xdg-mime, silent, missing desktop file, plugin entry, previous handler, Exec without URL code) forwards; a handler that cannot be spawned, and one that exits non-zero inside the settle window, each forward with exactly one log line; a successful handoff's log line carries neither the URI nor the link (R1)
- [ ] Unmappable Spotify paths, unrelated hosts and `--app` links forward with no xdg-mime call (R2)
- [ ] A listed host recognised by a rule still opens in the space (R3)
- [ ] Encoded captures, bad characters, non-default port and userinfo do not match (R5)
- [ ] Focused tests pass: `PATH=/usr/bin:$PATH python3 -m unittest tests.test_handoff tests.test_launch`
## Done summary
TBD

## Evidence
- Commits:
- Tests:
- PRs:
