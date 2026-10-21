---
satisfies: [R1, R2, R3, R4]
---
# fn-49-a-bare-url-as-the-first-argument-opens.1 Insert open before a URL or existing path

## Description
distractions main(): also insert open when argv[0] is not a command and has a URL scheme or names an existing path. Tests in the CLI test file.

## Acceptance
R1-R4 tested; full suite green.

## Done summary
`distractions` main() also inserts `open` when the first token is not a command and has a URL scheme or names an existing path, so `omarchy-launch-browser <url>` ($BROWSER) reaches open. Other unknown words still fail in argparse. Test in tests/test_launch.py covers https, file:, a trailing flag, a local path, a listed URL, a typo and a real command; it fails without the fix. Docs: reference open row, internals open paragraph. Version rides on 3.4.4 from fn-48 (stacked branch).

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: d5ff7bd
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: