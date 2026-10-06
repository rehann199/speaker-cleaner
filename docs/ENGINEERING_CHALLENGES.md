# Engineering Challenges

Each entry follows the same shape: the problem, the approach taken, the trade-offs, and the outcome. Identifiers from production code are generalised on purpose.

## 1. Continuous presets without audible seams

**Problem.** Auto, Water, and Dust modes chain many short segments with different frequency content. Restarting playback per segment would click and leave gaps, breaking the sense of one “cycle”.

**Approach.** Represent each mode as an ordered list of typed stages. A renderer walks the list once, writing into a single PCM buffer while carrying phase forward across sweep segments. Pulse stages fade each on-burst; pink noise uses its own envelope. A short overall fade wraps the entire buffer before WAV encoding.

**Trade-offs.** Memory spikes scale with total duration at 44.1 kHz mono int16 (tens of seconds is acceptable; the longest preset is the largest buffer). Any change to one stage requires regenerating the whole buffer for that run.

**Outcome.** One player invocation per preset run; UI still derives per-stage labels by indexing elapsed time into the stage list.

## 2. Keeping large WAV handoffs off the UI thread

**Problem.** Passing multi-megabyte byte arrays to the native player plugin on every play was implicated in main-thread stalls during screen transitions on some Android devices.

**Approach.** After rendering, write the WAV to a unique temp file and play by filesystem path. Delete the file on stop or when a new sequence replaces it. Keep in-memory bytes only for small chirps and manual chunks.

**Trade-offs.** Disk IO adds latency before the first sample, and temp cleanup must be best-effort on crash. Only one temp sequence file is tracked at a time.

**Outcome.** Platform channel payloads stay small for the heaviest modes; playback streams from disk on the native side.

## 3. When to stop audio vs when to keep it

**Problem.** Stopping cleaning tones whenever the app lifecycle flickers to `inactive` frustrates users who only opened the notification shade or a system dialog. Stopping too late leaves tones playing after the user switched apps.

**Approach.** Register a lifecycle observer on the audio engine that stops only on `paused` and `detached`, not on `inactive`. Pair with an audio session configuration that does not pause on transient duck events.

**Trade-offs.** If the user backgrounds the app without a clean pause event on an unusual OEM build, behaviour depends on OS delivery order; stop is still invoked on dispose when leaving a mode screen.

**Outcome.** Deliberate in-session tones survive benign overlays; backgrounding ends playback.

## 4. Manual mode frequency changes mid-run

**Problem.** Manual Clean must run for a fixed wall-clock duration while the user moves a frequency slider. A single pre-rendered file cannot track live slider changes.

**Approach.** Loop short sine chunks generated on demand; re-read the slider callback when building each chunk. Use the same gain and fade rules as other modes at chunk boundaries.

**Trade-offs.** Chunk boundaries are more frequent than preset stage boundaries, so fades must be tight to avoid modulation artifacts when frequency jumps quickly.

**Outcome.** Manual mode shares the engine’s stop/session semantics without a separate player implementation.

## 5. Rewarded gating without banner races

**Problem.** Rewarded ads use a native fullscreen surface, not a Flutter route. Banner widgets that reload when a Flutter loading dialog closes could appear underneath a still-visible rewarded ad.

**Approach.** Mode screens set a flag while the rewarded gate runs and omit banner construction entirely until the gate completes. Interstitial and rewarded loaders use non-dismissible loading overlays with explicit back suppression in tests.

**Trade-offs.** Free users see slightly delayed banner load on those screens. If rewarded load times out while online, the feature unlocks without an ad (product choice to avoid hard blocking).

**Outcome.** No banner reload under fullscreen ads; connectivity failures route to a blocking offline dialog that returns home instead of silently unlocking.

## 6. Remote ad switches that default to off

**Problem.** Ads must be toggled remotely, but a failed or slow config fetch must not show ads by mistake.

**Approach.** Maintain a “config confirmed” latch set only after a successful Firestore snapshot. Until then, every placement is treated as disabled. After confirmation, each placement requires an explicit true boolean; a global field can disable all formats. Premium bypasses requests entirely.

**Trade-offs.** A brand-new placement stays off until someone writes true in the cloud console. Offline first sessions show no ads even if the user would prefer the ad-supported path.

**Outcome.** Fail-closed ad behaviour aligned with privacy expectations during startup.

## 7. Premium status without blocking startup

**Problem.** Billing can be slow or unavailable; purchase state still needs to persist across restarts for ad gating.

**Approach.** Billing service initializes asynchronously during splash, swallows errors, and mirrors entitlement to local preferences. Widgets listen through app state’s premium getter.

**Trade-offs.** Entitlement is client-trusted until a server verifier exists. Restore purchases must be exposed in paywall for store policy.

**Outcome.** Startup never waits on billing success; ads and gates react as soon as the purchase stream reports.

## 8. Separate UI sounds from cleaning audio

**Problem.** Button feedback must not reconfigure or stop the cleaning session.

**Approach.** Dedicated low-latency-ineligible player for the bundled tap MP3, warmed at startup, respecting the sound-effects preference. Haptics use platform haptic APIs gated by a separate toggle.

**Trade-offs.** Two players consume slightly more native resources; tap sound fails softly if assets missing.

**Outcome.** Cleaning tones and UI clicks remain independent.
