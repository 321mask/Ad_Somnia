<div align="center">

<img src="assets/icon.png" width="120" alt="Ad Somnia app icon">

# Ad Somnia

### Lucid-dream training that times haptic and audio cues to your REM sleep.

<p>
<a href="https://testflight.apple.com/join/ZCQheBVj"><img src="https://img.shields.io/badge/iOS-17%2B-000000?logo=apple&logoColor=white" alt="iOS 17+"></a>
<img src="https://img.shields.io/badge/watchOS-10%2B-000000?logo=apple&logoColor=white" alt="watchOS 10+">
<img src="https://img.shields.io/badge/SwiftUI-2C2C2E?logo=swift&logoColor=F05138" alt="SwiftUI">
<img src="https://img.shields.io/badge/SwiftData-2C2C2E" alt="SwiftData">
<img src="https://img.shields.io/badge/HealthKit-FF2D55?logo=apple&logoColor=white" alt="HealthKit">
<img src="https://img.shields.io/badge/WatchConnectivity-5856D6" alt="WatchConnectivity">
<img src="https://img.shields.io/badge/status-TestFlight-7D4DFF" alt="Status: On the App Store">
</p>

<p>
<a href="https://apps.apple.com/fr/app/ad-somnia/id6776810198?l=en-GB">Try it on the App Store</a> ·
<a href="https://321mask.github.io/adsomnia-site/index.html">About</a> ·
<a href="#status">Support</a>
</p>

</div>

## The problem

Lucid dreaming — knowing you're dreaming while it's happening — is a learnable skill, but the techniques that work all share one requirement that's brutally hard to satisfy: you need a reminder *during REM sleep*, the exact moment you can't act for yourself. Classic methods lean on the dreamer's own discipline — reality checks, dream journaling, waking yourself at 4 a.m. — and lose precision right when it matters most.

Meanwhile, the Apple Watch on millions of wrists already records the sleep-stage data that pinpoints those REM windows. For most people that data just becomes a morning summary chart and nothing more.

Ad Somnia turns that latent signal into action. It reads your recent Apple Watch sleep history, models when you're most likely to be in REM, and delivers a gentle cue — a Watch haptic, an audio tone — inside that window. The cue is faint enough not to wake you, but distinct enough that, with practice, you learn to recognize it from inside the dream. Lucid dreaming stops being a discipline you maintain and becomes a signal you train yourself to catch.

## What it does

1. **Onboard and grant Health access.** The app explains that it uses about a week of Apple Watch sleep history to personalize cue timing — and works without it, so nothing is blocked on a fresh device.
2. **Calibrate the cue.** Tap the center power button to preview the three-tone audio cue (400 / 600 / 800 Hz) and set a comfortable volume; set Apple Watch haptic strength in Settings.
3. **Arm a session before bed.** One tap starts a background session. The home screen surfaces the live model: the next predicted cue window, when the next cue is scheduled, and how many cues have been delivered.
4. **Sleep.** Through the night, Ad Somnia delivers gentle, REM-timed cues to the paired Apple Watch (and/or audio at onset), nudging awareness without pulling you awake.
5. **Review and adapt.** The session graph shows where cue windows landed against your actual sleep, and the model sharpens over repeated nights.

## What I built

I designed, engineered, and shipped Ad Somnia end to end as a solo developer — the SwiftUI iOS app, the watchOS companion, the REM-timing model, and all of the sensor and background plumbing underneath. No team, no template; the architecture decisions below are mine.

### Genuinely hard part 1 — REM-probability modeling from Apple Watch sleep data

The core of the app is a per-user model that estimates *when* you're likely to be in REM. It reads sleep-analysis samples from HealthKit (REM / Core / Deep / Awake stages plus timing), reconciles the messy realities — fragmented samples, multiple sessions per night, day-to-day variability, differing data sources — into a coherent sleep timeline, and produces probability windows that drive cue scheduling. All of it runs on-device. The "Next window" countdown on the home screen is the live, user-facing surface of that model.

### Genuinely hard part 2 — watchOS extended-runtime & overnight background sessions

Delivering a cue at 3 a.m. means keeping a scheduled task alive across hours of background time on a battery, on a platform engineered to suspend you. I used HealthKit background delivery (background-delivery entitlement + a dedicated observer) to wake the app when new sleep data arrives so the model stays current, and watchOS extended-runtime sessions to hold execution through the cue window. The "Background session: ON / OFF" state is the honest readout of whether that machinery is armed.

### Genuinely hard part 3 — timed haptic + audio cue delivery

