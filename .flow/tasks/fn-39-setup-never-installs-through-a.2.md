---
satisfies: [R9]
---
# fn-39-setup-never-installs-through-a.2 Version bump and the docs that describe the refusal

## Description
Move `manifest.json` to 3.3.0 and describe the new refusal where the three docs already describe these installs. The version bump is required by CI, not optional: the version check fails a pull request that touches runtime files while the declared version already has a tag, and `v3.2.0` exists.

**Size:** S
**Files:** `manifest.json`, `docs/marketplace-submission.md`, `docs/reference.md`, `docs/internals.md`
**Touches:** [manifest.json, docs/marketplace-submission.md, docs/reference.md, docs/internals.md]

### Approach

- `manifest.json:5` — `"version": "3.2.0"` -> `"3.3.0"`. It is the only literal copy of the current version in the repository; the two version strings in `docs/marketplace-submission.md` are historical facts about past releases and stay.
- `docs/marketplace-submission.md:29` (`### User-level setup and data`) — state that the user-level installs carry the same discipline the privileged transaction already gets, in the vocabulary line `:15` uses for it (no-follow, regular-file checks): the slice unit and the WirePlumber script and fragment are installed through descriptors opened without following, and a symlinked destination or a symlinked directory beneath the XDG base is refused and reported rather than written through.
- `docs/marketplace-submission.md:39` — the remove sentence gains the clause that a symlinked destination is left in place and reported.
- `docs/reference.md:39` and `:41` — the slice-copy sentence and the WirePlumber-install sentence gain the refusal in user-facing terms; `:56` (`## Remove`) gains the leave-in-place behaviour. Match the file's register: what setup does, not how.
- `docs/internals.md:13` (`## The slice`) and `:117` (the WirePlumber paragraph) — the mechanism: the descriptor walk from the XDG base, what is judged and what is not, what is printed, and that a refusal fails the run while a remove's refusal does not.

### Investigation targets

**Required** (read before coding):
- `docs/marketplace-submission.md:13-25` — the privileged-transaction paragraph whose vocabulary the new sentence should echo
- `docs/reference.md:37-56` — the two install sentences and the remove paragraph
- `docs/internals.md:13`, `docs/internals.md:117` — the two mechanism paragraphs
- `.github/workflows/version-check.yml` — why the bump is mandatory

### Acceptance

- [ ] `manifest.json` declares 3.3.0 and no other file hardcodes a current version (R9)
- [ ] The submission doc states the user-level no-follow discipline and the remove behaviour in the same vocabulary as the privileged half (R9)
- [ ] `docs/reference.md` says setup refuses a symlinked destination or directory and that remove leaves one in place (R9)
- [ ] `docs/internals.md` describes the descriptor walk and the refusal in both the slice and WirePlumber sections (R9)
- [ ] The described behaviour matches what shipped in the first task, message wording included (R9)

## Acceptance
- [ ] TBD

## Done summary
`manifest.json` declares 3.3.0, which the version check needs for a runtime change under a version that already has a tag.

`docs/reference.md` says the slice unit and the two WirePlumber files are written without following a link, that a symlink at one of them or at a directory on the way to it is refused by name with the run failing, and that remove deletes a plain file at those three paths and leaves anything else where it is. `docs/internals.md` describes the helper in the slice section -- the descriptor walk from the XDG base, what is judged and what is not, the exclusive-create and rename, the removes -- and the WirePlumber section points at it and adds the fragment-keeps-its-script rule and the no-follow installed check. `docs/marketplace-submission.md` puts the three installs in the same no-follow vocabulary the privileged transaction already had, and says removal's provenance at those paths is the path rather than a record.

Two review rounds on this task, both on scope claims rather than the version: the first that "the files it wrote" claimed an ownership check that is not there for those three paths, the second that "every file setup installs under your directories" swept in the launcher entries and the Hyprland file, which keep their own writers and records. Both are corrected; the third round was SHIP.
## Evidence
- Commits: fe72ec4, c2aa78d, 47e34aa
- Tests: PATH=/usr/bin:$PATH python3 -m unittest discover -s tests, omarchy plugin validate <clean export of HEAD>
- PRs: