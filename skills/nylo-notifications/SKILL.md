---
name: nylo-notifications
description: Use when sending local notifications, scheduling reminders, configuring notification channels, handling notification taps, requesting notification permissions, or managing badge counts in a Nylo v7 Flutter app.
---

# Nylo Local Notifications

## Overview

Nylo v7 provides a `LocalNotification` class for sending immediate or scheduled local notifications on iOS and Android. It wraps platform-specific notification APIs behind a fluent builder pattern, with static convenience methods for common operations. Local notifications are not supported on web and will throw a `NotificationException` if attempted.

## When to Use

- Sending immediate local notifications to the user
- Scheduling notifications for a future date/time
- Configuring Android notification channels (importance, vibration, LEDs, sounds)
- Configuring iOS notification presentation (banners, alerts, badges, interruption level)
- Attaching media to iOS notifications
- Handling notification permissions
- Cancelling or clearing notifications
- Managing app badge counts
- When NOT to use: For push notifications from a remote server, use a dedicated push notification service (e.g., Firebase Cloud Messaging). This skill covers local-only notifications.

## Quick Reference

| Action | Code |
|--------|------|
| Send immediately | `await LocalNotification(title: "Hi", body: "World").send()` |
| Send with helper | `await localNotification("Hi", "World").send()` |
| Send via static method | `await LocalNotification.sendNotification(title: "Hi", body: "World")` |
| Schedule for later | `.send(at: DateTime.now().add(Duration(hours: 1)))` |
| Add payload | `.addPayload("order_id:456")` |
| Set notification ID | `.addId(42)` |
| Add subtitle | `.addSubtitle("From support")` |
| Set badge number | `.addBadgeNumber(3)` |
| Custom sound | `.addSound("alert.wav")` |
| iOS attachment | `.addAttachment("https://example.com/img.jpg", "img.jpg")` |
| Android config | `.setAndroidConfig(AndroidNotificationConfig(...))` |
| iOS config | `.setIOSConfig(IOSNotificationConfig(...))` |
| Cancel by ID | `await LocalNotification.cancelNotification(42)` |
| Cancel all | `await LocalNotification.cancelAllNotifications()` |
| Request permissions | `await LocalNotification.requestPermissions()` |
| Clear badge count | `await LocalNotification.clearBadgeCount()` |

## Sending Notifications

### Three Approaches

Nylo offers three equivalent ways to send a local notification:

```dart
// 1. Builder pattern (recommended for chaining options)
await LocalNotification(title: "Hello", body: "World").send();

// 2. Helper function shorthand
await localNotification("Hello", "World").send();

// 3. Static method (best for simple one-off sends)
await LocalNotification.sendNotification(
  title: "Hello",
  body: "World",
);
```

All three produce the same result. Use the builder pattern when you need to chain configuration methods before sending:

```dart
await LocalNotification(title: "New Photo", body: "Check this out!")
    .addPayload("photo_id:123")
    .addId(42)
    .addSubtitle("From your friend")
    .addBadgeNumber(3)
    .addSound("custom_sound.wav")
    .send();
```

### Available Chainable Methods

| Method | Parameters | Purpose |
|--------|------------|---------|
| `addPayload` | `String payload` | Attach custom data string for tap handling |
| `addId` | `int id` | Unique identifier for cancelling/updating |
| `addSubtitle` | `String subtitle` | Subtitle text below the title |
| `addBadgeNumber` | `int badgeNumber` | Set app icon badge count |
| `addSound` | `String sound` | Custom notification sound file |
| `addAttachment` | `String url, String fileName, {bool? showThumbnail}` | Media attachment (iOS only) |
| `setAndroidConfig` | `AndroidNotificationConfig config` | Android-specific settings |
| `setIOSConfig` | `IOSNotificationConfig config` | iOS-specific settings |

## Scheduling Notifications

Schedule a notification for a specific future date/time by passing the `at` parameter to `send()`:

```dart
final tomorrow = DateTime.now().add(Duration(days: 1));

// Builder pattern
await LocalNotification(
  title: "Reminder",
  body: "Don't forget your appointment!",
).send(at: tomorrow);

// Static method
await LocalNotification.sendNotification(
  title: "Reminder",
  body: "Don't forget your appointment!",
  at: tomorrow,
);
```

The `send()` method accepts optional `at` (`DateTime?`) and `androidScheduleMode` (`AndroidScheduleMode?`) parameters. Use `androidScheduleMode` to control scheduling precision on Android (exact vs. inexact alarms).

### Combining Scheduling with the Nylo Scheduler

Use `Nylo.scheduleOnce` or `Nylo.scheduleOnceDaily` to trigger notification logic at controlled intervals:

```dart
// Send a review prompt notification only once
Nylo.scheduleOnce('review_prompt', () async {
  await LocalNotification(
    title: "Enjoying the app?",
    body: "Leave us a review!",
  ).send();
});

// Send a daily reminder notification
Nylo.scheduleOnceDaily('daily_tip', () async {
  await LocalNotification(
    title: "Daily Tip",
    body: "Stay hydrated!",
  ).send();
});
```

## Android Configuration

Apply Android-specific settings using `AndroidNotificationConfig`:

```dart
await LocalNotification(
  title: "Android Notification",
  body: "With custom configuration",
)
.setAndroidConfig(AndroidNotificationConfig(
  channelId: "custom_channel",
  channelName: "Custom Channel",
  channelDescription: "Notifications from custom channel",
  importance: Importance.max,
  priority: Priority.high,
  enableVibration: true,
  vibrationPattern: [0, 1000, 500, 1000],
  enableLights: true,
  ledColor: Color(0xFF00FF00),
))
.send();
```

### Key AndroidNotificationConfig Properties

**Channel:** `channelId` (default `'default_channel'`), `channelName` (default `'Default Channel'`), `channelDescription`

**Behavior:** `importance` (default `Importance.max`), `priority` (default `Priority.high`), `playSound` (default `true`), `enableVibration` (default `true`), `vibrationPattern` (`List<int>?`), `enableLights` (default `false`), `ledColor`, `silent` (default `false`)

**Display:** `autoCancel` (default `true`), `ongoing` (default `false`), `color`, `largeIcon`, `visibility` (`NotificationVisibility?`), `fullScreenIntent` (default `false`)

**Grouping:** `groupKey`, `setAsGroupSummary` (default `false`)

**Progress:** `showProgress` (default `false`), `progress`, `maxProgress`, `indeterminate` (default `false`)

**Actions:** `actions` (`List<AndroidNotificationAction>?`)

### Android Notification Channels

Android 8.0+ requires notification channels. Use `channelId`, `channelName`, and `channelDescription` to define them:

```dart
// High-priority alerts channel
.setAndroidConfig(AndroidNotificationConfig(
  channelId: "alerts",
  channelName: "Alerts",
  channelDescription: "Urgent alerts and warnings",
  importance: Importance.max,
  priority: Priority.high,
))

// Silent updates channel
.setAndroidConfig(AndroidNotificationConfig(
  channelId: "updates",
  channelName: "Updates",
  channelDescription: "Background updates",
  importance: Importance.low,
  silent: true,
))
```

### Android Progress Notifications

Display a progress bar for ongoing operations:

```dart
await LocalNotification(
  title: "Downloading...",
  body: "50% complete",
)
.addId(100)
.setAndroidConfig(AndroidNotificationConfig(
  showProgress: true,
  progress: 50,
  maxProgress: 100,
  ongoing: true,
  autoCancel: false,
))
.send();
```

## iOS Configuration

Apply iOS-specific settings using `IOSNotificationConfig`:

```dart
await LocalNotification(
  title: "iOS Notification",
  body: "With custom configuration",
)
.setIOSConfig(IOSNotificationConfig(
  presentAlert: true,
  presentBanner: true,
  presentSound: true,
  threadIdentifier: "thread_1",
  interruptionLevel: InterruptionLevel.active,
))
.addBadgeNumber(1)
.addSound("custom_sound.wav")
.send();
```

### Key IOSNotificationConfig Properties

**Presentation:** `presentList` (default `true`), `presentAlert` (default `true`), `presentBadge` (default `true`), `presentSound` (default `true`), `presentBanner` (default `true`)

**Content:** `sound` (`String?`), `badgeNumber` (`int?`), `interruptionLevel` (`InterruptionLevel?`)

**Grouping:** `threadIdentifier` (`String?` -- groups notifications by thread), `categoryIdentifier` (`String?` -- links to custom action categories)

### iOS Attachments

Add media (images, audio, video) to iOS notifications by providing a download URL:

