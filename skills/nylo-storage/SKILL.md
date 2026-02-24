---
name: nylo-storage
description: Use when storing, reading, or deleting persistent data with NyStorage, using in-memory Backpack for fast synchronous access, caching data with expiration, managing collections, working with sessions, or implementing dual storage patterns in a Nylo v7 Flutter app.
---

# Nylo Storage

## Overview

Nylo v7 provides three storage mechanisms: `NyStorage` for secure persistent storage (backed by flutter_secure_storage), `Backpack` for fast in-memory synchronous access, and `cache()` for time-based caching. They can be combined for hybrid patterns where data is persisted and also available synchronously.

## When to Use

- Persisting data across app restarts (NyStorage)
- Fast synchronous reads during runtime (Backpack)
- Caching API responses or computed data with expiration (cache)
- Storing collections/lists of items
- Managing temporary session data (sessions)
- Dual storage: persistent + in-memory access
- When NOT to use: For authentication data specifically, use nylo-auth; for API response caching, see nylo-networking

## Quick Reference

| Action | Code |
|--------|------|
| Save to storage | `await NyStorage.save("key", value)` |
| Read from storage | `await NyStorage.read<String>("key")` |
| Delete from storage | `await NyStorage.delete("key")` |
| Save JSON | `await NyStorage.saveJson("key", map)` |
| Read JSON | `await NyStorage.readJson("key")` |
| Save with TTL | `await NyStorage.saveWithExpiry("key", val, ttl: Duration(hours: 1))` |
| Save to Backpack | `backpackSave("key", value)` |
| Read from Backpack | `backpackRead<String>("key")` |
| Save to both | `await NyStorage.save("key", value, inBackpack: true)` |
| Cache with expiry | `await cache().saveRemember("key", 300, () async => data)` |
| Cache forever | `await cache().saveForever("key", () async => data)` |
| Read cache | `await cache().get<String>("key")` |
| Clear cache | `await cache().clear("key")` |
| Add to collection | `await NyStorage.addToCollection("key", item: value)` |
| Read collection | `await NyStorage.readCollection<String>("key")` |

## NyStorage (Persistent Storage)

Stores data securely on device using flutter_secure_storage under the hood.

### Basic Operations

```dart
// Save
await NyStorage.save("coins", 100);
await NyStorage.save("username", "Anthony");

// Read with type
int? coins = await NyStorage.read<int>("coins");
String? username = await NyStorage.read<String>("username");

// Read with default value
String name = await NyStorage.read("name", defaultValue: "Guest") ?? "Guest";

// Delete
await NyStorage.delete("username");
await NyStorage.deleteMultiple(["key1", "key2", "key3"]);
await NyStorage.deleteAll();
await NyStorage.deleteAll(excludeKeys: ["auth_token"]);
```

### JSON Storage

```dart
Map<String, dynamic> user = {
  "name": "Anthony",
  "email": "anthony@example.com",
  "preferences": {"theme": "dark"},
};
await NyStorage.saveJson("user_data", user);
Map<String, dynamic>? userData = await NyStorage.readJson("user_data");
```

### TTL (Time-to-Live)

Values expire automatically after a duration:

```dart
await NyStorage.saveWithExpiry(
  "session_token",
  "abc123",
  ttl: Duration(hours: 1),
);

String? token = await NyStorage.readWithExpiry<String>("session_token");
Duration? remaining = await NyStorage.getTimeToLive("session_token");
int removed = await NyStorage.removeExpired();
```

### Batch Operations

```dart
// Save multiple
await NyStorage.saveAll({
  "username": "Anthony",
  "score": 1500,
  "level": 10,
});

// Read multiple
Map<String, dynamic?> values = await NyStorage.readMultiple<dynamic>([
  "username", "score", "level",
]);
```

### Collections

Store and manage lists under a single key:

```dart
// Add to collection
await NyStorage.addToCollection("favorites", item: "Product A");
await NyStorage.addToCollection("cart_ids", item: 123, allowDuplicates: false);

// Save entire collection
await NyStorage.saveCollection<int>("cart_ids", [1, 2, 3, 4, 5]);

// Read collection
List<String> favorites = await NyStorage.readCollection<String>("favorites");
bool isEmpty = await NyStorage.isCollectionEmpty("cart_ids");

// Update by index
await NyStorage.updateCollectionByIndex<int>(
  0,
  (item) => item + 10,
  key: "scores",
);

// Update by condition
await NyStorage.updateCollectionWhere<Map<String, dynamic>>(
  (item) => item['id'] == 5,
  key: "products",
  update: (item) {
    item['quantity'] = item['quantity'] + 1;
    return item;
  },
);

// Delete from collection
await NyStorage.deleteFromCollection<String>(0, key: "favorites");
await NyStorage.deleteValueFromCollection<int>("cart_ids", value: 123);
await NyStorage.deleteFromCollectionWhere<Map<String, dynamic>>(
  (item) => item['expired'] == true,
  key: "coupons",
);
```

### Model Storage

Models extending `Model` have built-in storage methods:

```dart
class User extends Model {
  String? name;
  String? email;

  static StorageKey key = 'user';
  User() : super(key: key);

  User.fromJson(dynamic data) : super(key: key) {
    name = data['name'];
    email = data['email'];
  }

  @override
  Map<String, dynamic> toJson() => {"name": name, "email": email};
}

// Save model
User user = User();
user.name = "Anthony";
await user.save();
await user.save(inBackpack: true); // Also save to Backpack

// Read model
User? user = await NyStorage.read<User>(User.key);

// Save to collection
await user.saveToCollection();
List<User> users = await NyStorage.readCollection<User>(User.key);
```

## Storage Keys Configuration

Define keys in `lib/config/storage_keys.dart`:

```dart
final class StorageKeysConfig {
  static StorageKey auth = 'SK_AUTH';
  static StorageKey coins = 'SK_COINS';
  static StorageKey themePreference = 'SK_THEME_PREFERENCE';

  static syncedOnBoot() => () async {
    return [
      coins.defaultValue(0),
      themePreference.defaultValue('light'),
    ];
  };
}
```

### StorageKey Extension Methods

```dart
// Save/Read
await StorageKeysConfig.coins.save(100);
await StorageKeysConfig.coins.save(100, inBackpack: true);
int? coins = await StorageKeysConfig.coins.read<int>();
int? coins = await StorageKeysConfig.coins.fromStorage<int>(defaultValue: 0);

// JSON
await StorageKeysConfig.coins.saveJson({"gold": 50});
Map? data = await StorageKeysConfig.coins.readJson<Map>();

// Delete
await StorageKeysConfig.coins.deleteFromStorage();
await StorageKeysConfig.coins.flush();

// Backpack (synchronous)
int? coins = StorageKeysConfig.coins.fromBackpack<int>();

// Collections
await StorageKeysConfig.coins.addToCollection<int>(100);
List<int> allCoins = await StorageKeysConfig.coins.readCollection<int>();
```

## Backpack (In-Memory Storage)

Backpack is a singleton in-memory store for fast synchronous access. Data is NOT persisted when the app closes.

### Basic Operations

```dart
// Save
backpackSave("user_token", "abc123");
backpackSave("user", userObject);
Backpack.instance.save("settings", {"darkMode": true});

// Read (synchronous)
String? token = backpackRead<String>("user_token");
User? user = backpackRead<User>("user");
var settings = Backpack.instance.read("settings");

// Read with default
String name = Backpack.instance.read<String>("name", defaultValue: "Guest") ?? "Guest";

// Check existence
bool hasTheme = Backpack.instance.contains("theme");

// Delete
backpackDelete("user_token");
backpackDeleteAll();
```

### Append to Lists

```dart
Backpack.instance.append("recent_searches", "Flutter");
Backpack.instance.append("recent_searches", "Dart");
Backpack.instance.append("recent_searches", "Nylo", limit: 10);
```

### Backpack Sessions

Organize related data into named groups:

```dart
// Update session values
Backpack.instance.sessionUpdate("cart", "item_count", 3);
Backpack.instance.sessionUpdate("cart", "total", 29.99);

// Read session values
int? itemCount = Backpack.instance.sessionGet<int>("cart", "item_count");
double? total = Backpack.instance.sessionGet<double>("cart", "total");

// Get all session data
Map<String, dynamic>? cartData = Backpack.instance.sessionData("cart");

// Remove individual key from session
Backpack.instance.sessionRemove("cart", "item_count");

// Clear entire session
Backpack.instance.sessionFlush("cart");
```

## Dual Storage Pattern

Save to both persistent storage and Backpack simultaneously:

```dart
// Save to both at once
await NyStorage.save("auth_token", "abc123", inBackpack: true);

// Read synchronously from Backpack (fast)
String? token = Backpack.instance.read("auth_token");

// Read asynchronously from storage (persistent)
String? token = await NyStorage.read("auth_token");
```

### Practical Example

```dart
Future<void> handleLogin(String token) async {
  // Persist and cache in memory
  await NyStorage.save("auth_token", token, inBackpack: true);
}

class ApiService extends NyApiService {
  Future<User?> getProfile() async {
    return await network<User>(
      request: (request) => request.get("/profile"),
      bearerToken: backpackRead("auth_token"), // synchronous access
    );
  }
}
```

### Sync Storage to Backpack

```dart
await NyStorage.syncToBackpack();
await NyStorage.syncToBackpack(overwrite: true);
```

## Sessions (Named In-Memory Groups)

Sessions group temporary data without persistence:

```dart
// Set session data
session('checkout')
    .add('items', ['Product A', 'Product B'])
    .add('total', 99.99)
    .add('coupon', 'SAVE10');

// Or set all at once
session('checkout', {
  'items': ['Product A', 'Product B'],
  'total': 99.99,
});

// Read session data
List<String>? items = session('checkout').get<List<String>>('items');
double? total = session('checkout').get<double>('total');
Map<String, dynamic>? allData = session('checkout').data();

// Delete from session
session('checkout').delete('coupon');
session('checkout').clear();

// Persist session to storage
await session('checkout').syncToStorage();
await session('checkout').syncFromStorage();
```

## Cache (Time-Based Caching)

### Cache with Expiration

```dart
// Cache for 300 seconds (5 minutes)
// Callback only executes on cache miss
Map<String, dynamic>? profile = await cache().saveRemember("profile", 300, () async {
  return await api<UserApiService>((r) => r.getProfile());
});
```

### Cache Forever

```dart
Map<String, dynamic>? config = await cache().saveForever("config", () async {
  return await api<ConfigApiService>((r) => r.getConfig());
});
```

### Direct Cache Storage

```dart
await cache().put("session_token", "abc123", seconds: 3600);
await cache().put("device_id", "xyz789"); // No expiration
```

### Read Cache

```dart
String? value = await cache().get<String>("my_key");
Map<String, dynamic>? data = await cache().get<Map<String, dynamic>>("user_data");
```

Returns `null` if expired or absent, and automatically removes expired items.

### Cache Management

```dart
await cache().clear("my_key");          // Remove single item
await cache().flush();                   // Remove all cached items
bool exists = await cache().has("my_key");
List<String> keys = await cache().documents();
int sizeInBytes = await cache().size();
```

### Platform Support

Cache is available on iOS, Android, macOS, Windows, and Linux. NOT available on Web (callbacks always execute).

```dart
if (cache().isAvailable) {
  // Use caching
}
```

## Storage Exceptions

| Exception | Meaning |
|-----------|---------|
| `StorageException` | Base exception with message/key |
| `StorageSerializationException` | Object serialization failed |
| `StorageDeserializationException` | Data deserialization failed |
| `StorageKeyNotFoundException` | Key not found |
| `StorageTimeoutException` | Operation exceeded time limit |

```dart
try {
  await NyStorage.read<User>("user");
} on StorageDeserializationException catch (e) {
  print("Failed to load user: ${e.message}");
  print("Expected type: ${e.expectedType}");
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Reading from Backpack after app restart without sync | Backpack is in-memory only; call `NyStorage.syncToBackpack()` on boot or use `inBackpack: true` when saving |
| Using `cache().saveRemember()` on web | Cache is not available on web platform; check `cache().isAvailable` first |
| Forgetting to register Model decoders | For `NyStorage.read<User>()` to work, the model must have a `fromJson` constructor and be registered in decoders |
| Using NyStorage for frequent synchronous reads | NyStorage is async; use Backpack for synchronous reads in hot paths like interceptors |
| Not handling null returns from `NyStorage.read()` | All read methods return nullable types; always handle the null case |
| Storing large objects in Backpack | Backpack lives in memory; store only what you need for quick access, not entire datasets |
| Using `cache().get()` without checking expiry | `get()` automatically removes expired items and returns null; always handle the null case |
| Confusing `cache()` with NyApiService caching | `cache()` is the standalone cache system; `cacheKey`/`cacheDuration` in `network()` is API-level caching |
