---
satisfies: [R1, R2, R3, R4, R5, R6]
---
# fn-44-summary-agents-run-text-only-with-no.1 Summary agents run without tools where proven, with tools only on acceptance

## Description
TBD

## Acceptance
- [ ] TBD

## Done summary
claude runs with --tools "" --strict-mcp-config, proven by a live canary. grok, codex, gemini, opencode and copilot keep their one-shot argv and run only when summary.allow_agent_tools is true (owner, 2026-10-06); otherwise they show the grouped count with a log line. The Settings toggle asks for that acceptance before turning the summary on with one of them and clears it when turning the summary off. The agent runs from an empty temp dir; a temp-dir failure falls back to the count; docs record the probe and the risk; 3.4.2. Codex review SHIP after one fix round on each version (the second round declined the stale drop-the-routes requirement, now superseded in the spec).

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: 26dd846, 378d814, 1873a33, 2572134
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: https://github.com/DanielKillenberger/omarchy-distraction-space/pull/40