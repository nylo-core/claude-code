---
name: nylo-themes
description: Use when working with themes, colors, dark/light mode, fonts, text styles, or asset management in a Nylo v7 Flutter project. Also applies when customizing app appearance, switching themes at runtime, or registering fonts and images.
---

# Nylo Themes, Styling & Assets

## Overview

Nylo v7 provides a centralized theming system with pre-configured Light and Dark themes that respond automatically to device settings. Themes are defined in `lib/resources/themes/`, colors in `lib/resources/themes/styles/`, and design defaults in `lib/config/design.dart`. Assets live in `assets/` and are referenced through helper widgets and functions.

## When to Use

- Setting up or modifying light/dark theme colors
- Creating a new custom theme (e.g. "brand theme")
- Switching themes at runtime
- Defining or changing the app font
- Using text style extensions (heading, body, label helpers)
- Adding images, icons, or custom asset folders
- Configuring the asset base path
- When NOT to use: for page-level widget styling unrelated to the theme system, use standard Flutter `ThemeData` overrides directly

## Quick Reference

| Task | Approach |
|---|---|
| Create new theme | `metro make:theme bright_theme` |
| Edit theme colors | `lib/resources/themes/styles/light_theme_colors.dart` |
| Switch theme at runtime | `NyTheme.set(context, id: "dark_theme")` |
| Access a theme color | `ThemeColor.get(context).background` |
| Change app font | Edit `appThemeFont` in `lib/config/design.dart` |
| Display a local image | `LocalAsset.image("photo.png")` |
| Get asset path | `getImageAsset("photo.png")` or `getAsset("file.json")` |
| Register asset folder | Add path under `flutter: assets:` in `pubspec.yaml` |

## Theme Architecture

### File Layout

```
lib/
  config/
    design.dart              # App font, logo, loader defaults
  resources/
    themes/
      light_theme.dart       # ThemeData for light mode
      dark_theme.dart        # ThemeData for dark mode
      styles/
        color_styles.dart    # Abstract ColorStyles interface
        light_theme_colors.dart
        dark_theme_colors.dart
```

### Theme Registration

Themes are registered in the `appThemes` list (typically in `lib/config/theme.dart` or the theme provider). Each entry is a `BaseThemeConfig<ColorStyles>`:

```dart
final appThemes = [
  BaseThemeConfig<ColorStyles>(
    id: 'Light Theme',
    description: "Light Theme",
    theme: lightTheme,
    colors: LightThemeColors(),
  ),
  BaseThemeConfig<ColorStyles>(
    id: 'Dark Theme',
    description: "Dark Theme",
    theme: darkTheme,
    colors: DarkThemeColors(),
  ),
];
```

## Defining Colors

All theme color classes implement the `ColorStyles` abstract class, which defines the required color properties.

### ColorStyles Interface

The interface includes these color categories:

- **General**: `background`, `content`, `primaryAccent`
- **Surface**: `surfaceBackground`, `surfaceContent`
- **App Bar**: app bar background and content colors
- **Buttons**: primary and secondary button background/content
- **Bottom Tab Bar**: background, icon, and label colors
- **Toast Notifications**: toast background and content

### Implementing a Color Class

```dart
class LightThemeColors implements ColorStyles {
  // General
  @override
  Color get background => const Color(0xFFFFFFFF);

  @override
  Color get content => const Color(0xFF212121);

  @override
  Color get primaryAccent => const Color(0xFF0045a0);

  // Surface
  @override
  Color get surfaceBackground => Colors.white;

  @override
  Color get surfaceContent => Colors.black;

  // Buttons
  @override
  Color get buttonPrimaryBackground => const Color(0xFF0045a0);

  @override
  Color get buttonPrimaryContent => Colors.white;

  // ... remaining overrides
}
```

### Accessing Colors in Widgets

```dart
// Dynamic - follows current theme
ThemeColor.get(context).background
ThemeColor.get(context).primaryAccent

// In a Text widget
Text(
  "Hello",
  style: TextStyle(color: ThemeColor.get(context).content),
)

// Direct access to a specific theme's colors (not reactive)
ThemeConfig.light().colors.content
ThemeConfig.dark().colors.primaryAccent
```

## Creating a Custom Theme

Generate a new theme with Metro CLI:

```bash
metro make:theme bright_theme
```

This creates:
1. `lib/resources/themes/bright_theme.dart` - ThemeData configuration
2. `lib/resources/themes/styles/bright_theme_colors.dart` - Color definitions

The theme is automatically added to `appThemes`:

```dart
BaseThemeConfig<ColorStyles>(
  id: 'Bright Theme',
  description: "Bright Theme",
  theme: brightTheme,
  colors: BrightThemeColors(),
)
```

Edit the generated color file to define your palette, then customize the ThemeData in the theme file.

## Switching Themes at Runtime

Use `NyTheme.set()` to change themes dynamically:

```dart
import 'package:nylo_framework/theme/helper/ny_theme.dart';

// Switch to dark theme
TextButton(
  onPressed: () {
    NyTheme.set(context, id: "dark_theme");
    setState(() {});
  },
  child: Text("Dark Theme"),
)

// Switch to a custom theme
NyTheme.set(context, id: "bright_theme");
setState(() {});
```

The device's system light/dark mode preference is handled automatically. Manual switching overrides the system preference.

## Font Configuration

### Setting the App Font

Edit `lib/config/design.dart`:

```dart
// Google Fonts (included by default)
final TextStyle appThemeFont = GoogleFonts.lato();

// Switch to another Google Font
final TextStyle appThemeFont = GoogleFonts.montserrat();

// Use a custom font bundled in assets/fonts/
final TextStyle appThemeFont = TextStyle(fontFamily: "ZenTokyoZoo");
```

For custom fonts, register them in `pubspec.yaml`:

```yaml
flutter:
  fonts:
    - family: ZenTokyoZoo
      fonts:
        - asset: assets/fonts/ZenTokyoZoo-Regular.ttf
```

### Design Defaults

`lib/config/design.dart` also exposes:

- `logo` - App logo widget (customizable in `resources/widgets/logo_widget.dart`)
- `loader` - Loading indicator widget (customizable in `resources/widgets/loader_widget.dart`)

These are used as defaults throughout the framework's helper methods.

## Text Style Extensions

Nylo provides chainable extensions on `Text` widgets for consistent typography.

### Heading & Display Styles

```dart
Text("Title").displayLarge()
Text("Title").displayMedium()
Text("Title").displaySmall()
Text("Title").headingLarge()
Text("Title").headingMedium()
Text("Title").headingSmall()
Text("Title").titleLarge()
Text("Title").titleMedium()
Text("Title").titleSmall()
```

### Body & Label Styles

```dart
Text("Body").bodyLarge()
Text("Body").bodyMedium()
Text("Body").bodySmall()
Text("Label").labelLarge()
Text("Label").labelMedium()
Text("Label").labelSmall()
```

### Formatting & Alignment

```dart
Text("Hello World")
  .displayLarge()
  .fontWeightBold()
  .setColor(context, (color) => color.primaryAccent)

Text("Centered").bodyMedium().alignCenter()
Text("Right").bodySmall().alignRight()
Text("Left").bodyLarge().alignLeft()
Text("Max 2 lines").bodyMedium().setMaxLines(2)
Text("Light").bodyMedium().fontWeightLight()
```

## Asset Management

### Directory Structure

```
assets/
  images/        # PNG, JPG, SVG images
  fonts/         # Custom font files
  videos/        # Video files
  data/          # JSON, CSV, or other data files
```

### Registering Assets

All asset folders must be declared in `pubspec.yaml`:

```yaml
flutter:
  assets:
    - assets/images/
    - assets/videos/
    - assets/data/
```

### Displaying Images

```dart
// Nylo helper widget - recommended
LocalAsset.image("nylo_logo.png")

// Using Flutter's Image.asset with Nylo path helper
Image.asset(getImageAsset("nylo_logo.png"))
```

### Custom Asset Subdirectories

Create dedicated constructors on `LocalAsset` for organized access:

```dart
const LocalAsset.icons(String assetName, {...})
  : assetName = "images/icons/$assetName";

// Usage
LocalAsset.icons("settings_icon.png")
```

### Generic Asset Access

```dart
// Get path to any asset file (video, JSON, etc.)
getAsset("data/config.json")
getAsset("videos/intro.mp4")
```

### Asset Base Path

The `ASSET_PATH` variable in `.env` defines the base directory (default: `assets`). If changed (e.g. to `res`), regenerate the environment config:

```bash
metro make:env
```

## Common Mistakes

| Mistake | Fix |
|---|---|
| Colors not updating after theme switch | Call `setState(() {})` after `NyTheme.set()` |
| Using hardcoded colors instead of theme colors | Use `ThemeColor.get(context).propertyName` for theme-aware colors |
| Forgetting to register a new theme in `appThemes` | `metro make:theme` does this automatically; if adding manually, add a `BaseThemeConfig` entry |
| Custom font not displaying | Ensure the font is declared under `flutter: fonts:` in `pubspec.yaml` and the asset path matches exactly |
| Asset not found at runtime | Register the folder under `flutter: assets:` in `pubspec.yaml` |
| Text style extensions not available | Ensure `import 'package:nylo_framework/nylo_framework.dart';` is present |
| Using `ThemeConfig.light().colors` expecting reactivity | This accesses a fixed theme's colors; use `ThemeColor.get(context)` for the current theme |
