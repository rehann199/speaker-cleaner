# Technical Case Study

This case study explains how Speaker Cleaner turns mode definitions into audible output on Android. It is written for engineers evaluating the project. **It describes synthesized tone sequences only** and does not assert that playing them removes water or dust or repairs a speaker.

## Problem shape

The product asks for several long, structured tone programs (tens of seconds each), a live-tunable manual mode, and a multi-channel speaker check. All audio must be generated on the phone because the app does not ship prerecorded cleaning clips (except a short UI click). Playback must feel continuous within a preset, respect system volume, survive common Android audio-focus quirks, and stop when the user leaves the app.

## Stage model

Each preset is an ordered list of **stages**. A stage has a duration and a kind:

| Kind | Behaviour |
|---|---|
| Sweep | Frequency moves linearly from a start Hz to an end Hz across the duration; phase accumulates sample-by-sample so there is no reset at stage boundaries when concatenated |
| Steady tone | Sweep with equal start and end frequency |
| Pulse | A fixed frequency gated on and off with configurable on/off durations; edges are faded to avoid clicks |
| Pink noise | Broadband noise via a deterministic multi-row generator so renders are reproducible |

Stage metadata also carries a **label kind** used only for UI strings (stage number, sweep, pulse, bass/treble labels in speaker test, and similar).

Preset builders assemble lists for Auto Clean, Water Eject, Dust Eject, and the speaker-test reference segment. Durations and frequency ranges are fixed in code rather than user-editable.

## PCM synthesis

A dedicated generator module renders stages into 16-bit PCM:

- Sample rate is **44.1 kHz** for all modes.
- Sweeps use continuous phase integration (sine of accumulated phase), not restarting phase each sample window.
- Each stage applies a short edge fade; the full sequence also gets a brief overall fade in and out.
- Pink noise uses a separate headroom scalar so it does not clip against tonal stages.
- **Channel routing** duplicates the mono synthesis into left, right, or both channels when encoding stereo WAV data for speaker test.

Manual mode bypasses the all-in-one sequence renderer for playback: it loops **small chunks** (each chunk is its own sine WAV in memory) and reads the current slider frequency when building the next chunk, with tiny fades at chunk boundaries.

Speaker test plays the reference sequence three times in a row with different channel modes, then optionally plays shorter fixed tones per channel in a separate helper used for the left/right/stereo pass labels.

## Playback engine

A singleton **audio engine** owns one native player instance and coordinates session lifecycle:

1. Configure an **audio session** tuned for utility playback (sonification content type, media usage, gain-style focus, do not pause when ducked).
2. Activate the session before play; deactivate after stop.
3. Assign a monotonically increasing **session id** so stale async work (file writes, play calls) cannot start after the user has already stopped or navigated away.
4. Expose position and state streams for UI progress.

**Sequence playback path:** render entire stage list to WAV bytes, write bytes to a uniquely named file under the OS temp directory, play via file path, delete the file on stop or when replaced. **Short playback path:** pass bytes directly for chirps, manual chunks, and speaker-test segments that stay small.

**Stop conditions:** explicit stop button, screen dispose, natural completion, session superseded, app lifecycle **paused** or **detached**. The engine deliberately **does not** stop on **inactive** (notification shade, volume HUD, permission dialog) so an in-progress tone is not cut off by transient overlays.

Digital **gain** is applied in the generator as a multiplier clamped before int16 conversion; the app still follows the device media volume.

## UI coupling

Mode screens subscribe to playback position, compute elapsed fraction of total duration, and map elapsed time into the active stage for labels (for example showing a sweep’s Hz range). Animated rings and wave decorations are cosmetic; the authoritative timer is the player position stream with a timeout safety net if completion events never fire.

A separate **UI feedback** helper plays the bundled tap MP3 on another player and drives periodic haptic ticks during cleaning when enabled in preferences.

## Startup and side services

Application state loads preferences, initializes Firebase, billing, and ads in parallel with a minimum splash delay. An **startup ad gate** runs consent gathering, mobile ads initialization, and an optional app-open ad before routing to first-run or home, with an overall bound so a stuck SDK cannot block launch indefinitely.

Billing listens to the purchase stream and mirrors premium status into preferences; widgets rebuild through a change notifier when premium flips.

Firestore snapshots supply live ad placement booleans; widgets consult a “config confirmed” flag before treating any placement as on.

## Testing surface

Automated tests include:

- Unit checks on WAV header layout, mono/stereo sample layout, and channel routing.
- Preference repository defaults and first-run flag logic.
- Widget tests for cold-start routing and paywall card layout.
- A widget test ensuring the ad loading dialog blocks back navigation until dismissed programmatically.

There is no end-to-end test that plays audio through device hardware in CI.

## Honest limits

- **No sensing:** the app does not measure speaker response or moisture; speaker test is subjective listening only.
- **No guaranteed outcome:** frequency choices in presets are product design, not validated physical procedures in this document.
- **Client-side premium flag:** tamper resistance depends on store APIs; there is no server entitlement service in the described architecture.
- **Platform variance:** actual loudness and distortion depend on handset hardware, case, and volume setting.

## See also

- [ARCHITECTURE.md](ARCHITECTURE.md) for layer diagram and module boundaries.
- [ENGINEERING_CHALLENGES.md](ENGINEERING_CHALLENGES.md) for trade-offs called out during development.
