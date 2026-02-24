---
name: flutter-app-icons
description: Use when the user wants to set up app launcher icons or splash screens in a Flutter project. Covers flutter_launcher_icons and flutter_native_splash packages with platform-specific configuration.
---

# Flutter App Icons and Splash Screens

## Overview

App launcher icons and splash screens are the first visual impressions of your app. Flutter uses `flutter_launcher_icons` for generating platform-specific icon assets and `flutter_native_splash` for native splash screens. Both are configured in `pubspec.yaml` and generate platform files via CLI commands.

## When to Use

- User wants to set or change the app launcher icon
- User wants to add or customize the splash screen
- User needs adaptive icons for Android
- App is being prepared for store submission and needs proper icons
- User is seeing the default Flutter icon and wants to replace it
- **NOT** for in-app icons or icon fonts (use `Icons` class or custom icon packages instead)

## Quick Reference

| Task | Package | Command |
|---|---|---|
| Generate launcher icons | `flutter_launcher_icons` | `dart run flutter_launcher_icons` |
| Generate splash screen | `flutter_native_splash` | `dart run flutter_native_splash:create` |
| Remove splash screen | `flutter_native_splash` | `dart run flutter_native_splash:remove` |

## Part 1: Launcher Icons (flutter_launcher_icons)

### Setup

Add to `pubspec.yaml`:

```yaml
dev_dependencies:
  flutter_launcher_icons: ^0.14.0
```

### Source Image Requirements

- **Format**: PNG (recommended), also supports JPEG
- **Minimum size**: 1024x1024 pixels
- **Shape**: Square (1:1 aspect ratio)
- **No transparency for iOS**: iOS does not support transparent icons; use a solid background
- **Transparency OK for Android**: Android adaptive icons can use transparency in the foreground layer

### Basic Configuration

Add to `pubspec.yaml` (at root level, not inside `dependencies`):

```yaml
flutter_launcher_icons:
  android: true
  ios: true
  image_path: "assets/icon/app_icon.png"
  min_sdk_android: 21
```

### Full Configuration with Adaptive Icons

```yaml
flutter_launcher_icons:
  # Enable platforms
  android: true
  ios: true

  # Default icon (used as fallback)
  image_path: "assets/icon/app_icon.png"

  # iOS-specific (no transparency, no alpha channel)
  image_path_ios: "assets/icon/app_icon_ios.png"

  # Android adaptive icon (Android 8.0+ / API 26+)
  adaptive_icon_foreground: "assets/icon/app_icon_foreground.png"
  adaptive_icon_background: "#FFFFFF"
  # OR use an image for background:
  # adaptive_icon_background: "assets/icon/app_icon_background.png"

  # Android monochrome icon (Android 13+ themed icons)
  adaptive_icon_monochrome: "assets/icon/app_icon_monochrome.png"

  # Minimum Android SDK (affects which icon types are generated)
  min_sdk_android: 21

  # Remove alpha channel for iOS (required)
  remove_alpha_ios: true

  # Web favicon
  web:
    generate: true
    image_path: "assets/icon/app_icon.png"
    background_color: "#FFFFFF"
    theme_color: "#000000"

  # macOS
  macos:
    generate: true
    image_path: "assets/icon/app_icon.png"

  # Windows
  windows:
    generate: true
    image_path: "assets/icon/app_icon.png"
    icon_size: 48
```

### Generate Icons

```bash
dart run flutter_launcher_icons
```

### What Gets Generated

**iOS** (`ios/Runner/Assets.xcassets/AppIcon.appiconset/`):
- Multiple PNG sizes from 20x20 to 1024x1024
- `Contents.json` mapping each size to its usage

**Android** (`android/app/src/main/res/`):
- `mipmap-mdpi/` — 48x48
- `mipmap-hdpi/` — 72x72
- `mipmap-xhdpi/` — 96x96
- `mipmap-xxhdpi/` — 144x144
- `mipmap-xxxhdpi/` — 192x192
- For adaptive icons: `mipmap-anydpi-v26/` with XML definitions
- For monochrome: additional monochrome layer resources

### Android Adaptive Icons Explained

Android 8.0+ uses adaptive icons with two layers:

```
┌─────────────────┐
│   Background     │  ← Solid color or image (full bleed)
│  ┌───────────┐  │
│  │ Foreground │  │  ← Logo/symbol (with safe zone padding)
│  └───────────┘  │
└─────────────────┘
```

- **Background layer**: 108x108dp, can be a color or image
- **Foreground layer**: 108x108dp, the main icon graphic
- **Safe zone**: Center 66dp circle — keep important content within this area
- The OS applies the mask shape (circle, squircle, rounded square, etc.)

**Foreground image guidelines**:
- 1024x1024 PNG with transparency
- Keep the main graphic within the center 66% (approximately 676x676 area centered)
- The outer area may be cropped by the adaptive icon mask

### Android 13+ Themed (Monochrome) Icons

Android 13 adds themed icons that match the user's wallpaper colors:

```yaml
adaptive_icon_monochrome: "assets/icon/app_icon_monochrome.png"
```

- Provide a single-color (white on transparent) version of your icon
- The system tints it to match the user's theme
- If not provided, the system uses your regular icon (not tinted)

### Using a Separate Configuration File

Instead of `pubspec.yaml`, create `flutter_launcher_icons.yaml` at the project root:

```yaml
flutter_launcher_icons:
  android: true
  ios: true
  image_path: "assets/icon/app_icon.png"
  adaptive_icon_foreground: "assets/icon/app_icon_foreground.png"
  adaptive_icon_background: "#FFFFFF"
```

Run with:

```bash
dart run flutter_launcher_icons
```

The package automatically detects the separate config file.

### Flavor-Specific Icons

For apps with multiple build flavors:

```yaml
# pubspec.yaml or flutter_launcher_icons.yaml
flutter_launcher_icons:
  android: true
  ios: true
  image_path: "assets/icon/app_icon.png"

# Create flutter_launcher_icons-development.yaml
# for a "development" flavor
flutter_launcher_icons:
  android: "launcher_icon_dev"
  ios: true
  image_path: "assets/icon/app_icon_dev.png"
```

```bash
dart run flutter_launcher_icons --flavor development
```

## Part 2: Splash Screens (flutter_native_splash)

### Setup

Add to `pubspec.yaml`:

```yaml
dev_dependencies:
  flutter_native_splash: ^2.4.0
```

### Basic Configuration

Add to `pubspec.yaml`:

```yaml
flutter_native_splash:
  color: "#FFFFFF"
  image: "assets/splash/splash_logo.png"
```

### Full Configuration

```yaml
flutter_native_splash:
  # Background color
  color: "#FFFFFF"

  # OR use a background image (mutually exclusive with color)
  # background_image: "assets/splash/background.png"

  # Center image (logo)
  image: "assets/splash/splash_logo.png"

  # Branding image (shown at bottom of splash)
  # branding: "assets/splash/branding.png"
  # branding_mode: bottom  # bottom, center, or bottomRight

  # Dark mode variants
  color_dark: "#121212"
  # background_image_dark: "assets/splash/background_dark.png"
  image_dark: "assets/splash/splash_logo_dark.png"
  # branding_dark: "assets/splash/branding_dark.png"

  # Android 12+ splash screen
  android_12:
    color: "#FFFFFF"
    color_dark: "#121212"
    image: "assets/splash/splash_icon.png"
    image_dark: "assets/splash/splash_icon_dark.png"
    # icon_background_color: "#FFFFFF"
    # icon_background_color_dark: "#121212"
    # branding: "assets/splash/branding_android12.png"
    # branding_dark: "assets/splash/branding_android12_dark.png"

  # Platform enable/disable
  android: true
  ios: true
  web: true

  # Keep splash screen visible until manually removed
  # (allows loading data before showing the app)
  android_gravity: center
  ios_content_mode: center

  # Fill the screen (use scaleToFill for the image)
  # android_gravity: fill
  # ios_content_mode: scaleToFill

  # Full screen (hide status bar during splash)
  fullscreen: true
```

### Generate Splash Screen

```bash
dart run flutter_native_splash:create
```

### Remove Splash Screen

```bash
dart run flutter_native_splash:remove
```

### Android 12+ Splash Screen

Android 12 introduced a new splash screen API with specific constraints:

- **Icon**: Center icon displayed in a circle, 240x240dp with 160dp visible area (inner 2/3)
- **Icon image size**: Provide at least 768x768px for best results
- **Background**: Single color only (no gradient or image)
- **Animation**: Optional animated icon (not supported via flutter_native_splash)
- **Branding**: Optional text/image at bottom

The icon is displayed inside a circular mask on Android 12+:

```
┌──────────────────┐
│                    │
│    ┌──────────┐   │
│    │  ○ Icon  │   │ ← 240dp container, 160dp visible circle
│    └──────────┘   │
│                    │
│    Branding text   │
└──────────────────┘
```

### Preserving the Splash Screen

To keep the splash visible while the app initializes (loading data, checking auth):

```dart
import 'package:flutter_native_splash/flutter_native_splash.dart';

void main() {
  // Keep the splash screen visible
  WidgetsBinding widgetsBinding = WidgetsFlutterBinding.ensureInitialized();
  FlutterNativeSplash.preserve(widgetsBinding: widgetsBinding);

  // Do initialization work...
  // await loadConfig();
  // await checkAuth();

  runApp(const MyApp());
}

// When ready to show the app (e.g., in your first screen's initState):
class _MyHomePageState extends State<MyHomePage> {
  @override
  void initState() {
    super.initState();
    _initializeApp();
  }

  Future<void> _initializeApp() async {
    // Do async initialization
    await Future.delayed(const Duration(seconds: 2));

    // Remove the splash screen
    FlutterNativeSplash.remove();
  }
}
```

### What Gets Generated

**iOS**:
- `ios/Runner/Assets.xcassets/LaunchImage.imageset/` — splash images
- `ios/Runner/Base.lproj/LaunchScreen.storyboard` — launch storyboard

**Android**:
- `android/app/src/main/res/drawable/` — splash screen XML definitions
- `android/app/src/main/res/values/styles.xml` — splash theme styles
- `android/app/src/main/res/values-night/styles.xml` — dark mode styles
- `android/app/src/main/res/values-v31/styles.xml` — Android 12+ styles

### Using a Separate Configuration File

Create `flutter_native_splash.yaml` at the project root:

```yaml
flutter_native_splash:
  color: "#FFFFFF"
  image: "assets/splash/splash_logo.png"
```

Run with:

```bash
dart run flutter_native_splash:create --path=flutter_native_splash.yaml
```

## Asset Preparation Checklist

### Launcher Icon Assets to Prepare

| Asset | Size | Format | Notes |
|---|---|---|---|
| Main icon | 1024x1024 | PNG | No transparency for iOS |
| iOS icon | 1024x1024 | PNG | No alpha channel, solid background |
| Android foreground | 1024x1024 | PNG | Transparency OK, keep content in center 66% |
| Android background | 1024x1024 | PNG or color | Solid color or full-bleed image |
| Monochrome icon | 1024x1024 | PNG | White on transparent, for Android 13+ |

### Splash Screen Assets to Prepare

| Asset | Recommended Size | Format | Notes |
|---|---|---|---|
| Splash logo | 768x768 | PNG | Centered, with padding |
| Splash logo (dark) | 768x768 | PNG | For dark mode |
| Android 12 icon | 768x768 | PNG | Will be masked to circle (center 2/3 visible) |
| Branding image | 400x100 | PNG | Optional, shown at bottom |

## Common Mistakes

1. **Using transparent PNG for iOS icon** — iOS does not support transparency in app icons. The system fills transparent areas with black, producing an ugly result. Always use a solid background for iOS icons.

2. **Foreground content outside adaptive icon safe zone** — Android adaptive icons can be masked to various shapes. Content outside the center 66% circle may be cropped. Keep your logo/symbol well within the safe zone.

3. **Not providing Android 12 splash configuration** — Android 12+ ignores the legacy splash screen setup. Without the `android_12` section, your splash screen will not display correctly on modern Android devices.

4. **Forgetting to run the generate command** — Editing `pubspec.yaml` config does not automatically update icons or splash screens. You must run `dart run flutter_launcher_icons` or `dart run flutter_native_splash:create` after every config change.

5. **Not running flutter clean after regeneration** — Cached build artifacts may show old icons. Run `flutter clean` followed by a fresh build to ensure the new assets are used.

6. **Splash image too large** — The splash image is centered, not stretched. If it is too large, it will be cropped on smaller screens. Design the splash logo to look good at various screen sizes with generous padding.

7. **Missing dark mode splash** — If your app supports dark mode but the splash screen only has light mode config, users see a jarring white flash when opening the app in dark mode. Always configure `color_dark` and `image_dark`.

8. **Not calling FlutterNativeSplash.remove()** — If you use `FlutterNativeSplash.preserve()` to hold the splash screen but forget to call `remove()`, the app appears frozen on the splash screen forever.

9. **Oversized monochrome icon** — The monochrome icon for Android 13 themed icons should be a simple, recognizable silhouette. Complex detailed icons become unreadable at small sizes when tinted.

10. **Forgetting to add assets to pubspec.yaml assets section** — The icon and splash source images (in `assets/`) need to be accessible to the build tools but do NOT need to be listed in `pubspec.yaml` under `flutter: assets:`. The generator tools read them directly from the file system. However, if your splash code references runtime assets, those do need to be declared.
