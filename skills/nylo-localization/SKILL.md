---
name: nylo-localization
description: Use when adding multi-language support, creating or editing translation files, using Text.tr or .tr() for translated strings, switching languages at runtime, configuring supported locales, setting up the LanguageSwitcher widget, handling RTL languages, or debugging missing translations in a Nylo v7 Flutter app.
---

# Nylo Localization

## Overview

Nylo v7 provides a JSON-based localization system configured in `lib/config/localization.dart`. Translation files live in the `lang/` directory, and strings are accessed throughout the app using the `.tr()` extension or the `TextTr` convenience widget.

## When to Use

- Adding multi-language (i18n) support to a Nylo project
- Creating or editing JSON translation files
- Displaying translated text with `Text.tr`, `TextTr`, or `.tr()`
- Passing dynamic arguments into translated strings
- Switching languages at runtime (programmatically or via LanguageSwitcher widget)
- Configuring supported locales, default locale, or fallback locale
- Handling right-to-left (RTL) languages like Arabic or Hebrew
- Debugging missing translation keys
- When NOT to use: For theming or styling text appearance unrelated to language, use nylo-themes instead

## Quick Reference

| Action | Code / Location |
|--------|----------------|
| Translation files directory | `lang/en.json`, `lang/es.json`, etc. |
| Localization config | `lib/config/localization.dart` |
| Set default locale | `DEFAULT_LOCALE=en` in `.env` |
| Use device locale | `LOCALE_TYPE=device` in `.env` |
| Debug missing keys | `DEBUG_TRANSLATIONS=true` in `.env` |
| Translate a string | `"welcome".tr()` |
| Translate with args | `"greeting".tr(arguments: {"name": "John"})` |
| Translate nested key | `"navigation.home".tr()` |
| TextTr widget | `TextTr("welcome")` |
| TextTr with args | `TextTr("greeting", arguments: {"name": "John"})` |
| Switch language (NyPage) | `changeLanguage('es')` |
| Switch language (anywhere) | `await NyLocalization.instance.setLanguage(context, language: 'es')` |
| Check if RTL | `LocalizationConfig.isRtl(languageCode)` |
| Verify key exists | `NyLocalization.instance.hasTranslation("key")` |
| Get all loaded keys | `NyLocalization.instance.getAllKeys()` |
| Rebuild env after changes | `metro make:env --force` |

## Project Setup

### 1. Create Language Files

Add JSON files to the `lang/` directory at the project root. Each file is named by its locale code:

```
lang/
  en.json
  es.json
  fr.json
  ar.json
```

### 2. Register Assets in pubspec.yaml

```yaml
flutter:
  assets:
    - lang/en.json
    - lang/es.json
    - lang/fr.json
    - lang/ar.json
```

### 3. Configure Environment Variables

In the `.env` file:

```
DEFAULT_LOCALE=en
LOCALE_TYPE=locale
DEBUG_TRANSLATIONS=true
```

| Variable | Values | Purpose |
|----------|--------|---------|
| `DEFAULT_LOCALE` | Any locale code (e.g., `en`, `es`) | Sets the app's default language |
| `LOCALE_TYPE` | `locale` or `device` | `locale` uses `DEFAULT_LOCALE`; `device` uses the system language |
| `DEBUG_TRANSLATIONS` | `true` or `false` | Logs warnings when translation keys are missing |

After changing `.env`, run:

```bash
metro make:env --force
```

### 4. Configure Supported Locales

In `lib/config/localization.dart`, define supported locales and the fallback:

```dart
class LocalizationConfig {
  static List<Locale> get supportedLocales => [
    Locale('en'),
    Locale('es'),
    Locale('fr'),
    Locale('ar'),
  ];

  static String get fallbackLanguageCode => 'en';
}
```

The `supportedLocales` list feeds into Flutter's `MaterialApp.supportedLocales`. The `fallbackLanguageCode` is used when a translation key is missing in the active locale, preventing raw keys from being displayed to users.

## Translation File Format

### Nested Keys with Dot Notation

Translation files are JSON with simple key-value pairs. Use nested objects for organization, accessed with dot notation:

