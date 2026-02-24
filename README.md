# nylo-claude-code

A Claude Code plugin with **19 skills** and **3 agents** for Flutter and Nylo framework development.

## Installation

```bash
# Add the marketplace
/plugin marketplace add nylo-core/claude-code

# Install the plugin
/plugin install nylo-claude-code@nylo-core-claude-code
```

## Skills

### Nylo Framework Skills

| Skill | Description | Usage |
|-------|-------------|-------|
| **nylo-routing** | Route definitions, navigation, route guards, parameters, NavigationHub | `/nylo-claude-code:nylo-routing` |
| **nylo-networking** | API services, network()/networkResponse(), interceptors, caching, token refresh | `/nylo-claude-code:nylo-networking` |
| **nylo-auth** | Auth.authenticate(), token storage, Backpack sync, authenticated routes | `/nylo-claude-code:nylo-auth` |
| **nylo-storage** | NyStorage, Backpack (in-memory), cache strategies, persistent storage | `/nylo-claude-code:nylo-storage` |
| **nylo-forms** | NyFormData, form validators, input field widget, form submission | `/nylo-claude-code:nylo-forms` |
| **nylo-themes** | Theme system, light/dark mode, colors, fonts, asset management | `/nylo-claude-code:nylo-themes` |
| **nylo-testing** | Widget tests, unit tests, integration tests in Nylo projects | `/nylo-claude-code:nylo-testing` |
| **nylo-widgets** | Alerts, modals, buttons, CollectionView, FutureWidget, spacing, text translation | `/nylo-claude-code:nylo-widgets` |
| **nylo-state-management** | NyState, NyPage, providers, events, decoders registration | `/nylo-claude-code:nylo-state-management` |
| **nylo-app-setup** | Project setup, env config, directory structure, Metro CLI, logging, scheduler | `/nylo-claude-code:nylo-app-setup` |
| **nylo-localization** | i18n setup, locale files, .tr() translations, language switching, RTL support | `/nylo-claude-code:nylo-localization` |
| **nylo-notifications** | Local notifications, scheduling, platform config, permission handling | `/nylo-claude-code:nylo-notifications` |

### Flutter Platform Skills

| Skill | Description | Usage |
|-------|-------------|-------|
| **flutter-app-config** | Change app name, bundle ID, package name across iOS and Android | `/nylo-claude-code:flutter-app-config` |
| **flutter-permissions** | Add iOS and Android permissions with correct keys and descriptions | `/nylo-claude-code:flutter-permissions` |
| **flutter-app-icons** | Launcher icons and splash screens setup | `/nylo-claude-code:flutter-app-icons` |
| **flutter-versioning** | Version and build number management across platforms | `/nylo-claude-code:flutter-versioning` |
| **flutter-signing** | iOS code signing and Android keystore configuration | `/nylo-claude-code:flutter-signing` |
| **flutter-release** | Build release artifacts, Play Store and App Store submission | `/nylo-claude-code:flutter-release` |

### Design Skill

| Skill | Description | Usage |
|-------|-------------|-------|
| **mobile-design** | Production-grade Flutter mobile interfaces for iOS and Android | `/nylo-claude-code:mobile-design` |

## Agents

### nylo-developer

Scaffolds and wires up Nylo components using Metro CLI with proper registrations. Creates pages, models, controllers, API services, providers, events, and forms following Nylo conventions.

**Example prompts:**
- "Create a new settings page for notifications"
- "Add an API endpoint to fetch leaderboard data"
- "Build a feature for daily challenges with a page, model, and API service"

### nylo-code-reviewer

Reviews Flutter/Dart/Nylo code for quality, correctness, security, and performance. Checks Dart static analysis, Nylo conventions, widget composition, NyState patterns, const constructors, and build optimization.

**Example prompts:**
- "Review the code I just wrote"
- "Check the new widget for issues"
- "Review my API service changes"

### nylo-v6-to-v7-migrator

Migrates Nylo Flutter apps from v6.x to v7.x with a methodical, phased approach covering dependency updates, boot sequence changes, theme system migration, and more.

**Example prompts:**
- "Upgrade my Nylo app to v7"
- "What do I need to change for Nylo v7?"

## License

MIT
