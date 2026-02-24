# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is a Claude Code plugin that provides the `mobile-design` skill. The skill guides Claude in creating distinctive, production-grade Flutter mobile interfaces for iOS and Android phones.

## Repository Structure

```
nylo/claude-code/
├── .claude-plugin/plugin.json  # Plugin metadata (name, version, author)
├── README.md                   # User-facing documentation
└── skills/mobile-design/
    └── SKILL.md                # The skill prompt (core content)
```

## How Claude Code Plugins Work

- `plugin.json` defines the plugin identity
- `SKILL.md` contains the prompt that gets injected when the skill is invoked
- Claude automatically uses this skill when users request Flutter mobile work

## Key Design Principles in SKILL.md

The skill enforces:
- **Platform awareness**: iOS uses Cupertino widgets, Android uses Material widgets
- **Touch targets**: iOS 44x44pt minimum, Android 48x48dp minimum
- **Thumb zone optimization**: Primary actions in bottom third of screen
- **Mandatory dark mode support**
- **Safe area handling** for notches, Dynamic Island, gesture bars

## Modifying the Skill

When editing `SKILL.md`:
- Preserve YAML frontmatter (name, description, license)
- Maintain the balance between platform conformity and creative expression
- Keep iOS/Android guidelines separate and accurate
- Update anti-patterns section when new generic AI aesthetics emerge
