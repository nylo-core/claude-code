---
name: flutter-permissions
description: Use when the user needs to add platform permissions for camera, photos, location, microphone, contacts, notifications, bluetooth, or other device capabilities in a Flutter app. Covers iOS Info.plist keys and Android AndroidManifest.xml permissions.
---

# Flutter Permissions

## Overview

Flutter apps must declare permissions in platform-specific configuration files and request them at runtime. iOS uses `Info.plist` usage description strings, Android uses `AndroidManifest.xml` permission declarations. Both platforms require runtime permission requests for sensitive capabilities.

## When to Use

- User needs to access camera, photos, location, microphone, contacts, or other device features
- App crashes with permission-related errors
- User is adding a plugin that requires platform permissions (e.g., image_picker, geolocator)
- Store review rejected the app for missing usage descriptions (iOS) or excessive permissions
- User wants to implement runtime permission handling
- **NOT** for internet permission alone on Android (included by default in Flutter)

## Quick Reference: Common Permissions

| Feature | iOS Info.plist Key | Android Manifest Permission |
|---|---|---|
| Camera | `NSCameraUsageDescription` | `android.permission.CAMERA` |
| Photo Library (read) | `NSPhotoLibraryUsageDescription` | `android.permission.READ_MEDIA_IMAGES` (API 33+) / `READ_EXTERNAL_STORAGE` (older) |
| Photo Library (write) | `NSPhotoLibraryAddUsageDescription` | `android.permission.WRITE_EXTERNAL_STORAGE` (API < 29) |
| Microphone | `NSMicrophoneUsageDescription` | `android.permission.RECORD_AUDIO` |
| Location (when in use) | `NSLocationWhenInUseUsageDescription` | `android.permission.ACCESS_FINE_LOCATION` |
| Location (always) | `NSLocationAlwaysAndWhenInUseUsageDescription` | `android.permission.ACCESS_BACKGROUND_LOCATION` |
| Contacts | `NSContactsUsageDescription` | `android.permission.READ_CONTACTS` |
| Calendar | `NSCalendarsUsageDescription` | `android.permission.READ_CALENDAR` |
| Reminders | `NSRemindersUsageDescription` | (use Calendar permission) |
| Bluetooth | `NSBluetoothAlwaysUsageDescription` | `android.permission.BLUETOOTH_CONNECT` (API 31+) / `BLUETOOTH` (older) |
| Notifications | Not in Info.plist (requested via API) | `android.permission.POST_NOTIFICATIONS` (API 33+) |
| Face ID | `NSFaceIDUsageDescription` | N/A |
| Speech Recognition | `NSSpeechRecognitionUsageDescription` | `android.permission.RECORD_AUDIO` |
| Motion & Fitness | `NSMotionUsageDescription` | `android.permission.ACTIVITY_RECOGNITION` (API 29+) |
| Local Network | `NSLocalNetworkUsageDescription` | N/A |
| Tracking (ATT) | `NSUserTrackingUsageDescription` | `com.google.android.gms.permission.AD_ID` |

## iOS Permissions (Info.plist)

### File Location

```
ios/Runner/Info.plist
```

### How to Add a Permission

Add key-value pairs inside the top-level `<dict>` tag. Every iOS permission requires a **usage description string** explaining why the app needs access. This string is shown to the user in the system permission dialog.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- Existing keys... -->

    <!-- Camera -->
    <key>NSCameraUsageDescription</key>
    <string>This app needs camera access to take photos for your profile.</string>

    <!-- Photo Library Read -->
    <key>NSPhotoLibraryUsageDescription</key>
    <string>This app needs access to your photos to let you choose a profile picture.</string>

    <!-- Photo Library Write -->
    <key>NSPhotoLibraryAddUsageDescription</key>
    <string>This app saves edited photos to your photo library.</string>

    <!-- Microphone -->
    <key>NSMicrophoneUsageDescription</key>
    <string>This app needs microphone access to record audio messages.</string>

    <!-- Location When In Use -->
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>This app uses your location to show nearby restaurants.</string>

    <!-- Location Always -->
    <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
    <string>This app uses your location in the background to send arrival notifications.</string>

    <!-- Contacts -->
    <key>NSContactsUsageDescription</key>
    <string>This app accesses your contacts to help you find friends.</string>

    <!-- Bluetooth -->
    <key>NSBluetoothAlwaysUsageDescription</key>
    <string>This app uses Bluetooth to connect to nearby fitness devices.</string>

    <!-- Face ID -->
    <key>NSFaceIDUsageDescription</key>
    <string>This app uses Face ID to securely authenticate your identity.</string>

    <!-- Tracking (App Tracking Transparency) -->
    <key>NSUserTrackingUsageDescription</key>
    <string>This app uses tracking to provide personalized ads.</string>

    <!-- Local Network -->
    <key>NSLocalNetworkUsageDescription</key>
    <string>This app discovers devices on your local network for casting.</string>

    <!-- Motion & Fitness -->
    <key>NSMotionUsageDescription</key>
    <string>This app uses motion data to count your steps.</string>

    <!-- Speech Recognition -->
    <key>NSSpeechRecognitionUsageDescription</key>
    <string>This app uses speech recognition for voice commands.</string>

    <!-- Calendars -->
    <key>NSCalendarsUsageDescription</key>
    <string>This app adds events to your calendar.</string>
</dict>
</plist>
```

### iOS Usage Description Guidelines

- **Be specific**: "This app needs camera access to scan QR codes for check-in" is better than "Camera access needed"
- **Explain the benefit to the user**: Frame it as what the user gains, not what the app needs
- **Apple reviews these strings**: Vague descriptions cause App Store rejection
- **Each permission needs its own description**: You cannot share descriptions between permissions
- **Localization**: For multi-language apps, localize usage descriptions via `InfoPlist.strings` files

### iOS Background Modes

For background capabilities, add to `Info.plist`:

```xml
<key>UIBackgroundModes</key>
<array>
    <string>location</string>        <!-- Background location -->
    <string>fetch</string>            <!-- Background fetch -->
    <string>remote-notification</string> <!-- Push notifications -->
    <string>audio</string>            <!-- Background audio -->
    <string>bluetooth-central</string> <!-- Bluetooth background -->
</array>
```

## Android Permissions (AndroidManifest.xml)

### File Location

```
android/app/src/main/AndroidManifest.xml
```

### How to Add a Permission

Add `<uses-permission>` elements inside the `<manifest>` tag but before `<application>`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Camera -->
    <uses-permission android:name="android.permission.CAMERA" />

    <!-- Photo/Media Access (Android 13+ / API 33+) -->
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    <uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />
    <uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />

    <!-- Photo/Media Access (Android 12 and below) -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
        android:maxSdkVersion="32" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
        android:maxSdkVersion="28" />

    <!-- Microphone -->
    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    <!-- Location -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />

    <!-- Contacts -->
    <uses-permission android:name="android.permission.READ_CONTACTS" />
    <uses-permission android:name="android.permission.WRITE_CONTACTS" />

    <!-- Calendar -->
    <uses-permission android:name="android.permission.READ_CALENDAR" />
    <uses-permission android:name="android.permission.WRITE_CALENDAR" />

    <!-- Bluetooth (Android 12+ / API 31+) -->
    <uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />

    <!-- Bluetooth (Android 11 and below) -->
    <uses-permission android:name="android.permission.BLUETOOTH"
        android:maxSdkVersion="30" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"
        android:maxSdkVersion="30" />

    <!-- Notifications (Android 13+ / API 33+) -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

    <!-- Phone -->
    <uses-permission android:name="android.permission.CALL_PHONE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />

    <!-- Activity Recognition (Android 10+ / API 29+) -->
    <uses-permission android:name="android.permission.ACTIVITY_RECOGNITION" />

    <!-- Biometrics -->
    <uses-permission android:name="android.permission.USE_BIOMETRIC" />

    <!-- Vibration -->
    <uses-permission android:name="android.permission.VIBRATE" />

    <application ...>
        ...
    </application>
</manifest>
```

### Android Permission Categories

**Normal Permissions** (granted automatically at install, no runtime prompt):
- `INTERNET` — already included by Flutter
- `VIBRATE`
- `WAKE_LOCK`
- `ACCESS_NETWORK_STATE`
- `RECEIVE_BOOT_COMPLETED`

**Dangerous Permissions** (require runtime request):
- `CAMERA`, `RECORD_AUDIO`
- `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION`, `ACCESS_BACKGROUND_LOCATION`
- `READ_CONTACTS`, `WRITE_CONTACTS`
- `READ_CALENDAR`, `WRITE_CALENDAR`
- `READ_EXTERNAL_STORAGE`, `READ_MEDIA_IMAGES`
- `CALL_PHONE`, `READ_PHONE_STATE`
- `BLUETOOTH_CONNECT`, `BLUETOOTH_SCAN`
- `POST_NOTIFICATIONS`
- `ACTIVITY_RECOGNITION`

### Using maxSdkVersion

Limit permissions to specific API levels when newer APIs provide alternatives:

```xml
<!-- Only needed on Android 12 and below; Android 13+ uses READ_MEDIA_IMAGES -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
    android:maxSdkVersion="32" />
```

## Runtime Permission Handling (permission_handler)

### Setup

Add to `pubspec.yaml`:

```yaml
dependencies:
  permission_handler: ^11.0.0
```

### iOS Podfile Configuration

In `ios/Podfile`, add the permissions your app uses inside `post_install`:

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)

    target.build_configurations.each do |config|
      config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= [
        '$(inherited)',
        ## Enable the permissions you need:
        'PERMISSION_CAMERA=1',
        'PERMISSION_PHOTOS=1',
        'PERMISSION_MICROPHONE=1',
        'PERMISSION_LOCATION=1',
        'PERMISSION_CONTACTS=1',
        'PERMISSION_NOTIFICATIONS=1',
        'PERMISSION_BLUETOOTH=1',
        'PERMISSION_APP_TRACKING_TRANSPARENCY=1',
        'PERMISSION_SPEECH_RECOGNIZER=1',
        'PERMISSION_SENSORS=1',
        'PERMISSION_CALENDARS=1',
        # 'PERMISSION_MEDIA_LIBRARY=1',
        # 'PERMISSION_REMINDERS=1',
      ]
    end
  end
