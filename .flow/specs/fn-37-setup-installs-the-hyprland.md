# Setup installs the Hyprland configuration

> HTML render lens: [.flow/artifacts/fn-37-setup-installs-the-hyprland/spec.html](../artifacts/fn-37-setup-installs-the-hyprland/spec.html) — regenerable, markdown is the record. <!-- flow-next:artifact-link -->

## Conversation Evidence

> user (turn 1): "why didn't we tick the standard installation acknowledgement?"
> user (turn 2): "can we not include the copy into the setup?"
> user (turn 3): "do it"

The design is the assistant's proposal in reply to turn 2, accepted in turn 3, so its substance carries `[inferred]`. Facts established in the conversation by reading the installed Omarchy: the plugin system has no Hyprland hook (plugin kinds are bar and bar-widget; the add and enable commands never touch the user's Hyprland config); Omarchy's Lua bootstrap puts the user's config directory on the Lua module path, so a module named for a file in the user's hypr directory resolves by name; Omarchy ships an optional-require helper that loads a module only when it can be found and returns nothing otherwise; the stock user hyprland.lua ends with a comment inviting personal configuration below it. Also established: the maintainer's own machine is a pasted install, with the workspace rule inside hyprland.lua and the bindings and autostart lines inside the per-file configs.

## Goal & Context
<!-- Source-tag breakdown: 10% [paraphrase] / 90% [inferred] -->

Installing the plugin today is three steps, and the middle one is by hand: add the plugin, copy three Lua snippets into the right files under the user's Hyprland config, run setup. The user asked for the copy to be part of setup. [paraphrase] The plugin has no way to have Omarchy load its config for it, so setup has to write into the user's config directory itself. [inferred] After this change a fresh install is the add command and one setup run, and the README says exactly that. [inferred]

The trigger was the marketplace form's standard-installation box, which stays unticked: setup still runs by hand and still asks for sudo, so the plugin still needs manual setup. This change removes the copy step, not the setup step. [inferred]

## Architecture & Data Models
<!-- Source-tag breakdown: 100% [inferred] -->

**One file, one line.** Setup writes a single Lua file into the user's Hyprland config directory whose content is the three shipped snippets in order (workspace rule, bindings, listener autostart), with a header comment naming the plugin and stating that setup owns the file. The shipped snippet files stay in the repository as the source setup reads from, so the README can still link to them and a person can still read what setup will write. Setup then appends one line to the user's hyprland.lua that loads that file through Omarchy's optional-require helper, followed by a marker comment that identifies the line as setup's. The optional helper is the point: when the file is gone, because the plugin was removed without running setup first, the line loads nothing and Hyprland starts cleanly. [inferred]

**Idempotent, marked, recorded.** A rerun rewrites the file only when its content differs and never adds a second line: the marker is the identity. Setup records the written file's path and content digest in the state directory the way it records launcher entries, so remove knows exactly what it wrote. Remove deletes the marked line and the file; a file whose digest no longer matches the record was edited by the person and is moved aside into the state directory's backup, the way a launcher entry the plugin did not write is, and remove says where it went. [inferred]

**Pasted installs.** Every 3.x install pasted the snippets by hand. Setup looks for the helper path in the user's bindings and autostart files and for the workspace rule in hyprland.lua before writing anything; when any of them is found, setup writes nothing, reports that the pasted snippets are in use and where, and leaves the config as it is. Writing anyway would start two listeners and register every bind twice. The README tells a pasted install how to move over: delete the pasted lines, rerun setup. [inferred]

**Unrecognised config.** A hyprland.lua setup cannot vouch for is left untouched and reported, and the rest of setup still runs: the file is missing or unreadable, or it already loads the plugin's module without the marker, or it does not load Omarchy's defaults at all and is therefore not the config layout this file is written for. Status reflects the skipped write the way it reports other incomplete setup work. [inferred]

**Reload.** After writing, setup asks Hyprland to reload its configuration through hyprctl, which is new for setup: today the listener applies window rules through hyprctl and setup never calls it. When Hyprland is not reachable, setup reports that a reload or re-login is needed. [inferred]

## Edge Cases & Constraints
<!-- Source-tag breakdown: 100% [inferred] -->

