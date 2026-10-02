# Setup never installs through a symlinked user path

## Conversation Evidence

> user (turn 1): "https://github.com/omacom/omarchy-plugin-marketplace/issues/7042#issuecomment-5731920072 pls fix in new spec review with sol in cursor agent and merge pr once SHIP and make a release and update the issue to get resolved"

The finding itself, quoted verbatim from the marketplace reviewer on the verification request for `cdc7dad` (3.2.0):

> HANCORE-linux (collaborator): "Blocked: ds/setup.py sync_hook() writes predictable WirePlumber paths with Path.write_bytes() without rejecting symlink targets or parents; a same-user process can redirect setup into overwriting another user-writable file."

Facts established by reading the code at `42efbc3`: `sync_hook()` writes the WirePlumber script and fragment with `Path.write_bytes()` after `mkdir(parents=True, exist_ok=True)`; `sync_slice()` writes the slice unit under the user's systemd config directory the same way; both removes gate on `exists()` and unlink whatever is at the name; every other user-level write in the repository already goes through `mkstemp` plus `os.replace`, which replaces a symlink instead of writing through it; the Hyprland block added in 3.2.0 already refuses a symlinked destination by name and says so.

## Goal & Context
<!-- Source-tag breakdown: 30% [quote] / 70% [inferred] -->

The marketplace verification request for 3.2.0 is blocked on one finding: setup installs the WirePlumber hook script and its config fragment with `Path.write_bytes()` at paths whose names are known in advance, and it checks neither the destination nor the directories it creates for symlinks. [quote] A process running as the same person can plant a symlink at either name, or at a directory setup would create beneath it, and setup then writes the plugin's bytes over whatever that link points at. [quote]

The slice unit install is the same three lines against the user's systemd config directory, and the two removes unlink the same predictable names. Fixing the hook alone would leave the identical shape one function away and invite a second round of review. [inferred]

The plugin already knows how to do this: every other user-level write goes through a temporary file and a rename, the Hyprland writer refuses a symlinked destination and says so, and the privileged transaction walks its destinations by descriptor and revalidates before the rename. This spec brings the two remaining direct writes up to the privileged half's bar and states the rule once, in a shared helper, so a later install path inherits it. [inferred]

## Architecture & Data Models
<!-- Source-tag breakdown: 100% [inferred] -->

**One helper for a file setup owns.** A private helper in `ds/setup.py` installs bytes at a user-level path setup owns. It takes the base directory the destination sits under, the destination, and the bytes. It answers with three outcomes -- the bytes were written, the destination already held them, or the install was refused -- and prints the reason itself on a refusal. Callers keep their own restart decisions and exit codes. [inferred]

**The walk is by descriptor, not by path.** The helper opens the base directory, then opens each component below it relative to the previous descriptor with no-follow and directory-only flags, creating a missing component relative to that descriptor and opening it the same way. A component that exists and is a symlink, or is not a directory, fails that open and is refused by name. Nothing in the walk is a bare path string re-checked between calls, so the destination the final write lands in is the directory each check actually passed, not one that shares its name. [inferred]

**The destination is checked and written relative to that descriptor.** The final component is inspected with a no-follow stat relative to the parent descriptor: a symlink, or anything that is not a regular file, is refused by name with the reason and nothing is written. A regular file whose bytes already match is left alone and reported as current. Otherwise the bytes go to a temporary name created relative to the same descriptor with exclusive-create at mode 0644, are flushed, and are renamed onto the destination relative to that descriptor -- the whole-file replace the rest of the plugin already uses, with the directory pinned open across it. A failed write removes the temporary file. [inferred]

**The base is the person's XDG root; everything below it is judged.** The script's base is the XDG data home, the fragment's and the slice unit's base is the XDG config home. The base itself is not the plugin's to judge, so a dotfiles install that links `~/.config` or `~/.local/share` keeps working. Every component below it is judged, including `wireplumber`, `wireplumber.conf.d`, `scripts`, `systemd`, and `user`: a dotfiles repository that links one of those is refused and told why, because setup would otherwise be writing into a directory it does not own. [inferred]

**The two call sites.** `sync_hook()` installs the script through the helper and then the fragment, keeping its order so a required feature never points at a missing script, and restarts WirePlumber once when either was written. `sync_slice()` installs the slice unit through the helper and reloads the user manager when it was written. A refusal is reported and returns the same non-zero code the existing write failure returns, and the caller stops exactly where it stops on a write failure today: a refused script never reaches the fragment, and no restart or reload runs. Because the helper decides per destination, the refusal is reported on a machine whose bytes already match as well -- the old code compared content first and could not see the link at all. [inferred]

**Remove treats a symlink as not ours.** `remove_hook()` and `remove_slice()` unlink only a regular file. A symlink at either name, including one whose target is gone, is left in place and reported, the way the Hyprland remove already leaves a symlinked Lua file and says so, and the remove still succeeds. Anything else that is not a regular file is left and reported the same way. [inferred]

**One reader is corrected with them.** The check that says whether the hook is installed resolves its two paths through a symlink today, so a foreign link pointing at any file would read as installed and the listener would report the hook live when it is not. It is changed to the same no-follow regular-file test the helper uses. [inferred]

## Edge Cases & Constraints
<!-- Source-tag breakdown: 100% [inferred] -->

