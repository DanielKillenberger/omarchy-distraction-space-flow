<!-- BEGIN FLOW-NEXT -->
<!-- flow-next:snippet:v2 -->
## Flow-Next

This project uses Flow-Next for ALL task tracking. `flowctl` comes from the flow-next plugin install — every flow-next skill resolves it itself, and on Claude Code it is also on PATH. Do NOT create markdown TODOs or use TodoWrite. Cold session: `flowctl brief` first — one bounded call (specs, ready tasks, memory); go deeper with `show`/`cat`/`anchor <task-id>`.

- Lifecycle: `flowctl list` / `show fn-N.M` / `start fn-N.M` / `done fn-N.M --summary-file s.md --evidence-json e.json` (e.json: `{"commits": ["<sha>"], "tests": ["<cmd>"], "prs": []}`)
- BEFORE any other flowctl operation, or when unsure of a flag: run `flowctl usage` (CLI cheatsheet + orchestration recipes) or `flowctl --help`.
- BEFORE bridging work to another model/CLI (`codex exec`, `cursor-agent`, `claude -p`, `grok`) or picking an implementation/review model: run `flowctl usage` and follow "Orchestration & model steering" exactly.
- Creating a spec: write it directly — `/flow-next:plan` is task breakdown only. `flowctl spec create --title "Short title" --plan-file plan.md --json`, then `/flow-next:plan <spec-id>`. Scaffold cascade (first match wins): `SPEC.md` -> `spec.md` -> bundled template.
- Substantial replies (reports, reviews, multi-section answers): invoke `/flow-next:prose` BEFORE drafting — the artifact prose contract applies to chat replies too. Short conversational turns skip it.
- If `flowctl` is not found: your shell lacks the plugin's `scripts/` dir on PATH (only Claude Code injects it). Resolve it the way the skills do - the plugin install's `scripts/flowctl` (Claude/Droid: plugin-root env var; Codex: `${CODEX_HOME:-$HOME/.codex}/scripts/flowctl`; Cursor/Grok: two levels above any flow-next SKILL.md) - or update/reinstall the flow-next plugin. A repo with no `.flow/` yet: run `/flow-next:setup`.
<!-- END FLOW-NEXT -->

## Where the flow state lives

`.flow/`, `CLAUDE.md`, and `AGENTS.md` are not part of the plugin repository. They live in the sister repository `omarchy-distraction-space-flow` (checked out beside it at `../omarchy-distraction-space-flow`, remote `DanielKillenberger/omarchy-distraction-space-flow`) and reach the plugin checkout as three symlinks, all listed in the plugin's `.gitignore`. The marketplace reviewer asked for development orchestration to stay out of the repository Omarchy installs (fn-36).

- Specs, tasks, memory, and receipts never show in the plugin's `git status` or in a PR diff, and `git add -A` there does not pick them up. Commit them in the sister repository, on its own history.
- A fresh clone or a new git worktree of the plugin has none of the three symlinks. Recreate them before running `flowctl` there: `ln -s ../omarchy-distraction-space-flow/.flow .flow`, and the same for `CLAUDE.md` and `AGENTS.md`; adjust the relative path for a worktree that sits somewhere else.
- Git refuses a path behind the symlink (`pathspec ... is beyond a symbolic link`), so run git commands for flow files from inside the sister repository.

## Pull requests (owner, 2026-09-20)

The format and the procedure are `pr-format.md` in the sister repository (`../omarchy-distraction-space-flow/pr-format.md` from a plugin checkout), taken over from Telperion. Every run of `/flow-next:make-pr`, by hand or under `flow --auto`, follows that file in place of the skill's body phases and does not read the skill's `workflow.md` or its companion files. The body is Change, Proof, Look here, Decisions and Open, 1,500 characters for a small diff and at most 4,000 for a large one, ending in the make-pr marker. No cognitive-aid artifact, no mermaid, no `pr.html`.

## Implementation routing

Technical tasks that need no taste run on grok, per the user's standing instruction (2026-09-02): mechanical renames, config plumbing, test scaffolds, wrapper or parser changes with a precise brief, docs that mirror code. The worker bridges them with `grok --always-approve --no-plan -m grok-4.6 --reasoning-effort high -p "<self-contained prompt>" </dev/null` from inside its isolated worktree (a trusted git dir), reviews the resulting diff, runs `PATH=/usr/bin:$PATH python3 -m unittest discover -s tests`, and commits. Tasks where judgment decides the outcome stay on the session model: spec and plan prose, README voice, UX or API shape, security boundaries, anything a reviewer would argue about. The reviewer tier is codex, as `.flow/config.json` sets it (`review.backend`, currently `codex:gpt-6-astra:medium`; owner, 2026-10-02).
