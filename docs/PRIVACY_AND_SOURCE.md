# Privacy and Source

## What this repository is

A portfolio showcase for a production Android application on Google Play (repository folder name `Spekaer-Cleaner` is a spelling variant). It describes software that **plays synthesized tone sequences**. It is not a source distribution and makes **no claims** that the app removes water or dust or repairs speakers.

## What is published

- Written descriptions of architecture, audio behaviour, design decisions and trade-offs
- Original vector diagrams in `assets/diagrams/`
- Real screenshots from the Android build, cropped to app content (system status bar and ad-placeholder strips removed), captured with sample data
- The app icon, resized to WebP in `assets/branding/`
- Static code-size counts in the README (Dart files and lines under the app library tree, excluding generated localisation, plus six automated test files)

## What is intentionally not published

- Production source code, in whole or in part
- Build and signing configuration, certificates and keystores
- Firebase and Google service configuration files, API keys and tokens
- Ad unit identifiers, mediation account details, and placement key strings
- Subscription product identifiers, price points, trial durations, and offer tokens
- Internal cloud document paths, field names, analytics event names, and parameter keys
- Exact proprietary stage timing tables beyond what appears in public product descriptions
- Version control history of the production repository
- User data of any kind

Line, file and test counts are static counts; no test results, performance figures, download numbers, user counts or store ratings are claimed. Diagram labels are conceptual and do not map one-to-one to production identifiers.

## Notes on figures

Counts of Dart files and lines were made against the application library tree with generated localisation and generated Firebase options excluded. Test file count includes six files under the project test folder. These numbers are snapshots for this portfolio, not live metrics.

## About the app's data handling

This repository does not restate or replace the application's privacy policy or the data safety section on the store listing; those documents are the authoritative sources for end users.

At the engineering level, the following properties apply to the described release:

- **Preferences and premium flag** stay on the device via the platform key-value store (language, first-run progress, sound/haptics toggles, notification prompt timestamps, update snooze, premium boolean).
- **Tone generation** is entirely on-device; no audio buffer is uploaded for synthesis.
- **Purchases** interact with Google Play Billing. Transaction handling stays within the store SDK; this portfolio does not document server-side receipt storage.
- **Advertisements** use Google's mobile ads SDK with a User Messaging Platform consent flow where required. Ad requests may share device and app identifiers as defined by the SDK and the user's consent choices.
- **Firebase** integrates Analytics, Crashlytics, Performance, Cloud Messaging, and Firestore reads for ad enablement flags. Events and crash reports follow SDK configurations shipped in the app; this repository does not enumerate which fields are collected or excluded.
- **Remote switches** read small cloud documents to enable or disable ad placements; these reads are not tied to a user account in the described design.
- **Push messages** may arrive via FCM; foreground display may be mirrored into local notifications depending on payload handling in the app.
- **External links** (rate app, share, privacy URLs, more-apps list) open system browsers or store listings via standard platform launchers.

No claim is made here that all processing is anonymous, that data is encrypted beyond platform defaults, or that the app complies with a specific regulation. Those questions belong to the published privacy policy and store disclosures.

## Images

- **Screenshots** are real captures from the Android build with placeholder or sample session data. They were re-encoded to WebP with metadata removed.
- **Diagrams** are original illustrations created for this showcase.
- **App icon** is the application's launcher artwork, resized.

## Access

Source access for hiring or due-diligence purposes can be discussed at the owner's discretion. See [RIGHTS.md](../RIGHTS.md) for usage terms on published materials.