```dart
await LocalNotification(
  title: "New Photo",
  body: "Check out this image!",
)
.addAttachment(
  "https://example.com/image.jpg",
  "photo.jpg",
  showThumbnail: true,
)
.send();
```

The file is downloaded from the URL and attached to the notification. The `showThumbnail` parameter controls whether a preview appears in the notification.

## Platform Setup

### iOS (Info.plist)

Add background notification mode to `ios/Runner/Info.plist`:

```xml
<key>UIBackgroundModes</key>
<array>
    <string>remote-notification</string>
</array>
```

### Android (AndroidManifest.xml)

Add required permissions to `android/app/src/main/AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<uses-permission android:name="android.permission.VIBRATE" />
<uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT" />
```

- `RECEIVE_BOOT_COMPLETED` -- Allows scheduled notifications to persist after device restart.
- `VIBRATE` -- Required for vibration patterns.
- `USE_FULL_SCREEN_INTENT` -- Required if using `fullScreenIntent: true`.

## Permission Handling

### Basic Permission Request

```dart
await LocalNotification.requestPermissions();
```

Permissions are automatically requested on the first notification send via Nylo's `NyScheduler.taskOnce`, so explicit calls are only needed if you want to prompt the user at a specific point in the app flow.

### Granular Permission Request

Request specific permission types with named parameters:

```dart
await LocalNotification.requestPermissions(
  alert: true,
  badge: true,
  sound: true,
  provisional: false,   // iOS: quiet/provisional authorization
  critical: false,      // iOS: bypass Do Not Disturb
  vibrate: true,        // Android
  enableLights: true,   // Android
  channelId: 'default_notification_channel_id',   // Android
  channelName: 'Default Notification Channel',    // Android
);
```

### Requesting Permissions Early

Prompt for permissions during onboarding instead of waiting for the first notification:

```dart
class _HomePageState extends NyPage<HomePage> {
  @override
  get init => () async {
    await LocalNotification.requestPermissions();
  };
}
```

## Cancelling and Managing Notifications

### Cancel a Specific Notification

Cancel by the ID assigned with `addId()`:

```dart
await LocalNotification.cancelNotification(42);
```

On Android, cancel by ID and tag:

```dart
await LocalNotification.cancelNotification(42, tag: "my_tag");
```

### Cancel All Notifications

Remove all pending and displayed notifications:

```dart
await LocalNotification.cancelAllNotifications();
```

### Clear Badge Count

Reset the app icon badge number to zero:

```dart
await LocalNotification.clearBadgeCount();
```

### Updating a Notification

Send a new notification with the same ID to replace an existing one:

```dart
// Send initial notification
await LocalNotification(title: "Downloading...", body: "25%")
    .addId(100)
    .send();

// Update it by reusing the same ID
await LocalNotification(title: "Downloading...", body: "75%")
    .addId(100)
    .send();
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using local notifications on web | Local notifications are not supported on web and will throw `NotificationException`. Gate notification code behind platform checks or only call from mobile-specific flows. |
| Not assigning an ID before cancelling | `cancelNotification()` requires the `int id` you set with `addId()`. Always assign an ID to notifications you intend to cancel or update later. |
| Forgetting Android manifest permissions | Add `RECEIVE_BOOT_COMPLETED` and `VIBRATE` permissions to `AndroidManifest.xml` or scheduled notifications and vibration will silently fail. |
| Creating too many Android channels | Android channels persist once created and the user manages them in Settings. Plan channel IDs carefully; do not create a new channel per notification. |
| Not requesting permissions before checking results | Call `requestPermissions()` before the first send if you need to handle the denied case gracefully; otherwise Nylo auto-requests on first send via `NyScheduler.taskOnce`. |
| Scheduling with a past DateTime | Passing a `DateTime` in the past to `send(at: ...)` will either fire immediately or be silently ignored depending on the platform. Always validate the date is in the future. |
| Sending iOS attachments with invalid URLs | `addAttachment()` downloads the file from the URL at send time. If the URL is unreachable, the notification may send without the attachment. Validate URLs before sending. |
| Not updating notifications correctly | To update a displayed notification, send a new one with the same `addId()` value. Using a different ID creates a separate notification instead of replacing the existing one. |
| Using `ongoing: true` without `autoCancel: false` | Ongoing Android notifications should set `autoCancel: false` so the notification persists after interaction. Otherwise the notification dismisses on tap despite being marked ongoing. |
