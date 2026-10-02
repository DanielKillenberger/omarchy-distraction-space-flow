---
satisfies: [R1, R2, R3, R4, R5, R6, R7, R8]
---
# fn-39-setup-never-installs-through-a.1 Descriptor-walked install helper behind the slice, hook, and both removes

## Description
Add the shared user-level install helper described in the spec and put `sync_slice()`, `sync_hook()`, `remove_slice()`, `remove_hook()` and the hook's installed-check behind it. One task because all four functions and the helper live within 100 lines of each other in `ds/setup.py` and share the refusal contract; splitting them would mean two passes over the same region.

**Size:** M
**Files:** `ds/setup.py`, `ds/wp.py`, `tests/test_setup.py`
**Touches:** [ds/setup.py, ds/wp.py, tests/test_setup.py]

### Approach

- Add the helper beside the other private writers in `ds/setup.py` (near `_replace_bytes` at `ds/setup.py:813`). Signature shape: `(base: Path, dest: Path, data: bytes) -> str | None` returning one of `"written"` / `"current"` / `None` for refused, printing the refusal itself. Type hints per the newer helpers in the file; one-line docstring plus a short rationale paragraph, matching `_replace_bytes` and `state.read_bounded`.
- Walk by descriptor: open the base with `os.open(base, os.O_RDONLY | os.O_DIRECTORY)`, then for each component of `dest.relative_to(base).parent` open it with `os.O_RDONLY | os.O_DIRECTORY | os.O_NOFOLLOW` and `dir_fd=<parent fd>`; on `FileNotFoundError` create it with `os.mkdir(name, 0o755, dir_fd=<parent fd>)` and open it the same way (re-open on `FileExistsError` rather than trusting the race). `ELOOP`/`ENOTDIR` from the no-follow open is the refusal. Close every descriptor in a `finally`.
- Destination: `os.lstat(dest.name, dir_fd=fd)`; `stat.S_ISLNK` or not `stat.S_ISREG` is a refusal. A regular file is read for comparison with `os.open(dest.name, os.O_RDONLY | os.O_NOFOLLOW | os.O_NONBLOCK, dir_fd=fd)` (bounded like `state.read_bounded`); equal bytes return `"current"`.
- Write: `os.open("<tmp name>", os.O_WRONLY | os.O_CREAT | os.O_EXCL, 0o644, dir_fd=fd)` with a name in the existing `.<name>.*.tmp` shape, write loop, `os.fsync`, close, `os.rename(tmp, dest.name, src_dir_fd=fd, dst_dir_fd=fd)`; unlink the temp with `dir_fd=fd` on any `OSError` and re-raise so callers keep the current `cannot install {path}: {e}` message.
- Bases: script -> `launch.data_home()`, fragment -> the XDG config base (the same expression `wp.config_dir()` and `_user_unit_dir()` build on -- factor the `$XDG_CONFIG_HOME or ~/.config` expression if it reads better, do not change its meaning), slice unit -> the same XDG config base.
- Refusal message voice follows `ds/setup.py:1039` and `:1104`: name the path, say setup never writes through a symlink, say deleting it and rerunning setup is the way out. Keep the `cannot install {path}` prefix for the non-symlink failures so `test_install_writes_the_script_before_the_fragment` (`tests/test_setup.py:1447`) still matches.
- `sync_hook()`: call the helper for the script, then the fragment; a `None` from either prints (helper did it) and returns 1 immediately, so a refused script never reaches the fragment and no restart runs -- the same stopping point as today's `OSError` path. Restart once when either returned `"written"`. The pre-computed `changed` gate goes away; the helper decides per destination, which is what makes R4 true.
- `sync_slice()`: same shape, reload-and-start when the helper wrote.
- `remove_hook()` / `remove_slice()`: replace the `path.exists()` gate with an `os.lstat` on the path itself. Missing -> skip, as today. Symlink (including dangling) or any non-regular file -> print in the `remove_hypr` voice (`ds/setup.py:1103`) and continue, returning 0 so `remove()` (`ds/setup.py:1968-1971`) does not short-circuit. Regular file -> unlink as today.
- `wp.installed()` (`ds/wp.py:53`): swap `is_file()` for a no-follow regular-file test on both paths.

