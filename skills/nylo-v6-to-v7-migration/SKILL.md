---
name: nylo-v6-to-v7-migration
description: Migrate a Nylo Flutter project from v6 to v7 using the nylo-v6-to-v7-migrator agent
disable-model-invocation: true
---

# Nylo v6 to v7 Migration

Spawn the `nylo-v6-to-v7-migrator` agent to migrate the current Nylo Flutter project from v6 to v7. The agent handles the full migration — discovery, task list, implementation, and verification.

## When to Use

- User wants to migrate a Nylo Flutter v6 project to v7
- User is getting errors after updating Nylo dependencies to v7
- User asks about v6-to-v7 breaking changes

## Steps

1. **Spawn the migrator agent** using the `nylo-v6-to-v7-migrator` agent type. The agent will:
   - Scan the v7 boilerplate and changelog
   - Analyze the user's v6 project
   - Present a migration task list for approval
   - Implement changes one category at a time

2. **After the agent completes**, run a post-migration verification pass using the maps below.

---

## Complete v6 → v7 Structure Map

### Root Level Changes

| v6 | v7 | Action |
|----|-----|--------|
| `public/` | `assets/` | **Renamed** |
| `public/app_icon/icon.png` | `assets/app_icon/icon.png` | Moved with parent |
| `public/fonts/` | `assets/fonts/` | Moved with parent |
| `public/images/logo.png` | `assets/images/logo.png` | Moved with parent |
| `public/postman/` | *(removed)* | **Deleted entirely** |
| `.env` | *(generated, gitignored)* | **Removed from repo** — generated via `make:env` |
| `lang/en.json` | `lang/en.json` | Stays, but must add required `nylo` translation keys |
| — | `lang/es.json` | **New** in v7 (optional extra locale) |

### lib/app/commands/

| v6 | v7 | Action |
|----|-----|--------|
| `custom_commands.json` | `commands.json` | **Renamed** |
| `current_time.dart` | `current_time.dart` | No change |
| `motivational_quote.dart` | `motivational_quote.dart` | No change |
| — | `download_fonts.dart` | **New** in v7 |

### lib/app/events/

| v6 | v7 | Action |
|----|-----|--------|
| `logout_event.dart` | `logout_event.dart` | No change |
| — | `authenticated_event.dart` | **New** in v7 (for post-auth navigation) |

### lib/app/forms/

| v6 | v7 | Action |
|----|-----|--------|
| `login_form.dart` | *(removed)* | **Removed** from boilerplate |
| `register_form.dart` | `register_form.dart` | Stays, but `NyFormData` → `NyFormWidget`, `validate:` → `validator:` |
| `style/form_style.dart` | *(removed)* | **Entire `style/` subdirectory removed** |

### lib/bootstrap/

| v6 | v7 | Action |
|----|-----|--------|
| `app.dart` (AppBuild widget) | *(removed)* | **Replaced** by `lib/resources/widgets/main_widget.dart` |
| `boot.dart` | `boot.dart` | **Rewritten** — new boot sequence with `BootConfig` |
| `extensions.dart` | `extensions.dart` | No change |
| `helpers.dart` | `helpers.dart` | No change |
| — | `decoders.dart` | **Moved from** `lib/config/decoders.dart` |
| — | `events.dart` | **Moved from** `lib/config/events.dart` |
| — | `providers.dart` | **Moved from** `lib/config/providers.dart` |
| — | `theme.dart` | **Moved from** `lib/config/theme.dart` |

### lib/config/

| v6 (10 files) | v7 (5 files) | Action |
|----------------|--------------|--------|
| `decoders.dart` | *(moved to bootstrap/)* | **Moved** |
| `design.dart` | `design.dart` | Stays |
| `events.dart` | *(moved to bootstrap/)* | **Moved** |
| `form_casts.dart` | *(removed)* | **Removed** |
| `keys.dart` | `storage_keys.dart` | **Renamed** |
| `localization.dart` | `localization.dart` | Stays |
| `providers.dart` | *(moved to bootstrap/)* | **Moved** |
| `theme.dart` | *(moved to bootstrap/)* | **Moved** |
| `toast_notification_styles.dart` | `toast_notification.dart` | **Renamed** |
| `validation_rules.dart` | *(removed)* | **Removed** |
| — | `app.dart` | **New** in v7 (`AppConfig`) |

### lib/resources/themes/

| v6 | v7 | Action |
|----|-----|--------|
| `dark_theme.dart` | `dark/dark_theme.dart` | **Moved** into `dark/` subdirectory |
| `light_theme.dart` | `light/light_theme.dart` | **Moved** into `light/` subdirectory |
| `styles/color_styles.dart` | `color_styles.dart` | **Moved** to themes root, `styles/` dir removed |
| `styles/dark_theme_colors.dart` | `dark/dark_theme_colors.dart` | **Moved** from `styles/` to `dark/` |
| `styles/light_theme_colors.dart` | `light/light_theme_colors.dart` | **Moved** from `styles/` to `light/` |
| `text_theme/default_text_theme.dart` | `default_text_theme.dart` | **Moved** to themes root, `text_theme/` dir removed |
| — | `base_theme.dart` | **New** in v7 |

### lib/resources/widgets/

| v6 | v7 | Action |
|----|-----|--------|
| `loader_widget.dart` | `loader_widget.dart` | No change |
| `logo_widget.dart` | `logo_widget.dart` | No change |
| `safearea_widget.dart` | *(removed)* | **Removed** |
| `splash_screen.dart` | `splash_screen.dart` | No change |
| `theme_toggle_widget.dart` | `theme_toggle_widget.dart` | No change |
| `toast_notification_widget.dart` | `toast_notification_widget.dart` | Stays, but may need rename to avoid framework conflict |
| `buttons/` | `buttons/` | Same structure (abstract + 8 partials) |
| — | `main_widget.dart` | **New** — replaces `lib/bootstrap/app.dart` |
| — | `local_asset_widget.dart` | **New** in v7 |
| — | `bottom_sheet_modals/` | **New** directory in v7 |

### test/

| v6 | v7 | Action |
|----|-----|--------|
| `widget_test.dart` | *(renamed)* | **Renamed** to `example_test.dart` |
| — | `home_page_test.dart` | **New** in v7 |

---

## Post-Migration Verification

After the migrator agent completes, verify the following:

### 1. Directory & File Renames Applied

Check that these v6 paths **no longer exist** (if they do, migration is incomplete):

- `public/` (should be `assets/`)
- `lib/app/commands/custom_commands.json` (should be `commands.json`)
- `lib/bootstrap/app.dart` (should be `lib/resources/widgets/main_widget.dart`)
- `lib/config/decoders.dart` (should be in `lib/bootstrap/`)
- `lib/config/events.dart` (should be in `lib/bootstrap/`)
- `lib/config/providers.dart` (should be in `lib/bootstrap/`)
- `lib/config/theme.dart` (should be in `lib/bootstrap/`)
- `lib/config/keys.dart` (should be `lib/config/storage_keys.dart`)
- `lib/config/toast_notification_styles.dart` (should be `lib/config/toast_notification.dart`)
- `lib/config/form_casts.dart` (should be removed)
- `lib/config/validation_rules.dart` (should be removed)
- `lib/resources/themes/styles/` (should be flattened)
- `lib/resources/themes/text_theme/` (should be flattened)
- `lib/resources/widgets/safearea_widget.dart` (should be removed)
- `lib/app/forms/style/` (should be removed)

Also grep for any remaining `public/` asset references in code — they should all be `assets/`.

### 2. Remaining v6 Code Patterns

Search the codebase for patterns that should no longer exist:

| Search For | Should Be |
|------------|-----------|
| `validate:` in form widget code | `validator:` |
| `NyFormData` | `NyFormWidget` |
| `Boot.finished` | removed (restructured boot) |
| `Boot.nylo()` returning `Future<Nylo>` | returns `BootConfig` |
| `afterBoot(` in providers | `boot(` |
| `context: context` in `api<>()` calls | removed |
| `pretty_dio_logger` import | removed (use `useNetworkLogger`) |
| `style: "compact"` in forms | removed |
| `url_launcher` import | `openUrl()` helper |
| `path_provider` import | removed |
| `scaffold_ui` import | removed |
| `rename` dependency | removed |
| `flutter_local_notifications` import | removed |
| `AppButton` class (not `StatefulAppButton`) | `StatefulAppButton` |
| `Keys` class reference | `StorageKeysConfig` |
| `context.color.X` (flat) | `context.color.general.X` (structured groups) |
| `appFont` reference | `DesignConfig.appFont` |
| `getEnv('SHOW_SPLASH_SCREEN')` | `AppConfig.showSplashScreen` |

### 3. New v7 Files Exist

Confirm these were created:

- `lib/config/app.dart` (AppConfig)
- `lib/config/storage_keys.dart`
- `lib/config/toast_notification.dart`
- `lib/resources/widgets/main_widget.dart`
- `lib/resources/themes/base_theme.dart`
- `lib/resources/themes/dark/` directory with theme + colors
- `lib/resources/themes/light/` directory with theme + colors
- `.env`, `.env.prod`, `.env.dev` generated and gitignored
- `lib/bootstrap/env.g.dart` generated and gitignored

### 4. Unknown/Custom Directories

Scan `lib/` for directories NOT in the standard Nylo v7 structure:

**Standard v7 directories:**
- `lib/app/` → `commands/`, `controllers/`, `events/`, `forms/`, `models/`, `networking/` (with `dio/interceptors/`), `providers/`
- `lib/bootstrap/`
- `lib/config/`
- `lib/resources/` → `pages/`, `themes/` (with `dark/`, `light/`), `widgets/` (with `buttons/`, `bottom_sheet_modals/`)
- `lib/routes/` → `guards/`

Any directory outside this list (e.g., `services/`, `common/`, `helpers/`, `utils/`, `data/`, `repositories/`, `blocs/`, `cubits/`) should be **reported to the user** with a summary of its contents, asking whether it contains Nylo framework code that needs migration or custom application code to leave as-is.

### 5. Report

After the verification pass, present:
- Checklist of v6 paths confirmed removed/renamed
- Any remaining v6 patterns found (with file:line references)
- Unknown directories and their contents
- Overall migration status: **Complete** or **Needs Attention** (with specific items)
