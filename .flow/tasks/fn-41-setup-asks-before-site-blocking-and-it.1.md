---
satisfies: [R1, R2, R3, R4, R5, R6, R7, R8, R9, R10, R11]
---
# fn-41-setup-asks-before-site-blocking-and-it.1 Implement Setup asks before site blocking, and it can be turned on later

## Description
TBD

## Acceptance
Every R-ID in the parent spec's ## Acceptance Criteria is satisfied; judge this task against the spec's criteria directly.

## Done summary
Setup now asks whether to block listed sites outside the space — once, right after the link question and before anything asks for a password — and the root transaction is the only step that follows the answer, so a no, a refusal, or a cancelled password still installs the slice, the hold hook, the Hyprland file, the launcher entries and the clone. `distractions site-block on|off` and the settings menu turn it on or off afterwards, through the same root transaction and the same passwordless wrapper; the menu hands the turn-on to Omarchy's floating terminal when a password is needed, so no second privileged path exists. Status tells "off by choice" from "never set up", and the listener's repeated unavailable notice is now for a helper that is installed and failing.

The manifest moves to 3.4.0 in its own chore(release) commit, as version-check.yml requires over the tagged 3.3.1.

Two spec clauses were amended during review rather than the implementation, each flagged in its commit and both accepted by the reviewer afterwards:

- R4 said a current helper is "treated as answered yes whatever the config file says about an explicit value". Read literally, `distractions setup` would turn blocking back on for anyone who ran `site-block off` — which keeps the helper by this spec's own boundary. R4 now says a helper answers whether the question was asked, never what was answered.
- R5 and the API contract did not enumerate an effect that does not happen. They now do: a flush that did not take, or a turn-on no listener applied, keeps the recorded answer, says which, and exits 1.

The last review round found the exit code for `on` was reading the wrong signal, in two ways that both fire on healthy machines: the listener's `ok` reply is composite (site blocking AND link routing AND the launcher sync), and `state.request_reload()` waits two seconds against a reconcile that can take twenty. It now asks with the listener's own budget and reports what the listener recorded for site blocking. Captured as bug/integration/a-composite-ok-reply-is-not-evidence-2026-09-20.

The test harness now pins the two root destinations into each sandbox, in-process and in every `distractions` child, so "is the firewall helper installed" is a fact about the test rather than about the machine running the suite. Sandboxes default to helper-installed, so tests that say nothing about site blocking behave as they did.

Tier: session (explicit IMPLEMENTER preserved: opus)

stage: impl-review - ran (cursor:gpt-5.6-sol-high; NEEDS_WORK, NEEDS_WORK, NEEDS_WORK, then SHIP after the owner reset the stalled loop)
## Evidence
- Commits: cd765bf8354d21a6fdf203d644828b5a265694cb, 330d7cc4e85785bf846c0e4fe70d8cef08795266, d5e4b3ddf01e9aea6e7a32209ee08a1f658d3f88, 8ffe3a7756958363b6d5291cd2994006001144eb, 99444cf98dd224a3ce73f4c72a68a298d5568600, d4197b57110f48e187d50658a93c66afa8f1bbac
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests (555 tests, OK, skipped=1, at the tree committed as d4197b5; baseline at 05fed60 was 534, OK), flowctl gate classify --base 05fed60 -> FULL (executable paths touched), so the full suite ran; no GATE_SKIPPED lines, flowctl gate receipt --gate unittest: refused to write, green-receipts dir resolves outside the repository through the symlinked .flow - recorded as a note, not a gate failure
- PRs: