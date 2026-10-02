---
title: A composite ok reply is not evidence that one subsystem applied
date: "2026-09-20"
track: bug
category: integration
module: "ds/setup.py,ds/listener.py"
tags: [ipc, listener, reload, timeout, site-block, review]
problem_type: integration
symptoms: site-block on exited 1 saying no listener is running while the block was applied; a healthy turn-on reported failure
root_cause: "used the listener's composite ok reply, and a 2s default ping timeout, as proof that site blocking specifically had been applied"
resolution_type: fix
related_to: [bug/integration/ask-once-setup-answer-fell-back-to-2026-09-05, bug/integration/pulse-prop-stripped-with-split-altered-2026-09-06]
---

## Problem

`distractions site-block on` had to end with the block applied, and the listener is what applies it. The obvious signal was there: `state.request_reload()` returns True when the listener answers `ok`. Both readings of it were wrong, and both fail on healthy machines.

The reply is composite. `listener.py:459` computes it as `result in ("on", "off") and item["links_ok"]`, then ands in the launcher-entry sync. A default browser another program had taken made the reply `error`, so a command that only wanted to know about site blocking reported "no listener is running" and exited 1 while the table was up and correct.

The timeout was the operation's, not the caller's. `state.request_reload()` defaults to `timeout=2.0`, while a reconcile resolves every listed host (`BATCH_DEADLINE` 10s) and then applies the table (`COMMAND_TIMEOUT` 10s). `listener._ask` exists precisely because of this and uses `_reload_wait()`. A two-second ping times out mid-reconcile and returns False, so a working turn-on reported failure.

## Solution

Ask with the listener's own budget, then read the listener's own record of the thing you asked about:

```python
since = state.now_iso()
state.request_reload(timeout=listener._reload_wait())
if _block_applied(since):   # state.json site_block in ("on","off") and observed_at >= since
    return 0
```

The listener writes `state.json` before it replies (`take_result`: `write_state()` at :458, `_reply_waiters` at :460), so one read after the answer needs no polling. `on` and `off` both count as applied policy, because a list with no hosts in it correctly reconciles to `off` — the same two values the listener itself counts as a clean reconcile. `state.listener_pid()` then tells "no listener at all" from "a listener that could not apply it", which are different messages to a person.

## Prevention

Before using an IPC acknowledgement as evidence, read what the responder ands into it. An `ok` that covers three subsystems is evidence for none of them individually; the per-subsystem record it wrote is.

A test that fakes the RPC must fake it in the responder's order — record, then answer — and must carry the reply and the recorded state as separate knobs. Faking only the reply cannot catch a conflated signal: the case that exposes it is reply=False with the state applied.

Check a default timeout against the operation it waits on. `request_reload`'s 2s default is fine for a fire-and-forget config ping and wrong for anything that waits for the work; the module that owns the work already had the right budget.
