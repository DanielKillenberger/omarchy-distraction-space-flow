# Links a native app can open hand off to that app

## Conversation Evidence

> user (turn 1, relayed bug report from another session): "when distraction-space is the default browser, open.spotify.com links open the Spotify web player in the browser instead of the Spotify desktop app."
> user (turn 1, relayed): "Expected: open.spotify.com/{track,album,playlist,artist,episode,show}/<id> (with or without the intl-xx/ prefix and ?si= query) opens in the Spotify app as spotify:<type>:<id>, via the spotify: scheme handler. Anything that can't be mapped (/user/..., the homepage, etc.), or a system with no spotify: handler, should keep forwarding to the browser as it does now."
> user (turn 2): "/flow-next:capture the generic spec that'll allow for any native app link to work properly"
> user (turn 3, selected): "Shipped + user config (Recommended)"
> user (turn 3, selected): "Spotify + common ones"

Facts established on 2026-10-20 by reading the code and the relayed diagnosis. The plugin's desktop entry is the default handler for http, https, about, unknown and text/html, so every web link the desktop opens goes through `distractions open`. Target resolution sends an http(s) URL whose host is listed into the space and forwards every other one to the previously recorded browser handler. On the reporting machine `x-scheme-handler/spotify` resolves to `spotify.desktop` (`spotify --uri=%u`), and the desktop app claims no https links, so before the plugin took over https the browser also loaded the web player.

## Goal & Context
<!-- scope: business -->

A link to something a native app on this machine can show opens in that app, not in the work browser [paraphrase]. The plugin sits in front of every http(s) open, so it is where these links can be routed [paraphrase]. The mechanism is generic: rules map web links to an app's own URI scheme, the plugin ships rules for Spotify and a few common apps, and the person can add their own in config so any native app link can be made to work without a plugin release [paraphrase].

## Architecture & Data Models
<!-- scope: technical -->

**A handoff rule** names an app, the hosts it applies to, the link shapes it recognises, and how a recognised link becomes a URI in the app's scheme. The shape language must express what the shipped apps need: an optional leading path segment such as Spotify's `intl-xx`, captured path segments, a captured query parameter, and wrapping the whole original URL (Steam's `steam://openurl/<url>`) [inferred]. Shipped rules sit beside the catalog as plugin data; the person's rules live in the plugin config [inferred].

**Resolution order.** A listed host still opens in the space, unchanged. Only a link that would otherwise forward is checked against the rules; the first rule that recognises it and whose scheme has a usable handler wins; otherwise the link forwards as today [inferred].

**A usable handler** is the desktop entry the system names as default for `x-scheme-handler/<scheme>`, existing on disk, and not the plugin's own entry [inferred].

## Edge Cases & Constraints
<!-- scope: technical -->

- A captured value only reaches the generated URI if it is made of the characters the rule allows for it; anything else means the rule does not match, so a crafted link cannot smuggle extra URI parts into the app [inferred].
- A rule may only target a scheme that is not http, https, file, about, javascript or data, so a rule cannot loop a link back through the browser path or into a local file [inferred].
- Handing off launches the app outside the space. That is right for an unlisted app and is why listed hosts keep their current path: a listed distraction never escapes containment through a rule [inferred].
- A malformed person-defined rule is skipped with one log line; the shipped rules and the person's other rules still apply [inferred].

## Acceptance Criteria
<!-- scope: both -->

- **R1:** An http(s) link with an unlisted host that a rule recognises opens through that rule's scheme handler as the mapped URI, and no browser opens; for Spotify, `open.spotify.com/{track,album,playlist,artist,episode,show}/<id>`, with or without the `intl-xx/` prefix and the `?si=` query, opens as `spotify:<type>:<id>`. Errors: no default handler for the scheme, a handler whose desktop file is missing, or a handler that is the plugin itself → the link forwards to the browser as today; the handler failing to start → the link forwards and one line is logged. [paraphrase]
- **R2:** A link no rule recognises, including a recognised host with an unmappable path (`/user/...`, the homepage), forwards to the browser exactly as today (no error surface beyond R1). [paraphrase]
- **R3:** A link whose host is listed opens in the space exactly as today, even when a rule would recognise it (no error surface beyond the existing listed path). [inferred]
- **R4:** The person can add rules in config, replace a shipped rule by its app name, and turn a shipped rule off. Errors: a malformed rule, or one whose target scheme is http, https, file, about, javascript or data → that rule is skipped with one log line and the others apply. [paraphrase]
- **R5:** A captured value containing a character its rule does not allow makes the rule not match, so the link forwards (no error surface beyond R2). [inferred]
- **R6:** The plugin ships rules for Spotify, Zoom, Slack, Steam, Discord and Telegram, each with tests mapping real example links of every recognised shape. An app whose mapping cannot be confirmed against its Linux client is left out and named in the pull request. [inferred]
- **R7:** `docs/reference.md` describes the handoff, the resolution order and the rule format with an example of a person-defined rule; `manifest.json` carries a version past the last tag. [inferred]

## Boundaries
<!-- scope: business -->

- Not changing what happens to a listed host's links, nor launching a listed app's native client from a link. [inferred]
- No per-open override to force the browser for a link a rule recognises; turning the rule off in config is the way. [inferred]
- No handoff for schemes other than http and https arriving at the plugin. [inferred]

## Decision Context
<!-- scope: both -->

A Spotify special case was rejected for a generic rule mechanism with Spotify as one entry, so apps can be added as data and by the person without a plugin release [paraphrase]. Shipping rules only in the plugin was rejected because the person wants any native app link to work [paraphrase].
