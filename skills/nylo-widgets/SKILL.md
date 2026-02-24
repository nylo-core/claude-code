---
name: nylo-widgets
description: Use when building UI with Nylo v7 built-in widgets such as alerts/toasts, modals/bottom sheets, buttons, CollectionView (lists/grids), FutureWidget, FadeOverlay, Pullable, StyledText, Spacing, TextTr (translation), LanguageSwitcher, or Connective (connectivity-aware widgets).
---

# Nylo Built-in Widgets

## Overview

Nylo v7 provides a rich widget library covering common UI patterns: toast notifications, bottom sheet modals, styled buttons with animation, list/grid rendering with pagination, async data display, pull-to-refresh, rich text, spacing, localization, and connectivity-aware layouts. These widgets integrate with Nylo's theme and state systems.

## When to Use

- Showing toast notifications or alert dialogs
- Displaying bottom sheet modals
- Creating buttons with loading states, animations, or form submission
- Rendering lists or grids with pagination, sorting, and filtering
- Loading async data with loading/error states
- Adding pull-to-refresh to scrollable content
- Building rich text with mixed styles and tap handlers
- Adding consistent spacing between elements
- Translating text or switching languages
- Showing different UI based on network connectivity
- When NOT to use: for custom widgets unrelated to Nylo's built-in library, use standard Flutter widgets

## Quick Reference

| Widget | Purpose |
|---|---|
| `showToastSuccess()` | Success toast notification |
| `showToastDanger()` | Error toast notification |
| `BottomSheetModal` | Bottom sheet modal dialog |
| `Button.primary()` | Primary styled button |
| `CollectionView<T>()` | Type-safe list rendering |
| `CollectionView<T>.pullable()` | List with pagination + pull-to-refresh |
| `CollectionView<T>.grid()` | Grid layout |
| `FutureWidget<T>()` | Async data loader with loading state |
| `FadeOverlay()` | Gradient fade overlay on images |
| `Pullable()` | Pull-to-refresh wrapper |
| `StyledText()` | Rich text with mixed styles |
| `StyledText.template()` | Template-based rich text |
| `Spacing.md` | 16px vertical space |
| `TextTr("key")` | Translated text widget |
| `LanguageSwitcher()` | Language selection dropdown |
| `Connective()` | Connectivity-aware widget |

---

## Alerts & Toast Notifications

### Convenience Methods (in NyState/NyController)

```dart
showToastSuccess(description: "Saved successfully");
showToastWarning(description: "Check your input");
showToastInfo(description: "New update available");
showToastDanger(description: "Something went wrong");
showToastOops(description: "Oops!");    // danger style
showToastSorry(description: "Sorry!");  // danger style
```

All accept optional `title` and `description` parameters.

### Global Function

```dart
showToastNotification(
  context,
  id: 'success',           // success | warning | info | danger | custom ID
  title: 'Saved',
  description: 'Your changes have been saved.',
  duration: Duration(seconds: 3),
  position: ToastNotificationPosition.top,  // top | bottom | center
  action: () => routeTo("/details"),
  onDismiss: () {},
  onShow: () {},
);
```

### Custom Toast Styles

Register in `AppProvider`:

```dart
await nylo.configure(
  toastNotifications: {
    ...ToastNotificationConfig.styles,
    'promo': ToastNotification.style(
      icon: Icon(Icons.local_offer, color: Colors.pink, size: 20),
      color: Colors.pink.shade50,
      defaultTitle: 'Special Offer',
      position: ToastNotificationPosition.bottom,
      duration: Duration(seconds: 8),
    ),
  },
);
```

Use with: `showToastCustom(id: "promo", description: "50% off today!")`.

### AlertTab Widget

Badge indicator for navigation tabs:

```dart
AlertTab(
  state: "notifications_tab",
  alertEnabled: true,
  icon: Icon(Icons.notifications),
  alertColor: Colors.red,
)
```

---

## Modals & Bottom Sheets

### Generate a Modal

```bash
metro make:bottom_sheet_modal payment_options
```

Creates a modal content widget and a static method on `BottomSheetModal`.

### Display a Modal

```dart
BottomSheetModal.showPaymentOptions(context);
```

### displayModal Parameters

```dart
displayModal<T>(
  context,
  child: MyContent(),
  actionsRow: [...],        // horizontal action buttons
  actionsColumn: [...],     // vertical action buttons
  height: 300,
  header: Text("Title"),
  showCloseButton: true,
  isScrollControlled: true, // enables scrolling
  showHandle: true,         // drag handle
  modalDecoration: BoxDecoration(...),
);
```

### Confirmation Pattern

```dart
bool? confirmed = await displayModal<bool>(
  context,
  child: Text("Are you sure?"),
  actionsRow: [
    TextButton(
      onPressed: () => Navigator.pop(context, false),
      child: Text("Cancel"),
    ),
    ElevatedButton(
      onPressed: () => Navigator.pop(context, true),
      child: Text("Confirm"),
    ),
  ],
);
```

---

## Buttons

Button definitions live in `lib/resources/widgets/buttons/buttons.dart`.

### Button Types

```dart
Button.primary(text: "Sign Up", onPressed: () {})
Button.secondary(text: "Learn More", onPressed: () {})
Button.outlined(text: "Cancel", borderColor: Colors.red, textColor: Colors.red, onPressed: () {})
Button.textOnly(text: "Skip", textColor: Colors.blue, onPressed: () {})
Button.icon(text: "Add to Cart", icon: Icon(Icons.shopping_cart), color: Colors.green, onPressed: () {})
Button.gradient(text: "Get Started", gradientColors: [Colors.purple, Colors.pink], onPressed: () {})
Button.rounded(text: "Continue", backgroundColor: Colors.teal, onPressed: () {})
Button.transparency(text: "Explore", color: Colors.white, onPressed: () {})
```

### Async Loading State

Return a `Future` from `onPressed` and the button auto-shows a loading indicator:

```dart
Button.primary(
  text: "Save Profile",
  onPressed: () async {
    await api<ApiService>((request) => request.updateProfile(name: "John"));
    showToastSuccess(description: "Profile saved!");
  },
)
```

### Loading Styles

```dart
loadingStyle: LoadingStyle.skeletonizer()   // shimmer skeleton (default)
loadingStyle: LoadingStyle.normal(child: Text("Please wait..."))
loadingStyle: LoadingStyle.none()           // disabled but visible
```

### Animation Styles

```dart
animationStyle: ButtonAnimationStyle.clickable()  // 3D press
animationStyle: ButtonAnimationStyle.bounce(scaleMin: 0.90)
animationStyle: ButtonAnimationStyle.pulse(pulseScale: 1.08)
animationStyle: ButtonAnimationStyle.squeeze()
animationStyle: ButtonAnimationStyle.jelly()
animationStyle: ButtonAnimationStyle.shine()
animationStyle: ButtonAnimationStyle.ripple()
animationStyle: ButtonAnimationStyle.morph(morphRadius: 30.0)
animationStyle: ButtonAnimationStyle.shake()     // good for error states
animationStyle: ButtonAnimationStyle.none()
```

### Splash Styles

```dart
splashStyle: ButtonSplashStyle.ripple(splashColor: Colors.blue, splashOpacity: 0.2)
splashStyle: ButtonSplashStyle.highlight()
splashStyle: ButtonSplashStyle.glow()
splashStyle: ButtonSplashStyle.ink()
splashStyle: ButtonSplashStyle.none()
```

### Form Submission

```dart
Button.primary(
  text: "Login",
  submitForm: (LoginForm(), (data) async {
    await api<AuthApiService>((request) => request.login(data));
  }),
  showToastError: false,
  onFailure: (error) { print(error); },
)
```

---

## CollectionView

Type-safe list/grid rendering with async loading, pagination, sorting, and filtering.

### Basic List

```dart
CollectionView<Todo>(
  data: () async {
    return await api<ApiService>((request) => request.fetchTodos());
  },
  builder: (context, item) {
    return ListTile(
      title: Text(item.data.title),
      subtitle: Text(item.isFirst ? "First item" : "Item ${item.index}"),
    );
  },
)
```

### Separated List

```dart
CollectionView<String>.separated(
  data: () => ['A', 'B', 'C'],
  builder: (context, item) => ListTile(title: Text(item.data)),
  separatorBuilder: (context, index) => Divider(),
)
```

### Grid Layout

```dart
CollectionView<Product>.grid(
  data: () async => await fetchProducts(),
  builder: (context, item) => ProductCard(product: item.data),
  crossAxisCount: 2,
  mainAxisSpacing: 8.0,
  crossAxisSpacing: 8.0,
)
```

### Pull-to-Refresh with Pagination

