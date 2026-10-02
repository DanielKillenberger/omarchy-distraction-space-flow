# Setup asks before site blocking, and it can be turned on later

## Conversation Evidence

> user: "Didn't we fix that the setup doesn't need manual setup anymore?"

> user: "but couldn't we do a setup that asks if you want site blocking enabled if yes -> do the sudo call and have everything done in one setup call so we're compliant with that?"

> user (after being told the marketplace box does not apply to this listing and the question is a product choice): "yes spec it but keep in mind that people might want to enable it later through a command or the gui"

Facts established by reading the code and the marketplace repository on 2026-09-20. Setup runs the root transaction (the firewall helper and its sudoers grant) before every user-level step, and a failed or refused transaction returns before the slice, the WirePlumber hook, the Hyprland file, the launcher entries, and the notification clone are installed, so a person who will not give a password gets nothing. Setup already asks one question once, about link routing, records the answer in the config file, and never asks again. `site_block.enabled` defaults to on; turning it off destroys the firewall table and stops resolving, and everything else keeps working. With it on and no helper, the listener reports site blocking unavailable and says to run setup. The settings menu already carries "Block listed sites outside the space". Omarchy ships a floating-terminal launcher its own menus use for a command that needs a password. The marketplace's standard-installation box applies only to a listing carrying a manual-installation override; this listing has none and already shows the standard install command, so this spec is not a compliance fix.

## Goal & Context
<!-- Source-tag breakdown: 30% [user] / 70% [inferred] -->

Site blocking is the one feature that needs root: a firewall helper and a sudoers grant that lets the listener run it without a password. Today setup demands that grant from everyone, first, and installs nothing at all for a person who declines. [paraphrase] A person wary of a sudoers grant should still get the space, the notification hold, the mute, and link routing, and should be told plainly what the password is for before being asked for it. [user]

Setup asks whether to block listed sites outside the space. A yes does the sudo step and the rest in the same run; a no skips the sudo step and installs everything else. [user] The answer is not final: site blocking can be turned on later with one command or from the settings menu, and either path ends with it working, not with an instruction to go and run something else. [user]

## Architecture & Data Models
<!-- Source-tag breakdown: 100% [inferred] -->

**The question follows the link question's shape.** It is asked once, before anything asks for a password, with a short explanation of what site blocking does and what the password installs. The answer is recorded in the config file under the existing site-block switch, and a rerun prints the current choice and how to change it instead of asking again. [inferred]

**What counts as already answered.** An explicit value for the switch in the config file is an answer. So is an installed, current firewall helper: a person who has it said yes under the old setup, and an update must not ask them to confirm it or take it away. Only a config file with no explicit value on a machine with no helper is unanswered. [inferred]

**A helper answers whether it was asked, never what was answered.** When the file carries an explicit value, that value is the answer and the helper only keeps the question from being asked again. The two stand side by side all the time: turning site blocking off leaves the helper installed, because removal belongs to `setup --remove`. Reading a standing helper as a standing yes would turn blocking back on at the next `distractions setup` — which people are told to rerun after an Omarchy update or a list edit — against a choice they recorded. [inferred, resolved during implementation review 2026-09-20]

**The root step depends on the answer, and nothing else depends on the root step.** A yes runs the root transaction exactly as today. A no skips it. In both cases every user-level step runs, and a root transaction that fails or is cancelled no longer stops them: the run finishes the rest, reports the failure, exits non-zero, and leaves the answer at yes so a rerun tries again. [inferred]

**One command turns it on or off later.** Turning it on records the yes, installs the helper when it is missing or out of date (the only moment it asks for a password), and has the listener apply the block. Turning it off records the no and the listener destroys the table, as the switch does today. Setting the config key directly stays a plain config write; when it sets the switch on and no helper is installed, it prints the command that finishes the job. [inferred]

**The settings menu uses the same command.** The existing "Block listed sites outside the space" entry turns the feature off directly. Turning it on when the helper is already installed is also direct. Turning it on when the helper is missing opens Omarchy's floating terminal running the turn-on command, so the password prompt has a terminal to appear in; the menu never handles a password itself. [inferred]

**Status tells "off" from "not set up".** Off by choice is healthy and reads as off. On with no helper installed reads as not set up and names the command, and the listener does not raise the repeated "unavailable" notice for it: that notice is for a helper that is installed and failing. [inferred]

## API Contracts
<!-- Source-tag breakdown: 100% [inferred] -->

`distractions setup` on a terminal, unanswered:

```
Block listed sites outside the space? This is the one part that needs your
password: it installs a firewall helper and a sudoers rule so the listener can
run it. Everything else works without it, and you can turn it on later with:
distractions site-block on
[Y/n]
```

`distractions site-block on` and `distractions site-block off`. Exit 0 when the choice is recorded and the block it asks for is in effect: for `on`, the helper is installed and current and a listener took the reload; for `off`, the table is gone. Exit 1 when the helper could not be installed, with the answer left at on. `on` without a terminal and with no helper installed installs nothing, exits 1, and says a terminal is needed once. [exit codes for the two effect failures — a reload nobody answered, a flush that did not take — resolved during implementation review 2026-09-20: the command never reports an effect that did not happen, and neither case removes the recorded answer]

`distractions setup --yes` and setup without a terminal never ask and never prompt for a password. An answered yes with a current helper proceeds as today. An answered yes with a missing helper tries without a password, reports the failure, still installs the rest, and exits 1. Unanswered is treated as not answered, not as yes: the rest installs, site blocking is left not set up, one line names `distractions site-block on`, and the exit code reflects only the steps that ran.

`distractions status` reports site blocking as `on`, `off`, `unavailable`, or `not set up`.

## Edge Cases & Constraints
<!-- Source-tag breakdown: 100% [inferred] -->

- An existing install is never asked and never loses site blocking on update: a current helper is a yes. [inferred]
- The answer is written before the password is requested. If it cannot be written, setup stops before installing anything, as the link question does today, because an unrecorded answer would be asked again or overridden by a later `--yes`. [inferred]
- A person who answers yes and then cancels the password prompt ends with everything but site blocking installed, a non-zero exit, and a line naming the command to retry. The switch stays on, so status reads not set up rather than off. [inferred]
- Turning site blocking off leaves the helper and the grant installed; `setup --remove` is what removes them. Turning it off and on again therefore asks for no second password. [inferred]
- The turn-on command and the menu path go through the same root transaction setup uses, with the same destination, ownership, and validation checks; no second privileged path exists, and nothing uses polkit or a graphical password prompt. [inferred]
- When the floating-terminal launcher is missing, the menu shows a notice naming the command instead of failing silently. [inferred]
- The link-routing question keeps its place and wording; the two questions are asked one after the other, both before any password. [inferred]
- The manifest version moves in the same pull request, since this changes runtime files. [inferred]

## Acceptance Criteria

