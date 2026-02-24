# nylo-claude-code

A Claude Code plugin providing the **mobile-design** skill and **Nylo framework agents** for Flutter development.

## Installation

```bash
# Add the marketplace
/plugin marketplace add nylo-core/claude-code

# Install the plugin
/plugin install nylo-claude-code@nylo-core-claude-code
```

## What's Included

### Skill: mobile-design

Guides Claude in creating distinctive, production-grade Flutter mobile interfaces for iOS and Android phones. Enforces platform conventions (Cupertino/Material), touch targets, thumb zone optimization, dark mode support, and safe area handling.

**Usage:**
```bash
/nylo-claude-code:mobile-design
```

Then describe the screen you want to build. The skill handles layout, theming, accessibility, and platform-specific patterns.

### Agents

#### nylo-developer

Scaffolds and wires up Nylo components using Metro CLI with proper registrations. Creates pages, models, controllers, API services, providers, events, and forms following Nylo conventions.

**Example prompts:**
- "Create a new settings page for notifications"
- "Add an API endpoint to fetch leaderboard data"
- "Build a feature for daily challenges with a page, model, and API service"

#### nylo-code-reviewer

Reviews recently written or modified code for quality, correctness, security, and adherence to project conventions. Covers PHP, JavaScript, Dart, and configuration files.

**Example prompts:**
- "Review the code I just wrote"
- "Check the new controller for issues"

#### nylo-localisation

Manages localisation across Nylo Flutter apps: scanning for translatable strings, synchronising locale files, generating translations, and validating consistency.

**Example prompts:**
- "Scan for untranslated strings"
- "Add German translations to my app"
- "Check if all translations are ready for release"

#### nylo-v6-to-v7-migrator

Migrates Nylo Flutter apps from v6.x to v7.x with a methodical, phased approach covering dependency updates, boot sequence changes, theme system migration, and more.

**Example prompts:**
- "Upgrade my Nylo app to v7"
- "What do I need to change for Nylo v7?"

## License

MIT
