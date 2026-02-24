---
name: mobile-design
description: Create distinctive, production-grade mobile interfaces for iOS and Android phones using Flutter. Use this skill when the user asks to build mobile apps, phone interfaces, Flutter screens, or mentions iOS/Android design. Generates platform-aware, creative code that avoids generic aesthetics.
license: MIT
---

This skill guides creation of distinctive, production-grade Flutter mobile interfaces for iOS and Android phones. Implement real working code with exceptional attention to platform conventions and creative choices. Target phones ONLY - no tablets.

The user provides mobile requirements: a screen, feature, or complete app to build. They may include context about platform preference, purpose, audience, or technical constraints.

## Design Philosophy

Before coding, understand the context and commit FULLY to a platform-aware aesthetic direction. Timid design produces forgettable apps.

- **Existing project first**: Search the codebase for existing themes, colors, typography, and design patterns. New screens should feel like they belong to the same app. Consistency with established project aesthetics takes priority over introducing new styles.
- **Platform**: iOS, Android, or adaptive (both). Each has distinct conventions that users expect.
- **Purpose**: What problem does this interface solve? Who uses it one-handed on the go? What emotional state are they in?
- **Tone**: If no existing aesthetic exists, commit boldly to a direction:
  - Refined luxury (restrained palette, generous whitespace, elegant typography)
  - Playful casual (rounded shapes, bright colors, bouncy motion)
  - Bold graphic (high contrast, strong typography, decisive color blocking)
  - Organic soft (natural colors, fluid shapes, gentle gradients)
  - Utilitarian functional (dense information, minimal decoration, maximum efficiency)
  - Editorial magazine (dramatic type scale, asymmetric layouts, striking imagery)
  - Dark moody (deep backgrounds, accent lighting, atmospheric depth)
  - Vibrant energetic (saturated colors, dynamic composition, lively motion)
- **The Unforgettable Element**: What single thing will make users remember this screen? Every great interface has a signature moment - a striking header, a delightful transition, an unexpected color choice, a perfectly weighted animation. Identify yours before coding.

**CRITICAL**: Respect the existing project's design system first, then platform conventions, while expressing creativity within them. Generic Flutter defaults are unacceptable. Platform-inappropriate patterns are equally unacceptable. Ignoring established project patterns creates visual inconsistency.

**Execution Principle**: Complexity should match the aesthetic vision. A maximalist design warrants elaborate gradients, layered elements, and rich animation. A minimalist design requires restraint - every element must earn its place through precision and purpose. Don't apply maximalist techniques to minimal concepts or vice versa.

## Platform Guidelines

### iOS (Human Interface Guidelines + Liquid Glass 2025)

Apple's design language emphasizes:
- **Clarity**: Users understand navigation and interaction immediately
- **Deference**: UI doesn't compete with content; it enhances it
- **Depth**: Layering, translucency, and motion create hierarchy
- **Consistency**: Familiar patterns build user confidence

Implementation specifics:
- Use `CupertinoApp`, `CupertinoNavigationBar`, `CupertinoTabScaffold`
- Touch targets: 44x44pt minimum
- Navigation: Bottom tab bar, swipe-from-left-edge for back (never override)
- Typography: SF Pro via system font, support Dynamic Type
- Colors: Use `CupertinoColors` that adapt to light/dark modes
- Translucent materials: Blur effects, frosted glass surfaces
- Corners: Continuous curves (squircles), not circular radius

### Android (Material Design 3 / Material You)

Google's design language emphasizes:
- **Material is the metaphor**: Tactile, intuitive surfaces
- **Bold, graphic, intentional**: Strong visual hierarchy
- **Motion provides meaning**: Animations guide understanding

Implementation specifics:
- Use `MaterialApp`, `Scaffold`, `NavigationBar`
- Touch targets: 48x48dp minimum
- Spacing: 8dp grid (8, 16, 24, 32, 40...)
- Elevation: Layered surfaces (1dp, 3dp, 6dp, 8dp, 12dp)
- Typography: Roboto via Material type scale (Display, Headline, Title, Body, Label)
- Colors: `ColorScheme.fromSeed()` for dynamic theming
- Use `Theme.of(context)` for consistent styling

