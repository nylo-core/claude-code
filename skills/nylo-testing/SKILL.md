---
name: nylo-testing
description: Use when writing, running, or debugging tests in a Nylo v7 Flutter project. Covers widget tests, unit tests, API mocking, factories, authentication simulation, time travel, state testing, and Nylo-specific assertions.
---

# Nylo Testing

## Overview

Nylo v7 provides a Laravel-inspired testing framework built on top of Flutter's test library. It includes helpers for widget testing, API mocking, authentication simulation, time manipulation, factories with fake data, and a rich set of custom assertions. Tests live in the `test/` directory and use `nyTest`, `nyWidgetTest`, and `nyGroup` as entry points.

## When to Use

- Writing unit tests for models, services, or utility functions
- Writing widget tests for NyPage or NyState-based pages
- Mocking API responses for network-dependent tests
- Simulating authenticated or guest users in tests
- Testing time-sensitive logic with time travel helpers
- Generating fake data with factories and NyFaker
- Asserting routes, state updates, toast messages, or lock/loading states
- When NOT to use: for standard Flutter widget tests that do not involve Nylo's state, routing, or API layers, use Flutter's built-in test utilities directly

## Quick Reference

| Task | Code |
|---|---|
| Initialize test file | `NyTest.init()` at top of test body |
| Unit test | `nyTest('name', () async { ... })` |
| Widget test | `nyWidgetTest('name', (tester) async { ... })` |
| Group tests | `nyGroup('name', () { ... })` |
| Pump a NyPage | `await tester.pumpNyWidget(child: MyPage())` |
| Pump and wait for init | `await tester.pumpNyWidgetAndWaitForInit(child: MyPage())` |
| Mock API response | `NyMockApi.respond('/users', {'id': 1})` |
| Simulate auth user | `NyTest.actingAs<User>(User(name: "Test"))` |
| Travel to a time | `NyTest.travel(DateTime(2025, 6, 15))` |
| Create factory model | `NyFactory.make<User>()` |
| Fire state update | `fireStateUpdate('HomePageState', data: {...})` |
| Debug test state | `NyTest.dump()` |

## Test File Structure

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:nylo_framework/nylo_framework.dart';

void main() {
  NyTest.init();

  nySetUpAll(() async {
    // Runs once before all tests
  });

  nySetUp(() async {
    // Runs before each test
  });

  nyTearDown(() async {
    // Runs after each test
  });

  nyTearDownAll(() async {
    // Runs once after all tests
  });

  nyTest('my unit test', () async {
    expect(1 + 1, equals(2));
  });

  nyWidgetTest('my widget test', (tester) async {
    await tester.pumpNyWidget(child: MyPage());
    expect(find.text('Hello'), findsOneWidget);
  });

  nyGroup('user tests', () {
    nyTest('can create user', () async {
      // ...
    });
  });
}
```

### Lifecycle Hooks

- `nySetUpAll()` - runs once before all tests in the file
- `nySetUp()` - runs before each individual test (state is auto-reset)
- `nyTearDown()` - runs after each individual test
- `nyTearDownAll()` - runs once after all tests in the file

`NyTest.init()` enables automatic state reset between tests.

## Widget Testing

### Pumping Widgets

```dart
// Full pump with MaterialApp, theme support, and Nylo initialization
await tester.pumpNyWidget(child: MyPage());

// Simple pump without custom fonts (faster, avoids font loading issues)
await tester.pumpNyWidgetSimple(child: MyWidget());

// Pump and wait for async init() to complete (waits for loading indicators to disappear)
await tester.pumpNyWidgetAndWaitForInit(child: MyPage());
```

### Testing a NyPage Directly

```dart
nyWidgetTest('home page loads correctly', (tester) async {
  await testNyPage(tester, MyHomePage(), (tester) async {
    expect(find.text('Welcome'), findsOneWidget);
    expect(find.byType(ListView), findsOneWidget);
  });
});
```

`testNyPage` pumps the page, waits for initialization, then runs the expectation callback.

### Simulating App Lifecycle

```dart
nyWidgetTest('handles background/foreground', (tester) async {
  await tester.pumpNyWidget(child: MyPage());
  await simulateLifecycleState(tester, AppLifecycleState.paused);
  await simulateLifecycleState(tester, AppLifecycleState.resumed);
});
```

### Checking Loading and Lock States

```dart
// Check if a named loading state is active
bool loading = isLoadingNamed('fetch-users');
bool locked = isLockedNamed('submit-form');
bool pageLoading = isLoading(); // general page loading
```

## API Mocking

### URL Pattern Matching

```dart
// Exact match
NyMockApi.respond('/users', {'id': 1, 'name': 'Test User'});

// Single-segment wildcard
NyMockApi.respond('/users/*', {'id': 1, 'name': 'User'});

// Multi-segment wildcard
NyMockApi.respond('/api/**', {'status': 'ok'});

// With options
NyMockApi.respond(
  '/users',
  {'id': 1},
  statusCode: 201,
  method: 'POST',
  headers: {'X-Custom': 'value'},
  delay: Duration(milliseconds: 500),
);
```

### Type-Based API Service Mocking

```dart
NyMockApi.register<UserApiService>((MockApiRequest request) async {
  return {'users': [{'id': 1, 'name': 'Mock User'}]};
});
```

### Call Tracking and Verification

```dart
// Enable call recording
NyMockApi.setRecordCalls(true);

// Run test code that makes API calls...

// Verify calls were made
expectApiCalled('/users');
expectApiCalled('/users', method: 'POST', times: 2);

// Get detailed call information
List<ApiCallInfo> calls = NyMockApi.getCallsFor('/users');
```

## Authentication Testing

```dart
// Simulate a logged-in user
NyTest.actingAs<User>(User(name: "Anthony", email: "anthony@test.com"));

// Assert authenticated
expectAuthenticated<User>();

// Access the acting user
User? user = NyTest.actingUser<User>();

// Log out
NyTest.logout();

// Assert guest
expectGuest();
```

## Time Travel

```dart
// Jump to a specific date
NyTest.travel(DateTime(2025, 6, 15));

// Move forward or backward
NyTest.travelForward(Duration(days: 30));
NyTest.travelBackward(Duration(hours: 2));

// Freeze time at current moment
NyTest.freezeTime();

// Scoped frozen time
NyTime.withFrozenTime(() {
  // Time is frozen within this block
});

// Boundary helpers
NyTest.travelToStartOfDay();
NyTest.travelToEndOfMonth();
```

## Factories and Fake Data

### Defining Factories

```dart
NyFactory.define<User>((NyFaker faker) => User(
  name: faker.name(),
  email: faker.email(),
));
```

### Creating Instances

```dart
// Single instance
User user = NyFactory.make<User>();

// With overrides
User user = NyFactory.make<User>(overrides: {'name': 'Custom Name'});
```

### NyFaker Data Methods

NyFaker provides realistic test data generation:

| Category | Methods |
|---|---|
| Identity | `name()`, `firstName()`, `lastName()`, `email()` |
| Numbers | `number()`, `randomInt()`, `randomDouble()` |
| Text | `sentence()`, `paragraph()`, `word()` |
| Dates | `date()`, `pastDate()`, `futureDate()` |
| Web | `url()`, `ipAddress()`, `domainName()` |
| Location | `address()`, `city()`, `country()`, `zipCode()` |
| Contact | `phoneNumber()`, `companyName()` |

## State Testing

### Firing State Updates and Actions

```dart
// Fire a state update with data
fireStateUpdate('HomePageState', data: {'items': ['a', 'b']});

// Fire a state action
fireStateAction('HomePageState', 'refresh-page');
```

### Asserting State Changes

```dart
expectStateUpdated('HomePageState');
expectStateAction('HomePageState', 'refresh-page');

// Assert widget data after state change
expectStateData(tester, find.byType(MyWidget), equals(42));

// Get tracked updates/actions
var updates = NyStateTestHelpers.getUpdatesFor('HomePageState');
var actions = NyStateTestHelpers.getActionsFor('HomePageState');
```

## Platform Channel Mocking

`NyMockChannels` automatically mocks common platform channels:

- `path_provider` - returns temporary directory paths
- `flutter_secure_storage` - in-memory key/value store
- `flutter_timezone` - returns a test timezone
- `flutter_local_notifications` - no-op notification channel
- `sqflite` - in-memory database

These are activated by `NyTest.init()` so tests run without platform errors.

## Route Guard Mocking

```dart
// Always passes
NyMockRouteGuard.pass();

// Always redirects
NyMockRouteGuard.redirect('/login');

// Custom logic
NyMockRouteGuard.custom((context) async {
  // return true to pass, false to block
});

// Check if guard was called
guard.wasCalled;    // bool
guard.callCount;    // int
guard.lastContext;   // BuildContext?
```

## Custom Assertions

### Route Assertions

```dart
expectRoute('/home');
expectNotRoute('/login');
expectRouteExists('/settings');
expectRouteInHistory('/onboarding');
```

### Backpack (Global State) Assertions

```dart
expectBackpackContains('user_token');
expectBackpackNotContains('expired_key');
```

### Environment Assertions

```dart
expectEnv('APP_NAME', 'MyApp');
expectEnvSet('API_KEY');
expectTestMode();
expectDebugMode();
expectProductionMode();
```

### Toast Assertions

```dart
// Requires setup
NyToastRecorder.setup();

// After an action that shows a toast
expectToastShown('Success');
expectNoToastShown();
```

### Lock/Loading Assertions

```dart
expectLocked('submit-form');
expectNotLocked('submit-form');
expectLoadingNamed('fetch-users');
expectNotLoadingNamed('fetch-users');
```

## Debugging Tools

```dart
// Print full test state: Backpack, auth user, time, API calls, locale
NyTest.dump();

// Dump and stop test execution
NyTest.dd();

// Store/retrieve values during a test
NyTest.set('my_key', 'my_value');
var val = NyTest.get('my_key');

// Pre-populate Backpack with test data
NyTest.seedBackpack({'token': 'abc123', 'user_id': 1});
```

## Test Modifiers

```dart
// Skip a test
nySkip('Reason for skipping');

// Mark a test as known-failing
nyFailing();

// Run only in CI environments
nyCi();

// Skip with nyTest options
nyTest('my test', () async { ... }, skip: 'Not ready yet');
nyTest('slow test', () async { ... }, timeout: Timeout(Duration(seconds: 30)));
```

## Custom Matchers

```dart
expect(result, isType<User>());
expect(widget, hasRouteName('/home'));
expect(state, backpackHas('token'));
expect(tracker, apiWasCalled('/users'));
```

## Common Mistakes

| Mistake | Fix |
|---|---|
| Forgetting `NyTest.init()` | Add it at the top of `main()` in every test file; it enables state reset and platform channel mocking |
| Using `tester.pumpWidget` instead of `pumpNyWidget` | `pumpNyWidget` wraps the widget in MaterialApp with Nylo theme support; use it for NyPage/NyState widgets |
| Async `init()` not completing before assertions | Use `pumpNyWidgetAndWaitForInit` or `testNyPage` to wait for async initialization |
| API mock not matching requests | Check URL pattern: exact match vs wildcard (`*` = one segment, `**` = multiple segments). Verify method matches too |
| Auth state leaking between tests | `NyTest.init()` resets state between tests; ensure each test calls `actingAs` independently |
| Platform channel errors in tests | `NyTest.init()` mocks common channels; if you use additional plugins, mock their channels manually |
| Time not resetting between tests | Time is auto-reset by `NyTest.init()`; avoid setting time in `setUpAll` if individual tests need different times |
