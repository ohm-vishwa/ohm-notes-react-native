# Expo Router — Complete Reference

A comprehensive reference for Expo Router v3+ (file-based routing for React Native & web).  
Covers file conventions, layout components, navigation hooks, typed routes, API routes, and common patterns.

> **Install:**
> 
> ```bash
> npx create-expo-app@latest my-app
> # or add to existing Expo app:
> npx expo install expo-router react-native-safe-area-context react-native-screens expo-linking expo-constants expo-status-bar
> ```

---

## Table of Contents

1. [Overview — How File-based Routing Works](#overview--how-file-based-routing-works)
2. [File & Directory Conventions](#file--directory-conventions)
3. [Special Files](#special-files)
4. [Layout Files — _layout.tsx](#layout-files--_layouttsx)
5. [Stack Navigator](#stack-navigator)
6. [Tabs Navigator](#tabs-navigator)
7. [Drawer Navigator](#drawer-navigator)
8. [Slot](#slot)
9. [Link Component](#link-component)
10. [Redirect Component](#redirect-component)
11. [useRouter Hook](#userouter-hook)
12. [useLocalSearchParams](#uselocalsearchparams)
13. [useGlobalSearchParams](#useglobalsearchparams)
14. [useSegments](#usesegments)
15. [usePathname](#usepathname)
16. [useNavigation](#usenavigation)
17. [useFocusEffect](#usefocuseffect)
18. [useRootNavigationState](#userootnavigationstate)
19. [Typed Routes](#typed-routes)
20. [API Routes (Server)](#api-routes-server)
21. [Error Boundaries](#error-boundaries)
22. [Loading States](#loading-states)
23. [Deep Linking & Linking Config](#deep-linking--linking-config)
24. [Authentication Pattern](#authentication-pattern)
25. [Common Patterns](#common-patterns)

---

## Overview — How File-based Routing Works

<details> <summary>Mental model — files become routes</summary>

Expo Router maps files in the `app/` directory to navigation routes automatically. Every file is a screen; every folder is a segment.

```
app/
├── _layout.tsx          → Root layout (wraps everything)
├── index.tsx            → Route: /
├── about.tsx            → Route: /about
├── profile/
│   ├── _layout.tsx      → Profile layout
│   ├── index.tsx        → Route: /profile
│   └── settings.tsx     → Route: /profile/settings
├── posts/
│   ├── index.tsx        → Route: /posts
│   └── [id].tsx         → Route: /posts/123  (dynamic)
├── (auth)/              → Route group — no URL segment
│   ├── login.tsx        → Route: /login
│   └── register.tsx     → Route: /register
└── +not-found.tsx       → 404 catch-all
```

> Expo Router is built on top of React Navigation — every layout is a navigator, every screen is a route. Deep linking, web URLs, and native navigation are unified.

</details> <details> <summary>app/ directory vs src/app/ directory</summary>

By default Expo Router looks for the `app/` folder at the project root. You can configure an `src/app/` layout:

```json
// app.json
{
  "expo": {
    "experiments": {
      "typedRoutes": true
    }
  }
}
```

```
project/
├── app/            ← default
│   └── index.tsx
└── src/
    └── app/        ← alternative (configure in package.json main field)
        └── index.tsx
```

</details>

---

## File & Directory Conventions

<details> <summary>Static routes — <code>filename.tsx</code></summary>

A plain file creates a static route. The filename becomes the URL path segment.

|File|Route|
|---|---|
|`app/index.tsx`|`/`|
|`app/about.tsx`|`/about`|
|`app/settings.tsx`|`/settings`|
|`app/blog/index.tsx`|`/blog`|
|`app/blog/post.tsx`|`/blog/post`|

```tsx
// app/about.tsx
import { View, Text } from 'react-native';

export default function AboutScreen() {
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>About Page</Text>
    </View>
  );
}
```

</details> <details> <summary>Dynamic routes — <code>[param].tsx</code></summary>

Square brackets create a dynamic route segment. The param name is accessible via `useLocalSearchParams`.

|File|Route|Params|
|---|---|---|
|`app/posts/[id].tsx`|`/posts/123`|`{ id: '123' }`|
|`app/users/[userId].tsx`|`/users/alice`|`{ userId: 'alice' }`|
|`app/[category]/[slug].tsx`|`/tech/expo-router`|`{ category: 'tech', slug: 'expo-router' }`|

```tsx
// app/posts/[id].tsx
import { useLocalSearchParams } from 'expo-router';
import { Text, View } from 'react-native';

export default function PostScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  return (
    <View style={{ flex: 1 }}>
      <Text>Post ID: {id}</Text>
    </View>
  );
}
```

</details> <details> <summary>Catch-all routes — <code>[...slug].tsx</code></summary>

Three dots before the param name match zero or more segments. The param is an array of strings.

|File|Route|Params|
|---|---|---|
|`app/docs/[...slug].tsx`|`/docs`|`{ slug: [] }`|
|`app/docs/[...slug].tsx`|`/docs/intro`|`{ slug: ['intro'] }`|
|`app/docs/[...slug].tsx`|`/docs/api/hooks`|`{ slug: ['api', 'hooks'] }`|

```tsx
// app/docs/[...slug].tsx
import { useLocalSearchParams } from 'expo-router';

export default function DocsPage() {
  const { slug } = useLocalSearchParams<{ slug: string[] }>();
  const path = Array.isArray(slug) ? slug.join('/') : slug;
  return <Text>Viewing: /docs/{path}</Text>;
}
```

</details> <details> <summary>Route groups — <code>(group)/</code></summary>

Parentheses around a folder name create a **route group** — the folder name is excluded from the URL. Used to share layouts between routes or to organise files without affecting URLs.

|File|Route|
|---|---|
|`app/(tabs)/home.tsx`|`/home` (not `/tabs/home`)|
|`app/(auth)/login.tsx`|`/login`|
|`app/(app)/profile.tsx`|`/profile`|

```
app/
├── _layout.tsx
├── (auth)/
│   ├── _layout.tsx      ← Auth layout (no header)
│   ├── login.tsx        → /login
│   └── register.tsx     → /register
└── (app)/
    ├── _layout.tsx      ← App layout (with tabs)
    ├── home.tsx         → /home
    └── profile.tsx      → /profile
```

</details>

---

## Special Files

<details> <summary><code>_layout.tsx</code> — layout / navigator wrapper</summary>

Every directory can have a `_layout.tsx` that wraps all sibling routes. Defines the navigator type (`Stack`, `Tabs`, `Drawer`). See [Layout Files](#layout-files--_layouttsx) for full details.

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return <Stack />;
}
```

</details> <details> <summary><code>+not-found.tsx</code> — 404 / unmatched routes</summary>

Rendered when no route matches the current URL. Works on both native (unmatched deep link) and web.

```tsx
// app/+not-found.tsx
import { Link, Stack } from 'expo-router';
import { View, Text } from 'react-native';

export default function NotFound() {
  return (
    <>
      <Stack.Screen options={{ title: 'Page Not Found' }} />
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text style={{ fontSize: 24 }}>404 — This page doesn't exist.</Text>
        <Link href="/" style={{ marginTop: 16, color: '#6366f1' }}>
          Go to Home
        </Link>
      </View>
    </>
  );
}
```

</details> <details> <summary><code>+html.tsx</code> — custom HTML shell (web only)</summary>

Customize the HTML `<head>` for the web build. Only rendered on web.

```tsx
// app/+html.tsx
import { ScrollViewStyleReset } from 'expo-router/html';

export default function Root({ children }) {
  return (
    <html lang="en">
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <ScrollViewStyleReset />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

</details> <details> <summary><code>+error.tsx</code> — error boundary for a route segment</summary>

Catches rendering errors in sibling routes. Works like React's `ErrorBoundary` but scoped to the route segment.

```tsx
// app/+error.tsx
import { useRouteError } from 'expo-router';
import { View, Text, Button } from 'react-native';

export default function ErrorBoundary({ error, retry }) {
  return (
    <View style={{ flex: 1, justifyContent: 'center', padding: 24 }}>
      <Text style={{ fontSize: 18, fontWeight: '600', marginBottom: 8 }}>
        Something went wrong
      </Text>
      <Text style={{ color: '#6b7280', marginBottom: 24 }}>{error.message}</Text>
      <Button title="Try again" onPress={retry} />
    </View>
  );
}
```

</details> <details> <summary><code>_sitemap.tsx</code> — development route listing</summary>

Auto-generated in development mode — shows all routes in the app. Available at `/_sitemap` when running in dev. No code needed; it's generated automatically.

```
# Accessible at:
http://localhost:8081/_sitemap
```

</details>

---

## Layout Files — _layout.tsx

<details> <summary>Root layout — <code>app/_layout.tsx</code></summary>

The root layout wraps the entire app. This is where you set up providers (theme, auth, redux, etc.) and define the root navigator type.

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { ThemeProvider } from '../context/theme';

export default function RootLayout() {
  return (
    <SafeAreaProvider>
      <ThemeProvider>
        <Stack
          screenOptions={{
            headerStyle: { backgroundColor: '#6366f1' },
            headerTintColor: '#fff',
          }}
        />
      </ThemeProvider>
    </SafeAreaProvider>
  );
}

// Optional: export unstable_settings to set the initial route
export const unstable_settings = {
  initialRouteName: 'index',
};
```

</details> <details> <summary>Nested layout — subdirectory <code>_layout.tsx</code></summary>

Each subdirectory can have its own layout defining a different navigator for that segment.

```tsx
// app/profile/_layout.tsx
import { Stack } from 'expo-router';

export default function ProfileLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Profile' }} />
      <Stack.Screen name="settings" options={{ title: 'Settings' }} />
      <Stack.Screen name="edit" options={{ title: 'Edit Profile', presentation: 'modal' }} />
    </Stack>
  );
}
```

</details>

---

## Stack Navigator

<details> <summary><code>Stack</code> — stack navigator in a layout</summary>

Creates a stack navigator where child routes are pushed on top of each other.

|Prop|Type|Description|
|---|---|---|
|`screenOptions`|`object \| function`|Default options for all screens|
|`initialRouteName`|`string`|First screen to show|

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function Layout() {
  return (
    <Stack
      screenOptions={{
        headerShown: true,
        animation: 'slide_from_right',
        headerStyle: { backgroundColor: '#fff' },
        headerShadowVisible: false,
      }}
    />
  );
}
```

</details> <details> <summary><code>Stack.Screen</code> — configure a specific screen from layout</summary>

Declare screen options for named routes inside the layout file.

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function Layout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home', headerShown: false }} />
      <Stack.Screen name="details" options={{ title: 'Details' }} />
      <Stack.Screen
        name="modal"
        options={{
          presentation: 'modal',
          headerLeft: () => <CloseButton />,
        }}
      />
    </Stack>
  );
}
```

</details> <details> <summary><code>Stack.Screen</code> — configure from inside the screen file</summary>

You can also set screen options directly inside the screen component using `<Stack.Screen>` as a child — no props are passed, just options.

```tsx
// app/details.tsx
import { Stack } from 'expo-router';
import { View, Text } from 'react-native';

export default function DetailsScreen() {
  return (
    <>
      {/* Sets header options for THIS screen */}
      <Stack.Screen
        options={{
          title: 'Detail View',
          headerRight: () => (
            <TouchableOpacity onPress={handleShare}>
              <Ionicons name="share-outline" size={22} />
            </TouchableOpacity>
          ),
        }}
      />
      <View style={{ flex: 1 }}>
        <Text>Detail content</Text>
      </View>
    </>
  );
}
```

</details>

---

## Tabs Navigator

<details> <summary><code>Tabs</code> — tab bar navigator</summary>

Creates a bottom tab navigator. Each tab corresponds to a route file in the same directory.

|Prop|Type|Description|
|---|---|---|
|`screenOptions`|`object \| function`|Default options for all tabs|
|`tabBar`|`function`|Custom tab bar component|
|`initialRouteName`|`string`|Initially selected tab|
|`backBehavior`|`enum`|Back button behavior|

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: '#6366f1',
        tabBarInactiveTintColor: '#9ca3af',
        tabBarStyle: {
          backgroundColor: '#fff',
          borderTopColor: '#e5e7eb',
        },
        headerShown: false,
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home-outline" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="explore"
        options={{
          title: 'Explore',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="search-outline" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="profile"
        options={{
          title: 'Profile',
          tabBarIcon: ({ color, size, focused }) => (
            <Ionicons
              name={focused ? 'person' : 'person-outline'}
              size={size}
              color={color}
            />
          ),
          tabBarBadge: 3, // notification badge
        }}
      />
    </Tabs>
  );
}
```

</details> <details> <summary><code>Tabs.Screen</code> options reference</summary>

|Option|Type|Description|
|---|---|---|
|`title`|`string`|Tab label text|
|`tabBarIcon`|`function`|Icon component|
|`tabBarLabel`|`string \| function`|Custom label (overrides `title`)|
|`tabBarBadge`|`number \| string`|Badge shown on tab icon|
|`tabBarBadgeStyle`|`StyleProp<ViewStyle>`|Style for the badge|
|`tabBarButton`|`function`|Completely custom tab button|
|`tabBarHideOnKeyboard`|`boolean`|Hide tab bar when keyboard opens|
|`tabBarStyle`|`StyleProp<ViewStyle>`|Style for tab bar (per tab override)|
|`tabBarVisible`|`boolean`|Show or hide this tab|
|`href`|`string \| null`|Route href; set `null` to hide tab|
|`headerShown`|`boolean`|Show screen header above tab content|

```tsx
<Tabs.Screen
  name="hidden"
  options={{
    href: null, // removes this tab from tab bar (but route still works)
  }}
/>
```

</details>

---

## Drawer Navigator

<details> <summary><code>Drawer</code> — side drawer navigator</summary>

Creates a side drawer (hamburger menu) navigator. Requires `@react-navigation/drawer` and `react-native-gesture-handler`.

```bash
npx expo install @react-navigation/drawer react-native-gesture-handler react-native-reanimated
```

```tsx
// app/_layout.tsx
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { Drawer } from 'expo-router/drawer';

export default function Layout() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <Drawer
        screenOptions={{
          drawerActiveTintColor: '#6366f1',
          drawerStyle: { backgroundColor: '#fff', width: 280 },
        }}
      >
        <Drawer.Screen
          name="index"
          options={{
            title: 'Home',
            drawerLabel: 'Home',
            drawerIcon: ({ color, size }) => (
              <Ionicons name="home-outline" size={size} color={color} />
            ),
          }}
        />
        <Drawer.Screen
          name="profile"
          options={{ drawerLabel: 'Profile', title: 'My Profile' }}
        />
      </Drawer>
    </GestureHandlerRootView>
  );
}
```

</details>

---

## Slot

<details> <summary><code>Slot</code> — render matched child route without a navigator</summary>

Renders the currently matched child route without adding any navigator UI (no header, no tab bar). Use when you want a custom layout that wraps content but doesn't need Stack/Tabs chrome.

```tsx
// app/(auth)/_layout.tsx
import { Slot } from 'expo-router';
import { View } from 'react-native';

export default function AuthLayout() {
  return (
    <View style={{ flex: 1, backgroundColor: '#6366f1' }}>
      {/* Custom auth branding at the top */}
      <View style={styles.logo}>
        <Logo />
      </View>
      {/* Renders login.tsx or register.tsx content here */}
      <Slot />
    </View>
  );
}
```

</details>

---

## Link Component

<details> <summary><code>Link</code> — declarative navigation</summary>

Renders a pressable link that navigates to the given route. Works like an `<a>` tag on web and `TouchableOpacity` on native.

|Prop|Type|Default|Description|
|---|---|---|---|
|`href`|`string \| object`|⚠️ Required|Route path or href object|
|`push`|`boolean`|`false`|Always push (never replace)|
|`replace`|`boolean`|`false`|Replace current route instead of push|
|`asChild`|`boolean`|`false`|Render child as the pressable instead|
|`style`|`StyleProp<TextStyle>`|—|Text/view style|
|`onPress`|`function`|—|Additional press handler|
|`className`|`string`|—|NativeWind / Tailwind class (web)|

```tsx
import { Link } from 'expo-router';
import { View, Text } from 'react-native';

// Static route
<Link href="/about">About Us</Link>

// Dynamic route
<Link href="/posts/123">View Post</Link>

// With query params
<Link href={{ pathname: '/search', params: { query: 'expo', page: '1' } }}>
  Search
</Link>

// Dynamic route object form
<Link href={{ pathname: '/posts/[id]', params: { id: post.id } }}>
  {post.title}
</Link>

// Replace instead of push
<Link href="/home" replace>
  Go Home
</Link>

// asChild — the child becomes the pressable
<Link href="/profile" asChild>
  <Pressable style={styles.button}>
    <Text>Open Profile</Text>
  </Pressable>
</Link>
```

</details>

---

## Redirect Component

<details> <summary><code>Redirect</code> — programmatic redirect on render</summary>

Redirects to another route immediately when rendered. Useful for conditional navigation in layouts (e.g. auth guards).

|Prop|Type|Description|
|---|---|---|
|`href`|`string \| HrefObject`|Destination route|

```tsx
import { Redirect } from 'expo-router';

// Redirect unauthenticated users
function ProtectedLayout() {
  const { isSignedIn } = useAuth();

  if (!isSignedIn) {
    return <Redirect href="/login" />;
  }

  return <Slot />;
}

// Redirect to dynamic route
<Redirect href={{ pathname: '/users/[id]', params: { id: user.id } }} />
```

</details>

---

## useRouter Hook

<details> <summary><code>useRouter()</code> — imperative navigation</summary>

Returns a router object for programmatic navigation. Use inside event handlers, effects, or anywhere you need to navigate without rendering a `<Link>`.

```tsx
import { useRouter } from 'expo-router';

function MyComponent() {
  const router = useRouter();
  // ...
}
```

</details> <details> <summary><code>router.navigate(href)</code></summary>

Navigate to a route. If already on that route, navigates back to it (same as React Navigation's `navigate`).

```tsx
router.navigate('/home');
router.navigate('/posts/123');
router.navigate({ pathname: '/posts/[id]', params: { id: '123' } });
router.navigate({ pathname: '/search', params: { q: 'expo' } });
```

</details> <details> <summary><code>router.push(href)</code></summary>

Always pushes a new screen onto the stack, even if the route already exists.

```tsx
router.push('/details');
router.push({ pathname: '/modal', params: { title: 'My Modal' } });
```

</details> <details> <summary><code>router.replace(href)</code></summary>

Replaces the current screen — no back navigation to the replaced screen.

```tsx
// After login — don't allow back to login screen
router.replace('/home');
router.replace({ pathname: '/users/[id]', params: { id: userId } });
```

</details> <details> <summary><code>router.back()</code></summary>

Go back to the previous screen in the stack.

```tsx
router.back();
```

</details> <details> <summary><code>router.canGoBack()</code></summary>

Returns `true` if there is a screen to go back to.

```tsx
if (router.canGoBack()) {
  router.back();
} else {
  router.replace('/home');
}
```

</details> <details> <summary><code>router.setParams(params)</code></summary>

Updates the search params for the current route without navigating. Useful for updating filters or state in the URL.

```tsx
// Update query string without navigation
router.setParams({ filter: 'active', sort: 'date' });
```

</details> <details> <summary><code>router.dismiss(count?)</code></summary>

Dismisses the current modal or screen. Optionally pass a number to dismiss multiple screens.

```tsx
router.dismiss();    // dismiss current modal
router.dismiss(2);   // dismiss 2 screens
```

</details> <details> <summary><code>router.dismissAll()</code></summary>

Dismisses all modals and returns to the nearest non-modal screen.

```tsx
router.dismissAll();
```

</details>

---

## useLocalSearchParams

<details> <summary><code>useLocalSearchParams&lt;T&gt;()</code> — read current screen's params</summary>

Returns the search params for the **current** route segment only. Re-renders only when this screen's params change. Preferred for reading dynamic route params.

|||
|---|---|
|**Returns**|`T extends Record<string, string \| string[]>`|

```tsx
// app/posts/[id].tsx
import { useLocalSearchParams } from 'expo-router';

export default function PostScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  return <Text>Post: {id}</Text>;
}

// app/search.tsx — query params
const { q, page = '1' } = useLocalSearchParams<{ q: string; page: string }>();

// app/docs/[...slug].tsx — catch-all
const { slug } = useLocalSearchParams<{ slug: string[] }>();
const path = Array.isArray(slug) ? slug.join('/') : slug ?? '';
```

> ✅ All param values are **strings** (or string arrays for catch-all). Parse numbers with `parseInt(id)`, booleans with `flag === 'true'`.

</details>

---

## useGlobalSearchParams

<details> <summary><code>useGlobalSearchParams&lt;T&gt;()</code> — read params from any route in the stack</summary>

Like `useLocalSearchParams`, but returns params from **all** segments in the current URL — including params from parent routes. Useful for accessing a parent dynamic segment from a nested screen.

```tsx
// URL: /users/alice/posts/123
// From the posts/[postId].tsx screen:

const { userId, postId } = useGlobalSearchParams<{
  userId: string;
  postId: string;
}>();
// userId = 'alice', postId = '123'
```

> ⚠️ Prefer `useLocalSearchParams` unless you specifically need parent params. `useGlobalSearchParams` re-renders the component whenever **any** param in the URL changes.

</details>

---

## useSegments

<details> <summary><code>useSegments()</code> — read URL segments as an array</summary>

Returns the current URL path split into segments (including group names). Useful for conditional logic based on route structure.

|||
|---|---|
|**Returns**|`string[]`|

```tsx
import { useSegments } from 'expo-router';

function AuthGuard() {
  const segments = useSegments();
  const router = useRouter();
  const { isSignedIn } = useAuth();

  useEffect(() => {
    const inAuthGroup = segments[0] === '(auth)';

    if (!isSignedIn && !inAuthGroup) {
      // Redirect to login if not in auth group
      router.replace('/login');
    } else if (isSignedIn && inAuthGroup) {
      // Redirect to home if already signed in
      router.replace('/home');
    }
  }, [isSignedIn, segments]);
}

// Example segment values:
// URL /          → []
// URL /about     → ['about']
// URL /posts/123 → ['posts', '123']
// URL /login     → ['(auth)', 'login']
```

</details>

---

## usePathname

<details> <summary><code>usePathname()</code> — current URL path as a string</summary>

Returns the current URL pathname as a string. Updates on navigation.

|||
|---|---|
|**Returns**|`string`|

```tsx
import { usePathname } from 'expo-router';

function BreadCrumbs() {
  const pathname = usePathname();
  // '/', '/about', '/posts/123'

  const parts = pathname.split('/').filter(Boolean);

  return (
    <View style={{ flexDirection: 'row' }}>
      {parts.map((part, i) => (
        <Text key={i}>{part} {'>'} </Text>
      ))}
    </View>
  );
}
```

</details>

---

## useNavigation

<details> <summary><code>useNavigation(routeName?)</code> — access React Navigation object</summary>

Returns the underlying React Navigation `navigation` object. Use for advanced navigation actions not covered by `useRouter`. Pass a route name to get the navigation object for a specific ancestor navigator.

```tsx
import { useNavigation } from 'expo-router';

function MyScreen() {
  const navigation = useNavigation();

  useEffect(() => {
    // Use React Navigation APIs directly
    navigation.setOptions({
      headerRight: () => <SaveButton />,
    });
  }, [navigation]);

  // Access parent navigator
  const parentNav = useNavigation('(tabs)');
}
```

</details>

---

## useFocusEffect

<details> <summary><code>useFocusEffect(callback)</code> — run effect when screen is focused</summary>

Runs a callback when the screen comes into focus. Cleanup runs when the screen loses focus. Wrap callback in `useCallback`.

```tsx
import { useFocusEffect } from 'expo-router';
import { useCallback } from 'react';

export default function FeedScreen() {
  useFocusEffect(
    useCallback(() => {
      // Screen is focused — refresh data
      fetchFeed();

      return () => {
        // Screen is unfocused — cleanup
        cancelPendingRequests();
      };
    }, [])
  );

  return <FeedList />;
}
```

</details>

---

## useRootNavigationState

<details> <summary><code>useRootNavigationState()</code> — read root navigation state</summary>

Returns the root navigation state. Mainly used to check if navigation is ready before performing imperative navigation in root-level effects.

```tsx
import { useRootNavigationState } from 'expo-router';

export default function RootLayout() {
  const navigationState = useRootNavigationState();

  useEffect(() => {
    if (!navigationState?.key) return; // not ready yet

    // Safe to navigate imperatively now
    performDeepLinkNavigation();
  }, [navigationState?.key]);
}
```

</details>

---

## Typed Routes

<details> <summary>Enable typed routes for autocomplete and type safety</summary>

Expo Router can auto-generate TypeScript types for all your routes, providing autocomplete and type-checking on `href` props and `useLocalSearchParams`.

```json
// app.json
{
  "expo": {
    "experiments": {
      "typedRoutes": true
    }
  }
}
```

```bash
# Generate types (runs automatically on expo start)
npx expo start
```

</details> <details> <summary>Using typed routes in components</summary>

```tsx
import { Link } from 'expo-router';

// ✅ TypeScript knows all valid routes
<Link href="/about">About</Link>
<Link href="/posts/123">Post</Link>

// ✅ Type-safe params
<Link href={{ pathname: '/posts/[id]', params: { id: '123' } }}>
  Post
</Link>

// ✅ useRouter is also typed
const router = useRouter();
router.push('/about');
router.push({ pathname: '/posts/[id]', params: { id: post.id } });

// ✅ useLocalSearchParams is typed per route
// In app/posts/[id].tsx:
const { id } = useLocalSearchParams<{ id: string }>();
```

</details>

---

## API Routes (Server)

<details> <summary>API routes — server-side endpoints</summary>

Files ending in `+api.ts` inside the `app/` directory create server-side API endpoints. Available when using Expo Router with a Node.js server or hosting on Expo's EAS.

```
app/
├── api/
│   ├── users+api.ts       → GET/POST /api/users
│   └── users/[id]+api.ts  → GET/PUT/DELETE /api/users/123
```

</details> <details> <summary>Defining HTTP handlers</summary>

Export named functions for each HTTP method. Receive a `Request` object (Web Fetch API) and return a `Response`.

```ts
// app/api/users+api.ts

export async function GET(request: Request) {
  const users = await db.users.findAll();
  return Response.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();
  const user = await db.users.create(body);
  return Response.json(user, { status: 201 });
}
```

</details> <details> <summary>Dynamic API routes with params</summary>

```ts
// app/api/users/[id]+api.ts
import { ExpoRequest, ExpoResponse } from 'expo-router/server';

export async function GET(
  request: ExpoRequest,
  { params }: { params: { id: string } }
) {
  const user = await db.users.findById(params.id);
  if (!user) {
    return Response.json({ error: 'Not found' }, { status: 404 });
  }
  return Response.json(user);
}

export async function DELETE(
  request: ExpoRequest,
  { params }: { params: { id: string } }
) {
  await db.users.delete(params.id);
  return new Response(null, { status: 204 });
}
```

</details>

---

## Error Boundaries

<details> <summary>Route-level error boundary with <code>ErrorBoundary</code> export</summary>

Export an `ErrorBoundary` component from any route file to handle errors thrown within that route and its children.

```tsx
// app/profile.tsx
import { View, Text, Button } from 'react-native';

// ErrorBoundary catches errors thrown by the default export
export function ErrorBoundary({ error, retry }: { error: Error; retry: () => void }) {
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center', padding: 24 }}>
      <Text style={{ fontSize: 20, fontWeight: '600', marginBottom: 12 }}>
        Failed to load profile
      </Text>
      <Text style={{ color: '#6b7280', textAlign: 'center', marginBottom: 24 }}>
        {error.message}
      </Text>
      <Button title="Try again" onPress={retry} />
    </View>
  );
}

export default function ProfileScreen() {
  // If this throws, ErrorBoundary is shown instead
  const profile = useProfile(); // might throw
  return <ProfileView profile={profile} />;
}
```

</details>

---

## Loading States

<details> <summary>Suspense and loading states</summary>

Expo Router supports React `Suspense` for code splitting and async data loading on web.

```tsx
// app/_layout.tsx — splash screen until app is ready
import { useEffect } from 'react';
import { SplashScreen, Stack } from 'expo-router';
import { useFonts } from 'expo-font';

// Prevent auto-hide
SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const [fontsLoaded] = useFonts({
    'Inter-Bold': require('../assets/fonts/Inter-Bold.ttf'),
  });

  useEffect(() => {
    if (fontsLoaded) {
      SplashScreen.hideAsync();
    }
  }, [fontsLoaded]);

  if (!fontsLoaded) return null;

  return <Stack />;
}
```

</details>

---

## Deep Linking & Linking Config

<details> <summary>Automatic deep linking</summary>

Expo Router automatically generates deep link configuration from your file structure. No manual linking config needed.

```json
// app.json
{
  "expo": {
    "scheme": "myapp",
    "web": {
      "bundler": "metro"
    }
  }
}
```

```
# These URLs automatically work:
myapp://               → app/index.tsx
myapp://about          → app/about.tsx
myapp://posts/123      → app/posts/[id].tsx   (id = '123')
myapp://search?q=expo  → app/search.tsx       (q = 'expo')

# On web:
https://myapp.com/
https://myapp.com/posts/123
```

</details> <details> <summary>Handle incoming links manually</summary>

```tsx
import * as Linking from 'expo-linking';
import { useEffect } from 'react';
import { useRouter } from 'expo-router';

export default function RootLayout() {
  const router = useRouter();

  useEffect(() => {
    const subscription = Linking.addEventListener('url', ({ url }) => {
      // URL is automatically handled by Expo Router
      // but you can intercept here if needed
      console.log('Incoming URL:', url);
    });
    return () => subscription.remove();
  }, []);

  return <Stack />;
}
```

</details>

---

## Authentication Pattern

<details> <summary>Auth flow with protected routes using segments</summary>

```
app/
├── _layout.tsx           ← Root layout with auth guard
├── (auth)/
│   ├── _layout.tsx       ← Auth layout (no tab bar)
│   ├── login.tsx         → /login
│   └── register.tsx      → /register
└── (app)/
    ├── _layout.tsx       ← Tab layout (protected)
    ├── home.tsx          → /home
    └── profile.tsx       → /profile
```

```tsx
// app/_layout.tsx
import { Slot, useSegments, useRouter } from 'expo-router';
import { useEffect } from 'react';
import { useAuthStore } from '../store/auth';

function AuthGuard() {
  const { token } = useAuthStore();
  const segments = useSegments();
  const router = useRouter();

  useEffect(() => {
    const inAuthGroup = segments[0] === '(auth)';

    if (!token && !inAuthGroup) {
      // Not signed in — redirect to login
      router.replace('/login');
    } else if (token && inAuthGroup) {
      // Signed in — redirect away from auth screens
      router.replace('/home');
    }
  }, [token, segments]);

  return null;
}

export default function RootLayout() {
  return (
    <>
      <AuthGuard />
      <Slot />
    </>
  );
}
```

</details> <details> <summary>Auth layout — no navigator chrome</summary>

```tsx
// app/(auth)/_layout.tsx
import { Stack } from 'expo-router';

export default function AuthLayout() {
  return (
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Screen name="login" />
      <Stack.Screen name="register" />
    </Stack>
  );
}
```

</details> <details> <summary>Login screen with router.replace</summary>

```tsx
// app/(auth)/login.tsx
import { useRouter } from 'expo-router';
import { useAuthStore } from '../../store/auth';

export default function LoginScreen() {
  const router = useRouter();
  const { signIn } = useAuthStore();

  const handleLogin = async () => {
    await signIn(email, password);
    // Replace so back button doesn't return to login
    router.replace('/home');
  };

  return (
    <View style={{ flex: 1, padding: 24 }}>
      <TextInput placeholder="Email" onChangeText={setEmail} />
      <TextInput placeholder="Password" secureTextEntry onChangeText={setPassword} />
      <Button title="Log In" onPress={handleLogin} />
      <Link href="/register">Create an account</Link>
    </View>
  );
}
```

</details>

---

## Common Patterns

<details> <summary>Pass params between screens</summary>

```tsx
// Sender — navigate with params
router.push({ pathname: '/products/[id]', params: { id: product.id, title: product.name } });
// or with Link:
<Link href={{ pathname: '/products/[id]', params: { id: product.id } }}>
  View Product
</Link>

// Receiver — app/products/[id].tsx
const { id, title } = useLocalSearchParams<{ id: string; title: string }>();
```

</details> <details> <summary>Modal screen</summary>

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function Layout() {
  return (
    <Stack>
      <Stack.Screen name="index" />
      <Stack.Screen
        name="create-post"
        options={{
          presentation: 'modal',
          headerShown: true,
          title: 'New Post',
        }}
      />
    </Stack>
  );
}

// Open modal from any screen
router.push('/create-post');

// Close modal from inside modal
// app/create-post.tsx
<Stack.Screen
  options={{
    headerLeft: () => (
      <TouchableOpacity onPress={() => router.back()}>
        <Text>Cancel</Text>
      </TouchableOpacity>
    ),
  }}
/>
```

</details> <details> <summary>Dynamic header title from route params</summary>

```tsx
// app/posts/[id].tsx
import { Stack, useLocalSearchParams } from 'expo-router';

export default function PostScreen() {
  const { id, title } = useLocalSearchParams<{ id: string; title: string }>();

  return (
    <>
      <Stack.Screen options={{ title: title ?? `Post ${id}` }} />
      <PostContent id={id} />
    </>
  );
}
```

</details> <details> <summary>Nested tabs inside a stack</summary>

```
app/
├── _layout.tsx          ← Root Stack
├── index.tsx            → /  (splash / onboarding)
└── (main)/
    ├── _layout.tsx      ← Tab navigator
    ├── home.tsx         → /home
    ├── search.tsx       → /search
    └── inbox/
        ├── _layout.tsx  ← Nested Stack
        ├── index.tsx    → /inbox
        └── [id].tsx     → /inbox/123
```

</details> <details> <summary>Tab with badge from state</summary>

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { useNotifications } from '../../hooks/useNotifications';

export default function TabLayout() {
  const { unreadCount } = useNotifications();

  return (
    <Tabs>
      <Tabs.Screen name="home" options={{ title: 'Home' }} />
      <Tabs.Screen
        name="inbox"
        options={{
          title: 'Inbox',
          tabBarBadge: unreadCount > 0 ? unreadCount : undefined,
          tabBarBadgeStyle: { backgroundColor: '#ef4444' },
        }}
      />
    </Tabs>
  );
}
```

</details> <details> <summary>Programmatic navigation after async operation</summary>

```tsx
// Navigate after API call
async function handleSubmit() {
  setLoading(true);
  try {
    const { id } = await createPost(formData);
    router.replace({ pathname: '/posts/[id]', params: { id } });
  } catch (e) {
    setError(e.message);
  } finally {
    setLoading(false);
  }
}
```

</details> <details> <summary>Redirect from index based on onboarding state</summary>

```tsx
// app/index.tsx
import { Redirect } from 'expo-router';
import { useOnboarding } from '../hooks/useOnboarding';

export default function IndexScreen() {
  const { isComplete, isLoading } = useOnboarding();

  if (isLoading) return <LoadingSpinner />;

  if (!isComplete) {
    return <Redirect href="/onboarding/step-1" />;
  }

  return <Redirect href="/home" />;
}
```

</details> <details> <summary>Back button with unsaved changes guard</summary>

```tsx
// app/edit-profile.tsx
import { Stack, useRouter } from 'expo-router';
import { usePreventRemove } from '@react-navigation/native';

export default function EditProfileScreen() {
  const [hasChanges, setHasChanges] = useState(false);
  const router = useRouter();

  usePreventRemove(hasChanges, ({ data }) => {
    Alert.alert('Discard changes?', 'You have unsaved changes.', [
      { text: 'Keep editing', style: 'cancel' },
      {
        text: 'Discard',
        style: 'destructive',
        onPress: () => router.back(),
      },
    ]);
  });

  return (
    <>
      <Stack.Screen options={{ title: 'Edit Profile' }} />
      <ProfileForm onChange={() => setHasChanges(true)} />
    </>
  );
}
```

</details>

---

## Quick-Reference Cheatsheet

|API / File|Use case|
|---|---|
|`app/index.tsx`|Root route `/`|
|`app/[param].tsx`|Dynamic route segment|
|`app/[...slug].tsx`|Catch-all route|
|`app/(group)/`|Route group — no URL segment|
|`app/_layout.tsx`|Layout / navigator wrapper|
|`app/+not-found.tsx`|404 page|
|`app/+error.tsx`|Error boundary|
|`app/route+api.ts`|Server API endpoint|
|`<Stack>`|Stack navigator|
|`<Tabs>`|Tab bar navigator|
|`<Drawer>`|Side drawer navigator|
|`<Slot>`|Render child route (no nav chrome)|
|`<Link href="/path">`|Declarative navigation|
|`<Redirect href="/path">`|Redirect on render|
|`useRouter()`|Imperative navigation|
|`router.push(href)`|Push new screen|
|`router.replace(href)`|Replace current screen|
|`router.back()`|Go back|
|`router.dismiss()`|Dismiss modal|
|`useLocalSearchParams()`|Read current screen's URL params|
|`useGlobalSearchParams()`|Read all URL params|
|`useSegments()`|URL path as string array|
|`usePathname()`|Current URL as string|
|`useFocusEffect(cb)`|Side effect on screen focus|
|`useNavigation()`|Raw React Navigation object|
|`SplashScreen.preventAutoHideAsync()`|Hold splash until ready|
|`typedRoutes: true`|TypeScript route autocomplete|

---

_Reference based on Expo Router v3. Always check the [official docs](https://docs.expo.dev/router/introduction/) for the latest updates._