```json
{
  "navigation": {
    "home": "Home",
    "profile": "Profile",
    "settings": "Settings"
  },
  "auth": {
    "login": "Log In",
    "register": "Sign Up",
    "forgot_password": "Forgot Password?"
  }
}
```

Access: `"navigation.home".tr()` returns `"Home"`.

### Parameterized Strings

Use `{{key}}` placeholders for dynamic values:

```json
{
  "greeting": "Hello, {{name}}!",
  "item_count": "You have {{total}} items in your cart"
}
```

### Styled Text Templates

Use `{{key:text}}` syntax for styled/tappable spans. The key stays stable across locales; the text is translated. Integrates with `StyledText.template()` for styling and tap handlers:

```json
{
  "learn": "Learn {{lang:Languages}}, {{read:Reading}} skills"
}
```

## Using Translations in Widgets

### The .tr() Extension

The most common way to display translations:

```dart
// Simple translation
Text("welcome".tr())

// With arguments
Text("greeting".tr(arguments: {"name": "Anthony"}))

// Nested keys
Text("navigation.home".tr())

// With arguments on nested keys
Text("item_count".tr(arguments: {"total": "5"}))
```

### The trans() Helper Function

An alternative to `.tr()`: `Text(trans("welcome"))` or `Text(trans("item_count", arguments: {"total": "5"}))`.

### The TextTr Widget

`TextTr` is a convenience wrapper around Flutter's `Text` widget that automatically calls `.tr()`:

```dart
// Instead of Text("welcome".tr()), use:
TextTr("welcome")

// With arguments
TextTr("greeting", arguments: {"name": "John"})

// With standard Text parameters
TextTr(
  "welcome",
  style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)
```

### TextTr Styled Constructors

TextTr provides named constructors that automatically apply theme-based text styles:

```dart
// Uses Theme.of(context).textTheme.displayLarge
TextTr.displayLarge("page_title")

// Uses Theme.of(context).textTheme.headlineLarge
TextTr.headlineLarge("section_header")

// Uses Theme.of(context).textTheme.bodyLarge
TextTr.bodyLarge("paragraph_text")

// Uses Theme.of(context).textTheme.labelLarge
TextTr.labelLarge("button_label")
```

## Language Switching

### Using the LanguageSwitcher Widget

The `LanguageSwitcher` widget auto-detects languages from your `lang/` directory and lets users switch between them. It persists the user's choice across sessions.

#### Dropdown in AppBar

```dart
@override
Widget view(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: TextTr("settings"),
      actions: [
        LanguageSwitcher(),
      ],
    ),
    body: ...
  );
}
```

#### Bottom Sheet Modal

```dart
// Show language picker as a bottom modal
LanguageSwitcher.showBottomModal(context);

// With custom height
LanguageSwitcher.showBottomModal(context, height: 300);
```

The bottom sheet displays all available languages with a checkmark on the currently selected one.

#### Customizing LanguageSwitcher Appearance

```dart
LanguageSwitcher(
  iconSize: 28,
  dropdownBgColor: Colors.white,
  elevation: 4,
  borderRadius: BorderRadius.circular(8),
  textStyle: TextStyle(fontSize: 16),
  dropdownBuilder: (Map<String, dynamic> language) {
    return Row(
      children: [
        Icon(Icons.language),
        SizedBox(width: 8),
        Text(language['name']),
      ],
    );
  },
  onLanguageChange: (Map<String, dynamic> language) {
    print('Switched to: ${language['name']}');
  },
)
```

### Programmatic Language Switching

#### From a NyPage or NyState Widget

```dart
// changeLanguage() is available directly in NyPage/NyState
changeLanguage('es');
```

#### From Anywhere (Including Controllers)

```dart
// Full language switch with UI rebuild
await NyLocalization.instance.setLanguage(context, language: 'es');

// From a controller, optionally prevent state restart
changeLanguage('fr', restartState: false);

// Silent locale switch (no UI rebuild)
NyLocalization.instance.setLocale(locale: Locale('fr'));
```

### Querying Language State

