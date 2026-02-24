---
name: nylo-routing
description: Use when defining routes, navigating between pages, adding route guards, passing parameters/data between pages, setting up deep links, configuring page transitions, building bottom/top navigation with NavigationHub, or creating onboarding journey flows in a Nylo v7 Flutter app.
---

# Nylo Routing

## Overview

Nylo v7 routing is defined in `lib/routes/router.dart` using `nyRoutes()`. Routes map URL paths to page widgets, support guards for access control, and integrate with NavigationHub for tab-based and journey navigation layouts.

## When to Use

- Defining or modifying app routes
- Navigating between pages (push, replace, pop)
- Passing data, query parameters, or path parameters between pages
- Protecting routes with guards (auth, roles, subscriptions)
- Setting up bottom navigation, top tabs, or onboarding journeys
- Configuring page transition animations
- Implementing deep linking
- When NOT to use: For state management within a single page, use NyState/NyPage instead

## Quick Reference

| Action | Code |
|--------|------|
| Define route | `router.add(MyPage.path)` |
| Set initial route | `router.add(MyPage.path).initialRoute()` |
| Navigate to page | `routeTo(MyPage.path)` |
| Navigate with data | `routeTo(MyPage.path, data: {"key": "val"})` |
| Navigate with query params | `routeTo(MyPage.path, queryParameters: {"tab": "1"})` |
| Navigate with path params | `routeTo(MyPage.path.withParams({"id": 7}))` |
| Go back | `pop()` |
| Go back with result | `pop(result: {"status": "done"})` |
| Replace current route | `routeTo(MyPage.path, navigationType: NavigationType.pushReplace)` |
| Clear stack and navigate | `routeTo(MyPage.path, navigationType: NavigationType.pushAndForgetAll)` |
| Navigate to initial route | `routeToInitial()` |
| Navigate to auth route | `routeToAuthenticatedRoute()` |
| Add route guard | `router.add(MyPage.path, routeGuards: [MyGuard()])` |
| Create page (CLI) | `metro make:page profile_page` |
| Create guard (CLI) | `metro make:route_guard auth` |
| Create nav hub (CLI) | `metro make:navigation_hub base` |

## Route Definitions

Routes live in `lib/routes/router.dart`:

```dart
appRouter() => nyRoutes((router) {
  router.add(HomePage.path).initialRoute();
  router.add(ProfilePage.path);
  router.add(SettingsPage.path);
});
```

### Special Route Types

```dart
// Initial route - first page on app launch
router.add(HomePage.path).initialRoute();

// Conditional initial route
router.add(OnboardingPage.path).initialRoute(
  when: () => !hasCompletedOnboarding()
);
router.add(HomePage.path).initialRoute(
  when: () => hasCompletedOnboarding()
);

// Authenticated route - shown after Auth.authenticate()
router.add(DashboardPage.path).authenticatedRoute();

// Unknown/404 route
router.add(NotFoundPage.path).unknownRoute();

// Preview route - dev only, remove before release
router.add(ProfilePage.path).previewRoute();
```

### Page with Path Parameters

```dart
class ProfilePage extends NyStatefulWidget<HomeController> {
  static RouteView path = ("/profile/{userId}", (_) => ProfilePage());
  ProfilePage() : super(child: () => _ProfilePageState());
}
```

## Navigation

### Basic Navigation

```dart
routeTo(SettingsPage.path);
```

### Navigation Types

```dart
// Push onto stack (default)
routeTo(MyPage.path, navigationType: NavigationType.push);

// Replace current route
routeTo(MyPage.path, navigationType: NavigationType.pushReplace);

// Pop current, then push
routeTo(MyPage.path, navigationType: NavigationType.popAndPushNamed);

// Push and clear entire stack
routeTo(MyPage.path, navigationType: NavigationType.pushAndForgetAll);
```

### Passing Data

```dart
// Send data
routeTo(ProfilePage.path, data: {"firstName": "Anthony"});

// Receive data in target page
class _ProfilePageState extends NyPage<ProfilePage> {
  @override
  get init => () {
    final data = widget.data();
    print(data["firstName"]); // Anthony
  };
}
```

### Path Parameters

```dart
// Navigate with path params (e.g., /profile/7)
routeTo(ProfilePage.path.withParams({"userId": 7}));

// Access in target page
class _ProfilePageState extends NyPage<ProfilePage> {
  @override
  get init => () {
    print(widget.queryParameters()); // {"userId": "7"}
  };
}
```

### Query Parameters

```dart
// Navigate with query params
routeTo(ProfilePage.path, queryParameters: {"user": "20", "tab": "posts"});

// Or use withQueryParams
routeTo(ProfilePage.path.withQueryParams({"user": "20", "tab": "posts"}));

// Access in target page
class _ProfilePageState extends NyPage<ProfilePage> {
  @override
  get init => () {
    final params = queryParameters();
    final userId = params["user"];   // "20"
    final tab = params["tab"];       // "posts"
  };
}
```

### Pop with Result

```dart
// Navigate and listen for result
routeTo(SettingsPage.path, onPop: (value) {
  print(value); // {"status": "COMPLETE"}
});

// In SettingsPage, return result
pop(result: {"status": "COMPLETE"});
```

### Conditional Navigation

```dart
routeIf(isLoggedIn, DashboardPage.path);

routeIf(
  hasPermission('view_reports'),
  ReportsPage.path,
  data: {'filters': defaultFilters},
  navigationType: NavigationType.push,
);
```

## Route Guards

Guards run before navigation to control access. Generate with CLI:

```bash
metro make:route_guard auth
```

### Basic Guard

```dart
class AuthRouteGuard extends NyRouteGuard {
  @override
  Future<GuardResult> onBefore(RouteContext context) async {
    bool isLoggedIn = await Auth.isAuthenticated();
    if (!isLoggedIn) {
      return redirect(LoginPage.path);
    }
    return next();
  }

  @override
  Future<void> onAfter(RouteContext context) async {
    Analytics.trackPageView(context.routeName);
  }

  @override
  Future<bool> onLeave(RouteContext context) async {
    if (hasUnsavedChanges) {
      return await showConfirmDialog();
    }
    return true; // allow leaving
  }
}
```

### Guard Actions

| Action | Purpose |
|--------|---------|
| `return next()` | Continue to route |
| `return redirect(LoginPage.path)` | Redirect to another route |
| `return abort()` | Cancel navigation, stay on current page |
| `setData({...})` | Enrich data for subsequent guards/target |

### Redirect with Options

```dart
return redirect(
  LoginPage.path,
  data: {"returnTo": context.routeName},
  navigationType: NavigationType.pushReplace,
  queryParameters: {"source": "guard"},
);
```

### Apply Guards to Routes

```dart
// Single guard
router.add(DashboardPage.path, routeGuards: [AuthRouteGuard()]);

// Multiple guards (run in sequence)
router.add(AdminPage.path, routeGuards: [AuthRouteGuard(), AdminRoleGuard()]);

// Fluent API
router.add(DashboardPage.path).addRouteGuard(MyGuard());
router.add(DashboardPage.path).addRouteGuards([Guard1(), Guard2()]);
```

### Parameterized Guards

```dart
class RoleGuard extends ParameterizedGuard<List<String>> {
  RoleGuard(super.params);

  @override
  Future<GuardResult> onBefore(RouteContext context) async {
    User? user = await Auth.user<User>();
    if (user == null || !params.contains(user.role)) {
      return redirect(UnauthorizedPage.path);
    }
    return next();
  }
}

// Usage
router.add(AdminPage.path, routeGuards: [
  RoleGuard(["admin", "super_admin"])
]);
```

### Guard Stacks (Reusable Combinations)

```dart
final protectedRoute = GuardStack([
  AuthRouteGuard(),
  VerifyEmailGuard(),
  TwoFactorGuard(),
]);

router.add(SecurePage.path, routeGuards: [protectedRoute]);
```

### Conditional Guards

```dart
router.add(BetaPage.path, routeGuards: [
  ConditionalGuard(
    condition: (context) => context.queryParameters.containsKey("beta"),
    guard: BetaAccessGuard(),
  ),
]);
```

## Route Groups

Share settings across multiple routes:

```dart
router.group(() => {
  "route_guards": [AuthRouteGuard()],
  "prefix": "/dashboard",
  "transition_type": TransitionType.fade(),
}, (router) {
  router.add(ChatPage.path);
  router.add(FollowersPage.path);
});
```

| Setting | Purpose |
|---------|---------|
| `route_guards` | Guards applied to all routes in group |
| `prefix` | URL prefix for all routes |
| `transition_type` | Shared transition animation |

## Page Transitions

```dart
router.add(SettingsPage.path,
  transitionType: TransitionType.bottomToTop()
);

// Or when navigating
routeTo(ProfilePage.path, transitionType: TransitionType.fade());
```

### Available Transitions

| Category | Types |
|----------|-------|
| Basic | `fade()`, `theme()` |
| Directional | `rightToLeft()`, `leftToRight()`, `topToBottom()`, `bottomToTop()` |
| With fade | `rightToLeftWithFade()`, `leftToRightWithFade()` |
| Transform | `scale(alignment:)`, `rotate(alignment:)`, `size(alignment:)` |
| Material | `sharedAxisHorizontal()`, `sharedAxisVertical()`, `sharedAxisScale()` |

Each transition accepts optional `curve`, `duration`, `reverseDuration`, `fullscreenDialog`, and `opaque` parameters.

## NavigationHub

NavigationHub manages multi-tab and journey-based navigation layouts. Generate with CLI:

```bash
metro make:navigation_hub base
# Choose: navigation_tabs or journey_states
```

### Bottom Navigation Hub

```dart
class BaseNavigationHub extends NyStatefulWidget with BottomNavPageControls {
  static RouteView path = ("/base", (_) => BaseNavigationHub());
  BaseNavigationHub()
      : super(child: () => _BaseNavigationHubState(),
              stateName: path.stateName());

  static NavigationHubStateActions stateActions =
    NavigationHubStateActions(path.stateName());
}

class _BaseNavigationHubState extends NavigationHub<BaseNavigationHub> {
  @override
  NavigationHubLayout? layout(BuildContext context) =>
    NavigationHubLayout.bottomNav();

  @override
  bool get maintainState => true;

  @override
  int get initialIndex => 0;

  _BaseNavigationHubState() : super(() => {
    0: NavigationTab.tab(title: "Home", page: HomeTab(), icon: Icon(Icons.home)),
    1: NavigationTab.tab(title: "Settings", page: SettingsTab(), icon: Icon(Icons.settings)),
  });

  @override
  onTap(int index) {
    super.onTap(index);
  }
}
```

### Layout Types

```dart
// Bottom navigation
NavigationHubLayout.bottomNav(
  backgroundColor: Colors.white,
  selectedItemColor: Colors.blue,
  unselectedItemColor: Colors.grey,
);

// Top navigation (tabs)
NavigationHubLayout.topNav(
  labelColor: Colors.blue,
  unselectedLabelColor: Colors.grey,
  indicatorColor: Colors.blue,
);

// Journey (onboarding/multi-step)
NavigationHubLayout.journey(
  progressStyle: JourneyProgressStyle(
    indicator: JourneyProgressIndicator.linear(),
  ),
);
```

### Tab Types

```dart
// Standard tab
NavigationTab.tab(title: "Home", page: HomeTab(), icon: Icon(Icons.home));

// Badge tab (shows count)
NavigationTab.badge(title: "Chats", page: ChatTab(),
  icon: Icon(Icons.message), initialCount: 10);

// Alert tab (shows dot indicator)
NavigationTab.alert(title: "Alerts", page: AlertTab(),
  icon: Icon(Icons.notification_important), alertEnabled: true);

// Journey tab
NavigationTab.journey(page: WelcomeStep());
```

### Controlling Hub from Anywhere

```dart
// Switch tab programmatically
MyHub.stateActions.currentTabIndex(2);

// Badge management
MyHub.stateActions.updateBadgeCount(tab: 0, count: 5);
MyHub.stateActions.incrementBadgeCount(tab: 0);
MyHub.stateActions.clearBadgeCount(tab: 0);

// Alert management
MyHub.stateActions.alertEnableTab(tab: 0);
MyHub.stateActions.alertDisableTab(tab: 0);
```

### Journey Navigation (Onboarding Flows)

Journey states extend `JourneyState` and provide lifecycle hooks:

```dart
class _WelcomeState extends JourneyState<Welcome> {
  _WelcomeState() : super(
    navigationHubState: OnboardingNavigationHub.path.stateName()
  );

  @override
  Future<bool> canContinue() async {
    if (nameController.text.isEmpty) {
      showToastSorry(description: "Please enter your name");
      return false;
    }
    return true;
  }

  @override
  Future<void> onBeforeNext() async {
    session('onboarding', {'name': nameController.text});
  }

  @override
  Widget view(BuildContext context) {
    return buildJourneyContent(
      content: Text("Welcome!"),
      nextButton: Button.primary(text: "Continue", onPressed: nextStep),
      backButton: isFirstStep ? null : Button.textOnly(
        text: "Back", onPressed: onBackPressed),
    );
  }
}
```

**Key JourneyState properties:** `isFirstStep`, `isLastStep`, `currentStep`, `totalSteps`, `completionPercentage`

**Key JourneyState methods:** `nextStep()`, `previousStep()`, `goToStep(index)`, `exitJourney()`

## Route History

```dart
Nylo.getRouteHistory();            // All visited routes
Nylo.getCurrentRouteName();        // Current route path
Nylo.getPreviousRouteName();       // Previous route path
Nylo.getCurrentRouteArguments();   // Current route data

// Programmatically update the navigation stack
NyNavigator.updateStack([
  HomePage.path,
  ProfilePage.path,
], replace: true, dataForRoute: {
  ProfilePage.path: {"userId": 42},
});
```

## Deep Linking

Routes automatically support deep links. Configure platform-specific settings (universal links for iOS, app links for Android), then handle events:

```dart
nylo.onDeepLink((String route, Map<String, String>? data) {
  if (route == ProfilePage.path) {
    NyNavigator.updateStack([
      HomePage.path,
      ProfilePage.path,
    ], replace: true, dataForRoute: {
      ProfilePage.path: data,
    });
  }
});
```

Test with CLI:
```bash
# Android
adb shell am start -a android.intent.action.VIEW \
  -d "https://yourdomain.com/profile?user=20" com.yourcompany.yourapp

# iOS Simulator
xcrun simctl openurl booted "https://yourdomain.com/profile?user=20"
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `super.onTap(index)` in NavigationHub `onTap` override | Always call `super.onTap(index)` to update the tab index |
| Using path params but accessing via `widget.data()` | Path params are accessed via `widget.queryParameters()` or `queryParameters()` |
| Not converting URL params from strings | All URL/query params arrive as strings; parse with `int.parse()`, `DateTime.parse()`, etc. |
| Leaving `previewRoute()` in production | Remove `.previewRoute()` before release builds |
| Calling `routeTo` without registering the route in `router.dart` | Every page must be registered with `router.add()` |
| Guards returning nothing | Always return `next()`, `redirect()`, or `abort()` from `onBefore` |
| Using `NavigationType.pushAndForgetAll` for regular navigation | Only use when you intentionally want to clear the entire nav stack (e.g., after login) |
