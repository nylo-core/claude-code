---
name: nylo-state-management
description: Use when working with state management in a Nylo v7 Flutter project. Covers NyState, NyPage lifecycle, state actions, updateState, loading/locking helpers, providers (NyProvider), events (NyEvent/NyListener), and decoder registration.
---

# Nylo State Management

## Overview

Nylo v7 state management is built on two core classes: `NyState` (for reusable widgets) and `NyPage` (for pages, extending NyState). State updates propagate through named state identifiers, state actions enable cross-widget communication, and the provider/event systems handle app-level initialization and event-driven logic. Decoders bridge API responses to typed models.

## When to Use

- Building stateful widgets that need to update from external triggers
- Managing page lifecycle (init, loading, reboot)
- Communicating between widgets via state actions
- Locking UI during async operations (e.g. preventing double-submit)
- Registering app services and packages via providers
- Dispatching and listening to application events
- Registering model decoders for API response parsing
- When NOT to use: for simple local state within a single widget, standard Flutter `setState` is sufficient

## Quick Reference

| Task | Code |
|---|---|
| Create stateful widget | `metro make:stateful_widget my_widget` |
| Create state-managed widget | `metro make:state_managed_widget cart` |
| Update a widget's state | `updateState(Cart.state)` |
| Update with data | `updateState(Cart.state, data: "value")` |
| Fire a state action | `stateAction('refresh', state: MyWidget.state)` |
| Lock during async | `lockRelease('key', perform: () async { ... })` |
| Check if locked | `isLocked('key')` |
| Check if loading | `isLoading(name: 'key')` |
| Reboot page init | `reboot()` |
| Create provider | `metro make:provider cache_provider` |
| Create event | `metro make:event payment_event` |
| Dispatch event | `event<PaymentEvent>(data: {...})` |

## NyState

### Basic Structure

```dart
class MyWidget extends StatefulWidget {
  static String state = "my_widget";

  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends NyState<MyWidget> {
  _MyWidgetState() {
    stateName = MyWidget.state;
  }

  @override
  get init => () async {
    // Async initialization - loader shown automatically
  };

  @override
  void stateUpdated(data) {
    // Called when updateState(MyWidget.state) is invoked
    reboot(); // Re-run init to refresh data
  }

  @override
  Widget view(BuildContext context) {
    return Scaffold(
      body: Text("My Widget"),
    );
  }
}
```

### Loading Styles

Configure what users see during `init()`:

```dart
// Default loader widget
@override
LoadingStyle get loadingStyle => LoadingStyle.normal();

// Custom loading widget
@override
LoadingStyle get loadingStyle => LoadingStyle.normal(
  child: Center(child: Text("Loading...")),
);

// Skeleton shimmer effect
@override
LoadingStyle get loadingStyle => LoadingStyle.skeletonizer();

// No loading indicator
@override
LoadingStyle get loadingStyle => LoadingStyle.none();
```

### Lifecycle Methods

| Method | When Called |
|---|---|
| `init` | During state initialization (async supported, shows loader) |
| `stateUpdated(data)` | When `updateState(stateName)` is called externally |
| `view(context)` | Renders the widget UI (replaces `build`) |

## NyPage

NyPage extends NyState with page-specific capabilities. Pages use `path` instead of `state` for identification.

```dart
class _HomePageState extends NyPage<HomePage> {
  @override
  get init => () async {
    // Page initialization
  };

  @override
  Widget view(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Home")),
      body: Text("Welcome"),
    );
  }
}
```

Enable state management on a page:

```dart
class _MyPageState extends NyPage<MyPage> {
  @override
  bool get stateManaged => true;

  @override
  get stateActions => {
    "refresh-page": () {
      reboot();
    },
  };
}
```

## Updating State

Trigger a widget rebuild from anywhere in the app:

```dart
// Simple update
updateState(Cart.state);

// Update with data
updateState(Cart.state, data: {"count": 5});
```

The target widget's `stateUpdated(data)` method is called with the provided data.

### Full Example: Cart Counter

```dart
// Widget definition
class _CartState extends NyState<Cart> {
  int _count = 0;

  _CartState() {
    stateName = Cart.state;
  }

  @override
  get init => () async {
    _count = int.parse(await getCartValue());
  };

  @override
  void stateUpdated(data) {
    reboot();
  }

  @override
  Widget view(BuildContext context) {
    return Text("$_count");
  }
}

// Update from anywhere
Future incrementCart() async {
  String count = await getCartValue();
  await storageSave(Keys.cart, (int.parse(count) + 1).toString());
  updateState(Cart.state);
}
```

## State Actions

Trigger specific behaviors on widgets without full state rebuilds.

### Defining Actions

```dart
class _MyWidgetState extends NyState<MyWidget> {
  _MyWidgetState() {
    stateName = MyWidget.state;
  }

  @override
  get stateActions => {
    "hello_world": () {
      print('Hello world');
    },
    "reset_data": (data) async {
      _textController.clear();
      setState(() {});
    },
    "show_high_score": (data) {
      _score = data["high_score"];
      setState(() {});
    },
  };
}
```

Alternative definition using `whenStateAction` in init:

```dart
@override
get init => () async {
  whenStateAction({
    "reset_badge": () {
      _count = 0;
      setState(() {});
    },
  });
};
```

### Firing Actions

```dart
// Without data
stateAction('hello_world', state: MyWidget.state);

// With data
stateAction('show_high_score', state: HighScore.state, data: {"high_score": 100});

// On a NyPage (use path)
stateAction('refresh-page', state: MyPage.path);
```

### StateAction Helper Class

Built-in actions available: `refreshPage()`, `pop()`, `showToastSuccess()`, `showToastDanger()`, `showToastWarning()`, `validate()`, `changeLanguage()`, `confirmAction()`.

## Helper Methods

### Loading State

```dart
// Check if page is loading
if (isLoading()) return AppLoader();

// Named loading states
setLoading(true, name: 'refreshing');
await fetchData();
setLoading(false, name: 'refreshing');

if (isLoading(name: 'refreshing')) { ... }

// afterLoad - show loader until init completes
@override
Widget view(BuildContext context) {
  return afterLoad(child: () {
    return Text("Loaded");
  });
}
```

### Lock/Release (Prevent Double-Submit)

```dart
_login() async {
  await lockRelease('login', perform: () async {
    await Future.delayed(Duration(seconds: 4));
    print('Login attempt...');
  });
}

// Show loader while locked
if (isLocked('login'))
  CircularProgressIndicator()
else
  Button.primary(text: "Login", onPressed: _login)

// afterNotLocked helper
afterNotLocked('login', child: () {
  return Button.primary(text: "Login", onPressed: _login);
})
```

### Reboot

Re-executes the `init` method to refresh page data:

```dart
reboot();
```

### afterNotNull

Display loader until a variable is populated:

```dart
User? _user;

@override
get init => () async {
  _user = await api<ApiService>((request) => request.fetchUser());
  setState(() {});
};

@override
Widget view(BuildContext context) {
  return afterNotNull(_user, child: () {
    return Text(_user!.name);
  });
}
```

### Validation

```dart
validate(rules: {
  "email address": [textEmail, "email"],
}, onSuccess: () {
  print('Validation passed');
});
```

### Environment-Conditional Code

```dart
@override
get init => () {
  whenEnv('developing', perform: () {
    _emailController.text = 'test@example.com';
  });
};
```

### Confirmation Dialog

```dart
confirmAction(() {
  logout();
}, title: "Logout of the app?");
```

---

## Providers (NyProvider)

Providers initialize services and packages before the app starts.

### Creating a Provider

```bash
metro make:provider cache_provider
```

### Provider Structure

```dart
class CacheProvider implements NyProvider {
  @override
  Future<Nylo?> setup(Nylo nylo) async {
    // Runs first during bootstrap
    // Initialize packages, register services
    await CacheManager.init();
    return nylo; // Must return Nylo or null
  }

  @override
  Future<void> boot(Nylo nylo) async {
    // Runs after ALL providers complete setup()
    // Safe to use services from other providers
    User user = await Auth.user();
    if (!user.isSubscribed) {
      await Auth.remove();
    }
  }
}
```

### Bootstrap Lifecycle

1. `Boot.nylo` loops through providers registered in `config/providers.dart`
2. Each provider's `setup()` runs in order
3. After all `setup()` calls complete, each provider's `boot()` runs
4. `Boot.finished` binds the Nylo instance to Backpack as `'nylo'`
5. Access via: `Backpack.instance.read('nylo')`

### Registration

Providers are registered in `lib/config/providers.dart`:

```dart
final providers = [
  AppProvider(),
  CacheProvider(),
  // ...
];
```

---

## Events (NyEvent / NyListener)

### Creating an Event

```bash
metro make:event PaymentSuccessfulEvent
```

### Event Structure

```dart
class PaymentSuccessfulEvent implements NyEvent {
  final listeners = {
    DefaultListener: DefaultListener(),
  };
}

class DefaultListener extends NyListener {
  handle(dynamic event) async {
    // Process event data
  }
}
```

### Multiple Listeners

```dart
class PaymentSuccessfulEvent implements NyEvent {
  final listeners = {
    NotificationListener: NotificationListener(),
    AnalyticsListener: AnalyticsListener(),
    OrderProcessingListener: OrderProcessingListener(),
  };
}
```

### Dispatching Events

```dart
// Without data
event<PaymentSuccessfulEvent>();

// With data
event<PaymentSuccessfulEvent>(data: {
  'user': user,
  'amount': amount,
  'transactionId': 'txn_123456',
});

// Broadcasting (for external listeners)
event<PaymentSuccessfulEvent>(
  data: {'user': user},
  broadcast: true,
);
```

### Listening to Events

```dart
// Manual subscription (requires manual cancel)
NyEventSubscription subscription = listenOn<PaymentSuccessfulEvent>((data) {
  showSuccessMessage("Payment received");
});
subscription.cancel(); // when done

// In NyPage/NyState (auto-cleanup on dispose)
@override
get init => () {
  listen<PaymentSuccessfulEvent>((data) {
    routeTo(OrderConfirmationPage.path);
  });
};
```

### Global Broadcasting

Enable auto-broadcast for all events in `AppProvider`:

```dart
@override
boot(Nylo nylo) async {
  nylo.broadcastEvents();
}
```

---

## Decoders

Decoders convert API responses into typed Dart objects. Configured in `lib/config/decoders.dart`.

### Model Decoders

```dart
final modelDecoders = {
  User: (data) => User.fromJson(data),
  List<User>: (data) => List.from(data)
    .map((json) => User.fromJson(json)).toList(),
};
```

Used automatically by `network()` in API services:

```dart
class ApiService extends NyApiService {
  ApiService({BuildContext? buildContext})
    : super(buildContext, decoders: modelDecoders);

  Future<User?> fetchUser() async {
    return await network<User>(
      request: (request) => request.get("/user"),
    );
  }
}
```

### API Decoders

Register API service instances for the `api()` helper:

```dart
final Map<Type, dynamic> apiDecoders = {
  ApiService: ApiService(),
  AuthApiService: AuthApiService(),
};
```

Usage:

```dart
User user = await api<ApiService>(
  (request) => request.fetchUser(),
);
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| `stateUpdated` not called | Ensure `stateName` is set in the constructor and matches the value passed to `updateState()` |
| State action not firing | Verify the action name string matches exactly between definition and `stateAction()` call |
| NyPage state actions not working | Set `stateManaged => true` in the NyPage subclass |
| Double-tap causing duplicate requests | Use `lockRelease('key', perform: ...)` to prevent concurrent execution |
| Loading indicator never disappears | Ensure `init` completes (resolves or throws); unhandled async errors can hang loading |
| Provider boot code failing | `boot()` runs after all `setup()` calls; ensure dependencies are initialized in `setup()` of their respective providers |
| Event listeners accumulating | Use `listen()` in NyPage/NyState (auto-cleanup) instead of `listenOn()` which requires manual `cancel()` |
| Model decoder returning null | Ensure the decoder function in `modelDecoders` matches the exact type used in `network<T>()` |
| `api<T>()` throwing "not found" | Register the API service in `apiDecoders` in `config/decoders.dart` |
| `updateState` from NyPage | Use the page's `path` for NyPage state actions, not a `state` field |