```dart
Map<String, dynamic>? lang = await LanguageSwitcher.currentLanguage();
Map<String, String>? data = LanguageSwitcher.getLanguageData("en"); // {"en": "English"}
List<Map<String, String>> all = await LanguageSwitcher.getLanguageList();
await LanguageSwitcher.storeLanguage(object: {"fr": "French"});
await LanguageSwitcher.clearLanguage();
```

## RTL Language Support

Nylo automatically detects RTL languages. The following languages are recognized as RTL: Arabic (ar), Hebrew (he), Persian (fa), Urdu (ur), Yiddish (yi), Pashto (ps), Kurdish (ku), Sindhi (sd), and Dhivehi (dv).

### Checking RTL Status

```dart
// Static check by language code
bool isRtl = LocalizationConfig.isRtl('ar'); // true

// Runtime check with context
bool isRtl = NyLocalization.instance.isDirectionRTL(context);
```

### NyLocaleHelper Utilities

```dart
NyLocaleHelper.isRtlLanguage('ar');             // true
NyLocaleHelper.getTextDirection('ar');           // TextDirection.rtl
NyLocaleHelper.getCurrentLocale(context);        // current Locale
NyLocaleHelper.getLanguageCode(context);         // e.g. "en"
NyLocaleHelper.matchesLocale(context, 'ar');     // bool
NyLocaleHelper.toLocale('en', 'US');             // Locale('en', 'US')
```

## NyLocalization API Reference

| Method / Property | Purpose |
|-------------------|---------|
| `translate(key, arguments)` | Core translation function |
| `hasTranslation(key)` | Check if a key exists in the current locale |
| `getAllKeys()` | List all loaded translation keys |
| `setLanguage(context, language: 'es')` | Switch language with UI rebuild |
| `setLocale(locale: Locale('fr'))` | Switch locale silently (no rebuild) |
| `languageCode` | Get current language code |
| `locale` | Get current Locale object |
| `delegates` | Get localization delegates |
| `isDirectionRTL(context)` | Check if current language is RTL |

## Complete Example: Settings Page with Language Switcher

```dart
class _SettingsPageState extends NyPage<SettingsPage> {
  @override
  Widget view(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: TextTr("settings.language"),
        actions: [
          LanguageSwitcher(
            onLanguageChange: (language) {
              showToastSuccess(
                description: "greeting".tr(arguments: {
                  "name": language['name'],
                }),
              );
            },
          ),
        ],
      ),
      body: SafeArea(
        child: Column(
          children: [
            TextTr.headlineLarge("app_title"),
            TextTr("cart.item_count", arguments: {"count": "3"}),
            TextTr.bodyLarge("cart.checkout"),
          ],
        ),
      ),
    );
  }
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Translation key shows as raw text instead of translated value | Ensure the JSON file is registered in `pubspec.yaml` under `flutter.assets` and the key exists in the locale file |
| Adding a new locale but it does not appear | Add the locale to `LocalizationConfig.supportedLocales` and create the corresponding `lang/{code}.json` file |
| Changing `.env` values but locale does not update | Run `metro make:env --force` after any `.env` changes to regenerate the environment config |
| Using `"key"` in `Text()` instead of `"key".tr()` | Always call `.tr()` on the string or use the `TextTr` widget; plain `Text("key")` displays the literal key |
| Arguments not interpolated (shows `{{name}}` literally) | Ensure the `arguments` map key matches the placeholder exactly: `"greeting".tr(arguments: {"name": "Val"})` for `{{name}}` |
| Nested key not resolving | Use dot notation in the `.tr()` call: `"navigation.home".tr()`, not `"navigation_home".tr()` |
| Language switch does not rebuild the UI | Use `NyLocalization.instance.setLanguage(context, language: 'es')` or `changeLanguage('es')` from NyPage; do not use `setLocale()` which switches silently |
| RTL layout not applied for Arabic | Verify `ar` is in `supportedLocales` and that the app wraps content with `Directionality` or relies on `MaterialApp`'s built-in direction support |
| LanguageSwitcher shows no languages | Ensure `lang/` JSON files are properly registered as assets in `pubspec.yaml` |
| Fallback locale not working | Confirm `fallbackLanguageCode` is set in `LocalizationConfig` and the corresponding JSON file exists with all keys |
