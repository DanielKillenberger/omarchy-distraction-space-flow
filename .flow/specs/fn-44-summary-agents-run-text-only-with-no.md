# Summary agents run text-only, with no tools

## Conversation Evidence

> maintainer on omacom/omarchy-plugin-marketplace#9667, at 3694ffc: "With the optional automatic summary enabled at `3694ffcb33ee3fbb505d11f64266ad31e3521327`, foreign notification titles/bodies flow through `ds/hold.py:402-415` into the prompt in `ds/summary.py:186-187`, and the built-in `opencode run` route does not disable the native default shell/write tools; notification text can therefore influence an agent with host permissions beyond summarization. Please enforce a text-only, tool-disabled summary route before requesting verification of a new commit"

> user, offered "keep only routes that can provably run with zero tools" (recommended), "remove the agent summary", or "wrap the agent in an OS sandbox": "yes"

> user, 2026-10-06, after the maintainer withdrew the objection on #9667: "i think we can just ship the improvement to disable tool calls for those who support it and have a warning for other cli's that they might still have tools enabled." Then: "don't disable them / just have users approve the risk". This supersedes dropping the unproven routes.

Facts established on 2026-10-02 by reading the code and the installed CLIs. `summary.command` defaults to `off`. With `auto`, `ds/summary.py` runs the Omarchy default agent once with a fixed instruction followed by the held records, one JSON object per line, on stdin: `grok -p`, `claude -p --output-format text`, `codex exec -s read-only --skip-git-repo-check -`, `gemini -p`, `opencode run`, or `copilot -p`. None of these disables the agent's tools; codex alone runs its shell in a read-only sandbox. The installed CLIs expose these controls: claude `--tools` (an empty value disables every built-in tool) and `--strict-mcp-config`; grok `--tools` (an allowlist) and `--disallowed-tools`; copilot `--available-tools` (an allowlist), `--disable-builtin-mcps`, and `--no-custom-instructions`; gemini `--policy` (policy-engine files) and `--approval-mode plan` (read-only, read tools remain); opencode no flag, only configuration, which an environment variable can supply; codex only a sandbox and a set of per-tool feature switches that changes between versions, with no allowlist.

## Goal & Context
<!-- Source-tag breakdown: 30% [user] / 70% [inferred] -->

The "While you were away" line is written from text other people control. Whatever agent writes it gets that text and nothing it can act with: no shell, no file reads or writes, no web, no MCP servers, no plugins or skills. [user, via the maintainer's finding] A route runs with no tools when its no-tools form is proven on the installed CLI. A route that cannot prove it stays, but `auto` runs it only after the person accepts that their agent may keep its tools; until then it shows the grouped count. [user, 2026-10-06]

## Architecture & Data Models
<!-- Source-tag breakdown: 100% [inferred] -->

**Each route is an allowlist of nothing, not a denylist.** A route qualifies only through a switch that names the tools the agent may have and is given none, so a tool a later CLI release adds is off by default. A denylist of today's tool names does not qualify. [inferred]

**Proof is a live canary, recorded.** For each route, the agent is asked from an empty directory to create a canary file and to print a secret from a file beside it. The route qualifies when the canary never appears, the secret never comes back, and an ordinary summary still answers. The result, the CLI version, and the date go into `docs/internals.md`. [inferred]

**Routes that cannot prove it run only on the person's acceptance.** codex has no tool allowlist, and grok, gemini, opencode and copilot have not passed the probe; each keeps its one-shot argv and runs only when `summary.allow_agent_tools` is true. The Settings toggle asks before it turns the summary on with such an agent and clears the acceptance when it turns the summary off. Without acceptance the log line says why the count is shown. [user, 2026-10-06]

**The agent runs outside the person's project.** The child's working directory is a fresh empty temporary directory, so no project instruction file or project configuration is picked up. [inferred]

**A custom command stays the person's own choice.** `summary.command` as an argv is used as given; the reference says the held text is untrusted and that the command must not have tools. [inferred]

## Edge Cases & Constraints
<!-- Source-tag breakdown: 100% [inferred] -->

- An agent CLI that rejects one of the flags (an older or newer release) fails the run; `ask` already logs it and falls back to the grouped count, so a mismatch shows the count, never an agent with tools. [inferred]
- No route may print the held text anywhere but the prompt on stdin, and no route writes configuration on disk; opencode's configuration comes from the environment of that one child. [inferred]
- `off` stays the default; nothing changes for a person who never opted in. [inferred]
- The manifest version moves in the same pull request, since this changes runtime files. [inferred]

## Acceptance Criteria

- **R1:** Every built-in route whose CLI has a proven switch that allows no tools passes it (today claude only, named in `NO_TOOLS`); a unit test pins each argv. [user, 2026-10-06]
- **R2:** The child runs in a fresh empty working directory that is removed afterwards. [inferred]
- **R3:** A live canary probe for every route in `NO_TOOLS`, on the installed CLI, records that no canary file was created, no secret was returned, and a plain summary was answered; the record names each CLI version and what the other routes did. [inferred]
- **R4:** `auto` with an agent outside `NO_TOOLS` runs it only when `summary.allow_agent_tools` is true, and otherwise logs why and shows the grouped count; turning the summary on from Settings with such an agent asks for that acceptance first, and turning it off there clears it. [user, 2026-10-06]
- **R5:** `docs/reference.md` lists the invocations, warns that agents outside `NO_TOOLS` may keep their tools and documents `summary.allow_agent_tools`, and warns that a custom command receives untrusted text and should have no tools; `docs/internals.md` carries the probe record. [user, 2026-10-06]
- **R6:** `manifest.json` carries a version past the last tag. [inferred]

## Boundaries

- Not sandboxing the agent at the operating-system level. [user]
- Not filtering or rewriting the notification text; the guarantee comes from the agent having no tools, not from cleaning its input. [inferred]