- Every check is a no-follow open or stat relative to a descriptor the helper holds; nothing resolves a destination and then trusts the resolved string. A link swapped in after a check fails the next fd-relative call rather than redirecting it. [inferred]
- A directory the person has symlinked at or above the base -- `~/.config` itself, or `~/.local/share` -- is theirs and stays legal. Only components below the base are judged, and a dotfiles install that links one of them is refused with the path named, not silently written through. [inferred]
- A refused hook install leaves the reactive mute path in place, which is what runs today on a machine without WirePlumber; the person loses the pre-emptive hook, not the feature. The refusal still fails the setup run, because it is a state only the person can fix. [inferred]
- `remove_hook()` keeps its ordering: the fragment goes before the script, so WirePlumber is never left requiring a feature whose script is gone. [inferred]
- A destination that is a fifo, socket, or device is refused by the same regular-file check, and the helper never opens the destination itself, so a fifo cannot stall setup. [inferred]
- A dangling symlink at a remove destination reads as absent to the current existence check and is silently skipped today; it is a symlink to the new check, so it is reported and left. [inferred]
- A person whose install was refused can clear it by deleting the link that was named and rerunning setup; nothing about the refusal is sticky. [inferred]
- The manifest version moves to 3.3.0 in the same pull request, because the version check fails a runtime change under a version that already has a tag, and `v3.2.0` exists. [inferred]

## Acceptance Criteria

- **R1:** With a symlink planted at the hook script path, at the fragment path, or at the slice unit path, setup writes nothing through it: the link's target keeps its bytes, the link is still a link, the refusal names the path and says setup does not write through a symlink and that deleting it and rerunning setup is the way out, and the call returns the same non-zero code a failed write returns. Errors: the refusal is the error surface; a refused script never reaches the fragment, and no restart or reload runs. [inferred]
- **R2:** With a symlink planted at any directory component beneath the XDG base -- `wireplumber`, `scripts`, `wireplumber.conf.d`, `systemd`, or `systemd/user` -- the install is refused the same way, naming that component, and no file is created inside the link's target. Errors: as R1. [inferred]
- **R3:** With a destination that exists as a fifo, a socket, or a directory, the install is refused by name without opening it. Errors: as R1. [inferred]
- **R4:** A refusal is reported whether or not the destination's target already holds the shipped bytes; the content comparison never precedes the symlink check. Errors: no error surface beyond R1. [inferred]
- **R5:** On an ordinary tree the three installs behave exactly as they do today: a first run creates the parents and writes the bytes at mode 0644, a rerun with matching bytes writes nothing and does not restart WirePlumber or reload the user manager, changed bytes rewrite the file and trigger the one restart or reload, and a base or component that is an ordinary directory is never refused. Errors: an `OSError` during the write is reported with the path and returns non-zero, leaving no temporary file behind. [inferred]
- **R6:** `setup --remove` unlinks the hook script, the fragment, and the slice unit only when each is a regular file; a symlink at any of those names, including a dangling one, and anything else that is not a regular file, is left in place and reported, and the remove still succeeds and still runs its later steps. Errors: an `OSError` during the unlink is reported and returns non-zero, as today. [inferred]
- **R7:** The check that reports whether the WirePlumber hook is installed treats a symlink at either path as not installed, so a foreign link is never read as the plugin's own file. Errors: no error surface beyond the report. [inferred]
- **R8:** All three user-level installs go through the one helper, and a test exercises each of R1, R2, R3, R4, and R5 against all three destinations, plus both remove cases in R6 and the reader in R7, against the harness's temporary XDG root with no write escaping it. Errors: none. [inferred]
- **R9:** `manifest.json` declares 3.3.0; `docs/marketplace-submission.md` states that setup's user-level installs carry the same no-follow discipline as the privileged transaction; `docs/reference.md` says setup refuses a symlinked destination or directory rather than writing through it and that remove leaves one in place; `docs/internals.md` describes the descriptor walk and the refusal in the slice and WirePlumber sections. Errors: none. [inferred]

## Boundaries

- No change to what the hook, the fragment, or the slice unit contain, nor to when WirePlumber is restarted or the user manager reloaded. [inferred]
- No change to the privileged transaction, the sudoers grant, or the nft wrapper: they already stage into root-owned directories and revalidate by descriptor, and the reviewer did not raise them. [inferred]
- No change to the Hyprland block, the launcher entry writes, or the state JSON writer. They replace through a rename, which lands on the link rather than on its target, so no write of theirs reaches a planted link's destination; bringing them onto the helper is a separate change with its own dotfiles compatibility question. [inferred]
- No status surface for a refused hook or slice install: the refusal is printed and fails the run, and a persisted note like the Hyprland one is out of scope. [inferred]
- Setup does not delete, move, or repair a symlink a person planted or a dotfiles repository placed; it refuses and reports. [inferred]

## Decision Context

Refuse and report, over resolving the link and writing to its target: a resolve-then-write cannot be made race-free from user space, and a plugin that follows a link into a directory it does not own is the finding restated rather than fixed. [inferred] A descriptor-relative walk, over an `lstat`-then-write on path strings: the string check can be defeated by a swap in the gap between the check and the write, which is the same class of bug one window narrower, and the privileged half of this same file already works by descriptor. [inferred] One shared helper, over a check pasted into each call site, because the next install path should inherit the rule instead of re-deriving it. [inferred] Judging only the components beneath the XDG base, over everything up to home, because a dotfiles install that links `~/.config` is ordinary and refusing it would break a legitimate machine for no security gain. [inferred] A refusal fails the setup run, unlike the Hyprland writer which reports and continues: there the person's own file is the destination and setup can hand them the one line to paste, so the degrade is complete, while a refused hook or slice leaves a feature simply not installed with no alternative offered. [inferred] Extending the fix to the slice install and both removes, over the single function the reviewer named, because they are the same three lines and a second blocked round costs more than the extra tests. [inferred]
