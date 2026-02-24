---
name: nylo-app-setup
description: Use when creating a new Nylo v7 project, configuring environment variables, understanding directory structure, using the Metro CLI, generating app icons, setting up logging, scheduling tasks, or building custom Metro commands.
---

# Nylo App Setup & Configuration

## Overview

Nylo v7 projects are created with the `nylo` CLI, configured via `.env` files encrypted into Dart, and scaffolded using the `metro` CLI tool. This skill covers project creation, environment configuration, directory structure, Metro CLI commands, app icons, logging, task scheduling, and custom commands.

## When to Use

- Creating a new Nylo project from scratch
- Configuring `.env` variables and generating encrypted env files
- Understanding where files belong in the Nylo directory structure
- Using Metro CLI to generate pages, widgets, models, services, etc.
- Setting up app icons for iOS and Android
- Configuring logging and debug output
- Scheduling one-time or daily tasks
- Building custom Metro CLI commands
- When NOT to use: for runtime state management, routing, or networking (see dedicated skills)

## Quick Reference

| Task | Command / Code |
|---|---|
| Install Nylo CLI | `dart pub global activate nylo_installer` |
| Create project | `nylo new my_app` |
| Initialize metro | `nylo init` |
| Generate encryption key | `metro make:key` |
| Generate env config | `metro make:env` |
| Generate for production | `metro make:env --file=".env.production" --force` |
| Create page | `metro make:page product_page` |
| Create model | `metro make:model product` |
| Create API service | `metro make:api_service user_api_service` |
| Create widget | `metro make:stateful_widget my_widget` |
| Generate app icons | `dart run flutter_launcher_icons` |
| Run app | `flutter run` |

---

## Project Creation

### Prerequisites

```bash
dart pub global activate nylo_installer
```

### Create and Initialize

```bash
nylo new my_app
cd my_app
nylo init
```

`nylo new` clones the Nylo template, configures the app name, and installs dependencies. `nylo init` configures the `metro` command for your project.

### Run the App

```bash
flutter run
```

---

## Environment Configuration

### .env File

The root `.env` file contains configuration variables:

```env
APP_KEY=your-32-character-key-here
APP_NAME=MyApp
APP_ENV=developing
APP_DEBUG=true
APP_URL=https://myapp.com
API_BASE_URL=https://api.myapp.com
ASSET_PATH=assets
DEFAULT_LOCALE=en
```

### Key Commands

```bash
# Generate a 32-character APP_KEY
metro make:key

# Generate encrypted env file (lib/bootstrap/env.g.dart)
metro make:env

# Regenerate after changes
metro make:env --force

# Generate for a specific env file
metro make:env --file=".env.production" --force

# Build-time key injection (for CI/CD)
metro make:env --dart-define
flutter build ios --dart-define=APP_KEY=your-secret-key
```

### Environment Flavors

Create environment-specific files:
- `.env` - development (default)
- `.env.staging`
- `.env.production`

### Accessing Config Values

```dart
// In lib/config/app.dart
final class AppConfig {
  static final String appName = getEnv('APP_NAME', defaultValue: 'Nylo');
  static final bool appDebug = getEnv('APP_DEBUG', defaultValue: false);
  static final String apiBaseUrl = getEnv('API_BASE_URL');
}

// Usage anywhere
String name = AppConfig.appName;
```

### Custom Config Classes

```bash
metro make:config RevenueCat
```

```dart
final class RevenueCatConfig {
  static final String apiKey = getEnv('REVENUECAT_API_KEY');
  static final String entitlementId = getEnv('REVENUECAT_ENTITLEMENT_ID');
}
```

### Type Handling

- Quoted strings become `String`
- `true`/`false` become `bool`
- `null` remains null
- Empty quotes `""` become empty string

---

## Directory Structure