- The written file and the marked line are created through a temporary file and rename, so an interrupted setup never leaves a half-written hyprland.lua; hyprland.lua is rewritten only to append or remove the one marked line and is otherwise byte-identical. [inferred]
- Remove with a missing file or a missing marked line is a no-op for that part and still succeeds; a record that names a file which no longer exists is dropped. [inferred]
- The marker survives a person editing other parts of hyprland.lua; setup never parses Lua, it matches the marker line. [inferred]
- The optional-require helper is part of Omarchy 4; a config that predates it is the unrecognised case above, never a crash. [inferred]
- The marketplace reviewed 3.0.0 on the stated basis that the plugin does not edit the user's Hyprland config. This change reverses that claim; it lands under the installer capability the reviewer already flagged, and the submission notes must say so before the next verification request. [inferred]
- The manifest version moves to 3.2.0 in the same pull request, because the version check refuses runtime changes under an already-tagged version. [inferred]

## Acceptance Criteria

- **R1:** A first setup run on a stock Omarchy config writes one Lua file in the user's Hyprland config directory containing the three shipped snippets, appends one marked optional-require line to hyprland.lua, and records the file's path and digest in the state directory. Errors: hyprland.lua missing, unreadable, already loading the module without the marker, or not loading Omarchy's defaults leaves both files untouched, is reported, and the rest of setup still runs; a failed write leaves hyprland.lua byte-identical. [inferred]
- **R2:** A second setup run on that config changes nothing: the file is rewritten only when the shipped snippets changed, hyprland.lua keeps exactly one marked line, and the record is unchanged. Errors: no error surface beyond R1. [inferred]
- **R3:** Setup with remove deletes the marked line and the recorded file and leaves every other byte of hyprland.lua in place; a file whose digest differs from the record is moved into the state directory's backup and its new location is printed. Errors: a missing file or missing line is a no-op for that part; the record is dropped either way. [inferred]
- **R4:** On a config that already carries the pasted snippets, found by the helper path in bindings or autostart or the workspace rule in hyprland.lua, setup writes nothing to the Hyprland config, reports which file holds the pasted lines, and the rest of setup runs; no config ever ends up with two autostart lines or two sets of binds. Errors: no error surface beyond the report. [inferred]
- **R5:** After a successful write setup asks Hyprland to reload through hyprctl; with Hyprland unreachable it reports that a reload or re-login is needed. Errors: a failed reload is reported and does not fail setup. [inferred]
- **R6:** When the written file is absent, the marked line loads nothing and hyprland.lua evaluates without error, verified by running the resulting config through a real Lua interpreter the way the existing Hyprland fragment tests do; the written file itself evaluates the same way against the fake Omarchy helpers. Errors: no error surface beyond a test failure. [inferred]
- **R7:** The README install section is the add command, one setup run, and done; the snippets are described as what setup writes and links to; the upgrade section tells a pasted 3.x install to delete its pasted lines and rerun setup; the remove section no longer asks the person to delete snippets by hand. Errors: none. [inferred]
- **R8:** The marketplace submission notes state that setup writes one file into the user's Hyprland config and one line into hyprland.lua, replacing the earlier claim that the plugin does not edit that directory, and that the standard-installation acknowledgment stays unticked because setup still runs by hand with sudo. Errors: none. [inferred]
- **R9:** Tests cover the fresh write, the rerun no-op, remove with a matching and with an edited file, each pasted-snippet detection path, each unrecognised hyprland.lua case, and the reload call and its unreachable fallback, all against a temporary config directory the harness owns. Errors: none. [inferred]

## Boundaries

- Setup does not rewrite, reorder, or reformat anything in the user's Hyprland config beyond the one marked line, and never edits the per-file configs for bindings, autostart, or windows. [inferred]
- No migration of a pasted install by setup itself: detection and a documented manual step, never deleting lines the person pasted. [inferred]
- The bindings themselves do not change, including the takeover of Super+Tab and Super+Shift+Tab; a person who wants different keys edits the written file, and setup's digest check then keeps remove from deleting it. [inferred]
- The standard-installation acknowledgment on the marketplace form stays unticked; this spec does not make the plugin a standard install. [inferred]

## Decision Context

One file plus one optional-require line, over one line per snippet or edits to the three per-file configs: a single file is one thing to read, one thing to remove, and one digest to check, and the optional helper makes plugin removal without setup safe by construction. [inferred] Rejected: appending the snippets into the per-file configs, because that touches three files the person owns and cannot be undone cleanly; a plain require line, because a missing module would stop Hyprland from loading the rest of the config; asking Omarchy to load plugin config, because no such hook exists. [inferred] Detection over migration for pasted installs, because setup deleting lines a person pasted into their own config is a worse failure than asking them to do it once. [inferred]
