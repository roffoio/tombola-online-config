# Tombola Online — app config

Runtime configuration for the Tombola Online mobile app. Replaces Firebase
Remote Config.

Published by GitHub Pages at:

    https://davideroffo.github.io/tombola-online-config/config.json

The app reads this file at launch and whenever it returns to the foreground,
caching the last known good values on the device. If this file is unreachable
the app keeps using the cached values, so a bad deploy here degrades quietly
instead of breaking the app.

## Editing

Edit `config.json` on github.com, commit to `main`, done. Pages redeploys in
under a minute; the CDN caches for up to ~10 minutes, and devices check at most
once every 5 minutes — so budget **~15 minutes** for a change to reach everyone.

`git revert` is the rollback.

## Keys

| Key | Type | Meaning |
|---|---|---|
| `maintenance_mode_enabled` | bool | `true` shows the maintenance screen instead of the app. |
| `required_app_version` | string | Users on a version **older** than this get the forced-update screen. Use a plain `major.minor.patch`, no build number. |

## Rules

- Keep it a flat JSON object. Nested objects are only read by keys that
  explicitly expect JSON.
- A missing or malformed key falls back to the default compiled into the app,
  so a typo is survivable — but a *valid* wrong value is not. Double-check
  `required_app_version` before committing: setting it too high locks users out
  of the app until they update.
- Never remove a key that a shipped version still reads.
