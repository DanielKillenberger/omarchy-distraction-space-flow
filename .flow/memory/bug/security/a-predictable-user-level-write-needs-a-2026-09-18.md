---
title: A predictable user-level write needs a descriptor walk, not a path check
date: "2026-09-18"
track: bug
category: security
module: "ds/setup.py,ds/wp.py"
tags: [symlink, xdg, no-follow, marketplace-review, setup]
problem_type: security
symptoms: Marketplace review blocked the 3.2.0 verification request
root_cause: sync_hook and sync_slice wrote fixed-name paths with Path.write_bytes() after mkdir(parents=True)
resolution_type: fix
---

## Problem
`sync_hook()` wrote `~/.local/share/wireplumber/scripts/<name>` and `~/.config/wireplumber/wireplumber.conf.d/<name>` with `Path.write_bytes()` after `mkdir(parents=True, exist_ok=True)`, and `sync_slice()` did the same for the slice unit. Every one of those names is fixed in advance, so a process running as the same person could plant a symlink at a name or at a directory below it and setup would write the plugin's bytes over a file it was never asked to touch. The marketplace reviewer blocked the verification request on it (omacom/omarchy-plugin-marketplace#7042).

## What Didn't Work
An `lstat`-then-write on path strings would have answered the finding as written and still lost the race: the link can be swapped between the check and the open. The old code was worse than it looked in a second way -- it compared content first with `path.read_bytes()`, which follows the link, so a planted link whose target already held the shipped bytes was invisible.

## Solution
One helper, `_install_user_file(base, dest, data)` (f422a34). The XDG base is opened as given, then every name below it is opened relative to the descriptor of the one before with `O_DIRECTORY | O_NOFOLLOW`, a missing directory created with `os.mkdir(..., dir_fd=)` and whatever won the race reopened rather than trusted. The destination is `os.lstat(name, dir_fd=)`; a symlink or a non-regular file is refused by name and the run fails. The write is `os.open(..., O_CREAT | O_EXCL, dir_fd=)`, `fchmod` 0644 so the umask does not decide, `fsync`, then `os.rename(..., src_dir_fd=, dst_dir_fd=)`. Removes use `lstat` and unlink only a regular file. `wp.installed()` stopped using `is_file()`, which resolves through a link.

## Prevention
Any write this plugin makes to a name it fixes in advance goes through `_install_user_file`; `Path.write_bytes()` and `mkdir(parents=True)` on such a path are the smell. The base is the only thing not judged, because a dotfiles repository may legitimately link `~/.config` or `~/.local/share` -- judging it would break a real machine for no gain. Two related traps the review round caught: a remove that leaves a foreign fragment must keep its script too (WirePlumber will not start with a required feature whose script is gone), and doc prose saying remove "deletes only the files it wrote" claims an ownership record these three paths do not have -- the guarantee is "a plain file at this path", nothing more.
