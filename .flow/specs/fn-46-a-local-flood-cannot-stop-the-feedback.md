# A local flood cannot stop the feedback listener

## Conversation Evidence

> maintainer on omacom/omarchy-plugin-marketplace#10257, at b53e0a5: "`ds/feedback.py:736–746` creates an unbounded thread for each unauthenticated loopback connection before the read timeout and later splice limit, allowing another local account to exhaust threads and terminate the acceptor through an uncaught thread-start failure"

> maintainer, 2026-10-18: closed the request as not planned after seven days without an answer; "please reopen it once the correction is ready."

> user, offered a bounded handler count, a caught thread-start failure and a flood test: "yes"

Facts established on 2026-10-19 by reading the code. `_accept_loop` starts one daemon thread per accepted connection with no limit, and `Thread.start()` raising `RuntimeError` escapes the loop, ending the HTTP or TLS acceptor for the rest of the session. The fn-43 input gate resets only redirected connections that land outside the person's cgroup; a direct connection from another account to the person's own listener on loopback still reaches `accept()`. A handler holds its thread for at most `READ_TIMEOUT` on the block path, and up to `IDLE_TIMEOUT` while a splice of at most `MAX_SPLICES` runs.

## Goal & Context

No number of connections to the feedback ports stops the listener from accepting. Handler threads are bounded, so a flood costs at most a bounded number of threads and, while it lasts, the closing of the connections over the bound. [user, via the maintainer's finding]

## Architecture & Data Models

**One shared bound on live handlers.** Both acceptors, on both families, claim a slot from a single counter before starting a handler, and the handler releases it when it returns, the same claim-and-release shape as the splice count. The bound is `MAX_SPLICES` plus 64, so every splice the pass-through allows still gets a handler, with room for block-path reads alongside. [inferred]

**Over the bound, the connection is closed at once.** No thread is started, the connection is closed, and the log records it at most once a minute, like the splice cap. [inferred]

**A failed thread start is not fatal.** `RuntimeError` from `Thread.start()` releases the slot, closes the connection, logs at most once a minute, and the loop continues. [inferred]

## Edge Cases & Constraints

- A flood still denies the person's own feedback while it lasts; the block path fails closed, which is what a blocked site already does. Authenticating loopback peers is out of scope. [inferred]
- Handlers that outlive a `stop()` keep releasing into the same counter, so a reload never leaks or double-counts slots. [inferred]
- The manifest version moves in the same pull request, since this changes runtime files. [inferred]

## Acceptance Criteria

- **R1:** At most `MAX_CONNS` handler threads run at once across both feedback ports; a connection over the bound is closed without a thread, and the log records the cap at most once a minute. [user]
- **R2:** A `RuntimeError` from starting a handler thread closes that connection and leaves the acceptor accepting. [user]
- **R3:** A test holds the bound full, shows the next connection closed, then frees it and shows an ordinary block-page request answered; a second test makes thread start fail and shows the following request still answered. [user]
- **R4:** `docs/reference.md` names the handler bound beside the splice cap. [inferred]
- **R5:** `manifest.json` carries a version past the last tag. [inferred]

## Boundaries

- Not authenticating loopback peers or extending the nftables gate to direct connections. [inferred]
- Not changing `MAX_SPLICES`, `READ_TIMEOUT` or `IDLE_TIMEOUT`. [inferred]
