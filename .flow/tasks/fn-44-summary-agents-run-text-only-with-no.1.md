---
satisfies: [R1, R2, R3, R4, R5, R6]
---
# fn-44-summary-agents-run-text-only-with-no.1 Built-in summary routes allow no tools, proven by a live canary

## Description
TBD

## Acceptance
- [ ] TBD

## Done summary
Built-in summary routes reduced to claude with --tools "" --strict-mcp-config, proven by a live canary; codex, grok, gemini, opencode and copilot show the grouped count with a log line; the agent runs from an empty temp dir; a temp-dir failure falls back to the count; docs record the probe; 3.4.2. Codex review SHIP after one fix round.

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: 26dd846, 378d814
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests, scratchpad probe.py live canary across claude, grok, copilot, gemini, opencode
- PRs: