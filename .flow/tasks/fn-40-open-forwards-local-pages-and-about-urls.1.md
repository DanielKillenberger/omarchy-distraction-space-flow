---
satisfies: [R1, R2, R3, R4, R5, R6, R7, R8]
---
# fn-40-open-forwards-local-pages-and-about-urls.1 Implement Open forwards local pages and about: URLs

## Description
TBD

## Acceptance
Every R-ID in the parent spec's ## Acceptance Criteria is satisfied; judge this task against the spec's criteria directly.

## Done summary
`distractions open` now forwards a `file:` or `about:` URL exactly as given, and a bare argument that matches no list entry or catalog name but names an existing regular file as its absolute percent-encoded `file://` URL, through the same forwarder as an unlisted http(s) link. Every other non-http(s) scheme and every non-file bare argument still prints the usage line and exits 2. The usage line, argparse help, and the reference's `open` row describe the new shapes.

Tests (tests/test_launch.py): `test_file_and_about_urls_forward_exactly_as_given` covers R1, R2, R6 and their fallback and exit-1 error cases. `test_an_existing_regular_file_forwards_as_its_file_url` covers R3 (relative, absolute, symlink, `..`, edge whitespace, control characters; missing, directory, fifo, dangling link refused) and R4. `test_browser_override_missing_browser_and_usage` gained the R5 schemes. The R1, R2 and R3 tests were seen failing on f422a34 (exit 2 with the usage line) before the change.

Review: two NEEDS_WORK rounds, both about the argument being trimmed before it was used as an as-given URL or a file name; fixed by reading the scheme and path from the raw argument. Captured as memory entry bug/runtime-errors/early-strip-of-an-argument-that-can-be-2026-09-20.

Gate note: `flowctl gate receipt` refused to write (green-receipts dir resolves outside the repository through the symlinked .flow); the full suite ran green (534 tests, suite_rc=0). Baseline: green.

Correction: the first `flowctl done` call for this task read a stale /tmp/summary.md left by another project's run; this text replaces it.

Follow-up, not built: a file whose name itself carries a scheme shape (`a:b.html`) needs `./` in front, the same answer the spec gives for a file named like a catalog product.

Tier: session (jev moderate 0.99)

stage: impl-review - ran (cursor:gpt-5.6-sol-high; NEEDS_WORK, NEEDS_WORK, SHIP)
## Evidence
- Commits: d401533, 0916836, 897aa22
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: