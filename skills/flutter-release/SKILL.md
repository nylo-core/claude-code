---
name: flutter-release
description: Use when the user wants to build release artifacts or submit a Flutter app to the App Store or Google Play. Covers building APKs, AABs, IPAs, and store submission requirements for both platforms.
---

# Flutter Release and Store Submission

## Overview

Releasing a Flutter app involves building platform-specific artifacts (AAB/APK for Android, IPA for iOS), then submitting them to Google Play Console or App Store Connect. Each store has specific requirements for listing metadata, screenshots, ratings, and compliance.

## When to Use

- User wants to build a release version of the app
- User is preparing for first submission to App Store or Google Play
- User needs to know what build commands to run
- User wants to understand store listing requirements
- User is submitting an update to an existing app
- **NOT** for debug builds or development testing (use `flutter run`)

## Quick Reference: Build Commands

| Artifact | Command | Output Location |
|---|---|---|
| Android APK | `flutter build apk` | `build/app/outputs/flutter-apk/app-release.apk` |
| Android APK (split per ABI) | `flutter build apk --split-per-abi` | `build/app/outputs/flutter-apk/app-*-release.apk` |
| Android App Bundle | `flutter build appbundle` | `build/app/outputs/bundle/release/app-release.aab` |
| iOS IPA | `flutter build ipa` | `build/ios/ipa/*.ipa` |
| iOS (no signing) | `flutter build ios --no-codesign` | `build/ios/iphoneos/Runner.app` |

## Pre-Release Checklist

Before building for release, verify:

- [ ] Version and build number incremented in `pubspec.yaml`
- [ ] Code signing configured (see flutter-signing skill)
- [ ] App icon set (see flutter-app-icons skill)
- [ ] Splash screen configured (see flutter-app-icons skill)
- [ ] App name and bundle ID finalized (see flutter-app-config skill)
- [ ] Debug-only code removed (print statements, debug banners)
- [ ] `flutter analyze` passes with no errors
- [ ] All tests pass: `flutter test`
- [ ] Permissions configured with proper usage descriptions (see flutter-permissions skill)
- [ ] ProGuard/R8 rules configured if using code shrinking (Android)
- [ ] `minSdkVersion` and deployment target set appropriately

## Building for Android

### Android App Bundle (AAB) — Recommended for Google Play

```bash
flutter build appbundle
```

The AAB is Google Play's preferred format. Google Play generates optimized APKs for each device configuration, reducing download size.

**Output**: `build/app/outputs/bundle/release/app-release.aab`

### APK — For Direct Distribution

```bash
# Fat APK (all architectures in one file)
flutter build apk

# Split APK (one per architecture, smaller size)
flutter build apk --split-per-abi
```

Split per ABI produces three APKs:
- `app-armeabi-v7a-release.apk` — 32-bit ARM devices
- `app-arm64-v8a-release.apk` — 64-bit ARM devices (most modern phones)
- `app-x86_64-release.apk` — x86 devices (emulators, some Chromebooks)

### Build Flags

```bash
flutter build appbundle \
  --release \                     # Release mode (default for build commands)
  --build-name=1.2.3 \           # Override version name
  --build-number=42 \             # Override build number
  --target-platform android-arm64 \  # Target specific architecture
  --obfuscate \                   # Obfuscate Dart code
  --split-debug-info=build/debug-info  # Required with --obfuscate
```

### Obfuscation

For release builds, obfuscate Dart code to protect intellectual property:

```bash
flutter build appbundle \
  --obfuscate \
  --split-debug-info=build/debug-info
```

- `--obfuscate` — renames Dart symbols to make reverse engineering harder
- `--split-debug-info` — extracts debug symbols (required for obfuscation, also used to symbolicate crash reports)

**Keep the debug info directory** — you need it to decode stack traces from crash reports.

### ProGuard / R8 Configuration

If `minifyEnabled true` is set in `build.gradle`, create `android/app/proguard-rules.pro`:

```proguard
## Flutter specific
-keep class io.flutter.app.** { *; }
-keep class io.flutter.plugin.** { *; }
-keep class io.flutter.util.** { *; }
-keep class io.flutter.view.** { *; }
-keep class io.flutter.** { *; }
-keep class io.flutter.plugins.** { *; }

## Keep annotations
-keepattributes *Annotation*

## Keep your model classes if using JSON serialization
# -keep class com.yourpackage.models.** { *; }
```

### Testing the Release Build

```bash
# Install release APK on connected device
flutter install --release

# Or run in release mode
flutter run --release
```

## Building for iOS

### Build IPA

```bash
flutter build ipa
```

This builds an Xcode archive and exports an IPA file.

**Output**: `build/ios/ipa/YourApp.ipa`

### Build with Export Options

For CI/CD or specific distribution methods:

```bash
flutter build ipa --export-options-plist=ios/ExportOptions.plist
```

See the flutter-signing skill for ExportOptions.plist configuration.

### Build Flags

```bash
flutter build ipa \
  --release \                     # Release mode (default)
  --build-name=1.2.3 \           # Override version name
  --build-number=42 \             # Override build number
  --obfuscate \                   # Obfuscate Dart code
  --split-debug-info=build/debug-info  # Required with --obfuscate
  --export-options-plist=ios/ExportOptions.plist
```

### Build Without Code Signing (CI First Step)

```bash
flutter build ios --no-codesign
```

Useful when you want to verify the build compiles before setting up signing.

### Upload to App Store Connect

**Option 1: Xcode (GUI)**

1. Open `build/ios/archive/Runner.xcarchive` in Xcode (double-click)
2. In the Xcode Organizer (Window → Organizer), select the archive
3. Click **Distribute App**
4. Choose **App Store Connect**
5. Select **Upload** (or **Export** for manual upload)
6. Follow the prompts

**Option 2: Command Line (xcrun)**

```bash
xcrun altool --upload-app \
  --type ios \
  --file build/ios/ipa/YourApp.ipa \
  --apiKey YOUR_API_KEY_ID \
  --apiIssuer YOUR_ISSUER_ID
```