Two cue channels, two devices. Audio plays at sleep onset on iPhone — a three-tone signal (400 / 600 / 800 Hz) at a calibrated volume. Haptics fire during REM windows on the Apple Watch, reached from the phone over WatchConnectivity, which requires the Watch to be on-wrist and reachable. The design problem underneath the plumbing: a cue strong enough to register inside a dream but soft enough not to wake the sleeper, fired at a precise scheduled instant from a suspended state.

## Tech highlights

| Area | Stack & decisions |
| --- | --- |
| **UI** | SwiftUI across iPhone and Apple Watch, one design language (deep-indigo, glowing cue button). The home screen is state-driven and renders the live model directly — next window, schedule, cues delivered. |
| **Sensor / health data** | HealthKit, **read-only**, a single category (`HKCategoryTypeIdentifierSleepAnalysis`). No write access and no other health types requested — the smallest footprint that supports the feature. |
| **Background runtime** | HealthKit background delivery (entitlement + `HealthKitBackgroundObserver`) wakes the app on new sleep data; a watchOS extended-runtime session holds execution through the scheduled cue window. |
| **Audio & haptics** | Three-tone onset audio (400 / 600 / 800 Hz) with user-calibrated volume on iPhone; configurable-strength haptics on the Apple Watch, dispatched via WatchConnectivity. |
| **Persistence** | SwiftData models (`SleepNight`, `RemModelState`) in a local store; `@AppStorage` / `UserDefaults` for lightweight prefs (`hasSeenOnboarding`, `hapticStrength`). No CloudKit, no iCloud sync. |
| **Concurrency** | Async Health queries and scheduling off the main thread, with UI state updates marshalled back to the main actor. |
| **Networking** | None. No `URLSession`, no analytics SDK, no third-party trackers — the only data movement is peer-to-peer iPhone ↔ Watch over WatchConnectivity. |

## Status

- **App Store** — <a href="https://apps.apple.com/fr/app/ad-somnia/id6776810198?l=en-GB">Try it on the App Store</a>.
- **Source** — private repository, kept closed by design. The full app is available for review via the TestFlight build above.
- **Built by** — solo, end to end: design, iOS + watchOS engineering, the REM-timing model, and submission.
- **Support** — `https://321mask.github.io/adsomnia-site/support.html`.

## Reliability as architecture

A lucid-dreaming app lives or dies on one question: *can it fire a cue at 3 a.m., mid-REM, with the phone locked and the user asleep, on a platform that has every incentive to suspend it?* The idea is the easy part — the timing is the entire product. A cue that's late, early, or missing makes every other screen irrelevant.

That single constraint shaped the architecture more than any feature did.

First, the app has to **know** when to fire, which rules out a morning batch job — the model has to stay current as the night unfolds. HealthKit background delivery wakes the app when new sleep data lands so the next REM window is re-estimated without anyone touching the phone.

Second, the app has to **survive** long enough to act. Background execution gets reclaimed aggressively, so holding a precise overnight wake-up isn't free; the cue path is built on watchOS extended-runtime sessions and an iPhone ↔ Watch handoff over WatchConnectivity, so the haptic actually plays on-wrist at the scheduled moment.

Third, it does all of this **with no server** — no cloud cron job to offload scheduling to, every decision made on-device, on a battery, overnight. That's a harder version of background work than most apps face, and making it trustworthy (a "Background session: ON" you can actually rely on) is the piece I'm proudest of.

It's the same lesson good privacy engineering teaches: the thing that defines the product isn't a screen the user looks at — it's a property of the system that has to hold true while they're not looking.

## Screens

<p>
<img src="assets/Apple Watch cues.png" width="240" alt="Home screen with cue button and next-window countdown">
<img src="assets/Lucid Dreaming.png" width="240" alt="Volume calibration for the audio cue">
<img src="assets/Measure dreamtime.png" width="240" alt="Session graph of cue windows against sleep">
<img src="assets/Relax before sleep.png" width="240" alt="Apple Watch haptic configuration">
</p>

| Screen | What it shows |
| --- | --- |
| **Home** | The armed session: cue button, next predicted REM window, schedule, and cues delivered. |
| **Calibration** | Previewing the three-tone audio cue and setting a comfortable volume. |
| **Graph** | The night's cue windows plotted against recorded sleep. |
| **Watch** | Configuring and test-firing the haptic cue on the paired Apple Watch. |

<div align="center">
<sub>Built with SwiftUI · HealthKit · WatchConnectivity · SwiftData — iOS + watchOS, fully on-device, no servers.</sub>
</div>