end
```

### Requesting Permissions in Dart

```dart
import 'package:permission_handler/permission_handler.dart';

// Check current status
final status = await Permission.camera.status;
if (status.isDenied) {
  // Request permission
  final result = await Permission.camera.request();
  if (result.isGranted) {
    // Permission granted - proceed
  } else if (result.isPermanentlyDenied) {
    // User permanently denied - open app settings
    await openAppSettings();
  }
}

// Request multiple permissions at once
Map<Permission, PermissionStatus> statuses = await [
  Permission.camera,
  Permission.microphone,
].request();

// Check if granted
if (statuses[Permission.camera]!.isGranted &&
    statuses[Permission.microphone]!.isGranted) {
  // Both permissions granted
}
```

### Permission Status Values

| Status | Meaning |
|---|---|
| `granted` | Permission is granted |
| `denied` | Permission is denied (can request again) |
| `permanentlyDenied` | Permission denied and cannot be requested again (Android). Must open app settings. |
| `restricted` | Permission is restricted by OS (iOS, e.g., parental controls) |
| `limited` | Limited access granted (iOS 14+ photo library) |
| `provisional` | Provisional permission (iOS notifications) |

### Common Permission Request Pattern

```dart
Future<bool> requestPermission(Permission permission) async {
  // Check if already granted
  if (await permission.isGranted) {
    return true;
  }

  // Request the permission
  final status = await permission.request();

  if (status.isGranted) {
    return true;
  }

  if (status.isPermanentlyDenied) {
    // Show dialog explaining why permission is needed
    // and offer to open settings
    final opened = await openAppSettings();
    return false;
  }

  return false;
}

// Usage
if (await requestPermission(Permission.camera)) {
  // Take photo
}
```

### Location Permission (Step-by-Step)

Location has a special flow — you must request foreground before background:

```dart
// Step 1: Request when-in-use location
var status = await Permission.locationWhenInUse.request();
if (!status.isGranted) return;

// Step 2: If you need background location, request it separately
// (Only after when-in-use is granted)
if (await Permission.locationAlways.isDenied) {
  status = await Permission.locationAlways.request();
}
```

### Notification Permission (Android 13+ / iOS)

```dart
// Android 13+ requires explicit notification permission
// iOS always requires it
final status = await Permission.notification.request();
if (status.isGranted) {
  // Can show notifications
}
```

## Common Mistakes

1. **Missing iOS usage description** — Adding the permission to the Podfile preprocessor macros but forgetting the `Info.plist` usage description string. The app will crash when requesting the permission.

2. **Vague iOS usage descriptions** — Apple rejects apps with generic descriptions like "This app needs camera access." Be specific about why and how the feature benefits the user.

3. **Not configuring Podfile for permission_handler** — The `permission_handler` package requires `GCC_PREPROCESSOR_DEFINITIONS` in the Podfile for iOS. Without this, permissions silently fail or the app crashes.

4. **Requesting background location first** — On Android, you must first obtain `ACCESS_FINE_LOCATION` or `ACCESS_COARSE_LOCATION` before requesting `ACCESS_BACKGROUND_LOCATION`. Requesting background first will be denied.

5. **Forgetting maxSdkVersion for storage permissions** — On Android 13+, `READ_EXTERNAL_STORAGE` is replaced by granular media permissions (`READ_MEDIA_IMAGES`, etc.). Include `android:maxSdkVersion="32"` on the old permission to avoid requesting unnecessary access.

6. **Not handling permanentlyDenied** — On Android, after the user denies a permission twice, it becomes permanently denied. The only option is to direct users to app settings via `openAppSettings()`. Failing to handle this leaves users stuck.

7. **Requesting permissions at app launch** — Requesting all permissions on first launch before the user understands why leads to denials. Request permissions contextually, right before the feature that needs them.

8. **Not declaring Android permissions in manifest** — Even with `permission_handler`, you must still declare permissions in `AndroidManifest.xml`. The runtime request will silently fail without the manifest declaration.

9. **Missing Bluetooth permissions for Android 12+** — Android 12 (API 31) introduced new granular Bluetooth permissions (`BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT`, `BLUETOOTH_ADVERTISE`) replacing the old `BLUETOOTH` and `BLUETOOTH_ADMIN`. Apps targeting API 31+ must use the new permissions.

10. **Notification permission on Android < 13** — The `POST_NOTIFICATIONS` permission only exists on Android 13+. On older versions, notifications work without any permission. The `permission_handler` package handles this automatically, but don't be confused if the permission shows as granted on older devices.
