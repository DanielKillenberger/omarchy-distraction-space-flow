# Feedback ports only reach the person's own listener

## Conversation Evidence

> maintainer on omacom/omarchy-plugin-marketplace#8628, at 268018a: "`ds/feedback.py:99–140` continues after a failed bind to the predictable HTTP port, while `distractions-nft:106–140` still installs redirects to that port. With site blocking enabled, another local user can reserve `127.0.0.1:28080` and receive requests to a listed plaintext HTTP site, including their request data, so this commit cannot be verified yet."

> user, offered "only redirect while bound" or "kernel-enforced port ownership" (recommended): "yes"

Facts established on 2026-10-02 by reading the code and a live test. The wrapper's `output_nat` chain redirects TCP 80 and 443 for listed addresses to 28080 and 28443 on every output path of the machine, not only the person's. The listener binds those ports unprivileged and, on a failed bind, notifies and carries on. `feedback.stop()` closes the listeners on every reload while the redirect stays loaded, so the ports are free for a moment even when every bind succeeds. The running listener sits in `user.slice/user-1000.slice/user@1000.service/session.slice/wayland-wm@hyprland.desktop.service`. In an unprivileged network namespace on kernel 7.2.3 with nftables 1.1.7, an `input` rule `socket cgroupv2 level N "<path>" accept` followed by a reset accepted a redirected connection to a listener inside the named cgroup and refused one outside it, but only when the rule was limited to the opening SYN: the handshake's final ACK finds a request socket, which carries no cgroup, and is reset.

## Goal & Context
<!-- Source-tag breakdown: 30% [user] / 70% [inferred] -->

Traffic the firewall redirects to the feedback ports reaches only a socket the person owns. Whoever holds 28080 or 28443 when the redirect fires, a socket outside the person's login cgroup never completes a connection on them, so no other account receives a listed site's request. [user, via the maintainer's finding] The fix is enforced by the kernel rather than by the listener's bind succeeding, so the restart window and the failed bind are closed by the same rule. [inferred]

## Architecture & Data Models
<!-- Source-tag breakdown: 100% [inferred] -->

**An input chain gates new connections to the feedback ports.** The wrapper's table gains `chain input`, a filter hook on input at the filter priority with policy accept, holding two rules: TCP to destination ports 28080 and 28443 with the SYN flag set and ACK clear is accepted when the receiving socket is in `user.slice/user-<uid>.slice` (`socket cgroupv2 level 2`), and the same match otherwise is reset. Packets after the SYN pass untouched, since a connection that got past the SYN belongs to the person's socket. [inferred]

**The cgroup is the person's whole login, not the listener's unit.** Level 2 is the boundary another account cannot cross: only root and that uid's own manager place processes under `user-<uid>.slice`. The listener's own unit varies with how Hyprland was started, and processes of the same person are already trusted with everything the listener has. The path is derived from `SUDO_UID` like the slice path, never from input. [inferred]

**`check ds` verifies the new chain.** The expected listing includes the input chain, so a missing or altered gate reads as drift and the listener repairs it the way it repairs the other chains today. [inferred]

**The listener keeps its bind behaviour.** A failed bind still notifies and still reports pass-through as unavailable; with the gate in place the redirected connection is reset, which is what "blocked sites still fail fast" already promises. [inferred]

## Edge Cases & Constraints
<!-- Source-tag breakdown: 100% [inferred] -->

- The gate keys on the destination port only; nothing else in the table, and no address, changes. The two ports are this plugin's, so the reset touches no other service. [inferred]
- IPv4 and IPv6 are both covered: the `inet` table's input hook sees both families, and the match carries no family. [inferred]
- An installed helper keeps the old table until `distractions setup` reinstalls the wrapper; the update's release notes name that rerun. [inferred]
- `flush ds` is unchanged: destroying the table removes the gate together with the redirect it guards. [inferred]
- The manifest version moves in the same pull request, since this changes runtime files. [inferred]

## Acceptance Criteria

- **R1:** `render_table` emits an input chain whose first rule accepts a SYN without ACK to TCP 28080 or 28443 when the receiving socket is in `user.slice/user-<uid>.slice`, and whose second rule resets the same match otherwise. [inferred]
- **R2:** `check ds` accepts the live listing of that table and reports drift when the input chain or either rule is missing or altered. [inferred]
- **R3:** In a live namespace test, a connection redirected to 28080 reaches a listener inside the person's cgroup and is refused for a listener outside it, for IPv4 and IPv6. [inferred]
- **R4:** `docs/internals.md` and `docs/reference.md` describe the gate and why it covers the failed bind and the reload window. [inferred]
- **R5:** `manifest.json` carries a version past the last tag. [inferred]

## Boundaries

- Not narrowing the redirect to the person's own traffic; other accounts' traffic to listed addresses being redirected is a separate design question. [inferred]
- Not changing how an outdated installed helper is detected. [inferred]
