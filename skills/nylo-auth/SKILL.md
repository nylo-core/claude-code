---
name: nylo-auth
description: Use when implementing user authentication, login/logout flows, storing auth tokens, checking authentication state, managing multiple auth sessions, accessing device IDs, or configuring authenticated routes in a Nylo v7 Flutter app.
---

# Nylo Authentication

## Overview

Nylo v7 authentication is managed through the `Auth` class, which stores user session data securely and syncs it to Backpack (an in-memory key-value store) for fast synchronous access. It supports multiple named sessions, token management, and integrates with the router for authenticated routes.

## When to Use

- Implementing login/logout flows
- Storing and retrieving auth tokens
- Checking if a user is authenticated
- Updating stored auth data (e.g., token refresh)
- Managing multiple auth sessions (user, device, admin)
- Setting up routes that require authentication
- Generating unique device identifiers
- When NOT to use: For general-purpose key-value storage unrelated to auth, use nylo-storage instead

## Quick Reference

| Action | Code |
|--------|------|
| Authenticate user | `await Auth.authenticate(data: user)` |
| Get all auth data | `Auth.data()` |
| Get specific field | `Auth.data(field: 'token')` |
| Check if authenticated | `await Auth.isAuthenticated()` |
| Update auth data | `await Auth.set((data) { data['token'] = newToken; return data; })` |
| Logout | `await Auth.logout()` |
| Logout all sessions | `await Auth.logoutAll(sessions: ['device', 'admin'])` |
| Sync to Backpack | `await Auth.syncToBackpack()` |
| Get device ID | `await Auth.deviceId()` |
| Set authenticated route | `router.add(HomePage.path).authenticatedRoute()` |
| Navigate to auth route | `routeToAuthenticatedRoute()` |

## Auth.authenticate()

Stores user session data. Accepts a Map, a Model instance, or no data (timestamp only):

```dart
// Store a Map
await Auth.authenticate(data: {
  "token": "ey2sdm...",
  "userId": 123,
  "email": "user@example.com",
});

// Store a Model instance
User user = User(id: 123, email: "user@example.com", token: "ey2sdm...");
await Auth.authenticate(data: user);

// Authenticate with no data (stores timestamp only)
await Auth.authenticate();

// Named session
await Auth.authenticate(
  data: {"deviceToken": "abc123"},
  session: 'device',
);
```

## Auth.data()

Retrieves stored auth data synchronously from Backpack:

```dart
// All auth data
dynamic userData = Auth.data();

// Specific field
String? token = Auth.data(field: 'token');
int? userId = Auth.data(field: 'userId');

// Named session
dynamic deviceData = Auth.data(session: 'device');
String? deviceToken = Auth.data(field: 'deviceToken', session: 'device');
```

## Auth.isAuthenticated()

Checks if a user is currently authenticated:

```dart
bool isLoggedIn = await Auth.isAuthenticated();

// Check specific session
bool deviceRegistered = await Auth.isAuthenticated(session: 'device');
bool isAdmin = await Auth.isAuthenticated(session: 'admin');
```

## Auth.set()

Updates or modifies existing auth data:

```dart
// Update specific field
await Auth.set((data) {
  data['token'] = 'new_token_value';
  return data;
});

// Add new fields
await Auth.set((data) {
  data['refreshToken'] = 'refresh_abc123';
  data['lastLogin'] = DateTime.now().toIso8601String();
  return data;
});

// Replace entire data
await Auth.set((data) => {
  'token': newToken,
  'userId': userId,
});
```

## Auth.logout()

Removes authenticated user from default or named session:

```dart
// Default session
await Auth.logout();

// Named session
await Auth.logout(session: 'device');

// All sessions
await Auth.logoutAll(sessions: ['device', 'admin']);
```

## Complete Login Flow

```dart
class _LoginPageState extends NyPage<LoginPage> {

  Future<void> handleLogin(String email, String password) async {
    // 1. Call API
    User? user = await api<AuthApiService>((request) =>
      request.login(email: email, password: password)
    );

    if (user == null) {
      showToastDanger(description: "Invalid credentials");
      return;
    }

    // 2. Store authenticated user
    await Auth.authenticate(data: user);

    // 3. Navigate to authenticated route (clears stack)
    routeToAuthenticatedRoute();
  }
}
```

## Complete Logout Flow

```dart
Future<void> handleLogout() async {
  await Auth.logout();
  routeTo(LoginPage.path, navigationType: NavigationType.pushAndForgetAll);
}
```

## Multiple Named Sessions

Nylo supports separate authentication contexts for different purposes:

```dart
// Default user session
await Auth.authenticate(data: {
  "token": "user_token_123",
  "userId": 42,
});

// Device session
await Auth.authenticate(
  data: {"deviceToken": "abc123"},
  session: 'device',
);

// Admin session
await Auth.authenticate(
  data: {"adminToken": "admin_xyz", "role": "superadmin"},
  session: 'admin',
);

// Access each session independently
String? userToken = Auth.data(field: 'token');            // default session
String? deviceToken = Auth.data(field: 'deviceToken', session: 'device');
String? adminRole = Auth.data(field: 'role', session: 'admin');

// Check each session
bool isLoggedIn = await Auth.isAuthenticated();
bool isDeviceRegistered = await Auth.isAuthenticated(session: 'device');
bool isAdmin = await Auth.isAuthenticated(session: 'admin');

// Logout specific session
await Auth.logout(session: 'device');
```

## Token Storage and Backpack Sync

Auth data is stored securely (persistent storage) and automatically synced to Backpack (in-memory store) for fast synchronous reads. If you need to manually sync:

```dart
// Sync default session
await Auth.syncToBackpack();

// Sync specific session
await Auth.syncToBackpack(session: 'device');

// Sync all sessions
await Auth.syncAllToBackpack(sessions: ['device', 'admin']);
```

**How it works:**
1. `Auth.authenticate()` writes data to secure persistent storage AND syncs to Backpack
2. `Auth.data()` reads from Backpack (in-memory) for instant access
3. `Auth.set()` updates both persistent storage and Backpack
4. `Auth.logout()` removes from both stores

## Token Refresh Pattern

Combine Auth with NyApiService's token refresh:

```dart
class ApiService extends NyApiService {
  @override
  Future<RequestHeaders> setAuthHeaders(RequestHeaders headers) async {
    String? token = Auth.data(field: 'token');
    if (token != null) {
      headers.addBearerToken(token);
    }
    return headers;
  }

  @override
  Future<bool> shouldRefreshToken() async {
    // Check if token is expired
    return false;
  }

  @override
  Future<void> refreshToken(Dio dio) async {
    dynamic response = (await dio.post(
      "https://example.com/refresh-token"
    )).data;

    // Update stored token
    await Auth.set((data) {
      data['token'] = response['token'];
      return data;
    });
  }
}
```

## Device ID

Generate and persist a unique device identifier across sessions:

```dart
String deviceId = await Auth.deviceId();
// Example: "550e8400-e29b-41d4-a716-446655440000-1704067200000"
```

### Register Device with Backend

```dart
Future<void> registerDevice() async {
  String deviceId = await Auth.deviceId();
  String? pushToken = await FirebaseMessaging.instance.getToken();

  await api<DeviceApiService>((request) =>
    request.register(deviceId: deviceId, pushToken: pushToken)
  );

  await Auth.authenticate(
    data: {"deviceId": deviceId, "pushToken": pushToken},
    session: 'device',
  );
}
```

## Authenticated Routes

Set a route as the initial page for authenticated users in `lib/routes/router.dart`:

```dart
appRouter() => nyRoutes((router) {
  router.add(LoginPage.path).initialRoute();
  router.add(HomePage.path).authenticatedRoute();
  router.add(ProfilePage.path);
});
```

After authentication, navigate to the authenticated route:

```dart
await Auth.authenticate(data: user);
routeToAuthenticatedRoute();
```

### Conditional Authenticated Route

```dart
router.add(HomePage.path).authenticatedRoute(
  when: () => hasCompletedSetup()
);
```

### Auth Route Guard

Protect routes that require authentication:

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
}

// Apply to routes
router.add(ProfilePage.path, routeGuards: [AuthRouteGuard()]);

// Apply to route group
router.group(() => {
  "route_guards": [AuthRouteGuard()],
  "prefix": "/dashboard",
}, (router) {
  router.add(DashboardPage.path);
  router.add(SettingsPage.path);
});
```

## Helper Functions

Nylo provides convenience functions that mirror `Auth` class methods:

```dart
// Authenticate
await authAuthenticate(data: user);
await authAuthenticate(data: device, session: 'device');

// Read data
dynamic userData = authData();
String? token = authData(field: 'token');
dynamic deviceData = authData(session: 'device');

// Update data
await authSet((data) {
  data['token'] = newToken;
  return data;
});

// Check auth state
bool isLoggedIn = await authIsAuthenticated();
bool deviceAuth = await authIsAuthenticated(session: 'device');

// Logout
await authLogout();
await authLogoutAll(sessions: ['device', 'admin']);

// Sync
await authSyncToBackpack();

// Device ID
String deviceId = await authDeviceId();

// Auth key
String key = authKey();
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Calling `Auth.data()` before `Auth.authenticate()` | Always authenticate first; `Auth.data()` returns null if no session exists |
| Not awaiting `Auth.authenticate()` | It is async; always use `await` to ensure data is stored before proceeding |
| Navigating before auth completes | `await Auth.authenticate(data: user)` must finish before calling `routeToAuthenticatedRoute()` |
| Forgetting to clear nav stack on logout | Use `NavigationType.pushAndForgetAll` or `routeToInitial()` after logout to prevent back navigation |
| Using `Auth.data()` in an API interceptor without null check | Token may be null if user is not authenticated; always check before adding to headers |
| Not setting up `authenticatedRoute()` in router | The route must be registered with `.authenticatedRoute()` for `routeToAuthenticatedRoute()` to work |
| Storing large objects in Auth | Auth is for credentials and tokens; store user profiles in Backpack or NyStorage separately |
| Confusing session names | Named sessions are string-based; use consistent constants to avoid typos |
