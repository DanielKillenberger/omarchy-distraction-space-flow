---
satisfies: [R7, R12]
---
# fn-38-readme-shows-the-features-and-drops-the.2 The site-block feature shows the HTTPS banner, not the block page

## Description
The maintainer found that a listed site typed into the work browser shows "This site can't be reached", not the block page: browsers reach nearly every site over HTTPS, where the plugin can only close the connection and raise the "Blocked here" banner. The README's site-block line and its block-page image show the plain-HTTP case as if it were the normal one. Replace the image with the maintainer-approved banner capture and reword the line so the README neither shows nor promises the block page.

**Touches:** README.md, docs/images/block-page.png (removed), docs/images/blocked-banner.png (added)

## Acceptance
R7 and R12 in the parent spec hold: the site-block feature line describes the failed load and the "Blocked here" banner, blocked-banner.png sits beside it with alt text saying what is shown, block-page.png is gone from the repository and from every link, and the reference doc keeps its technical account of the block page.

## Done summary
The README's site-block line now describes what a listed site typed into the work browser actually does over HTTPS: the page fails to load and a "Blocked here" banner offers to open it in the space. The maintainer-approved banner capture replaces the block-page image, block-page.png is removed, and the known-gaps sentence scopes the banner-only limit to HTTPS. docs/reference.md is unchanged and keeps the technical account of the block page and `nudges.block_page`.

The banner claim is taken from `ds/feedback.py` `blocked()`: title "Blocked here", body "<Product> opens in the distraction space. Super+Ctrl+Shift+D enters.", action `distractions open https://<host>/`.

GATE_SKIPPED:unittest:docs-only - cumulative diff classified tier-B (no executable paths touched)

stage: impl-review - ran (cursor:gpt-5.6-sol-high; round 1 NEEDS_WORK, a P2 because the gap sentence covered HTTP sites too, fixed in 4eddf7a; round 2 SHIP)
## Evidence
- Commits: 2b4a18b28f9798641d529af543a95c3d804356f3, 4eddf7a623eeeb04c5e9a4ae9b6bfc449452d3c9
- Tests: baseline: green (PATH=/usr/bin:$PATH python3 -m unittest discover -s tests: 517 tests, OK, 1 skipped, pre-edit at 8efbe07), GATE_SKIPPED:unittest:docs-only - cumulative diff classified tier-B (no executable paths touched), git grep block-page.png: no matches; every README docs/ link resolves
- PRs: