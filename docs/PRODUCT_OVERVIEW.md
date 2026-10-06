# Product Overview

Speaker Cleaner is an Android utility distributed on Google Play. At the engineering level it is a **tone-sequence player**: the app synthesizes audio on the device and routes it to the loudspeaker. This document describes screens and behaviour. It does **not** claim that any mode removes water or dust, restores muffled audio, or repairs hardware. Store and in-app marketing text may use those words; they are not repeated here as facts.

## Platforms and release posture

The public release is **Android on Google Play** only. The codebase is Flutter and includes optional iOS-oriented packages, but there is no App Store link in this portfolio and no statement that an iOS build ships to consumers.

## First-run experience

Cold start moves through a short splash while core services initialize, then a phase machine decides what the user still needs:

1. **Language selection** — pick one of eleven supported languages, persisted locally.
2. **Onboarding** — three illustrated pages introducing the product (visual marketing only in the app; not quoted here).
3. **Tutorial** — four steps explaining how to hold the phone and what each mode category represents in the UI.
4. **Home** — the main hub once all three first-run flags are set.

Returning users skip straight to Home after splash. Preferences failures fall back to defaults rather than blocking launch.

## Home hub

Home is the navigation root after first-run. It presents:

- A **Get Premium** chip (hidden once the user has an active subscription entitlement in local billing state).
- A hero **Auto Clean** card with a one-tap entry into the longest preset sequence.
- A **Cleaning Tools** grid with four tiles: Manual Clean, Water Eject, Dust Eject, and Speaker Test. Three tiles show a small **AD** badge in the free tier to signal that opening them may involve advertising or premium gating.

Back from Home shows an exit confirmation sheet; confirming exit may show an interstitial when remote ad switches allow it.

Other home-time prompts (not all shown in portfolio screenshots) include an optional notification-permission sheet with snooze and “do not ask again”, and a Play in-app update sheet when the store reports a newer version.

## Mode screens (audio behaviour)

Each mode screen shares a similar layout: back navigation, a short description line, a central progress visualization, countdown or stage label, start/stop control, and optional banner ad slot at the bottom for free users. Haptic ticks during an active run respect the haptics toggle in settings.

### Auto Clean

Runs the **general preset**: six stages totaling about eighty seconds, combining low and mid frequency sweeps and a closing steady tone. Progress and labels derive from playback position mapped onto the stage list. Completing a cycle may trigger an interstitial for free users when enabled remotely.

Auto Clean and Manual Clean are **not** behind the watch-ad-or-premium dialog on entry.

### Manual Clean

Runs a **fixed thirty-second** cycle. The user sets frequency on a slider (roughly 100 Hz to 2.5 kHz). While running, the engine generates short sine chunks sequentially and re-samples the slider each chunk so frequency changes apply without restarting the whole cycle. An interstitial may appear after completion for free users.

### Water Eject (UI name)

Plays a **low-frequency preset** of about forty-five seconds: ramps, repeating sweeps in the low hundreds of Hz, a pulsed segment, and closing sweeps. The engineering content is synthesized tone only; this document does not describe liquid movement.

Free users pass through a **premium gate dialog** on the home tile (watch a rewarded ad or open the paywall). After navigation, a **rewarded ad gate** may run on screen open before controls unlock.

### Dust Eject (UI name)

Plays a **higher-frequency preset** of about fifty seconds: ascending sweeps, a wide debris-labelled sweep stage, short steady tones at several frequencies, and a closing downward sweep. Again, only audio is emitted; no claim is made about physical dust.

Same **premium gate** and **rewarded gate** pattern as Water Eject.

### Speaker Test

Runs a **reference sequence** per channel: several named steady tones, a wide logarithmic sweep, and a pink-noise segment, then repeats the full sequence for left, right, and stereo routing. The user steps through phases with explicit channel labels.

Same entry gating as Water and Dust for free users. Banners are suppressed while the rewarded gate overlay is active so a banner does not reload under a fullscreen ad.

## Paywall and premium

The paywall presents monthly and yearly subscription options resolved from Play product details (localized price labels come from the store). Purchasing or restoring sets a **local premium flag** persisted on device; ads hide and gated modes open without the home dialog. Entitlement verification is client-side through the store purchase stream; server-side receipt validation would be a natural hardening step but is not described here.

## Settings

Settings exposes toggles for **sound effects** (UI tap click) and **haptics** (including the cleaning pulse), language change, privacy policy viewer, share link, rate-app link, more-apps catalog, and version info. Privacy policy content is bundled per language in the app.

## Monetisation summary (free tier)

| Element | When it appears |
|---|---|
| Banner slots | Several mode screens and home-adjacent layouts when remote config confirms and user is not premium |
| Rewarded gate | On open of Water Eject, Dust Eject, Speaker Test after home navigation |
| Interstitial | After some mode completions, on exit confirm, and other placement keys when enabled |
| App open | During startup ad gate after consent, bounded by timeout |
| Premium subscription | Removes ads and bypasses rewarded gates |

Remote ad switches live in a cloud document; until a successful read, placements stay off.

## Visual and accessibility notes

The app uses a dark theme, bundled Plus Jakarta Sans font files (avoiding runtime font fetch flicker), and SVG mode illustrations on home and tutorials. This portfolio does not claim WCAG conformance or a formal accessibility audit.

## Relationship to other documents

- [TECHNICAL_CASE_STUDY.md](TECHNICAL_CASE_STUDY.md) — synthesis and playback mechanics.
- [ARCHITECTURE.md](ARCHITECTURE.md) — modules and startup.
- [ENGINEERING_CHALLENGES.md](ENGINEERING_CHALLENGES.md) — selected design problems.
- [PRIVACY_AND_SOURCE.md](PRIVACY_AND_SOURCE.md) — data handling at a high level.