### Investigation targets

**Required** (read before coding):
- `ds/setup.py:693-788` — `sync_slice`, `remove_slice`, `sync_hook`, `remove_hook` as they stand
- `ds/setup.py:813-840` — `_replace_bytes`, the atomic-write shape to match
- `ds/setup.py:186-217` — the privileged `stage`/`activate` pair, the descriptor-revalidation precedent
- `ds/state.py:43-79` — `read_bounded`, the no-follow bounded read
- `ds/setup.py:1036-1042`, `ds/setup.py:1100-1106` — the two symlink message voices to copy

**Optional**:
- `tests/test_setup.py:1150-1230` — existing slice tests
- `tests/test_setup.py:1417-1492` — existing hook tests, including the ordering tests the new paths must not break

### Key context

- `os.open`, `os.mkdir`, `os.lstat`, `os.rename` and `os.unlink` all take `dir_fd` on Linux; `tempfile.mkstemp` does not, which is why the temp file is created by hand here.
- The old `changed` computation used `path.read_bytes()`, which follows a symlink — that is exactly why a planted link with matching content is invisible today.
- Run the suite as `PATH=/usr/bin:$PATH python3 -m unittest discover -s tests`; a mise `python3` shim defeats the fake binaries the harness puts on PATH.

### Acceptance

- [ ] A symlink at the script, fragment, or slice-unit path is refused, the target keeps its bytes, the link survives, and setup returns non-zero (R1)
- [ ] A symlink at `wireplumber`, `scripts`, `wireplumber.conf.d`, `systemd`, or `systemd/user` is refused by name with nothing created in its target (R2)
- [ ] A fifo, socket, or directory at a destination is refused without being opened (R3)
- [ ] The refusal fires even when the link's target already holds the shipped bytes (R4)
- [ ] On an ordinary tree the first install, the no-op rerun, and the drift rewrite behave exactly as the existing tests assert, mode 0644, no temp file left behind on a failed write (R5)
- [ ] Remove unlinks only regular files; a symlink, a dangling symlink, or another irregular file is left and reported, and remove still succeeds and runs its later steps (R6)
- [ ] The hook's installed-check reads a symlinked path as not installed (R7)
- [ ] Tests cover every case above for all three destinations inside the harness sandbox, and the full suite passes (R8)

## Acceptance
- [ ] TBD

## Done summary
All three user-level installs -- the WirePlumber hook script, its config fragment, and the systemd slice unit -- now go through one helper in `ds/setup.py`. It opens every name below the XDG base relative to the descriptor of the one before it without following a link, creates a missing directory there, and refuses by name anything that is a symlink or not a directory; the destination is checked the same way, and the bytes go to a temporary file created relative to that same descriptor and renamed onto the destination at mode 0644. The XDG root itself stays the person's, opened as given, so a dotfiles repository that links `~/.config` keeps working.

A refusal fails the run and says which path to delete. It fires whether or not the link's target already holds the shipped bytes, which the old content-first comparison could not see.

Both removes unlink only a regular file. A symlink, a dangling one included, or anything else holding one of those names is left in place and reported. A fragment that is not ours to delete keeps its script with it, because the feature that fragment still requires would otherwise have no script to load -- the first review round caught that. `wp.installed()` stopped resolving its two paths through a link, so a planted one no longer reads as this plugin's own hook.

Eleven new tests cover the symlinked destination, the irregular destination (fifo, socket, directory), the symlinked directory component at each of the six names below the two XDG bases, the link whose target already holds the shipped bytes, the drift rewrite, the failed write that leaves no temporary file, mode 0644, and the three remove cases -- all inside the harness's temporary XDG root. The suite is 532 tests, green.

Reviewed by gpt-5.6-sol-high through cursor: NEEDS_WORK on the fragment/script removal order, the incomplete matrix, and a missing remedy line on the race path; SHIP after the fixes.
## Evidence
- Commits: ffcf9bd, 8ca867a
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests
- PRs: