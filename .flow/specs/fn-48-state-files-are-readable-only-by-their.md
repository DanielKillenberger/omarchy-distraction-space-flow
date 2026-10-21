# State files are readable only by their owner

## Problem
Marketplace review on omacom/omarchy-plugin-marketplace#12365 (at 0b9a65f): `held.jsonl` stores the titles and bodies of held notifications, and the log records what the plugin did. Both are appended with default modes, and the state directory is created with default modes. Under umask 022 they come out 0644 in a 0755 directory, so another local user can read private notification content.

## Requirements
- **R1:** The plugin's state directory is owner-only (0700): created that way, and an existing directory with group or other bits is tightened the next time the plugin uses it.
- **R2:** `held.jsonl` and the default log are created 0600 whatever the umask, an existing one with group or other bits is tightened on the next append, and the rewrite that trims `held.jsonl` keeps 0600.
- **R3:** The JSON state files keep their current 0600 (atomic write through a private temp file); a test pins it.
- **R4:** A failure to change a mode never stops the plugin: appends that failed before still fail quietly, and nothing new raises.
- **R5:** `manifest.json` carries an unreleased version.

## Boundaries
- A log path the person configured elsewhere gets 0600 on creation and the same tightening; its parent directory is theirs and is left alone.
- The runtime directory (`$XDG_RUNTIME_DIR`, already 0700) is out of scope.
