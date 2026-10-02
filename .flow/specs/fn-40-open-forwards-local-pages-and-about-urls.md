# Open forwards local pages and about: URLs

## Conversation Evidence

> user (turn 5): "my agents struggle to xdg-open links for me i think?"

Pasted with that turn, from another agent's session notes dated 2026-09-20:

> "The owner's xdg-open wrapper accepts HTTP URLs and rejected a local HTML path while returning success. Hosting the four existing images and a side-by-side page in a dedicated temporary loopback directory opened successfully"

> user (turn 6, answering "Do you want me to spec this and fix it through Flow-Next?" after the scope was proposed as `file` and `about` only): "yes /flow-next:flow"

Facts established by reproducing on the installed plugin at `f422a34` on 2026-09-20: `distractions open /path/t.html` and `distractions open file:///path/t.html` both print the usage line and exit 2; `~/.config/mimeapps.list` names the plugin's handler for `text/html`, `x-scheme-handler/http`, `x-scheme-handler/https`, `x-scheme-handler/about`, and `x-scheme-handler/unknown`, which is what `xdg-settings set default-web-browser` writes; `xdg-open /path/t.html` then fell through to the next `text/html` handler, which on this machine is nvim, and hung until it was killed. Without a terminal nvim exits at once, which is the "rejected while returning success" the pasted note describes. An unlisted http(s) URL is forwarded to the recorded previous browser and opens.

## Goal & Context
<!-- Source-tag breakdown: 20% [quote] / 80% [inferred] -->

Setup makes the plugin the default browser, and `xdg-settings` hands a default browser `text/html` and the `about` scheme along with http and https. `open` accepts only http(s) URLs and names, so a local page sent through `xdg-open` is refused, and `xdg-open` quietly tries the next handler on the machine. [paraphrase] The person sees nothing open, and an agent that asked for the page is told it worked. [quote]

The plugin took the default-browser seat, so it owes the person what the browser it displaced did with a local page: open it. Nothing about a local file or an `about:` page belongs to the distraction space, so both go where every unlisted link already goes, the recorded previous browser. [inferred]

## Architecture & Data Models
<!-- Source-tag breakdown: 100% [inferred] -->

Target resolution gains two forwardable shapes beside the unlisted http(s) URL. [inferred]

**A URL whose scheme is `file` or `about`** resolves to a forward target carrying the URL exactly as given. No host is read from it and it is never matched against the list, so it can never open in the space. [paraphrase]

**A bare argument that names an existing regular file** resolves to a forward target carrying that file's absolute `file://` URL, percent-encoded. Name lookup keeps its place: a list entry name, then a catalog name, are tried first, and the file test runs only when neither matched, so no existing invocation changes meaning. [inferred]

Forwarding itself is unchanged. The forward target goes through the same forwarder as an unlisted link: the recorded previous handler's Exec line with the URL in its field code, the fallback forwarder when no usable handler is recorded, never inside the slice, with the plugin's audio identity stripped. [inferred]

## API Contracts
<!-- Source-tag breakdown: 100% [inferred] -->

`distractions open [--app] [url | path | name] [browser flags...]`

| Target | Result | Exit |
|---|---|---|
| `file://` URL | Forwarded as given | 0, or 1 when no browser could be started |
| `about:` URL | Forwarded as given | 0, or 1 |
| Existing regular file, no list or catalog name matches | Forwarded as its absolute `file://` URL | 0, or 1 |
| Any other non-http(s) scheme | Usage line on stderr, nothing launched | 2 |
| Bare argument that is no name and no existing regular file | Usage line on stderr, nothing launched | 2 |

`--app` and browser flags apply to these forward targets exactly as they apply to an unlisted http(s) URL. The usage line names the local page beside the http(s) URL. [inferred]

## Edge Cases & Constraints
<!-- Source-tag breakdown: 100% [inferred] -->

- A `file://` URL is forwarded without testing that the file exists; the browser reports a missing file the way it does when it is the default itself. The existence test applies only to a bare path, where it is what tells a path from a mistyped name. [inferred]
- A bare path to a directory, a fifo, a socket, or a device is not a regular file and exits 2. The test follows a symlink, because the browser will open whatever the link reaches. [inferred]
- A path is made absolute against the caller's working directory before it is forwarded, because the forwarded browser does not share it. Spaces, `#`, `?`, and `%` in a file name are percent-encoded so the browser reads them as part of the path. [inferred]
- The file is never opened, read, or sniffed for type; any existing regular file forwards, as it would with the previous browser as the default. [inferred]
- A local page never enters the slice and never gets the distraction profile, whatever it links to; links followed inside it are the forwarded browser's business. [inferred]
- Site blocking, link routing for listed hosts, and exit codes for http(s) targets are untouched. [inferred]

## Acceptance Criteria

- **R1:** `open file:///abs/page.html` forwards exactly that URL to the recorded previous handler and exits 0, and nothing is launched in the slice. Errors: no recorded or usable handler takes the fallback forwarder, and no startable browser exits 1 with the existing notice, both exactly as an unlisted http(s) URL does today. [paraphrase]
- **R2:** `open about:blank` forwards exactly that URL the same way and exits 0. Errors: as R1. [paraphrase]
- **R3:** `open <path>` where the path is an existing regular file and matches no list entry or catalog name forwards the file's absolute, percent-encoded `file://` URL and exits 0; a relative path is resolved against the caller's working directory, and a name containing a space or `#` arrives as one intact URL. Errors: a path that does not exist, or names a directory or other non-regular file, prints the usage line and exits 2 with nothing launched. [inferred]
- **R4:** A bare argument that matches a list entry or catalog name opens that entry even when a file of the same name exists in the working directory. Errors: no error surface beyond R3. [inferred]
- **R5:** Every other non-http(s) scheme, `mailto:`, `javascript:`, `data:`, and `ftp:` among them, still prints the usage line and exits 2 with nothing launched. Errors: this criterion is the error surface. [paraphrase]
- **R6:** `--app` and browser flags reach a `file://` forward target the way they reach an unlisted http(s) URL. Errors: no error surface beyond R1. [inferred]
- **R7:** The usage line and the reference's `open` row describe local pages and `about:` URLs as forwarded. Errors: none. [paraphrase]
- **R8:** A test fails on the current code for each of R1, R2, and R3 and passes after the change, and tests cover R4, R5, and R6. Errors: none. [paraphrase]

## Boundaries
<!-- Source-tag breakdown: 100% [inferred] -->

- No other scheme is forwarded. `x-scheme-handler/unknown` stays claimed by `xdg-settings` and stays refused. [paraphrase]
- Setup keeps calling `xdg-settings`; it does not rewrite the MIME defaults to give `text/html` back. [inferred]
- What `xdg-open` does after a refusal, including which handler it tries next, is not the plugin's to change. [inferred]
- No list matching for local pages: a local file is never a distraction. [inferred]

## Decision Context

Two smaller fixes were on the table. Releasing `text/html` at setup would stop the plugin receiving local pages at all, but `xdg-settings` sets those defaults as one unit, and undoing part of it by hand means owning MIME bookkeeping that remove would then have to reverse. Forwarding every scheme the handler receives would also fix it, and would make the plugin pass `javascript:` and `data:` URLs along blindly to a browser; a handler that refuses what it does not understand is the safer default, and `file` and `about` are the two the default-browser seat demonstrably receives. [paraphrase]

Names resolve before paths so the change cannot alter what any existing call does. The cost is that a file named like a catalog product needs `./` in front of it, which is the ordinary shell answer to the same ambiguity. [inferred]
