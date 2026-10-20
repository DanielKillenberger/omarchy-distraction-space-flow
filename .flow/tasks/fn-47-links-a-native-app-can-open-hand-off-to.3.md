---
satisfies: [R6]
---
# fn-47-links-a-native-app-can-open-hand-off-to.3 Shipped rules for Zoom, Slack, Steam, Discord and Telegram

Touches: handoff.json, tests/test_handoff_rules.py

## Description
Add the remaining shipped rules as data, each with example-link tests. Pure data plus table-driven tests on `handoff.match`.

**Size:** M
**Files:** `handoff.json`, `tests/test_handoff_rules.py` (new; .2 owns `tests/test_handoff.py`)

### Approach
- For each app, take the scheme mapping from its vendor's own documentation or its Linux client's registered handler, and put the source in a comment next to its test (and later the PR). Candidates: Zoom `zoom.us/j/<id>?pwd=` → `zoommtg://zoom.us/join?confno=<id>&pwd=<pwd>` (also `<sub>.zoom.us` hosts listed explicitly); Slack `app.slack.com/client/<team>/<channel>` → `slack://channel?team=<team>&id=<channel>`; Steam store/community URLs → `steam://openurl/{url}` restricted to Steam hosts; Discord `discord.com/channels/<g>/<c>[/<m>]` → `discord://-/channels/...`; Telegram `t.me/<name>` → `tg://resolve?domain=<name>`.
- Table-driven tests in the style of `ClassifyTests` (`tests/test_launch.py:1023`): every recognised shape maps, at least one unmappable link per app does not.
- An app whose mapping cannot be sourced is left out; record which and why in the task summary for the PR.

### Investigation targets
**Required:** `handoff.json` and `ds/handoff.py` from .1
**Optional:** `tests/test_launch.py:1023` (table-driven style)

### Gotchas
- Discord and Telegram are on the default distraction list, so R3 keeps their listed links in the space; their rules only act when unlisted. Test `match` directly, not through a listed expansion.
## Acceptance
- [ ] Each of Zoom, Slack, Steam, Discord and Telegram either ships with tests for every recognised shape and an unmappable link, or is named as left out with the reason (R6)
- [ ] Steam's whole-URL rule refuses non-Steam hosts and URLs with forbidden characters (R6, R5)
- [ ] Focused tests pass: `PATH=/usr/bin:$PATH python3 -m unittest tests.test_handoff_rules`
## Done summary
Shipped rules now cover Spotify, Zoom, Slack, Steam and Telegram, each tested in tests/test_handoff_rules.py against every recognised shape and several unmappable links, with its source named beside the tests:
- Zoom: `zoom.us`/`usNNweb.zoom.us` `/j|w/<id>[?pwd=]` → `zoommtg://zoom.us/join?action=join&confno=<id>[&pwd=]` (Zoom developer forum; the Linux client registers zoommtg). Vanity subdomains are not covered since hosts are exact.
- Slack: `app.slack.com/client/<T…>[/<C|G|D…>]` → `slack://channel?team=&id=` / `slack://open?team=` (docs.slack.dev deep linking).
- Steam: `store.steampowered.com/app/<id>[/slug]` → `steam://store/<id>` (Valve's Steam browser protocol page). The planned whole-URL `steam://openurl/<url>` was dropped: Valve documents it as opening the system's default browser, which is this plugin, so the R6 note about Steam's whole-URL rule no longer applies; `{url}` stays available and tested for person rules.
- Telegram: `t.me`/`telegram.me` `/<username>[/<post>]` → `tg://resolve?domain=[&post=]` (core.telegram.org/api/links), reserved paths excluded.
- Left out: Discord. Its `discord://-/channels/...` routes are documented only in an unofficial gist, so R6 names it for the PR.
stage: impl-review - skipped(policy: deferred to one spec-level review after .4, the tasks share one feature surface)

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: ec701fc2e4a2cf820f3cc316a3ac5833d9ab808e
- Tests: PATH=/usr/bin:$PATH python3 -m unittest tests.test_handoff_rules tests.test_handoff
- PRs: