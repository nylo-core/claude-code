---
name: flutter-app-config
description: Use when the user wants to change the app name, bundle ID, package name, or app display name in a Flutter project. Covers iOS and Android platform-specific configuration files.
---

# Flutter App Configuration

## Overview

Changing a Flutter app's name, bundle identifier, or package name requires modifying multiple platform-specific files. A missed file causes build failures or store rejection. This skill covers every file that needs updating on both iOS and Android.

## When to Use

- User wants to change the app display name (what appears under the icon)
- User wants to change the iOS bundle identifier (e.g., `com.company.appName`)
- User wants to change the Android application ID / package name (e.g., `com.company.app_name`)
- User is preparing an app for release and needs correct identifiers
- User is cloning/forking a project and needs to rebrand it
- **NOT** for changing the Dart package name in `pubspec.yaml` alone (that only affects Dart imports)

## Quick Reference

| What to Change | iOS Files | Android Files |
|---|---|---|
| Display name | `ios/Runner/Info.plist` → `CFBundleDisplayName`, `CFBundleName` | `android/app/src/main/AndroidManifest.xml` → `android:label` |
| Bundle ID / App ID | `ios/Runner.xcodeproj/project.pbxproj` → `PRODUCT_BUNDLE_IDENTIFIER` | `android/app/build.gradle` → `applicationId` (or `namespace` in newer Gradle) |
| Package name (Android) | N/A | `AndroidManifest.xml` → `package`, Kotlin/Java directory structure, `build.gradle` → `namespace` |

## Changing the App Display Name

The display name is what users see under the app icon on their home screen.

### iOS

Edit `ios/Runner/Info.plist`:

```xml
<key>CFBundleDisplayName</key>
<string>My App Name</string>
<key>CFBundleName</key>
<string>My App Name</string>
```

- `CFBundleDisplayName` — the name shown on the home screen (primary)
- `CFBundleName` — short bundle name, used in places where space is limited (max 15 characters recommended)
- If `CFBundleDisplayName` is not present, iOS falls back to `CFBundleName`

### Android

Edit `android/app/src/main/AndroidManifest.xml`:

```xml
<application
    android:label="My App Name"
    ...>
```

The `android:label` attribute on the `<application>` tag sets the app name displayed on the launcher.

## Changing the Bundle Identifier (iOS)

The bundle identifier uniquely identifies your app on iOS (e.g., `com.company.myApp`).

### Files to Modify

**1. `ios/Runner.xcodeproj/project.pbxproj`**

Search for all occurrences of `PRODUCT_BUNDLE_IDENTIFIER` and replace:

```
PRODUCT_BUNDLE_IDENTIFIER = com.newcompany.newAppName;
```

There are typically **3-6 occurrences** — one for each build configuration (Debug, Release, Profile) for each target. Update ALL of them.

**2. `ios/Runner/Info.plist`** (if explicitly set)

Check for a hardcoded `CFBundleIdentifier`. By default it uses `$(PRODUCT_BUNDLE_IDENTIFIER)` which reads from the project file:

```xml
<key>CFBundleIdentifier</key>
<string>$(PRODUCT_BUNDLE_IDENTIFIER)</string>
```

If it contains a hardcoded value instead of `$(PRODUCT_BUNDLE_IDENTIFIER)`, update it.

### Verification

After changing, verify with:

```bash
grep -r "PRODUCT_BUNDLE_IDENTIFIER" ios/Runner.xcodeproj/project.pbxproj
```

All results should show the new identifier.

## Changing the Application ID (Android)

### Modern Approach (AGP 7.0+ / Gradle Kotlin DSL)

**1. `android/app/build.gradle` (or `build.gradle.kts`)**

```groovy
android {
    namespace "com.newcompany.new_app_name"

    defaultConfig {
        applicationId "com.newcompany.new_app_name"
        ...
    }
}
```

- `applicationId` — uniquely identifies the app on Google Play (the "application ID")
- `namespace` — used for R class and BuildConfig generation (AGP 7.0+)

**2. `android/app/src/main/AndroidManifest.xml`**

In older projects, the `package` attribute is set:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.newcompany.new_app_name">
```

In newer Flutter projects using AGP 7.0+, the `package` attribute may be removed from the manifest (replaced by `namespace` in build.gradle). If present, update it to match.

**3. Kotlin/Java Source Directory Structure**

The main activity file lives in a directory matching the package name:

```
android/app/src/main/kotlin/com/newcompany/new_app_name/MainActivity.kt
```

Steps:
1. Create the new directory structure matching the new package name
2. Move `MainActivity.kt` (or `MainActivity.java`) to the new path
3. Update the `package` declaration at the top of the file:

```kotlin
package com.newcompany.new_app_name
```

4. Delete the old empty directories

**4. `android/app/src/debug/AndroidManifest.xml`** and **`android/app/src/profile/AndroidManifest.xml`**

These overlay manifests may also contain the old package name. Check and update if present.

### Verification

```bash
# Check all references to old package name
grep -r "com.oldcompany.old_app_name" android/
```

All results should be eliminated or updated.

## Using rename_app_flutter Package (Automated Approach)

The `rename_app_flutter` package automates most of the renaming process.

### Installation and Usage

```bash
# Add as dev dependency
flutter pub add rename_app_flutter --dev

# Change app name (display name on both platforms)
dart run rename_app_flutter:main all="My New App Name"

# Change bundle ID for iOS
dart run rename_app_flutter:main ios_id="com.newcompany.newApp"

# Change application ID for Android
dart run rename_app_flutter:main android_id="com.newcompany.new_app"

# Change everything at once
dart run rename_app_flutter:main all="My New App Name" android_id="com.newcompany.new_app" ios_id="com.newcompany.newApp"
```

### Alternative: change_app_package_name

For Android package name changes specifically:

```bash
flutter pub add change_app_package_name --dev
dart run change_app_package_name:main com.newcompany.new_app_name
```

This handles:
- `build.gradle` applicationId and namespace
- `AndroidManifest.xml` package attribute
- Kotlin/Java directory structure and package declarations

### After Automated Rename

Always verify after running any automated tool:

```bash
# Rebuild to ensure no broken references
flutter clean
flutter pub get
flutter build ios --no-codesign   # Test iOS build
flutter build apk                  # Test Android build
```

## Full Rename Checklist

When doing a complete rebrand (new name + new identifiers):

1. **`pubspec.yaml`** — update `name:` (Dart package name, snake_case, affects imports)
2. **`pubspec.yaml`** — update `description:`
3. **iOS `Info.plist`** — update `CFBundleDisplayName` and `CFBundleName`
4. **iOS `project.pbxproj`** — update all `PRODUCT_BUNDLE_IDENTIFIER` entries
5. **Android `build.gradle`** — update `applicationId` and `namespace`
6. **Android `AndroidManifest.xml`** — update `package` attribute and `android:label`
7. **Android source directory** — rename directory structure to match new package
8. **Android `MainActivity.kt`** — update `package` declaration
9. **Debug/Profile manifests** — check and update if they reference the old package
10. **Firebase config** (if applicable):
    - `ios/Runner/GoogleService-Info.plist` — re-download from Firebase console
    - `android/app/google-services.json` — re-download from Firebase console
11. **Deep links / URL schemes** — update any custom URL scheme configurations
12. **`flutter clean && flutter pub get`** — rebuild from clean state

## Platform-Specific File Paths Summary

### iOS Files

| File | Keys/Fields |
|---|---|
| `ios/Runner/Info.plist` | `CFBundleDisplayName`, `CFBundleName`, `CFBundleIdentifier` |
| `ios/Runner.xcodeproj/project.pbxproj` | `PRODUCT_BUNDLE_IDENTIFIER` (all occurrences) |

### Android Files

| File | Keys/Fields |
|---|---|
| `android/app/build.gradle` | `applicationId`, `namespace` |
| `android/app/src/main/AndroidManifest.xml` | `package`, `android:label` |
| `android/app/src/main/kotlin/.../MainActivity.kt` | `package` declaration |
| `android/app/src/debug/AndroidManifest.xml` | `package` (if present) |
| `android/app/src/profile/AndroidManifest.xml` | `package` (if present) |

## Common Mistakes

1. **Missing occurrences in project.pbxproj** — There are multiple `PRODUCT_BUNDLE_IDENTIFIER` entries (Debug, Release, Profile). Missing one causes builds to fail for that configuration. Always search and replace ALL occurrences.

2. **Not updating Android directory structure** — Changing `applicationId` in build.gradle but leaving `MainActivity.kt` in the old directory path causes `ClassNotFoundException` at runtime.

3. **Forgetting the namespace in build.gradle** — On AGP 7.0+, `namespace` is separate from `applicationId`. If you only change `applicationId`, the R class and BuildConfig still reference the old namespace.

4. **Case sensitivity mismatch** — iOS bundle IDs are case-sensitive. `com.Company.App` and `com.company.app` are different identifiers. Android application IDs should be lowercase.

5. **Not cleaning after rename** — Old cached artifacts cause confusing errors. Always run `flutter clean && flutter pub get` after renaming.

6. **Firebase config mismatch** — If the project uses Firebase, the bundle ID / application ID in `GoogleService-Info.plist` and `google-services.json` must match exactly. Re-download these files from the Firebase console after changing identifiers.

7. **Changing pubspec.yaml name without updating imports** — The `name` field in pubspec.yaml affects Dart package imports. If you change it from `old_name` to `new_name`, every `import 'package:old_name/...'` must be updated to `import 'package:new_name/...'`.

8. **Android package name with hyphens** — Android package names cannot contain hyphens. Use underscores: `com.company.my_app` not `com.company.my-app`.

9. **iOS bundle ID starting with number** — Each component of an iOS bundle identifier must start with a letter. `com.company.2048game` is invalid; use `com.company.game2048`.