For API key authentication, create an App Store Connect API key:
1. Go to [appstoreconnect.apple.com](https://appstoreconnect.apple.com) → Users and Access → Integrations → App Store Connect API
2. Generate a new key with "App Manager" or "Admin" role
3. Download the `.p8` file
4. Place it at `~/.appstoreconnect/private_keys/AuthKey_YOUR_KEY_ID.p8`

**Option 3: Transporter App**

Download the Transporter app from the Mac App Store. Drag and drop the IPA file to upload.

### TestFlight

After uploading to App Store Connect:

1. The build appears in **TestFlight** section (may take 15-30 minutes for processing)
2. If the build uses encryption, answer the **Export Compliance** questions
3. For **Internal Testing**: Add internal testers (up to 100 Apple Developer team members). No review required.
4. For **External Testing**: Create a test group, add testers by email (up to 10,000). First build requires Beta App Review.
5. Testers receive an email invitation to install via the TestFlight app

## Google Play Store Submission

### Google Play Console Setup

1. Create a [Google Play Developer account](https://play.google.com/console) ($25 one-time fee)
2. Create a new app: **All apps** → **Create app**
3. Fill in app details: name, default language, app type (app/game), free/paid

### Store Listing Requirements

| Requirement | Specification |
|---|---|
| App name | Max 30 characters |
| Short description | Max 80 characters |
| Full description | Max 4000 characters |
| App icon | 512x512 PNG, 32-bit, no alpha |
| Feature graphic | 1024x500 PNG or JPEG |
| Phone screenshots | Min 2, max 8. JPEG or PNG, min 320px, max 3840px, aspect ratio max 2:1 |
| 7-inch tablet screenshots | Optional but recommended, same specs |
| 10-inch tablet screenshots | Optional but recommended, same specs |
| App category | Select from predefined categories |
| Content rating | Complete IARC questionnaire |
| Privacy policy URL | Required for all apps |
| Target audience | Declare target age group |

### Uploading to Google Play

1. Go to **Release** → **Production** (or Testing track)
2. Click **Create new release**
3. Upload the AAB file (`app-release.aab`)
4. Add release notes (what's new in this version)
5. Click **Review release**
6. Click **Start rollout to Production**

### Release Tracks

| Track | Purpose | Audience |
|---|---|---|
| Internal testing | Quick testing, no review | Up to 100 testers |
| Closed testing | Beta testing with limited audience | Invite-only testers |
| Open testing | Public beta | Anyone can join |
| Production | Full release | All users |

Recommended flow: Internal → Closed → Open → Production

### Staged Rollouts (Production)

Instead of releasing to 100% of users immediately:

1. Start with a small percentage (e.g., 5%, 10%, 25%)
2. Monitor crash reports and user feedback
3. Gradually increase to 100%
4. Can halt the rollout if issues are found

### Content Rating (IARC)

Required before publishing:

1. Go to **Policy** → **App content** → **Content rating**
2. Complete the IARC questionnaire about your app's content:
   - Violence
   - Sexuality
   - Language
   - Controlled substances
   - User-generated content
   - Personal information collection
3. Receive automatic ratings for all regions (ESRB, PEGI, etc.)

### Data Safety Section

Required for all apps:

1. Go to **Policy** → **App content** → **Data safety**
2. Declare what data your app collects, shares, and how it handles it:
   - Data types collected (location, contacts, photos, etc.)
   - Whether data is shared with third parties
   - Whether data is encrypted in transit
   - Whether users can request data deletion
3. Must accurately reflect your app's behavior

## App Store Submission (iOS)

### App Store Connect Setup

1. Sign in to [App Store Connect](https://appstoreconnect.apple.com)
2. Go to **My Apps** → **+** → **New App**
3. Fill in: Platform (iOS), Name, Primary Language, Bundle ID, SKU

### Store Listing Requirements

| Requirement | Specification |
|---|---|
| App name | Max 30 characters |
| Subtitle | Max 30 characters |
| Promotional text | Max 170 characters (can be updated without new build) |
| Description | Max 4000 characters |
| Keywords | Max 100 characters (comma-separated) |
| App icon | 1024x1024 PNG, no alpha, no rounded corners (system applies them) |
| 6.7" screenshots | Required (iPhone 15 Pro Max). Min 3, max 10. |
| 6.5" screenshots | Required (iPhone 15 Plus). Min 3, max 10. |
| 5.5" screenshots | Optional (iPhone 8 Plus). |
| iPad Pro 12.9" screenshots | Required if app runs on iPad. Min 3, max 10. |
| Privacy policy URL | Required |
| Support URL | Required |
| App category | Primary and optional secondary |
| Age rating | Complete Apple's age rating questionnaire |
| Copyright | Year and copyright holder |

### Screenshot Specifications

| Device | Size (pixels) | Required |
|---|---|---|
| iPhone 6.7" display | 1290 x 2796 | Yes |
| iPhone 6.5" display | 1284 x 2778 or 1242 x 2688 | Yes |
| iPhone 5.5" display | 1242 x 2208 | Optional |
| iPad Pro 12.9" (6th gen) | 2048 x 2732 | If iPad supported |
| iPad Pro 12.9" (2nd gen) | 2048 x 2732 | If iPad supported |

### Submitting for Review

1. In App Store Connect, go to your app
2. Under the version section, fill in all required metadata
3. Upload screenshots for all required device sizes
4. Add build (select from uploaded builds)
5. Complete the **App Review Information**:
   - Contact information for the reviewer
   - Demo account credentials (if app requires login)
   - Notes for the reviewer (explain special features, testing instructions)
6. Answer **Export Compliance** questions (encryption usage)
7. Answer **Content Rights** questions
8. Set **Release Options**: Manual release, automatic after approval, or scheduled
9. Click **Submit for Review**

### App Review Timeline

- Typical review: 24-48 hours (can be longer)
- Expedited review: Request via [reportaproblem.apple.com](https://reportaproblem.apple.com) for critical fixes
- Rejection: Apple provides specific reasons. Fix and resubmit.

### Common Rejection Reasons

- **Crashes or bugs** — test thoroughly on real devices
- **Incomplete information** — provide demo accounts if login is required
- **Privacy violations** — missing privacy policy, collecting data without consent
- **Misleading metadata** — screenshots or description don't match actual functionality
- **Minimum functionality** — app must provide value beyond a simple website wrapper
- **Permissions without justification** — requesting permissions not clearly needed

## Automated Deployment

### Fastlane

Fastlane automates building and deploying for both platforms.

**iOS Fastfile** (`ios/fastlane/Fastfile`):

```ruby
default_platform(:ios)

platform :ios do
  desc "Deploy to TestFlight"
  lane :beta do
    build_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store"
    )
    upload_to_testflight
  end

  desc "Deploy to App Store"
  lane :release do
    build_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store"
    )
    upload_to_app_store(
      skip_metadata: false,
      skip_screenshots: true
    )
  end
end
```

**Android Fastfile** (`android/fastlane/Fastfile`):

```ruby
default_platform(:android)

platform :android do
  desc "Deploy to Google Play internal testing"
  lane :beta do
    upload_to_play_store(
      track: 'internal',
      aab: '../build/app/outputs/bundle/release/app-release.aab'
    )
  end

  desc "Deploy to Google Play production"
  lane :release do
    upload_to_play_store(
      track: 'production',
      aab: '../build/app/outputs/bundle/release/app-release.aab'
    )
  end
end
```

### Shorebird (Code Push for Flutter)

Shorebird enables over-the-air updates for Dart code without going through store review:

```bash
# Install Shorebird
curl --proto '=https' --tlsv1.2 https://raw.githubusercontent.com/shorebirdtech/install/main/install.sh -sSf | bash

# Initialize in your project
shorebird init

# Create a release
shorebird release android
shorebird release ios

# Push a patch (OTA update)
shorebird patch android
shorebird patch ios
```

Note: Shorebird can only update Dart code, not native code or assets.

## Common Mistakes

1. **Uploading APK instead of AAB to Google Play** — Google Play strongly prefers AAB format. APKs are still accepted but AAB provides smaller downloads through dynamic delivery. Use `flutter build appbundle`.

2. **Not testing the release build on a real device** — Release mode behaves differently from debug mode (no hot reload, tree shaking, code stripping). Always test `flutter run --release` on a physical device before submitting.

3. **Missing ProGuard rules** — If `minifyEnabled true` is set in build.gradle, R8/ProGuard may strip classes needed at runtime (especially for reflection-based libraries like JSON serialization). If the release build crashes but debug works, check ProGuard rules.

4. **Forgetting debug info for crash reporting** — When using `--obfuscate`, crash stack traces are unreadable without the debug symbols from `--split-debug-info`. Store these files and upload them to your crash reporting service (Firebase Crashlytics, Sentry, etc.).

5. **Screenshots don't match the app** — App Store and Google Play reviewers compare screenshots to actual app behavior. Outdated or misleading screenshots cause rejection.

6. **Not answering export compliance (iOS)** — Every build uploaded to App Store Connect requires an answer to "Does this app use encryption?" Most apps using HTTPS should answer "Yes" but qualify for an exemption. Not answering this blocks TestFlight distribution.

7. **Submitting without privacy policy** — Both stores require a privacy policy URL. Even if your app collects no data, you need a privacy policy stating that fact.

8. **Version/build number not incremented** — Both stores reject uploads with the same version and build number as a previous upload. Always increment before building.

9. **Not handling the Android back button** — Google Play reviewers test the back button. If pressing back on the main screen does nothing or behaves unexpectedly, it can cause rejection. Use `WillPopScope` or `PopScope` to handle back navigation properly.

10. **Uploading to the wrong track** — Accidentally uploading to Production instead of Internal Testing on Google Play. Always double-check the track before uploading. Use staged rollouts for production releases.