- **R1:** On a terminal, with no explicit site-block value in the config file and no helper installed, setup explains what site blocking needs and asks once, before any password prompt; the answer is recorded in the config file and a rerun prints the choice and the command that changes it instead of asking. Errors: an answer that cannot be written stops setup before anything is installed, with the path and the reason. [user]
- **R2:** A no installs every user-level step, runs no root transaction, asks for no password, exits 0 when those steps succeed, and leaves status reading site blocking off with the listener healthy. Errors: a failing user-level step is reported and exits non-zero as today. [user]
- **R3:** A yes runs the root transaction and every user-level step in the same run. Errors: a failed or cancelled root transaction is reported, the user-level steps still run, the exit is non-zero, and the recorded answer stays yes. [user]
- **R4:** A machine with a current helper installed has already answered, whether or not the config file carries an explicit value: setup does not ask it again. With no explicit value the helper's answer is yes, so site blocking stays on across the update; with an explicit value that value is the answer, so a machine that turned blocking off and kept its helper stays off. Errors: a helper that is installed but out of date is updated as today. [inferred, wording resolved during implementation review 2026-09-20; the original "treated as answered yes whatever the config file says about an explicit value" read two ways, and the reading that overrides an explicit no contradicts this spec's own boundary that turning off keeps the helper]
- **R5:** `distractions site-block on` records yes, installs the helper when it is missing or out of date, and ends with the listener applying the block; `distractions site-block off` records no and the table is destroyed. Errors: `on` with no terminal and no helper installs nothing, exits 1, and names the need for a terminal; a failed root transaction exits 1 with the answer left at on; an unknown argument exits 2 with usage; an effect that did not happen — a reload no listener answered, or a flush that did not take — keeps the recorded answer, says which, and exits 1. [user; the effect-failure clause resolved during implementation review 2026-09-20]
- **R6:** From the settings menu, turning site blocking off, and turning it on with a helper installed, take effect without a terminal; turning it on with no helper opens Omarchy's floating terminal running the turn-on command, and after it succeeds the menu shows site blocking on. Errors: a missing launcher shows a notice naming the command; a cancelled password leaves the menu showing not set up. [user]
- **R7:** `--yes` and a run without a terminal never ask either question and never prompt for a password; unanswered is left unanswered, the user-level steps install, and one line names the turn-on command. Errors: an answered yes with a missing helper reports the failed passwordless attempt, installs the rest, and exits 1. [inferred]
- **R8:** Status and the settings menu distinguish off, on, unavailable, and not set up; not set up names the turn-on command, and the listener raises no repeated unavailable notice for a helper that was never installed. Errors: none beyond the report. [inferred]
- **R9:** `config set` of the site-block switch stays a plain config write; setting it on with no helper installed prints the turn-on command. Errors: as `config set` today. [inferred]
- **R10:** The README's install section, the reference's setup and command sections, and the marketplace submission note describe the question, the no path, and the two ways to turn it on later. Errors: none. [inferred]
- **R11:** Tests cover R1 to R9 in the harness with a fake root transaction, including the existing-install case in R4 and both non-interactive cases in R7, and no test asks for a real password. Errors: none. [inferred]

## Boundaries
<!-- Source-tag breakdown: 100% [inferred] -->

- Turning site blocking off does not uninstall the helper or the sudoers grant. [inferred]
- No polkit rule, no graphical password dialog, and no second privileged installer. [inferred]
- Setup is still a command the person runs after `omarchy plugin add`; nothing here runs it automatically, and the marketplace verification form's standard-installation box stays unticked because it does not apply to this listing. [paraphrase]
- The other setup steps are not made optional; only the root step is. [inferred]
- The privileged transaction, the helper, and the grant are unchanged. [inferred]

## Decision Context

The request began as a way to tick the marketplace's standard-installation box. Reading the marketplace's form and registry showed the box belongs to listings with a manual-installation override, which this one does not have, so nothing here is owed to the marketplace. It is worth doing for the person installing: the sudoers grant is the most intrusive thing the plugin asks for, and today declining it costs every other feature. [paraphrase]

A dedicated on/off command was chosen over making `config set` install the helper, because a config write that sometimes asks for a password is a surprise, and over telling the person to rerun setup, because the owner asked that turning it on later be one action from a command or the menu. The menu opens a terminal instead of using a graphical password prompt because Omarchy's own menus do the same and because a second privileged path is a second thing for the marketplace reviewer to audit. [inferred]

Leaving the helper installed on off keeps off-and-on from costing a second password and keeps removal in the one place that already does it. A person who wants the grant gone has `setup --remove`. [inferred]

An installed helper counts as a yes so no existing install is asked a question it already answered or silently loses site blocking. [inferred]
