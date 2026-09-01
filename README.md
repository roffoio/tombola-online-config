# Tombola Online — app config

Runtime configuration for the Tombola Online mobile app. Replaces Firebase
Remote Config.

Published by GitHub Pages at:

    https://roffoio.github.io/tombola-online-config/config.json

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

### App

| Key                        | Type   | Default   | Meaning                                                                                                               |
| -------------------------- | ------ | --------- | --------------------------------------------------------------------------------------------------------------------- |
| `maintenance_mode_enabled` | bool   | `false`   | `true` shows the maintenance screen instead of the app.                                                               |
| `required_app_version`     | string | app's own | Users on a version **older** than this get the forced-update screen. Use a plain `major.minor.patch`, no build number. |

### Ads

Read by the app release that carries the ad stack refactor, and every version
after it. Earlier versions ignore these keys, so they are safe to publish before
that build ships — and publishing them first is the point, so the policy is
already in force the moment it does.

The *Default* column is the value compiled into the app, used when this file is
unreachable or the key is missing. What is in `config.json` right now may
deliberately differ — see [Ad rollout](#ad-rollout).

#### Switches

| Key                   | Type | Default | Meaning                                                                                            |
| --------------------- | ---- | ------- | -------------------------------------------------------------------------------------------------- |
| `ads_enabled`         | bool | `true`  | Master kill switch. `false` removes **every** ad format. Reach for this first if anything goes wrong. |
| `ads_banner_enabled`  | bool | `true`  | The banners on the games list and the game screen.                                                  |
| `ads_app_open_enabled`| bool | `true`  | The full-screen ad on returning to the app. Off takes effect at once; on takes effect from the next resume. |
| `ads_rewarded_enabled`| bool | `true`  | The opt-in "ad-free break" offer. On takes effect on the next screen the player opens.              |

None of these affect the paid ad-free upgrade: a player who bought `remove_ads_1`
sees no ads regardless of anything in this file.

#### Interstitials — where

| Key                          | Type   | Default  | Values                                             |
| ---------------------------- | ------ | -------- | -------------------------------------------------- |
| `ads_interstitial_placement` | string | `"both"` | `"room_entry"`, `"game_over"`, `"both"`, `"off"`   |

- `"room_entry"` — on entering a room, once the game screen is ready.
- `"game_over"` — when the match ends, before the results.
- `"both"` — both triggers, with the caps below doing the balancing.
- `"off"` — no interstitials at all. Banners, app open and rewarded are unaffected.

Anything else — a typo, a wrong case — falls back to `"both"`.

#### Interstitials — how often

All four caps are checked before every interstitial. Any one of them can refuse.

| Key                                      | Type | Default | Meaning                                                                                                     |
| ---------------------------------------- | ---- | ------- | ------------------------------------------------------------------------------------------------------------- |
| `ads_interstitial_min_interval_seconds`  | int  | `90`    | Minimum gap between any two full-screen ads (interstitial **or** app open). The cap that usually binds.       |
| `ads_interstitial_every_n_matches`       | int  | `1`     | At most one per N matches. A match counts at whichever of room entry or match end comes first, so both triggers count the same unit of play. |
| `ads_interstitial_skip_first_sessions`   | int  | `1`     | No interstitial at all during a new install's first N app sessions. `0` disables the grace period.           |
| `ads_interstitial_skip_first_room_entry` | bool | `true`  | Never one on the first room entry of a session — the first thing a returning player sees should be the game. |

Two more rules are **not** configurable and always apply: no interstitial in a
room the player reached through an invite link (that room only, not the rest of
their session), and never on top of a purchase flow, a dialog or the win
animation.

#### App open and rewarded

| Key                                 | Type | Default | Meaning                                                                                     |
| ----------------------------------- | ---- | ------- | --------------------------------------------------------------------------------------------- |
| `ads_app_open_min_interval_seconds` | int  | `300`   | Minimum gap since the last full-screen ad before an app open ad may show on resume.          |
| `ads_rewarded_break_minutes`        | int  | `30`    | How long one rewarded view buys with no interstitials and no app open ads. Banners continue. |

App open ads never show on a first cold start, never on top of a game in
progress, and never on the resume that follows an invite link.

## Ad rollout

`config.json` currently ships **`ads_app_open_enabled: false` and
`ads_rewarded_enabled: false`**, which is deliberate and differs from the app's
own defaults.

TICKET-001 §8 releases the ad stack in two steps: consent, banners and
interstitials first, then app open and rewarded once the first step is stable.
The app's ad unit IDs are now all populated, so nothing in the code holds the
second step back — this file does.

To release step two, flip both to `true`. No build required.

To pull the whole thing in a hurry, set `ads_enabled: false`; to soften it
without turning it off, raise `ads_interstitial_min_interval_seconds` and
`ads_interstitial_every_n_matches`. This app's audience is older and loyal and
there is already a review complaining about ads, so watch the store rating and
the crash-free rate alongside revenue, and loosen from here rather than shipping
a build.

## Rules

- Keep it a flat JSON object. Nested objects are only read by keys that
  explicitly expect JSON.
- A missing or malformed key falls back to the default compiled into the app,
  so a typo is survivable — but a _valid_ wrong value is not. Double-check
  `required_app_version` before committing: setting it too high locks users out
  of the app until they update.
- Never remove a key that a shipped version still reads.
- Ints must be plain non-negative numbers. A negative value, a decimal or a
  quoted number that will not parse falls back to the compiled default.
- Booleans may be `true`/`false` or the strings `"true"`/`"false"`. Anything
  else falls back to the default.
- Ad changes reach a device on the same ~15 minute budget as everything else,
  and apply from the next time the app evaluates them — an ad already on screen
  is not taken away mid-view.