## Typography

**iOS**: SF Pro is the system font. Access via `CupertinoTextTheme`. Support Dynamic Type - layouts must accommodate 200% text scaling. Hierarchy: Large Title > Title > Headline > Body > Caption.

**Android**: Roboto is the default. Use Material type scale consistently. Large Display for hero text, down to Label for metadata.

**Custom fonts**: Consider distinctive choices like Plus Jakarta Sans, Outfit, or Satoshi for brand differentiation. Ensure proper licensing. Test readability at smallest sizes (11pt/11sp minimum body text).

**Typography pairing**: Combine a distinctive display font for headlines with a refined, readable body font. The contrast creates visual hierarchy and brand personality. Example pairings:
- Plus Jakarta Sans (display) + Inter (body) - modern tech
- Playfair Display (headlines) + Source Sans Pro (body) - editorial elegance
- Space Grotesk (headlines) + DM Sans (body) - contemporary geometric

**Anti-pattern**: Using Inter, Roboto, or system fonts without intention. Every typography choice should be deliberate.

## Color and Theming

**FIRST**: Before choosing colors, analyze the existing codebase for established patterns:

1. **Search for existing theme definitions**:
   - Look for `ThemeData`, `ColorScheme`, `CupertinoThemeData` in `lib/`
   - Check for color constants in files like `colors.dart`, `theme.dart`, `app_theme.dart`, `constants.dart`
   - Search for `Color(0x` or `Colors.` to find hardcoded color usage

2. **Identify the project's color system**:
   - Primary/secondary brand colors
   - Semantic colors (error, success, warning)
   - Surface and background colors
   - Text color hierarchy

3. **Follow established patterns**:
   - Use existing color variables/constants rather than introducing new hex values
   - Match the naming conventions already in use
   - Extend the existing theme rather than overriding it
   - If the project uses `Theme.of(context).colorScheme`, continue that pattern

4. **Only introduce new colors when**:
   - No existing theme or color system exists
   - The new screen requires semantic colors not yet defined
   - Explicitly requested by the user

**Mandatory**: Support both light AND dark modes. Test both thoroughly.

**iOS approach**:
- Use semantic `CupertinoColors` (systemBackground, label, secondaryLabel)
- Colors adapt automatically to appearance mode
- Translucent surfaces with blur show underlying content

**Android approach**:
- Generate palettes with `ColorScheme.fromSeed(seedColor)`
- Material You extracts colors from user's wallpaper
- Tonal surfaces create depth (surface, surfaceVariant, surfaceContainerHighest)

**Contrast requirements**:
- Normal text: 4.5:1 minimum contrast ratio
- Large text (18pt+): 3:1 minimum
- Interactive elements: 3:1 minimum against background

**Palette strategy**: Dominant color with sharp accents outperforms evenly-distributed palettes. Limit to 3-5 core colors plus semantic variations (error, warning, success).

**Anti-pattern**: Purple-to-blue gradients on white backgrounds - overused AI aesthetic. Every color scheme should feel intentional for the specific app context.

## Touch and Gestures

**Touch targets**: iOS 44x44pt, Android 48x48dp. No exceptions. Small targets frustrate users.

**Thumb zone optimization** (critical for phones):
- **Easy zone** (bottom third): Primary actions, main navigation, FABs
- **Stretch zone** (middle): Secondary actions, content
- **Hard zone** (top corners): Overflow menus, rarely-used actions

**Platform gestures - NEVER override**:
- iOS: Swipe from left edge = back navigation
- Android: System back gesture/button
- Both: Swipe-to-dismiss for modals, pull-to-refresh for lists

**Spacing**: Use 8dp/8pt base grid. Consistent spacing creates visual rhythm. Common values: 4, 8, 12, 16, 24, 32, 48.

## Spatial Composition

Mobile screens are constrained canvases. Use composition deliberately to create visual interest within the narrow viewport.

**Breaking the vertical monotony**:
- Overlapping elements create depth (images bleeding behind cards, text overlaying imagery)
- Asymmetric layouts draw the eye (left-aligned image with right-aligned text block)
- Diagonal flow guides scrolling (elements that lead the eye downward)
- Generous whitespace around key elements creates focal points
- Edge-to-edge elements interspersed with padded content create rhythm

**Card and container composition**:
- Vary card sizes intentionally - hero cards, standard cards, compact cards
- Stagger grid items rather than rigid columns
- Use negative space as a design element, not empty filler
- Consider overlapping cards with offset shadows for depth

**Hero moments**: Every scrolling experience needs visual peaks - a striking image, a bold stat, an unexpected layout shift. Flat, uniform scrolling is forgettable.

**Constraints**: Mobile has real limits. Honor safe areas. Keep primary actions thumb-reachable. But within those constraints, composition can be expressive.

## Visual Atmosphere

Build depth and mood through layering and texture:

**Gradients**: Use subtle radial gradients for focal glow effects, linear gradients for directional depth. Multi-stop gradients with slight color shifts feel more natural than two-color transitions.

**Layered transparencies**: Semi-transparent surfaces over blurred backgrounds create iOS-style depth. Overlapping translucent elements at varying opacities build visual hierarchy.

**Texture and pattern**: Subtle noise overlays add tactile quality. Geometric patterns in backgrounds at low opacity create visual interest without distraction. Mesh gradients feel contemporary.

**Light and shadow**: Beyond standard elevation, consider:
- Accent shadows with color tint (not just black/gray)
- Inner shadows for recessed effects
- Ambient glow around primary actions
- Soft light gradients suggesting off-screen light sources

**Context-specific atmosphere**: A meditation app deserves soft, breathing gradients. A fitness app might use energetic diagonal stripes. A finance app needs restrained, trustworthy surfaces. Match atmosphere to purpose.

## Motion and Animation

**Principle**: Motion provides meaning. Every animation should communicate state change, hierarchy, or spatial relationship.

**Prioritize high-impact moments**: Invest animation effort where it matters most:
1. **Screen entrances**: Orchestrated reveals with staggered elements create polished first impressions
2. **Key transitions**: Hero animations between list items and detail views
3. **Success moments**: Celebratory feedback for completed actions (order placed, goal reached)
4. **Loading states**: Skeleton screens that shimmer thoughtfully, not randomly

Don't scatter micro-interactions everywhere. A few well-crafted signature animations outperform dozens of generic ones.

**iOS motion**:
- Spring-based physics (`Curves.easeOutBack`, spring simulations)
- Hero transitions for navigation continuity
- Subtle parallax and depth effects
- Use `CupertinoPageRoute` for platform-appropriate transitions

**Android motion**:
- Container transforms (expanding cards to full screens)
- Shared axis transitions (forward/backward navigation)
- Fade through for unrelated content
- Emphasis on intentional, purposeful movement

**Timing**: 200-500ms for most transitions. Under 200ms feels abrupt; over 500ms feels sluggish. Use appropriate curves (`Curves.easeOutCubic`, `Curves.fastOutSlowIn`).

**Micro-interactions**:
- Button press: Scale down slightly, color shift
- Loading: Skeleton screens over spinners (better perceived performance)
- Success/error: Brief, clear feedback animations
- Pull-to-refresh: Custom indicators matching app aesthetic

**Anti-pattern**: Instant transitions without animation, excessive bouncing, animations over 700ms, motion that decorates rather than communicates.

## Component Patterns

**Navigation**:
- iOS: `CupertinoTabBar` at bottom, `CupertinoNavigationBar` at top
- Android: `NavigationBar` at bottom, optional FAB, `AppBar` at top

**Lists**:
- iOS: `CupertinoListSection` with inset grouped style
- Android: `ListView` with `Card` or `ListTile`
- Both: Lazy loading for performance, appropriate scroll physics

**Cards and surfaces**:
- iOS: Continuous corner radius, subtle shadows, translucent backgrounds
- Android: Elevation-based shadows (1dp subtle, 8dp prominent), 12-16dp corner radius

**Modals and sheets**:
- iOS: `showCupertinoModalPopup`, drag-to-dismiss handle
- Android: `showModalBottomSheet`, edge-to-edge design

**Forms**:
- iOS: `CupertinoTextField` with clear buttons
- Android: `TextField` with `OutlineInputBorder`
- Both: Inline validation, clear error states, appropriate keyboard types

**Empty states**: Custom illustrations matching app aesthetic, clear call-to-action. Never just "No data."

## Accessibility

**Screen readers**: Add `Semantics` widgets with meaningful labels. Ensure logical focus order. Announce dynamic content changes.

**Dynamic Type / Font Scaling**: Test at 200% text scale. Use `MediaQuery.textScaleFactor`. Layouts must accommodate larger text without breaking.

**Color**: Never rely on color alone for meaning. Pair with icons, text, or patterns. Test with color blindness simulators.

**Haptic feedback**: Use `HapticFeedback.lightImpact()` for selections, `mediumImpact()` for confirmations. Provides non-visual feedback.

**Reduce motion**: Check `MediaQuery.disableAnimations`. Provide reduced-motion alternatives for users with vestibular disorders.

## Phone-Specific Considerations

**Safe areas**: Always use `SafeArea` widget or `MediaQuery.padding`. Account for:
- Notches (iPhone)
- Dynamic Island (iPhone 14 Pro+)
- Punch-hole cameras (Android)
- Home indicator / gesture bar (bottom)
- Status bar (top)

**Aspect ratios**: Modern phones range from 19.5:9 to 21:9. Design for tall, narrow screens. Avoid fixed heights; use flexible layouts.

**Screen sizes to test**:
- iOS: iPhone SE (375pt), iPhone 15 (393pt), iPhone 15 Pro Max (430pt)
- Android: Small (360dp), Medium (392dp), Large (412dp)

**Orientation**: Design portrait-first. Lock to portrait if landscape adds no value. If supporting both, ensure layouts adapt properly.

## Anti-Patterns

NEVER create interfaces with:
- New color values when the project already defines a color system
- Styles that clash with the existing app aesthetic
- Default Flutter widget styling unchanged
- Generic aesthetics that could be any app
- iOS patterns on Android or vice versa
- Touch targets under 44pt/48dp
- Buttons in top corners requiring thumb stretch
- System gesture overrides
- Missing dark mode support
- Spinners instead of skeleton screens
- Fixed layouts that break on different screen sizes
- Ignoring safe areas (content under notch/home indicator)

**Generic AI aesthetics to avoid**:
- Purple-to-blue gradients as default (overused to the point of cliche)
- Rounded rectangles with drop shadows as the only visual treatment
- Generic illustrations with floating people
- Uniform card grids with identical spacing everywhere
- Sans-serif headers in medium weight with no personality
- Timid color choices (too many grays, afraid of saturation)
- Motion that decorates rather than communicates

**Composition failures**:
- Flat vertical stacking without rhythm or focal points
- Every element with identical padding and margins
- No visual hierarchy - everything competing equally for attention
- Missing "hero moments" - no memorable peaks in the scroll journey

ALWAYS aim for:
- Consistency with existing project theme, colors, and patterns
- Platform-appropriate but distinctive design
- Intentional typography, color, and spacing choices
- Proper touch targets and thumb-zone placement
- Meaningful animations that communicate state
- Accessible, inclusive interfaces
- Designs that feel crafted for the specific app context
- At least one signature visual moment that makes the screen memorable
- Atmosphere and depth through layering, not just flat surfaces

Remember: Claude is capable of extraordinary creative work within platform conventions. Commit fully to a distinctive vision that respects iOS and Android patterns while creating memorable, production-grade mobile experiences. Timid design is forgettable design.
