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
TBD

## Evidence
- Commits:
- Tests:
- PRs:
