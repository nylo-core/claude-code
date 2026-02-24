---
name: nylo-networking
description: Use when making HTTP requests, creating API services, configuring interceptors, caching API responses, handling token refresh, uploading files, or working with NyApiService and NyResponse in a Nylo v7 Flutter app.
---

# Nylo Networking

## Overview

Nylo v7 networking is built on Dio and centered around `NyApiService` classes in `lib/app/networking/`. Services provide `network()` for typed responses and `networkResponse()` for full `NyResponse<T>` objects with status codes, headers, and error details.

## When to Use

- Creating or modifying API service classes
- Making GET, POST, PUT, DELETE, PATCH requests
- Adding request/response interceptors
- Caching API responses with cache policies
- Handling authentication tokens and refresh logic
- Uploading or downloading files
- Configuring retry logic, timeouts, or connectivity checks
- When NOT to use: For local storage without network calls, use nylo-storage instead

## Quick Reference

| Action | Code |
|--------|------|
| Create API service (CLI) | `metro make:api_service user` |
| Create with model | `metro make:api_service user --model="User"` |
| Create interceptor (CLI) | `metro make:interceptor logging` |
| Simple GET | `await get<User>("/users/1")` |
| Simple POST | `await post<User>("/users", data: payload)` |
| network() call | `await network<User>(request: (r) => r.get("/users/1"))` |
| networkResponse() call | `await networkResponse<User>(request: (r) => r.get("/users/1"))` |
| Use from page | `await api<ApiService>((r) => r.fetchUser())` |
| Cache response | `network(..., cacheKey: "key", cacheDuration: Duration(hours: 1))` |
| Bearer token | `network(..., bearerToken: "token123")` |
| Retry requests | `network(..., retry: 3, retryDelay: Duration(seconds: 2))` |
| Clear cache | `await apiService.clearCache("key")` |

## API Service Structure

Services live in `lib/app/networking/` and extend `NyApiService`:

```dart
class ApiService extends NyApiService {
  ApiService({BuildContext? buildContext})
      : super(
          buildContext,
          decoders: modelDecoders,
        );

  @override
  String get baseUrl => getEnv('API_BASE_URL');

  @override
  Map<Type, Interceptor> get interceptors => {
    ...super.interceptors,
  };

  Future<User?> fetchUser(int id) async {
    return await network<User>(
      request: (request) => request.get("/users/$id"),
    );
  }

  Future<List<User>?> fetchUsers() async {
    return await network<List<User>>(
      request: (request) => request.get("/users"),
    );
  }

  Future<User?> createUser(Map<String, dynamic> data) async {
    return await network<User>(
      request: (request) => request.post("/users", data: data),
    );
  }
}
```

Generate a new service with CLI:

```bash
metro make:api_service user
metro make:api_service user --model="User"
```

## Request Methods

### Convenience Methods

Shorthand wrappers around `network()`:

```dart
// GET
Future<User?> fetchUser(int id) async {
  return await get<User>("/users/$id", queryParameters: {"include": "profile"});
}

// POST
Future<User?> createUser(Map<String, dynamic> data) async {
  return await post<User>("/users", data: data);
}

// PUT
Future<User?> updateUser(int id, Map<String, dynamic> data) async {
  return await put<User>("/users/$id", data: data);
}

// DELETE
Future<bool?> deleteUser(int id) async {
  return await delete<bool>("/users/$id");
}

// PATCH
Future<User?> patchUser(int id, Map<String, dynamic> data) async {
  return await patch<User>("/users/$id", data: data);
}
```

### network() Method

Use for advanced features like caching, retries, custom headers:

```dart
Future<User?> fetchUser(int id) async {
  return await network<User>(
    request: (request) => request.get("/users/$id"),
    bearerToken: "token123",
    headers: {"X-Custom": "value"},
    retry: 3,
    retryDelay: Duration(seconds: 2),
    cacheKey: "user_$id",
    cacheDuration: Duration(minutes: 30),
    cachePolicy: CachePolicy.staleWhileRevalidate,
  );
}
```

**Key parameters:**

| Parameter | Type | Purpose |
|-----------|------|---------|
| `request` | `Function(Dio)` | HTTP operation (required) |
| `bearerToken` | `String?` | Bearer auth token |
| `baseUrl` | `String?` | Override service base URL |
| `headers` | `Map<String, dynamic>?` | Custom request headers |
| `retry` | `int?` | Number of retry attempts |
| `retryDelay` | `Duration?` | Delay between retries |
| `retryIf` | `bool Function(DioException)?` | Conditional retry |
| `cacheKey` | `String?` | Cache identifier |
| `cacheDuration` | `Duration?` | Cache TTL |
| `cachePolicy` | `CachePolicy?` | Caching strategy |
| `checkConnectivity` | `bool?` | Check network before request |
| `handleSuccess` | `Function(NyResponse<T>)?` | Success callback |
| `handleFailure` | `Function(NyResponse<T>)?` | Failure callback |

### networkResponse() Method

Returns `NyResponse<T>` with full response details:

```dart
Future<NyResponse<User>> fetchUser(int id) async {
  return await networkResponse<User>(
    request: (request) => request.get("/users/$id"),
  );
}

// Usage
NyResponse<User> response = await apiService.fetchUser(1);
if (response.isSuccessful) {
  User? user = response.data;
  print('Status: ${response.statusCode}');
} else {
  print('Error: ${response.errorMessage}');
}
```

## NyResponse Properties and Methods

### Status Checks

| Property | Checks |
|----------|--------|
| `isSuccessful` | 200-299 |
| `isClientError` | 400-499 |
| `isServerError` | 500-599 |
| `isRedirect` | 300-399 |
| `isUnauthorized` | 401 |
| `isForbidden` | 403 |
| `isNotFound` | 404 |
| `isTimeout` | 408 |
| `isRateLimited` | 429 |
| `hasData` | Data is not null |

### Data Access

| Property/Method | Returns |
|-----------------|---------|
| `data` | Decoded model instance (`T?`) |
| `rawData` | Unprocessed response content |
| `statusCode` | HTTP status code |
| `headers` | Response headers |
| `errorMessage` | Error description |

### Helper Methods

```dart
// Get data or throw
User user = response.dataOrThrow('User not found');

// Get data with fallback
User user = response.dataOr(User.guest());

// Execute on success
String? greeting = response.ifSuccessful((user) => 'Hello ${user.name}');

// Pattern matching
String result = response.when(
  success: (user) => 'Welcome, ${user.name}!',
  failure: (response) => 'Error: ${response.statusMessage}',
);

// Extract specific header
String? auth = response.getHeader('Authorization');
```

## Using API Services from Pages

### Direct Instantiation

```dart
class _MyPageState extends NyPage<MyPage> {
  ApiService _apiService = ApiService();

  @override
  get init => () async {
    List<User>? users = await _apiService.fetchUsers();
  };
}
```

### api() Helper

```dart
class _MyPageState extends NyPage<MyPage> {
  @override
  get init => () async {
    User? user = await api<ApiService>((request) => request.fetchUser(1));
  };
}
```

### api() with Callbacks

```dart
await api<ApiService>(
  (request) => request.fetchUser(1),
  onSuccess: (response, data) {
    // data is the morphed User instance
  },
  onError: (DioException err) {
    // Handle error
  },
  cacheKey: "user_1",
  cacheDuration: Duration(minutes: 30),
);
```

## Model Decoders

Register JSON-to-model mappings in `lib/bootstrap/decoders.dart`:

```dart
final Map<Type, dynamic> modelDecoders = {
  User: (data) => User.fromJson(data),
  List<User>: (data) =>
      List.from(data).map((json) => User.fromJson(json)).toList(),
};
```

The type argument in `network<User>()` matches the decoder key to automatically morph JSON responses into model instances.

## Interceptors

Interceptors modify all requests/responses globally. Located in `lib/app/networking/dio/interceptors/`.

### Register Interceptors

```dart
@override
Map<Type, Interceptor> get interceptors => {
  ...super.interceptors,
  BearerAuthInterceptor: BearerAuthInterceptor(),
  LoggingInterceptor: LoggingInterceptor(),
};
```

### Create Custom Interceptor

```bash
metro make:interceptor logging
```

```dart
class LoggingInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    print('REQUEST[${options.method}] => PATH: ${options.path}');
    super.onRequest(options, handler);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    print('RESPONSE[${response.statusCode}] => PATH: ${response.requestOptions.path}');
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    print('ERROR[${err.response?.statusCode}] => PATH: ${err.requestOptions.path}');
    handler.next(err);
  }
}
```

## Caching

### Basic Caching

```dart
Future<List<Country>?> fetchCountries() async {
  return await network<List<Country>>(
    request: (request) => request.get("/countries"),
    cacheKey: "app_countries",
    cacheDuration: Duration(hours: 1),
  );
}
```

### Cache Policies

| Policy | Behavior | Use When |
|--------|----------|----------|
| `CachePolicy.networkOnly` | Always fetch from network | Data must be real-time |
| `CachePolicy.cacheFirst` | Use cache, fallback to network | Rarely-changing data, prioritize speed |
| `CachePolicy.networkFirst` | Try network, fallback to cache | Must-be-fresh data, offline fallback |
| `CachePolicy.cacheOnly` | Cache only, error if empty | Offline-only mode |
| `CachePolicy.staleWhileRevalidate` | Return cache immediately, refresh in background | Need instant UI, accept brief staleness |

```dart
return await network<List<Country>>(
  request: (request) => request.get("/countries"),
  cacheKey: "app_countries",
  cacheDuration: Duration(hours: 1),
  cachePolicy: CachePolicy.staleWhileRevalidate,
);
```

### Clear Cache

```dart
await apiService.clearCache("app_countries"); // Specific key
await apiService.clearAllCache();             // All cached responses
```

## Authentication Headers

### Auto-Attach Auth Headers

```dart
class ApiService extends NyApiService {
  @override
  Future<RequestHeaders> setAuthHeaders(RequestHeaders headers) async {
    String? token = Auth.data(field: 'token');
    if (token != null) {
      headers.addBearerToken(token);
    }
    headers.addHeader('X-App-Version', '1.0.0');
    return headers;
  }
}
```

### Skip Auth Headers for Public Endpoints

```dart
await network(
  request: (request) => request.get("/public-endpoint"),
  shouldSetAuthHeaders: false,
);
```

## Token Refresh

```dart
class ApiService extends NyApiService {
  @override
  Future<bool> shouldRefreshToken() async {
    // Check if token is expired
    return false;
  }

  @override
  Future<void> refreshToken(Dio dio) async {
    // dio is a fresh instance without interceptors (prevents loops)
    dynamic response = (await dio.post(
      "https://example.com/refresh-token"
    )).data;

    await Auth.set((data) {
      data['token'] = response['token'];
      return data;
    });
  }
}
```

## Retry Logic

```dart
// Basic retry
await network(
  request: (request) => request.get("/users"),
  retry: 3,
);

// With delay
await network(
  request: (request) => request.get("/users"),
  retry: 3,
  retryDelay: Duration(seconds: 2),
);

// Conditional retry (only on 500 errors)
await network(
  request: (request) => request.get("/users"),
  retry: 3,
  retryIf: (DioException err) {
    return err.response?.statusCode == 500;
  },
);
```

## File Operations

### Upload

```dart
Future<UploadResponse?> uploadAvatar(String filePath) async {
  return await upload<UploadResponse>(
    '/upload',
    filePath: filePath,
    fieldName: 'avatar',
    additionalFields: {'userId': '123'},
    onProgress: (sent, total) {
      print('Progress: ${(sent / total * 100).toStringAsFixed(0)}%');
    },
  );
}

// Multiple files
Future<UploadResponse?> uploadDocuments() async {
  return await uploadMultiple<UploadResponse>(
    '/upload',
    files: {
      'avatar': '/path/to/avatar.jpg',
      'document': '/path/to/doc.pdf',
    },
  );
}
```

### Download

```dart
await download(
  url,
  savePath: savePath,
  onProgress: (received, total) {
    if (total != -1) {
      print('Progress: ${(received / total * 100).toStringAsFixed(0)}%');
    }
  },
  deleteOnError: true,
);
```

## Timeouts and Base Options

```dart
class ApiService extends NyApiService {
  ApiService({BuildContext? buildContext}) : super(
    buildContext,
    decoders: modelDecoders,
    baseOptions: (BaseOptions opts) {
      return opts
        ..connectTimeout = Duration(seconds: 5)
        ..sendTimeout = Duration(seconds: 5)
        ..receiveTimeout = Duration(seconds: 5);
    },
  );
}
```

Runtime adjustments:

```dart
apiService.setConnectTimeout(Duration(seconds: 10));
apiService.setReceiveTimeout(Duration(seconds: 30));
apiService.setSendTimeout(Duration(seconds: 10));
```

## Connectivity Checks

Fail fast when offline instead of waiting for timeout:

```dart
// Service-level
class ApiService extends NyApiService {
  @override
  bool get checkConnectivityBeforeRequest => true;
}

// Per-request
await network(
  request: (request) => request.get("/users"),
  checkConnectivity: true,
);
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting to register model in `decoders.dart` | Add both `User` and `List<User>` decoder entries for the model type |
| Using `network()` when you need status codes | Use `networkResponse()` to get `NyResponse<T>` with `statusCode`, `headers`, etc. |
| Token refresh interceptor loop | The `refreshToken()` method receives a fresh Dio instance without interceptors; always use that instance |
| Not handling null return from `network()` | `network<T>()` returns `T?`; always handle the null case |
| Caching without a `cacheKey` | Both `cacheKey` and `cacheDuration` are required for caching to work |
| Using `cacheOnly` without pre-populating cache | `cacheOnly` throws an error if no cached data exists; use `cacheFirst` or `networkFirst` for initial requests |
| Hardcoding base URL | Use `getEnv('API_BASE_URL')` from `.env` file for environment-specific configuration |
| Not calling `handler.next()` in interceptors | Always call `handler.next(response)` or `super.onRequest(options, handler)` to continue the chain |
