# Leaving the space never lands on a special workspace

## Conversation Evidence

> user (turn 1): "distration space is conflicting with the super+s space. when i'm on dspace and i wanna get out by super+ctrl+shift+d it opens/closes the super+s space. Smth weird is going on"

> user (turn 2): "should have done this in a new spec and branch"

Facts established on 2026-09-23 from the live session: `hyprctl workspaces -j` listed `1` (id 1, 3 windows), `distraction` (id -1337, 4 windows), and `special:scratchpad` (id -98, 1 window). Super+S is Omarchy's scratchpad toggle.

## Goal & Context
<!-- Source-tag breakdown: 30% [quote] / 70% [inferred] -->

Super+Ctrl+Shift+D on the space is meant to take the person back to a normal workspace. Instead it opens or closes the Super+S scratchpad. [quote]

`toggle` on the space calls `leave`, which calls `hypr.cycle("next")`. The cycle keeps every workspace with windows except the space itself and sorts them by id. Hyprland gives named and special workspaces negative ids, so from the space (-1337) the next id is the scratchpad (-98). Focusing `name:special:scratchpad` toggles it. Super+Tab and Super+scroll share the same cycle, so they can reach the scratchpad too. [inferred]

## Architecture & Data Models
<!-- Source-tag breakdown: 100% [inferred] -->

`hypr.cycle` excludes any workspace whose name starts with `special:`, beside the existing exclusion of the space. Nothing else changes: destination order, the wrap-around, and the no-op when no other workspace has windows stay as they are. [inferred]

## Edge Cases & Constraints
<!-- Source-tag breakdown: 100% [inferred] -->

- When the only workspace with windows other than the space is a special one, leaving is a no-op and returns success, as it does today when no other workspace has windows. [inferred]
- A visible special workspace does not change `activeworkspace`, so the cycle's starting point is unaffected. [inferred]

## Acceptance Criteria

- **R1:** From the space with ids as in the live session, `cycle("next")` focuses `name:1` and never a `special:` workspace. Errors: none. [paraphrase]
- **R2:** `cycle("next")` and `cycle("prev")` from a numbered workspace never focus a `special:` workspace. Errors: none. [inferred]
- **R3:** A test fails on the current code for R1 and passes after the change. Errors: none. [inferred]

## Boundaries
<!-- Source-tag breakdown: 100% [inferred] -->

- Out of scope: the keybinds themselves, and whether the space should remember the workspace it was entered from. [inferred]
