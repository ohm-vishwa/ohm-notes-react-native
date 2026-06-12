# react-native-mmkv — Complete Reference

A comprehensive reference for `react-native-mmkv` v3 (latest).  
Covers storage instances, all read/write APIs, React hooks, encryption, multi-instance, state management integrations, and common patterns.

> **Install:**
> 
> ```bash
> # Expo
> npx expo install react-native-mmkv
> 
> # Bare React Native
> npm install react-native-mmkv
> cd ios && pod install
> ```

---

## Table of Contents

1. [Overview — Why MMKV](#overview--why-mmkv)
2. [Creating a Storage Instance](#creating-a-storage-instance)
3. [Writing Values](#writing-values)
4. [Reading Values](#reading-values)
5. [Checking & Deleting](#checking--deleting)
6. [Encryption](#encryption)
7. [React Hooks](#react-hooks)
8. [useMMKVString](#usemmkvstring)
9. [useMMKVNumber](#usemmkvnumber)
10. [useMMKVBoolean](#usemmkvboolean)
11. [useMMKVObject](#usemmkvobject)
12. [useMMKVBuffer](#usemmkvbuffer)
13. [useMMKVListener](#usemmkvlistener)
14. [Multi-Instance Storage](#multi-instance-storage)
15. [TypeScript — Typed Storage](#typescript--typed-storage)
16. [Zustand Integration](#zustand-integration)
17. [Redux Persist Integration](#redux-persist-integration)
18. [Jotai Integration](#jotai-integration)
19. [Common Patterns](#common-patterns)

---

## Overview — Why MMKV

<details> <summary>MMKV vs AsyncStorage — key differences</summary>

MMKV is a key-value storage framework by WeChat, backed by memory-mapped files. It is **synchronous**, **extremely fast**, and **type-safe** — making it the recommended replacement for `AsyncStorage` in production apps.

|Feature|AsyncStorage|MMKV|
|---|---|---|
|Thread|Async (JS thread)|Synchronous (JSI)|
|Speed|~1ms read|~0.01ms read (100× faster)|
|Encoding|JSON strings only|Native types (string, number, boolean, binary)|
|Max size|~6MB (Android)|Virtually unlimited|
|Encryption|❌|✅ AES-256|
|Multi-instance|❌|✅|
|React hooks|❌|✅ Built-in|
|Type safety|❌|✅|
|Persistence|✅|✅|
|Web support|✅|❌ Native only|

> ✅ Use MMKV for: tokens, user preferences, settings, cached data, session state.  
> ⚠️ MMKV requires JSI — not available in Expo Go. Use a development build.

</details>

---

## Creating a Storage Instance

<details> <summary>Default instance — singleton</summary>

The simplest setup. Creates a shared default storage instance. Fine for most apps.

```ts
import { MMKV } from 'react-native-mmkv';

// Create a default instance (id: 'mmkv.default')
export const storage = new MMKV();
```

> ✅ Create the instance **outside** of React components — at module level — so it's a stable singleton.

</details> <details> <summary><code>id</code> — named instance</summary>

Give the storage a custom identifier. Different IDs are completely isolated — separate files on disk.

|||
|---|---|
|**Type**|`string`|
|**Default**|`'mmkv.default'`|

```ts
// Per-feature storages
export const authStorage     = new MMKV({ id: 'auth' });
export const settingsStorage = new MMKV({ id: 'settings' });
export const cacheStorage    = new MMKV({ id: 'cache' });

// Per-user storage
const userId = 'user_abc123';
export const userStorage = new MMKV({ id: `user.${userId}` });
```

</details> <details> <summary><code>path</code> — custom storage directory</summary>

Override where MMKV writes its data file on disk.

|||
|---|---|
|**Type**|`string`|
|**Default**|App's default data directory|

```ts
import { MMKV } from 'react-native-mmkv';
import { DocumentDirectoryPath } from 'react-native-fs'; // optional

export const storage = new MMKV({
  id: 'secure-data',
  path: `${DocumentDirectoryPath}/mmkv`,
});
```

</details> <details> <summary><code>encryptionKey</code> — AES-256 encrypted storage</summary>

Encrypts all stored data with AES-256. The key is a string used to derive the encryption secret.

|||
|---|---|
|**Type**|`string`|
|**Default**|`undefined` (no encryption)|

```ts
import { MMKV } from 'react-native-mmkv';

export const secureStorage = new MMKV({
  id: 'secure',
  encryptionKey: 'my-super-secret-key-32-chars-long',
});

// Store sensitive data — transparently encrypted on disk
secureStorage.set('auth.token', accessToken);
secureStorage.set('auth.refresh', refreshToken);
```

> 🔐 In production, generate the encryption key using `expo-crypto` or `react-native-keychain` and store it in the device's secure enclave — not hardcoded in source.

</details> <details> <summary><code>accessGroup</code> — iOS App Groups</summary>

Shares storage between an iOS app and its extensions (widgets, share extensions) using an App Group.

|||
|---|---|
|**Type**|`string`|
|**Platform**|iOS|

```ts
export const sharedStorage = new MMKV({
  id: 'shared',
  accessGroup: 'group.com.mycompany.myapp', // must match Xcode entitlements
});
```

</details> <details> <summary>Full constructor options reference</summary>

```ts
const storage = new MMKV({
  id: 'my-storage',                    // unique name → separate file
  path: '/custom/path',                // custom directory
  encryptionKey: 'aes-256-key',        // encrypt all data
  accessGroup: 'group.com.example',    // iOS App Groups / extensions
});
```

</details>

---

## Writing Values

<details> <summary><code>storage.set(key, value)</code> — string</summary>

Stores a string value. Synchronous — no await needed.

|||
|---|---|
|**Type**|`(key: string, value: string) => void`|

```ts
storage.set('user.name', 'Alice');
storage.set('auth.token', 'eyJhbGci...');
storage.set('app.theme', 'dark');
storage.set('last.url', 'https://example.com/page');
```

</details> <details> <summary><code>storage.set(key, value)</code> — number</summary>

Stores a number (integer or float) natively — no JSON serialization.

|||
|---|---|
|**Type**|`(key: string, value: number) => void`|

```ts
storage.set('user.age', 28);
storage.set('settings.volume', 0.75);
storage.set('cart.itemCount', 3);
storage.set('session.expiresAt', Date.now() + 3600_000);
```

</details> <details> <summary><code>storage.set(key, value)</code> — boolean</summary>

Stores a boolean natively — no string conversion.

|||
|---|---|
|**Type**|`(key: string, value: boolean) => void`|

```ts
storage.set('user.isPremium', true);
storage.set('onboarding.completed', true);
storage.set('notifications.enabled', false);
storage.set('app.darkMode', true);
```

</details> <details> <summary><code>storage.set(key, value)</code> — ArrayBuffer (binary)</summary>

Stores raw binary data. Useful for encrypted blobs, images, or binary protocols.

|||
|---|---|
|**Type**|`(key: string, value: ArrayBuffer) => void`|

```ts
const buffer = new ArrayBuffer(16);
const view = new Uint8Array(buffer);
view.fill(42);

storage.set('binary.data', buffer);
```

</details> <details> <summary>Storing objects — JSON stringify</summary>

MMKV doesn't have a native object type. Serialize objects to JSON strings manually, or use `useMMKVObject` hook which does it automatically.

```ts
// Manual JSON serialization
const user = { id: '123', name: 'Alice', role: 'admin' };
storage.set('user.profile', JSON.stringify(user));

// Retrieve
const raw = storage.getString('user.profile');
const parsed = raw ? JSON.parse(raw) : null;

// ✅ Or use the hook for automatic serialization (see useMMKVObject)
```

</details>

---

## Reading Values

<details> <summary><code>storage.getString(key)</code></summary>

Returns a string or `undefined` if the key doesn't exist.

|||
|---|---|
|**Type**|`(key: string) => string \| undefined`|

```ts
const name = storage.getString('user.name');
// 'Alice' | undefined

// With default value
const theme = storage.getString('app.theme') ?? 'light';

// Type-narrow
if (name !== undefined) {
  console.log(`Hello, ${name}`);
}
```

</details> <details> <summary><code>storage.getNumber(key)</code></summary>

Returns a number or `undefined` if the key doesn't exist.

|||
|---|---|
|**Type**|`(key: string) => number \| undefined`|

```ts
const age = storage.getNumber('user.age');
// 28 | undefined

const volume = storage.getNumber('settings.volume') ?? 1.0;
const expiresAt = storage.getNumber('session.expiresAt');

// Check token expiry
if (expiresAt && Date.now() > expiresAt) {
  storage.delete('auth.token');
}
```

</details> <details> <summary><code>storage.getBoolean(key)</code></summary>

Returns a boolean or `undefined` if the key doesn't exist.

|||
|---|---|
|**Type**|`(key: string) => boolean \| undefined`|

```ts
const isPremium = storage.getBoolean('user.isPremium');
// true | false | undefined

// With default
const darkMode = storage.getBoolean('app.darkMode') ?? false;
const onboarded = storage.getBoolean('onboarding.completed') ?? false;
```

</details> <details> <summary><code>storage.getBuffer(key)</code></summary>

Returns an `ArrayBuffer` or `undefined` if the key doesn't exist.

|||
|---|---|
|**Type**|`(key: string) => ArrayBuffer \| undefined`|

```ts
const buffer = storage.getBuffer('binary.data');
if (buffer) {
  const view = new Uint8Array(buffer);
  console.log(view[0]); // 42
}
```

</details>

---

## Checking & Deleting

<details> <summary><code>storage.contains(key)</code></summary>

Returns `true` if the key exists in storage (regardless of value type).

|||
|---|---|
|**Type**|`(key: string) => boolean`|

```ts
if (storage.contains('auth.token')) {
  // User has a stored token
  proceedToApp();
} else {
  navigateToLogin();
}
```

</details> <details> <summary><code>storage.delete(key)</code></summary>

Removes a single key and its value.

|||
|---|---|
|**Type**|`(key: string) => void`|

```ts
storage.delete('auth.token');
storage.delete('auth.refresh');
storage.delete('session.expiresAt');
```

</details> <details> <summary><code>storage.clearAll()</code></summary>

Removes **all** keys and values from this storage instance. Other instances are unaffected.

|||
|---|---|
|**Type**|`() => void`|

```ts
// On logout — clear all user data
storage.clearAll();

// Clear a specific instance only
authStorage.clearAll();    // only auth data
cacheStorage.clearAll();   // only cache
// settingsStorage is untouched
```

</details> <details> <summary><code>storage.getAllKeys()</code></summary>

Returns an array of all keys currently stored in this instance.

|||
|---|---|
|**Type**|`() => string[]`|

```ts
const keys = storage.getAllKeys();
// ['user.name', 'user.age', 'app.theme', ...]

// Delete all keys matching a prefix
const userKeys = storage.getAllKeys().filter(k => k.startsWith('user.'));
userKeys.forEach(key => storage.delete(key));

// Inspect storage contents (debug)
keys.forEach(key => {
  console.log(key, storage.getString(key));
});
```

</details> <details> <summary><code>storage.size</code></summary>

Returns the total size of the storage file in bytes.

|||
|---|---|
|**Type**|`number` (bytes)|

```ts
const bytes = storage.size;
const kb = (bytes / 1024).toFixed(2);
console.log(`Storage size: ${kb} KB`);
```

</details>

---

## Encryption

<details> <summary>Create an encrypted instance</summary>

Pass `encryptionKey` to the constructor. All values in this instance are transparently encrypted with AES-256 on disk.

```ts
import { MMKV } from 'react-native-mmkv';

export const secureStorage = new MMKV({
  id: 'secure',
  encryptionKey: process.env.STORAGE_KEY, // from environment / secure config
});

// Usage is identical to unencrypted storage
secureStorage.set('auth.token', token);
const token = secureStorage.getString('auth.token');
```

</details> <details> <summary><code>storage.recrypt(encryptionKey)</code> — change or remove encryption</summary>

Re-encrypts existing data with a new key. Pass `undefined` to remove encryption entirely.

|||
|---|---|
|**Type**|`(encryptionKey: string \| undefined) => void`|

```ts
// Change encryption key
storage.recrypt('new-stronger-key');

// Remove encryption (decrypt all data)
storage.recrypt(undefined);

// Add encryption to a previously unencrypted instance
storage.recrypt('my-new-key');
```

</details> <details> <summary>Production key management with react-native-keychain</summary>

Never hardcode encryption keys. Generate and store them in the device's secure enclave.

```ts
import * as Keychain from 'react-native-keychain';
import { MMKV } from 'react-native-mmkv';
import * as Crypto from 'expo-crypto';

async function getOrCreateEncryptionKey(): Promise<string> {
  const credentials = await Keychain.getGenericPassword({ service: 'mmkv-key' });

  if (credentials) {
    return credentials.password;
  }

  // Generate a new random key
  const key = await Crypto.digestStringAsync(
    Crypto.CryptoDigestAlgorithm.SHA256,
    `${Date.now()}-${Math.random()}`
  );

  await Keychain.setGenericPassword('mmkv', key, { service: 'mmkv-key' });
  return key;
}

// Initialize storage with the secure key
const encryptionKey = await getOrCreateEncryptionKey();
export const secureStorage = new MMKV({ id: 'secure', encryptionKey });
```

</details>

---

## React Hooks

<details> <summary><code>useMMKV(configuration?)</code> — get or create a storage instance in a component</summary>

Returns a stable MMKV instance. Use when you need the raw instance inside a component — for example to call `getAllKeys()`, `clearAll()`, or pass to other hooks.

|||
|---|---|
|**Returns**|`MMKV`|

```ts
import { useMMKV } from 'react-native-mmkv';

// Default instance
function MyComponent() {
  const storage = useMMKV();

  const clearCache = () => storage.clearAll();
  const keys = storage.getAllKeys();
}

// Named instance
function UserComponent() {
  const storage = useMMKV({ id: 'user-prefs' });
}

// Encrypted instance
function SecureComponent() {
  const storage = useMMKV({ id: 'secure', encryptionKey: 'my-key' });
}
```

> ✅ `useMMKV` returns the **same instance** on every render — it's stable and doesn't re-create the storage.

</details>

---

## useMMKVString

<details> <summary><code>useMMKVString(key, storage?)</code> — reactive string value</summary>

Returns `[value, setter]`. Re-renders the component whenever the value for `key` changes (from any part of the app).

|||
|---|---|
|**Returns**|`[string \| undefined, (value: string \| undefined) => void]`|

```ts
import { useMMKVString } from 'react-native-mmkv';

function ProfileScreen() {
  const [name, setName] = useMMKVString('user.name');
  const [token, setToken] = useMMKVString('auth.token');

  return (
    <View>
      <Text>Hello, {name ?? 'Guest'}</Text>
      <TextInput
        value={name}
        onChangeText={setName}       // auto-persists on every keystroke
        placeholder="Your name"
      />
      <Button title="Logout" onPress={() => setToken(undefined)} />
    </View>
  );
}

// With a specific storage instance
function SettingsScreen() {
  const storage = useMMKV({ id: 'settings' });
  const [theme, setTheme] = useMMKVString('app.theme', storage);
}
```

</details>

---

## useMMKVNumber

<details> <summary><code>useMMKVNumber(key, storage?)</code> — reactive number value</summary>

Returns `[value, setter]`. Re-renders when the number value changes.

|||
|---|---|
|**Returns**|`[number \| undefined, (value: number \| undefined) => void]`|

```ts
import { useMMKVNumber } from 'react-native-mmkv';

function VolumeControl() {
  const [volume, setVolume] = useMMKVNumber('settings.volume');

  return (
    <Slider
      value={volume ?? 1}
      onValueChange={setVolume}
      minimumValue={0}
      maximumValue={1}
    />
  );
}

function CartBadge() {
  const [count] = useMMKVNumber('cart.itemCount');
  return <Badge count={count ?? 0} />;
}
```

</details>

---

## useMMKVBoolean

<details> <summary><code>useMMKVBoolean(key, storage?)</code> — reactive boolean value</summary>

Returns `[value, setter]`. Re-renders when the boolean changes.

|||
|---|---|
|**Returns**|`[boolean \| undefined, (value: boolean \| undefined) => void]`|

```ts
import { useMMKVBoolean } from 'react-native-mmkv';

function DarkModeToggle() {
  const [isDark, setIsDark] = useMMKVBoolean('app.darkMode');

  return (
    <Switch
      value={isDark ?? false}
      onValueChange={setIsDark}
    />
  );
}

function OnboardingGuard({ children }) {
  const [completed] = useMMKVBoolean('onboarding.completed');

  if (!completed) {
    return <Redirect href="/onboarding/step-1" />;
  }

  return children;
}

function NotificationSettings() {
  const [pushEnabled, setPushEnabled]  = useMMKVBoolean('notifications.push');
  const [emailEnabled, setEmailEnabled] = useMMKVBoolean('notifications.email');

  return (
    <View>
      <Switch value={pushEnabled  ?? true}  onValueChange={setPushEnabled} />
      <Switch value={emailEnabled ?? true}  onValueChange={setEmailEnabled} />
    </View>
  );
}
```

</details>

---

## useMMKVObject

<details> <summary><code>useMMKVObject&lt;T&gt;(key, storage?)</code> — reactive typed object</summary>

Automatically serializes/deserializes JSON. Returns `[value, setter]`. Re-renders when the object changes.

|||
|---|---|
|**Returns**|`[T \| undefined, (value: T \| undefined) => void]`|

```ts
import { useMMKVObject } from 'react-native-mmkv';

interface UserProfile {
  id: string;
  name: string;
  email: string;
  avatar: string;
  role: 'admin' | 'user';
}

function ProfileCard() {
  const [profile, setProfile] = useMMKVObject<UserProfile>('user.profile');

  const updateName = (name: string) => {
    setProfile(prev => prev ? { ...prev, name } : undefined);
  };

  return (
    <View>
      <Text>{profile?.name ?? 'Loading...'}</Text>
      <Text>{profile?.email}</Text>
    </View>
  );
}

// Storing arrays
interface CartItem { id: string; qty: number; price: number }

function useCart() {
  const [items, setItems] = useMMKVObject<CartItem[]>('cart.items');

  const addItem = (item: CartItem) => {
    setItems(prev => [...(prev ?? []), item]);
  };

  const removeItem = (id: string) => {
    setItems(prev => prev?.filter(i => i.id !== id));
  };

  return { items: items ?? [], addItem, removeItem };
}
```

</details>

---

## useMMKVBuffer

<details> <summary><code>useMMKVBuffer(key, storage?)</code> — reactive binary data</summary>

Returns `[value, setter]` for `ArrayBuffer` values. Use for raw binary data.

|||
|---|---|
|**Returns**|`[ArrayBuffer \| undefined, (value: ArrayBuffer \| undefined) => void]`|

```ts
import { useMMKVBuffer } from 'react-native-mmkv';

function BinaryDataView() {
  const [data, setData] = useMMKVBuffer('binary.chunk');

  const saveData = (buffer: ArrayBuffer) => {
    setData(buffer);
  };

  return (
    <View>
      <Text>Buffer size: {data?.byteLength ?? 0} bytes</Text>
    </View>
  );
}
```

</details>

---

## useMMKVListener

<details> <summary><code>useMMKVListener(listener, storage?)</code> — listen to any key change</summary>

Calls the listener whenever **any** key in the storage changes. Useful for cross-component sync, logging, or reacting to external writes.

|||
|---|---|
|**Type**|`(onValueChanged: (key: string) => void, storage?: MMKV) => void`|

```ts
import { useMMKVListener, useMMKV } from 'react-native-mmkv';

// Listen on default storage
function StorageLogger() {
  useMMKVListener((key) => {
    console.log(`[MMKV] Key changed: ${key}`);
  });

  return null;
}

// Listen on a specific instance
function AuthMonitor() {
  const authStorage = useMMKV({ id: 'auth' });

  useMMKVListener((key) => {
    if (key === 'auth.token') {
      const token = authStorage.getString('auth.token');
      updateAuthHeader(token);
    }
  }, authStorage);

  return null;
}

// Track changes for analytics
function ChangeTracker() {
  useMMKVListener((key) => {
    analytics.track('storage_change', { key, timestamp: Date.now() });
  });
}
```

</details>

---

## Multi-Instance Storage

<details> <summary>Isolated storage per feature</summary>

Create separate instances for different concerns. Each instance is a separate file — clearing one doesn't affect others.

```ts
// storage/index.ts
import { MMKV } from 'react-native-mmkv';

// Separate, isolated storage instances
export const authStorage     = new MMKV({ id: 'auth' });
export const settingsStorage = new MMKV({ id: 'settings' });
export const cacheStorage    = new MMKV({ id: 'cache' });
export const onboardStorage  = new MMKV({ id: 'onboarding' });

// Encrypted instance for sensitive data only
export const secureStorage = new MMKV({
  id: 'secure',
  encryptionKey: ENCRYPTION_KEY,
});
```

```ts
// Logout — clear sensitive data, keep settings
export function logout() {
  authStorage.clearAll();    // tokens gone
  cacheStorage.clearAll();   // cache gone
  // settingsStorage untouched — user preferences stay
}
```

</details> <details> <summary>Per-user storage — isolated data per account</summary>

```ts
import { MMKV } from 'react-native-mmkv';

// Create a new storage instance scoped to the logged-in user
function createUserStorage(userId: string) {
  return new MMKV({ id: `user.${userId}` });
}

// auth/useUserStorage.ts
function useUserStorage() {
  const { userId } = useAuth();
  return useMemo(() => createUserStorage(userId), [userId]);
}

// Use in component
function PreferencesScreen() {
  const storage = useUserStorage();
  const storage2 = useMMKV({ id: `user.${userId}` });

  const [theme, setTheme] = useMMKVString('theme', storage2);
}
```

</details>

---

## TypeScript — Typed Storage

<details> <summary>Strongly-typed storage wrapper</summary>

Create a typed wrapper around MMKV to get autocomplete and type safety on all keys.

```ts
// storage/typed.ts
import { MMKV } from 'react-native-mmkv';

// Define all keys and their types
interface StorageSchema {
  'auth.token':             string;
  'auth.refresh':           string;
  'auth.expiresAt':         number;
  'user.id':                string;
  'user.name':              string;
  'user.email':             string;
  'user.isPremium':         boolean;
  'settings.theme':         'light' | 'dark' | 'system';
  'settings.language':      string;
  'settings.notifications': boolean;
  'onboarding.completed':   boolean;
}

class TypedMMKV {
  private storage: MMKV;

  constructor(config?: ConstructorParameters<typeof MMKV>[0]) {
    this.storage = new MMKV(config);
  }

  set<K extends keyof StorageSchema>(key: K, value: StorageSchema[K]): void {
    if (typeof value === 'boolean') {
      this.storage.set(key, value);
    } else if (typeof value === 'number') {
      this.storage.set(key, value);
    } else {
      this.storage.set(key, value as string);
    }
  }

  getString<K extends Extract<keyof StorageSchema, string>>(
    key: StorageSchema[K] extends string ? K : never
  ): StorageSchema[K] | undefined {
    return this.storage.getString(key) as StorageSchema[K] | undefined;
  }

  getBoolean<K extends Extract<keyof StorageSchema, string>>(
    key: StorageSchema[K] extends boolean ? K : never
  ): boolean | undefined {
    return this.storage.getBoolean(key);
  }

  getNumber<K extends Extract<keyof StorageSchema, string>>(
    key: StorageSchema[K] extends number ? K : never
  ): number | undefined {
    return this.storage.getNumber(key);
  }

  delete<K extends keyof StorageSchema>(key: K): void {
    this.storage.delete(key);
  }

  contains<K extends keyof StorageSchema>(key: K): boolean {
    return this.storage.contains(key);
  }

  clearAll(): void {
    this.storage.clearAll();
  }
}

export const typedStorage = new TypedMMKV();

// Usage — fully typed, autocomplete on all keys
typedStorage.set('auth.token', 'eyJ...');         // ✅
typedStorage.set('user.isPremium', true);          // ✅
typedStorage.set('auth.expiresAt', Date.now());    // ✅
// typedStorage.set('auth.token', 123);            // ❌ TypeScript error
```

</details>

---

## Zustand Integration

<details> <summary>Zustand persist middleware with MMKV</summary>

Use MMKV as the storage backend for Zustand's `persist` middleware for a sync, fast, persisted store.

```ts
// store/storage.ts
import { MMKV } from 'react-native-mmkv';
import { StateStorage } from 'zustand/middleware';

const mmkv = new MMKV({ id: 'zustand' });

export const zustandStorage: StateStorage = {
  setItem: (key, value) => {
    mmkv.set(key, value);
  },
  getItem: (key) => {
    return mmkv.getString(key) ?? null;
  },
  removeItem: (key) => {
    mmkv.delete(key);
  },
};
```

```ts
// store/useAuthStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { zustandStorage } from './storage';

interface AuthState {
  token: string | null;
  userId: string | null;
  isSignedIn: boolean;
  signIn: (token: string, userId: string) => void;
  signOut: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      userId: null,
      isSignedIn: false,

      signIn: (token, userId) =>
        set({ token, userId, isSignedIn: true }),

      signOut: () =>
        set({ token: null, userId: null, isSignedIn: false }),
    }),
    {
      name: 'auth-store',
      storage: createJSONStorage(() => zustandStorage),
    }
  )
);
```

```ts
// store/useSettingsStore.ts
interface SettingsState {
  theme: 'light' | 'dark' | 'system';
  language: string;
  setTheme: (theme: SettingsState['theme']) => void;
  setLanguage: (lang: string) => void;
}

export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      theme: 'system',
      language: 'en',
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
    }),
    {
      name: 'settings-store',
      storage: createJSONStorage(() => zustandStorage),
      // Only persist specific fields
      partialize: (state) => ({
        theme: state.theme,
        language: state.language,
      }),
    }
  )
);
```

</details>

---

## Redux Persist Integration

<details> <summary>Redux Persist storage adapter</summary>

Replace Redux Persist's default `AsyncStorage` with MMKV for synchronous, faster state rehydration.

```ts
// store/mmkvStorage.ts
import { MMKV } from 'react-native-mmkv';
import { Storage } from 'redux-persist';

const mmkv = new MMKV({ id: 'redux' });

export const reduxMMKVStorage: Storage = {
  setItem: (key, value) => {
    mmkv.set(key, value);
    return Promise.resolve(true);
  },
  getItem: (key) => {
    const value = mmkv.getString(key);
    return Promise.resolve(value ?? null);
  },
  removeItem: (key) => {
    mmkv.delete(key);
    return Promise.resolve();
  },
};
```

```ts
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { persistStore, persistReducer } from 'redux-persist';
import { reduxMMKVStorage } from './mmkvStorage';
import rootReducer from './rootReducer';

const persistConfig = {
  key: 'root',
  storage: reduxMMKVStorage,
  whitelist: ['auth', 'settings'], // only persist these slices
};

const persistedReducer = persistReducer(persistConfig, rootReducer);

export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({ serializableCheck: false }),
});

export const persistor = persistStore(store);
```

</details>

---

## Jotai Integration

<details> <summary>Jotai atom with MMKV persistence</summary>

```ts
// atoms/mmkvAtom.ts
import { atom, useAtom } from 'jotai';
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV({ id: 'jotai' });

// Factory — create a persisted atom for any key
function atomWithMMKV<T>(key: string, initialValue: T) {
  const getInitial = (): T => {
    if (typeof initialValue === 'boolean') {
      return (storage.getBoolean(key) ?? initialValue) as T;
    }
    if (typeof initialValue === 'number') {
      return (storage.getNumber(key) ?? initialValue) as T;
    }
    if (typeof initialValue === 'string') {
      return (storage.getString(key) ?? initialValue) as T;
    }
    const raw = storage.getString(key);
    return raw ? (JSON.parse(raw) as T) : initialValue;
  };

  const baseAtom = atom<T>(getInitial());

  const derivedAtom = atom(
    (get) => get(baseAtom),
    (get, set, update: T) => {
      set(baseAtom, update);
      if (typeof update === 'boolean') storage.set(key, update);
      else if (typeof update === 'number') storage.set(key, update);
      else if (typeof update === 'string') storage.set(key, update);
      else storage.set(key, JSON.stringify(update));
    }
  );

  return derivedAtom;
}

// Persisted atoms
export const themeAtom      = atomWithMMKV('app.theme', 'system');
export const languageAtom   = atomWithMMKV('app.language', 'en');
export const isPremiumAtom  = atomWithMMKV('user.isPremium', false);

// Usage in component
function SettingsScreen() {
  const [theme, setTheme] = useAtom(themeAtom);
  return <ThemePicker value={theme} onChange={setTheme} />;
}
```

</details>

---

## Common Patterns

<details> <summary>Auth token management</summary>

```ts
// auth/tokenStorage.ts
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV({ id: 'auth', encryptionKey: ENCRYPTION_KEY });

export const tokenStorage = {
  getAccessToken:  () => storage.getString('auth.accessToken'),
  getRefreshToken: () => storage.getString('auth.refreshToken'),
  getExpiresAt:    () => storage.getNumber('auth.expiresAt'),

  saveTokens: (access: string, refresh: string, expiresIn: number) => {
    storage.set('auth.accessToken', access);
    storage.set('auth.refreshToken', refresh);
    storage.set('auth.expiresAt', Date.now() + expiresIn * 1000);
  },

  clearTokens: () => {
    storage.delete('auth.accessToken');
    storage.delete('auth.refreshToken');
    storage.delete('auth.expiresAt');
  },

  isExpired: () => {
    const expiresAt = storage.getNumber('auth.expiresAt');
    return !expiresAt || Date.now() > expiresAt;
  },
};
```

</details> <details> <summary>User preferences hook</summary>

```ts
// hooks/usePreferences.ts
import { useMMKVString, useMMKVBoolean, useMMKV } from 'react-native-mmkv';

export function usePreferences() {
  const storage = useMMKV({ id: 'prefs' });

  const [theme, setTheme]               = useMMKVString('theme', storage);
  const [language, setLanguage]         = useMMKVString('language', storage);
  const [pushEnabled, setPushEnabled]   = useMMKVBoolean('push', storage);
  const [analytics, setAnalytics]       = useMMKVBoolean('analytics', storage);

  const reset = () => storage.clearAll();

  return {
    theme:        theme ?? 'system',
    language:     language ?? 'en',
    pushEnabled:  pushEnabled ?? true,
    analytics:    analytics ?? true,
    setTheme,
    setLanguage,
    setPushEnabled,
    setAnalytics,
    reset,
  };
}

// Usage
function SettingsScreen() {
  const { theme, setTheme, pushEnabled, setPushEnabled } = usePreferences();

  return (
    <View>
      <ThemePicker value={theme} onChange={setTheme} />
      <Switch value={pushEnabled} onValueChange={setPushEnabled} />
    </View>
  );
}
```

</details> <details> <summary>TTL-based cache</summary>

```ts
// utils/mmkvCache.ts
import { MMKV } from 'react-native-mmkv';

const cache = new MMKV({ id: 'cache' });

export const ttlCache = {
  set: <T>(key: string, value: T, ttlMs: number) => {
    cache.set(`data.${key}`, JSON.stringify(value));
    cache.set(`exp.${key}`, Date.now() + ttlMs);
  },

  get: <T>(key: string): T | null => {
    const expiresAt = cache.getNumber(`exp.${key}`);
    if (!expiresAt || Date.now() > expiresAt) {
      cache.delete(`data.${key}`);
      cache.delete(`exp.${key}`);
      return null;
    }
    const raw = cache.getString(`data.${key}`);
    return raw ? JSON.parse(raw) : null;
  },

  delete: (key: string) => {
    cache.delete(`data.${key}`);
    cache.delete(`exp.${key}`);
  },

  clearExpired: () => {
    const keys = cache.getAllKeys().filter(k => k.startsWith('exp.'));
    keys.forEach(expKey => {
      const expiresAt = cache.getNumber(expKey);
      if (expiresAt && Date.now() > expiresAt) {
        const dataKey = expKey.replace('exp.', 'data.');
        cache.delete(expKey);
        cache.delete(dataKey);
      }
    });
  },
};

// Usage
const TTL_5MIN = 5 * 60 * 1000;

async function getUserProfile(userId: string) {
  const cached = ttlCache.get<UserProfile>(`profile.${userId}`);
  if (cached) return cached;

  const profile = await api.getProfile(userId);
  ttlCache.set(`profile.${userId}`, profile, TTL_5MIN);
  return profile;
}
```

</details> <details> <summary>Migration from AsyncStorage</summary>

```ts
// utils/migrateStorage.ts
import AsyncStorage from '@react-native-async-storage/async-storage';
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();
const MIGRATION_KEY = 'migration.v1.done';

export async function migrateFromAsyncStorage() {
  // Skip if already migrated
  if (storage.getBoolean(MIGRATION_KEY)) return;

  const keys = await AsyncStorage.getAllKeys();
  const pairs = await AsyncStorage.multiGet(keys);

  pairs.forEach(([key, value]) => {
    if (value !== null) {
      storage.set(key, value); // store as string (parse later if needed)
    }
  });

  storage.set(MIGRATION_KEY, true);
  console.log(`Migrated ${pairs.length} keys from AsyncStorage to MMKV`);
}

// Call once in your root component or _layout.tsx
useEffect(() => {
  migrateFromAsyncStorage();
}, []);
```

</details> <details> <summary>Storage inspector — debug utility</summary>

```ts
// utils/storageInspector.ts
import { MMKV } from 'react-native-mmkv';

export function inspectStorage(storage: MMKV, label = 'MMKV') {
  const keys = storage.getAllKeys();
  const sizeKB = (storage.size / 1024).toFixed(2);

  console.group(`[${label}] ${keys.length} keys — ${sizeKB} KB`);
  keys.forEach(key => {
    const str  = storage.getString(key);
    const num  = storage.getNumber(key);
    const bool = storage.getBoolean(key);
    const val  = str ?? num ?? bool ?? '<binary>';
    console.log(`  ${key}:`, val);
  });
  console.groupEnd();
}

// Usage in development
if (__DEV__) {
  inspectStorage(storage, 'App Storage');
}
```

</details> <details> <summary>Delete all keys with a prefix</summary>

```ts
function deleteByPrefix(storage: MMKV, prefix: string) {
  const keys = storage.getAllKeys().filter(k => k.startsWith(prefix));
  keys.forEach(k => storage.delete(k));
  return keys.length; // number of deleted keys
}

// Usage
deleteByPrefix(storage, 'user.');   // delete all user data
deleteByPrefix(storage, 'cache.');  // clear all cache entries
deleteByPrefix(cacheStorage, `profile.${userId}.`);
```

</details> <details> <summary>Reactive settings across the app</summary>

```ts
// One storage instance — multiple components all react to changes
const settingsStorage = new MMKV({ id: 'settings' });

// Component A — changes the theme
function ThemeToggle() {
  const [isDark, setIsDark] = useMMKVBoolean('dark', settingsStorage);
  return <Switch value={isDark ?? false} onValueChange={setIsDark} />;
}

// Component B — somewhere else in the tree — auto re-renders when A changes it
function ThemedHeader() {
  const [isDark] = useMMKVBoolean('dark', settingsStorage);
  return (
    <View style={{ backgroundColor: isDark ? '#111' : '#fff' }}>
      <Text>My App</Text>
    </View>
  );
}
```

</details>

---

## Quick-Reference Cheatsheet

|API|Use case|
|---|---|
|`new MMKV()`|Default storage instance|
|`new MMKV({ id })`|Named / isolated instance|
|`new MMKV({ encryptionKey })`|AES-256 encrypted storage|
|`storage.set(key, string)`|Write a string|
|`storage.set(key, number)`|Write a number|
|`storage.set(key, boolean)`|Write a boolean|
|`storage.set(key, ArrayBuffer)`|Write binary data|
|`storage.getString(key)`|Read string or undefined|
|`storage.getNumber(key)`|Read number or undefined|
|`storage.getBoolean(key)`|Read boolean or undefined|
|`storage.getBuffer(key)`|Read ArrayBuffer or undefined|
|`storage.contains(key)`|Check if key exists|
|`storage.delete(key)`|Remove one key|
|`storage.clearAll()`|Remove all keys in this instance|
|`storage.getAllKeys()`|List all keys|
|`storage.size`|Size in bytes|
|`storage.recrypt(key)`|Change or remove encryption|
|`useMMKV(config?)`|Get storage instance in component|
|`useMMKVString(key, storage?)`|`[string, setter]` — reactive|
|`useMMKVNumber(key, storage?)`|`[number, setter]` — reactive|
|`useMMKVBoolean(key, storage?)`|`[boolean, setter]` — reactive|
|`useMMKVObject<T>(key, storage?)`|`[T, setter]` — auto JSON|
|`useMMKVBuffer(key, storage?)`|`[ArrayBuffer, setter]`|
|`useMMKVListener(fn, storage?)`|React to any key change|

---

_Reference based on `react-native-mmkv` v3. Always check the [official docs](https://github.com/mrousavy/react-native-mmkv) for the latest updates._