```
my_app/
  android/                    # Android platform code
  ios/                        # iOS platform code
  assets/
    app_icon/                 # Source icon (1024x1024 PNG)
    fonts/                    # Custom font files
    images/                   # Image assets
  lang/                       # Translation files (en.json, es.json, etc.)
  lib/
    app/
      commands/               # Custom Metro CLI commands
      controllers/            # Page controllers (business logic)
      events/                 # NyEvent classes
      forms/                  # NyFormData validation classes
      models/                 # Data model classes
      networking/             # API services and interceptors
        dio/interceptors/     # Dio HTTP interceptors
      providers/              # NyProvider classes (boot-time init)
      services/               # Utility services
    bootstrap/
      boot.dart               # App startup sequence
      decoders.dart           # Model and API decoder registration
      env.g.dart              # Generated encrypted env config
      events.dart             # Event registration
      extensions.dart         # Custom extensions
      helpers.dart            # Helper functions
      providers.dart          # Provider registration
      theme.dart              # Theme registration
    config/
      app.dart                # Core app settings
      design.dart             # Font, logo, loader config
      localization.dart       # Language settings
      storage_keys.dart       # Local storage key definitions
      toast_notification.dart # Toast notification config
    resources/
      pages/                  # Page widgets (screens)
      themes/                 # Theme definitions
        styles/               # Color style classes
      widgets/                # Reusable widgets
        buttons/              # Button definitions and partials
    routes/
      router.dart             # Route definitions
      guards/                 # Route guard classes
    main.dart                 # Entry point
  test/                       # Tests
  .env                        # Environment variables
  pubspec.yaml                # Flutter dependencies
```

---

## Metro CLI Commands

### Page Generation

```bash
metro make:page product_page
metro make:page product_page --controller    # with controller
metro make:page product_page --auth          # auth page
metro make:page product_page --initial       # initial/launch page
metro make:page product_page --force         # overwrite existing
```

### Widget Generation

```bash
metro make:stateless_widget product_card
metro make:stateful_widget product_card
metro make:state_managed_widget cart_badge
metro make:bottom_sheet_modal payment_options
metro make:button checkout_button
```

### Model & Service Generation

```bash
metro make:model product
metro make:model product --json         # JSON-to-model generation
metro make:api_service user_api_service
metro make:api_service user --model="User"  # with paired model
```

### Other Generators

```bash
metro make:controller profile_controller
metro make:provider firebase_provider
metro make:event login_event
metro make:form car_advert_form
metro make:theme bright_theme
metro make:route_guard premium_content
metro make:config shopping_settings
metro make:interceptor auth_interceptor
metro make:command my_command
metro make:command deploy --category="project"
metro make:navigation_hub dashboard
metro make:journey_widget welcome,dob,photos --parent="onboarding"
```

### Environment Commands

```bash
metro make:key                           # Generate APP_KEY
metro make:key --file=.env.production    # For specific env file
metro make:env                           # Generate encrypted config
metro make:env --force                   # Regenerate
metro make:env --file=".env.staging"     # Specific env file
metro make:env --dart-define             # Build-time injection mode
```

---

## App Icons

### Setup

1. Create a 1024x1024 PNG icon (no transparency)
2. Place in `assets/app_icon/`
3. Configure `pubspec.yaml`:

```yaml
flutter_launcher_icons:
  android: true
  ios: true
  image_path: "assets/app_icon/icon.png"
```

4. Generate:

```bash
dart run flutter_launcher_icons
```

Icons are generated to:
- iOS: `ios/Runner/Assets.xcassets/AppIcon.appiconset/`
- Android: `android/app/src/main/res/mipmap-*/`

### Badge Count Management

```dart
await setBadgeNumber(5);    // Set badge count
await clearBadgeNumber();   // Clear badge
```

---

## Logging

### Log Methods

```dart
printInfo("General information");     // Blue
printDebug("Debug details");          // Cyan
printError("Error message");          // Red
```

All logs are suppressed when `APP_DEBUG=false` in `.env`.

### Advanced Logging

```dart
// Force output regardless of debug mode
printInfo("Always visible", alwaysPrint: true);

// Print next log once then suppress
showNextLog();
printInfo("This prints once");

// JSON output
printJson({"key": "value"}, prettyPrint: true);

// Stack traces with errors
try {
  throw Exception("Something failed");
} catch (e, stackTrace) {
  printError(e.toString(), stackTrace: stackTrace);
}
```

### Debug Helpers

```dart
// Print any value to console
dump(myObject);

// Print and immediately exit
dd(myObject);
```

### Log Listeners (Crash Reporting Integration)

```dart
NyLogger.onLog = (entry) {
  // entry.message, entry.type, entry.dateTime, entry.stackTrace
  // Forward to Sentry, Firebase Crashlytics, etc.
};
```

### Configuration

```dart
// Show timestamps in logs
Nylo.instance.showDateTimeInLogs(true);

// Disable colored output
NyLogger.useColors = false;
```

---

## Scheduler

Run tasks once, daily, or after a specific date:

```dart
// Run once (ever)
Nylo.scheduleOnce('onboarding_info', () {
  print("This runs only the first time");
});

// Run once after a specific date
Nylo.scheduleOnceAfterDate('app_review_prompt', () {
  print("Show review prompt");
}, date: DateTime(2025, 04, 10));

// Run once per day
Nylo.scheduleOnceDaily('daily_reward', () {
  print("Grant daily coins");
});
```

Each task requires a unique string identifier. The framework tracks execution state internally.

---

## App Usage Monitoring

Enable in `AppProvider`:

```dart
nylo.configure(monitorAppUsage: true);
```

Available metrics:

```dart
int? launches = await Nylo.appLaunchCount();
DateTime? firstLaunch = await Nylo.appFirstLaunchDate();
int days = await Nylo.appTotalDaysSinceFirstLaunch();
```

---

## Custom Metro Commands

### Create a Command

```bash
metro make:command deploy --category="project"
```

### Command Structure

```dart
import 'package:nylo_framework/metro/ny_cli.dart';

void main(arguments) => _DeployCommand(arguments).run();

class _DeployCommand extends NyCustomCommand {
  _DeployCommand(super.arguments);

  @override
  CommandBuilder builder(CommandBuilder command) {
    command.addOption('environment', abbr: 'e',
      defaultValue: 'development',
      allowed: ['development', 'staging', 'production']);
    command.addFlag('verbose', abbr: 'v', defaultValue: false);
    return command;
  }

  @override
  Future<void> handle(CommandResult result) async {
    final env = result.getString('environment');
    final verbose = result.getBool('verbose');

    info('Deploying to $env');

    await runTasksWithSpinner([
      CommandTask('Clean', () => flutterClean()),
      CommandTask('Build', () => flutterBuild('web', args: ['--release'])),
    ]);

    success('Deployment complete');
  }
}
```

### Run Custom Commands

```bash
metro project:deploy --environment=production
metro project:deploy -e production --verbose
metro project:deploy --help
```

### Key Helper Methods in Commands

| Category | Methods |
|---|---|
| Output | `info()`, `error()`, `success()`, `warning()`, `alert()`, `line()` |
| Input | `prompt()`, `confirm()`, `select()`, `multiSelect()`, `promptSecret()` |
| Process | `runProcess()`, `flutterPubGet()`, `flutterClean()`, `flutterBuild()`, `flutterTest()` |
| Files | `readFile()`, `writeFile()`, `fileExists()`, `directoryExists()`, `scaffold()` |
| JSON/YAML | `readJson()`, `writeJson()`, `readYaml()`, `appendToJsonArray()` |
| Dart files | `addImport()`, `insertBeforeClosingBrace()`, `fileContains()` |
| Packages | `addPackage()`, `addPackages()` |
| Naming | `snakeCase()`, `camelCase()`, `pascalCase()`, `titleCase()`, `kebabCase()` |
| Paths | `modelsPath`, `pagesPath`, `widgetsPath`, `controllersPath`, `projectPath()` |
| Progress | `withSpinner()`, `createSpinner()`, `progressBar()`, `withProgress()` |
| Tasks | `runTasks()`, `runTasksWithSpinner()` |
| Validation | `isValidDartIdentifier()`, `cleanClassName()`, `cleanFileName()` |

**Important**: Custom commands run outside Flutter runtime. Import `ny_cli.dart`, not `nylo_framework.dart`.

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| `metro` command not found | Run `nylo init` in the project root first |
| `.env` changes not reflected | Run `metro make:env --force` to regenerate `env.g.dart` |
| Missing `APP_KEY` | Run `metro make:key` to generate a 32-character key |
| App icons not updating | Ensure icon is 1024x1024 PNG with no transparency, then run `dart run flutter_launcher_icons` |
| Custom command not appearing in `metro` | Check `lib/app/commands/commands.json` has the entry with correct `name`, `category`, and `script` |
| Logs not printing | Verify `APP_DEBUG=true` in `.env` or use `alwaysPrint: true` |
| Scheduler task running multiple times | Ensure each task has a unique string identifier |
| Importing `nylo_framework.dart` in commands | Custom commands run outside Flutter; use `import 'package:nylo_framework/metro/ny_cli.dart'` instead |
| Env variables returning wrong types | Check quoting: `"true"` is a String, `true` (no quotes) is a bool |
| Production secrets in `.env` committed to git | Add `.env` to `.gitignore`; use `metro make:env --dart-define` for CI/CD |