```dart
CollectionView<Post>.pullable(
  data: (int iteration) async {
    return await api<ApiService>((request) => request.get('/posts?page=$iteration'));
  },
  builder: (context, item) => PostCard(post: item.data),
  onRefresh: () => print('Refreshed!'),
  headerStyle: 'WaterDropHeader',
)
```

Also available: `CollectionView.pullableSeparated()`, `CollectionView.pullableGrid()`.

### CollectionItem Properties

Each builder receives a `CollectionItem<T>` with: `data`, `index`, `totalItems`, `isFirst`, `isLast`, `isOdd`, `isEven`, `progress` (0.0-1.0), `isAt(int)`, `isInRange(start, end)`, `isMultipleOf(int)`.

### Sorting & Filtering

```dart
CollectionView<Task>(
  data: () async => await fetchTasks(),
  builder: (context, item) => TaskTile(task: item.data),
  sort: (List<Task> items) {
    items.sort((a, b) => a.dueDate.compareTo(b.dueDate));
    return items;
  },
  transform: (List<Task> tasks) => tasks.where((t) => t.isActive).toList(),
)
```

### State Management

```dart
CollectionView<Item>(
  stateName: "my_list",
  // ...
)

// Programmatic control
CollectionView.stateReset("my_list");
CollectionView.removeFromIndex("my_list", 2);
updateState("my_list");
```

---

## FutureWidget

```dart
FutureWidget<User>(
  future: fetchCurrentUser(),
  child: (context, data) {
    return Text(data!.name);
  },
  loadingStyle: LoadingStyle.skeletonizer(
    child: UserCard(user: User.placeholder()),
    effect: SkeletonizerEffect.shimmer,
  ),
  onError: (context, error) {
    return Text("Failed to load user");
  },
)
```

Loading styles: `LoadingStyle.normal()`, `LoadingStyle.skeletonizer()`, `LoadingStyle.none()`.

---

## FadeOverlay

Gradient fade effect over child widgets (e.g. text over images):

```dart
FadeOverlay(
  strength: 0.2,       // 0.0 - 1.0
  color: Colors.black,
  child: Image.asset("photo.jpg"),
)

// Directional variants
FadeOverlay.top(child: myImage)
FadeOverlay.bottom(child: myImage)
FadeOverlay.left(child: myImage)
FadeOverlay.right(child: myImage)
```

---

## Pullable (Pull-to-Refresh)

Wrap any scrollable widget with pull-to-refresh and optional load-more:

```dart
Pullable(
  onRefresh: () async => await refreshData(),
  child: ListView(...),
)

// With load-more pagination
Pullable(
  onRefresh: () async => await refreshData(),
  onLoading: () async => await loadMore(),
  enablePullUp: true,
  child: ListView(...),
)

// Extension method on any widget
myListView.pullable(onRefresh: () async => await fetchItems())
```

### Header Styles

`Pullable()` (waterDrop default), `Pullable.classicHeader()`, `Pullable.materialClassicHeader()`, `Pullable.waterDropMaterialHeader()`, `Pullable.bezierHeader()`, `Pullable.noBounce()`, `Pullable.custom()`, `Pullable.builder()`.

---

## StyledText

### Children Mode

```dart
StyledText(
  style: TextStyle(fontSize: 16, color: Colors.black),
  children: [
    Text("Your order "),
    Text("#1234", style: TextStyle(fontWeight: FontWeight.bold)),
    Text(" has been "),
    Text("confirmed", style: TextStyle(color: Colors.green, fontWeight: FontWeight.bold)),
    Text("."),
  ],
)
```

### Template Mode

```dart
StyledText.template(
  "By continuing, you agree to our {{Terms of Service}} and {{Privacy Policy}}.",
  style: TextStyle(color: Colors.grey),
  styles: {
    "Terms of Service|Privacy Policy": TextStyle(
      color: Colors.blue,
      decoration: TextDecoration.underline,
    ),
  },
  onTap: {
    "Terms of Service": () => routeTo("/terms"),
    "Privacy Policy": () => routeTo("/privacy"),
  },
)
```

Use `|` in style/tap keys to apply to multiple placeholders. Use `{{key:text}}` syntax for localization keys.

---

## Spacing

```dart
// Vertical presets (for Columns)
Spacing.zero   // 0px
Spacing.xs     // 4px
Spacing.sm     // 8px
Spacing.md     // 16px
Spacing.lg     // 24px
Spacing.xl     // 32px

// Horizontal presets (for Rows)
Spacing.xsHorizontal  // 4px
Spacing.smHorizontal  // 8px
Spacing.mdHorizontal  // 16px
Spacing.lgHorizontal  // 24px
Spacing.xlHorizontal  // 32px

// Custom
Spacing.vertical(12)
Spacing.horizontal(20)
Spacing(width: 10, height: 20)

// In slivers
Spacing.md.asSliver()
```

---

## TextTr (Text Translation)

```dart
// Simple translation
TextTr("welcome_message")

// With arguments (maps to "Hello, {{name}}!" in lang JSON)
TextTr("greeting", arguments: {"name": "John"})

// Styled constructors
TextTr.headlineLarge("section_heading")
TextTr.bodyLarge("body_text")
TextTr.displayLarge("hero_title")
TextTr.labelLarge("label_text")
```

Translation files live in `/lang/en.json`, `/lang/es.json`, etc.

---

## LanguageSwitcher

```dart
// Dropdown (e.g. in AppBar actions)
actions: [LanguageSwitcher()]

// Bottom sheet modal
LanguageSwitcher.showBottomModal(context, height: 300);

// Static methods
await LanguageSwitcher.currentLanguage();
await LanguageSwitcher.getLanguageList();
await LanguageSwitcher.clearLanguage();
```

---

## Connective (Connectivity-Aware)

```dart
Connective(
  onWifi: Text('Connected via WiFi'),
  onMobile: Text('Connected via Mobile Data'),
  onNone: Text('No internet connection'),
  child: Text('Connected'),
  onConnectivityChanged: (state, results) {
    if (state == NyConnectivityState.none) {
      showSnackbar('You went offline');
    }
  },
)

// Builder pattern
Connective.builder(
  builder: (context, state, results) {
    if (state == NyConnectivityState.none) return OfflineWidget();
    return OnlineContent();
  },
)

// Offline banner
OfflineBanner(
  message: 'Check your connection',
  backgroundColor: Colors.orange,
  animate: true,
)

// Static checks
if (await NyConnectivity.isOnline()) { ... }
if (await NyConnectivity.isWifi()) { ... }

// Conditional execution
final data = await NyConnectivity.when(
  online: () async => await api.fetchData(),
  offline: () async => await cache.getData(),
);

// Widget extensions
MyContent().connectiveOr(offline: Text('Unavailable'))
SyncButton().onlyOnline()
OfflineMessage().onlyOffline()
```

---

## Text Style Extensions

Chainable extensions available on any `Text` widget:

```dart
Text("Welcome Back")
  .headingLarge(color: Colors.black)
  .fontWeightBold()
  .alignCenter()
  .setMaxLines(2)
  .setColor(context, (color) => color.primaryAccent)
```

Typography methods: `displayLarge()`, `displayMedium()`, `displaySmall()`, `headingLarge()`, `headingMedium()`, `headingSmall()`, `titleLarge()`, `titleMedium()`, `titleSmall()`, `bodyLarge()`, `bodyMedium()`, `bodySmall()`, `labelLarge()`, `labelMedium()`, `labelSmall()`.

Utility methods: `fontWeightBold()`, `fontWeightLight()`, `alignLeft()`, `alignRight()`, `alignCenter()`, `setMaxLines(int)`, `setFontFamily(String)`, `setFontSize(double)`, `paddingOnly(...)`, `copyWith(...)`.

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Toast not showing | Ensure you have `BuildContext` available; use `showToastNotification(context, ...)` outside NyState |
| CollectionView not refreshing | Assign a `stateName` and use `CollectionView.stateReset("name")` to reload |
| Button loading indicator never stops | Ensure the `onPressed` Future completes (resolves or throws); unhandled hangs keep loading |
| Pullable not triggering refresh | Child must be a scrollable widget (ListView, SingleChildScrollView, etc.) |
| StyledText template placeholders not styled | Keys in `styles` map must exactly match text between `{{ }}`; use `|` for multiple keys |
| TextTr showing key instead of translation | Verify the key exists in `/lang/en.json` and the language file is registered |
| Connective not detecting changes | Ensure `connectivity_plus` is properly configured in the project |
| CollectionView.pullable pagination stops | The `data` callback must return an empty list when no more pages exist |
| LanguageSwitcher not updating UI | The widget triggers a full rebuild; ensure state management propagates the locale change |
