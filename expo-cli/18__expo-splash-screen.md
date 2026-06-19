# expo-splash-screen — Complete Reference

A comprehensive reference for `expo-splash-screen` (SDK 50+, new SplashScreen API).  
Covers configuration in `app.json`, the imperative API, hooks, transitions to a custom splash/animated screen, and common patterns.

> **Install:**
> 
> ```bash
> npx expo install expo-splash-screen
> ```

---

## Table of Contents

1. [Overview — Native Splash vs Animated Splash](#overview--native-splash-vs-animated-splash)
2. [app.json Configuration](#appjson-configuration)
3. [Plugin Configuration Options](#plugin-configuration-options)
4. [Imperative API](#imperative-api)
5. [preventAutoHideAsync](#preventautohideasync)
6. [hideAsync](#hideasync)
7. [hide (deprecated sync)](#hide-deprecated-sync)
8. [setOptions](#setoptions)
9. [useSplashScreen Pattern](#usesplashscreen-pattern)
10. [Custom Animated Splash Screen](#custom-animated-splash-screen)
11. [Expo Router Integration](#expo-router-integration)
12. [Asset Preloading Before Hiding](#asset-preloading-before-hiding)
13. [Common Patterns](#common-patterns)
14. [Troubleshooting](#troubleshooting)

---

## Overview — Native Splash vs Animated Splash

<details> <summary>The two-phase splash screen model</summary>

Expo apps have **two distinct splash phases** that work together:

|Phase|Controlled by|Duration|
|---|---|---|
|**1. Native splash**|`app.json` config + native OS|From app launch until JS bundle starts executing|
|**2. JS-controlled splash**|`expo-splash-screen` API in your code|From JS start until you call `hideAsync()`|

```
App Tap
  │
  ▼
┌─────────────────────────┐
│  Native Splash (Phase 1) │  ← OS shows app.json's "image" instantly
│  Static image, no JS yet │
└─────────────────────────┘
  │  JS bundle loads & executes
  ▼
┌─────────────────────────┐
│  Same image held by      │  ← preventAutoHideAsync() keeps it visible
│  preventAutoHideAsync()  │     while you load fonts/data/auth state
└─────────────────────────┘
  │  Your app calls hideAsync()
  ▼
┌─────────────────────────┐
│  App UI renders          │
└─────────────────────────┘
```

> ✅ Always call `preventAutoHideAsync()` immediately at the top of your root layout/App component — before any other code runs — to avoid a flash of blank screen between phase 1 and phase 2.

</details> <details> <summary>When you need a custom animated splash screen</summary>

The native splash can only show a **static image**. If you want a Lottie animation, custom transition, or anything dynamic, you need a **second JS-rendered splash component** that:

1. Native splash shows the static image (phase 1)
2. JS loads, calls `preventAutoHideAsync()`
3. Your JS renders a custom splash UI matching the native one (no visible jump)
4. Once your custom animation finishes, swap to the real app UI

See [Custom Animated Splash Screen](#custom-animated-splash-screen) for the full pattern.

</details>

---

## app.json Configuration

<details> <summary>Legacy <code>splash</code> key (SDK &lt; 53, still works)</summary>

The simpler config object directly under `expo`. Still supported but the `expo-splash-screen` plugin (below) is the modern recommended approach.

```json
{
  "expo": {
    "splash": {
      "image": "./assets/images/splash-icon.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    }
  }
}
```

|Field|Type|Description|
|---|---|---|
|`image`|`string`|Path to splash image|
|`resizeMode`|`'contain' \| 'cover' \| 'native'`|How the image fits the screen|
|`backgroundColor`|`string`|Background color behind/around the image|

</details> <details> <summary>Modern <code>expo-splash-screen</code> plugin config (recommended)</summary>

Add the plugin to `app.json` for finer per-platform control, dark mode support, and tablet-specific images.

```json
{
  "expo": {
    "plugins": [
      [
        "expo-splash-screen",
        {
          "image": "./assets/images/splash-icon.png",
          "imageWidth": 200,
          "resizeMode": "contain",
          "backgroundColor": "#ffffff",
          "dark": {
            "image": "./assets/images/splash-icon-dark.png",
            "backgroundColor": "#000000"
          }
        }
      ]
    ]
  }
}
```

> ⚠️ If using the plugin, remove the legacy `splash` key from `app.json` to avoid conflicts.

</details>

---

## Plugin Configuration Options

<details> <summary><code>image</code> — splash image path</summary>

Path to the image shown during native splash. PNG recommended.

|||
|---|---|
|**Type**|`string`|
|**Required**|✅|

```json
{
  "image": "./assets/images/splash-icon.png"
}
```

> 💡 Use a small centered logo (e.g. 200×200) rather than a full-screen image — full-bleed splash images often look stretched or cropped differently across device sizes.

</details> <details> <summary><code>imageWidth</code> — control image size</summary>

Sets the rendered width of the splash image in points/dp. Height is calculated automatically to preserve aspect ratio.

|||
|---|---|
|**Type**|`number`|
|**Default**|Original image width|

```json
{
  "image": "./assets/images/splash-icon.png",
  "imageWidth": 200
}
```

</details> <details> <summary><code>resizeMode</code> — how the image fits</summary>

|||
|---|---|
|**Type**|`'contain' \| 'cover' \| 'native'`|
|**Default**|`'contain'`|

|Value|Behavior|
|---|---|
|`'contain'`|Image centered, scaled to fit without cropping (recommended)|
|`'cover'`|Image fills the screen, may crop edges|
|`'native'`|Uses platform-native sizing behavior|

```json
{
  "resizeMode": "contain"
}
```

</details> <details> <summary><code>backgroundColor</code> — splash background</summary>

|||
|---|---|
|**Type**|`string` (hex color)|
|**Default**|`'#ffffff'`|

```json
{
  "backgroundColor": "#6366f1"
}
```

</details> <details> <summary><code>dark</code> — dark mode variant</summary>

Provides a separate image and/or background color shown when the device is in dark mode.

|Field|Type|Description|
|---|---|---|
|`dark.image`|`string`|Dark mode splash image|
|`dark.backgroundColor`|`string`|Dark mode background color|
|`dark.resizeMode`|`string`|Dark mode resize mode override|

```json
{
  "image": "./assets/images/splash-light.png",
  "backgroundColor": "#ffffff",
  "dark": {
    "image": "./assets/images/splash-dark.png",
    "backgroundColor": "#0f0f0f"
  }
}
```

</details> <details> <summary><code>ios</code> / <code>android</code> — per-platform overrides</summary>

Override any of the above settings for a specific platform.

```json
{
  "image": "./assets/images/splash-icon.png",
  "imageWidth": 200,
  "resizeMode": "contain",
  "backgroundColor": "#ffffff",
  "ios": {
    "backgroundColor": "#fafafa"
  },
  "android": {
    "imageWidth": 180,
    "backgroundColor": "#ffffff",
    "dark": {
      "backgroundColor": "#000000"
    }
  }
}
```

</details> <details> <summary>Full example with all options</summary>

```json
{
  "expo": {
    "plugins": [
      [
        "expo-splash-screen",
        {
          "image": "./assets/images/splash-icon.png",
          "imageWidth": 200,
          "resizeMode": "contain",
          "backgroundColor": "#ffffff",
          "dark": {
            "image": "./assets/images/splash-icon-dark.png",
            "backgroundColor": "#0f0f0f"
          },
          "ios": {
            "enableFullScreenImage_legacy": false
          },
          "android": {
            "imageWidth": 180
          }
        }
      ]
    ]
  }
}
```

> ⚠️ After editing the plugin config, you must rebuild native code: `npx expo prebuild --clean` (bare workflow) or create a new development/production build via EAS.

</details>

---

## Imperative API

<details> <summary>Importing the module</summary>

```ts
import * as SplashScreen from 'expo-splash-screen';
```

</details>

---

## preventAutoHideAsync

<details> <summary><code>SplashScreen.preventAutoHideAsync()</code> — keep splash visible</summary>

Prevents the splash screen from automatically hiding once the JS bundle has loaded. Must be called **before** your root component renders — typically at module scope, outside any component function.

|||
|---|---|
|**Returns**|`Promise<boolean>` — `true` if successful|

```tsx
import * as SplashScreen from 'expo-splash-screen';

// ✅ Call immediately at module level — before component definition
SplashScreen.preventAutoHideAsync();

export default function App() {
  const [appIsReady, setAppIsReady] = useState(false);

  useEffect(() => {
    async function prepare() {
      try {
        // Load fonts, fetch data, check auth, etc.
        await loadFontsAsync();
        await checkAuthStatus();
      } finally {
        setAppIsReady(true);
      }
    }
    prepare();
  }, []);

  useEffect(() => {
    if (appIsReady) {
      SplashScreen.hideAsync();
    }
  }, [appIsReady]);

  if (!appIsReady) {
    return null; // Splash screen still showing natively
  }

  return <MainApp />;
}
```

> ⚠️ If `preventAutoHideAsync()` throws or this call is skipped, the splash screen will hide automatically as soon as the first frame renders — even if your data isn't ready yet, causing a flash of incomplete UI.

</details>

---

## hideAsync

<details> <summary><code>SplashScreen.hideAsync()</code> — hide the splash screen</summary>

Hides the splash screen, revealing your app's UI. Call this once your app has finished loading whatever it needs (fonts, auth state, initial data).

|||
|---|---|
|**Returns**|`Promise<boolean>`|

```tsx
import * as SplashScreen from 'expo-splash-screen';

async function onAppReady() {
  await SplashScreen.hideAsync();
}

// Inside a component
useEffect(() => {
  if (fontsLoaded && authChecked) {
    SplashScreen.hideAsync();
  }
}, [fontsLoaded, authChecked]);
```

> 💡 Calling `hideAsync()` without first calling `preventAutoHideAsync()` is a no-op — there's nothing to hide since it would have auto-hidden already.

</details>

---

## hide (deprecated sync)

<details> <summary><code>SplashScreen.hide()</code> — synchronous variant (deprecated)</summary>

The older, synchronous version of `hideAsync`. Still works in most SDK versions but `hideAsync` is preferred going forward.

```tsx
// ⚠️ Deprecated — prefer hideAsync()
SplashScreen.hide();
```

</details>

---

## setOptions

<details> <summary><code>SplashScreen.setOptions(options)</code> — configure fade-out transition (SDK 51+)</summary>

Configures a smooth fade transition when the splash screen hides, instead of an abrupt cut. Call this once at startup, alongside `preventAutoHideAsync()`.

|Option|Type|Default|Description|
|---|---|---|---|
|`duration`|`number`|`400`|Fade-out duration in milliseconds|
|`fade`|`boolean`|`false`|Enable fade transition when hiding|

```tsx
import * as SplashScreen from 'expo-splash-screen';

SplashScreen.preventAutoHideAsync();
SplashScreen.setOptions({
  duration: 400,
  fade: true,
});

export default function App() {
  // ... loading logic
  // hideAsync() will now fade smoothly instead of cutting abruptly
}
```

</details>

---

## useSplashScreen Pattern

<details> <summary>Custom hook wrapping the splash lifecycle</summary>

A reusable hook pattern that encapsulates "prevent → load → hide" logic cleanly.

```tsx
// hooks/useAppReady.ts
import { useState, useEffect, useCallback } from 'react';
import * as SplashScreen from 'expo-splash-screen';
import { useFonts } from 'expo-font';

SplashScreen.preventAutoHideAsync();

export function useAppReady() {
  const [authChecked, setAuthChecked] = useState(false);
  const [fontsLoaded, fontError] = useFonts({
    'Inter-Regular': require('../assets/fonts/Inter-Regular.ttf'),
    'Inter-Bold': require('../assets/fonts/Inter-Bold.ttf'),
  });

  useEffect(() => {
    async function checkAuth() {
      try {
        await restoreAuthSession();
      } finally {
        setAuthChecked(true);
      }
    }
    checkAuth();
  }, []);

  const appIsReady = (fontsLoaded || !!fontError) && authChecked;

  const onLayoutRootView = useCallback(async () => {
    if (appIsReady) {
      await SplashScreen.hideAsync();
    }
  }, [appIsReady]);

  return { appIsReady, onLayoutRootView };
}
```

```tsx
// App.tsx
import { useAppReady } from './hooks/useAppReady';
import { View } from 'react-native';

export default function App() {
  const { appIsReady, onLayoutRootView } = useAppReady();

  if (!appIsReady) {
    return null;
  }

  return (
    <View style={{ flex: 1 }} onLayout={onLayoutRootView}>
      <MainApp />
    </View>
  );
}
```

> ✅ Using `onLayout` to trigger `hideAsync()` (instead of a `useEffect`) ensures the splash hides only **after** the first frame of your real UI has been measured and is ready to paint — eliminating any flash of blank screen.

</details>

---

## Custom Animated Splash Screen

<details> <summary>Full pattern — native splash → animated JS splash → app</summary>

For a Lottie animation, custom branding sequence, or any dynamic splash experience, layer a JS-rendered splash screen on top that visually matches the native one, then transition out.

```tsx
// App.tsx
import { useState, useCallback } from 'react';
import { View, StyleSheet } from 'react-native';
import * as SplashScreen from 'expo-splash-screen';
import { useFonts } from 'expo-font';
import AnimatedSplash from './components/AnimatedSplash';
import MainApp from './MainApp';

// Hide native splash immediately once JS takes over —
// our custom AnimatedSplash component will take its place visually
SplashScreen.preventAutoHideAsync();

export default function App() {
  const [appIsReady, setAppIsReady] = useState(false);
  const [splashAnimationDone, setSplashAnimationDone] = useState(false);
  const [fontsLoaded] = useFonts({
    'Inter-Bold': require('./assets/fonts/Inter-Bold.ttf'),
  });

  const onLayoutRootView = useCallback(async () => {
    if (fontsLoaded) {
      // Hide the native splash — our AnimatedSplash matches it visually
      // so there's no visible jump
      await SplashScreen.hideAsync();
      setAppIsReady(true);
    }
  }, [fontsLoaded]);

  if (!fontsLoaded) {
    return null;
  }

  return (
    <View style={{ flex: 1 }} onLayout={onLayoutRootView}>
      {appIsReady && <MainApp />}
      {!splashAnimationDone && (
        <AnimatedSplash onAnimationFinish={() => setSplashAnimationDone(true)} />
      )}
    </View>
  );
}
```

```tsx
// components/AnimatedSplash.tsx
import { View, StyleSheet, Image, Animated } from 'react-native';
import { useEffect, useRef } from 'react';

interface Props {
  onAnimationFinish: () => void;
}

export default function AnimatedSplash({ onAnimationFinish }: Props) {
  const opacity = useRef(new Animated.Value(1)).current;
  const scale = useRef(new Animated.Value(1)).current;

  useEffect(() => {
    // Hold for a moment, then animate out
    Animated.sequence([
      Animated.delay(800),
      Animated.parallel([
        Animated.timing(opacity, {
          toValue: 0,
          duration: 400,
          useNativeDriver: true,
        }),
        Animated.timing(scale, {
          toValue: 1.1,
          duration: 400,
          useNativeDriver: true,
        }),
      ]),
    ]).start(() => onAnimationFinish());
  }, []);

  return (
    <Animated.View
      style={[
        styles.container,
        { opacity, transform: [{ scale }] },
      ]}
      pointerEvents="none"
    >
      {/* Match this exactly to your native splash image/background */}
      <Image
        source={require('../assets/images/splash-icon.png')}
        style={{ width: 200, height: 200 }}
        resizeMode="contain"
      />
    </Animated.View>
  );
}

const styles = StyleSheet.create({
  container: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: '#ffffff',  // match app.json's backgroundColor exactly
    justifyContent: 'center',
    alignItems: 'center',
    zIndex: 999,
  },
});
```

> ✅ **Key principle:** your custom JS splash must render with the **exact same image, position, and background color** as the native splash. If they don't match pixel-for-pixel, the user will see a visible "jump" when the native splash hides and your component takes over.

</details> <details> <summary>Lottie-based animated splash</summary>

```tsx
// components/LottieSplash.tsx
import { View, StyleSheet } from 'react-native';
import LottieView from 'lottie-react-native';

interface Props {
  onAnimationFinish: () => void;
}

export default function LottieSplash({ onAnimationFinish }: Props) {
  return (
    <View style={styles.container} pointerEvents="none">
      <LottieView
        source={require('../assets/animations/logo-intro.json')}
        autoPlay
        loop={false}
        onAnimationFinish={(isCancelled) => {
          if (!isCancelled) onAnimationFinish();
        }}
        style={{ width: 250, height: 250 }}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: '#ffffff',
    justifyContent: 'center',
    alignItems: 'center',
    zIndex: 999,
  },
});
```

</details>

---

## Expo Router Integration

<details> <summary>Root layout setup with Expo Router</summary>

```tsx
// app/_layout.tsx
import { useEffect, useCallback } from 'react';
import { Stack } from 'expo-router';
import * as SplashScreen from 'expo-splash-screen';
import { useFonts } from 'expo-font';
import { View } from 'react-native';

SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const [fontsLoaded, fontError] = useFonts({
    'Inter-Regular': require('../assets/fonts/Inter-Regular.ttf'),
    'Inter-Bold': require('../assets/fonts/Inter-Bold.ttf'),
  });

  useEffect(() => {
    if (fontsLoaded || fontError) {
      SplashScreen.hideAsync();
    }
  }, [fontsLoaded, fontError]);

  if (!fontsLoaded && !fontError) {
    return null;
  }

  return <Stack />;
}
```

</details> <details> <summary>With auth check and redirect (Expo Router)</summary>

```tsx
// app/_layout.tsx
import { useEffect, useState } from 'react';
import { Slot, useRouter, useSegments } from 'expo-router';
import * as SplashScreen from 'expo-splash-screen';
import { useFonts } from 'expo-font';
import { useAuthStore } from '../store/auth';

SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const [fontsLoaded] = useFonts({
    'Inter-Bold': require('../assets/fonts/Inter-Bold.ttf'),
  });
  const [authChecked, setAuthChecked] = useState(false);
  const { token, restoreSession } = useAuthStore();
  const segments = useSegments();
  const router = useRouter();

  // Restore auth session on mount
  useEffect(() => {
    restoreSession().finally(() => setAuthChecked(true));
  }, []);

  // Hide splash once everything is ready
  useEffect(() => {
    if (fontsLoaded && authChecked) {
      SplashScreen.hideAsync();
    }
  }, [fontsLoaded, authChecked]);

  // Redirect based on auth state — runs after splash hides
  useEffect(() => {
    if (!authChecked) return;

    const inAuthGroup = segments[0] === '(auth)';
    if (!token && !inAuthGroup) {
      router.replace('/login');
    } else if (token && inAuthGroup) {
      router.replace('/home');
    }
  }, [authChecked, token, segments]);

  if (!fontsLoaded || !authChecked) {
    return null;
  }

  return <Slot />;
}
```

</details>

---

## Asset Preloading Before Hiding

<details> <summary>Preload images alongside fonts</summary>

```tsx
import { useState, useEffect, useCallback } from 'react';
import * as SplashScreen from 'expo-splash-screen';
import { useFonts } from 'expo-font';
import { Asset } from 'expo-asset';

SplashScreen.preventAutoHideAsync();

const imageAssets = [
  require('../assets/images/onboarding-1.png'),
  require('../assets/images/onboarding-2.png'),
  require('../assets/images/hero-bg.png'),
];

export default function App() {
  const [appIsReady, setAppIsReady] = useState(false);
  const [fontsLoaded] = useFonts({
    'Inter-Bold': require('../assets/fonts/Inter-Bold.ttf'),
  });

  useEffect(() => {
    async function loadResources() {
      try {
        // Preload images so they don't flicker on first render
        await Asset.loadAsync(imageAssets);
      } catch (e) {
        console.warn('Asset preload error:', e);
      } finally {
        setAppIsReady(true);
      }
    }
    loadResources();
  }, []);

  const onLayoutRootView = useCallback(async () => {
    if (fontsLoaded && appIsReady) {
      await SplashScreen.hideAsync();
    }
  }, [fontsLoaded, appIsReady]);

  if (!fontsLoaded || !appIsReady) {
    return null;
  }

  return (
    <View style={{ flex: 1 }} onLayout={onLayoutRootView}>
      <MainApp />
    </View>
  );
}
```

</details> <details> <summary>Comprehensive prepare function — fonts, assets, auth, remote config</summary>

```tsx
async function prepareApp(): Promise<void> {
  const tasks = [
    loadFontsAsync(),
    Asset.loadAsync(imageAssets),
    restoreAuthSession(),
    fetchRemoteConfig(),
    AsyncStorage.getItem('onboarding_complete'),
  ];

  // Run in parallel, but don't let one slow failure block forever
  const results = await Promise.allSettled(tasks);

  results.forEach((result, i) => {
    if (result.status === 'rejected') {
      console.warn(`Prepare task ${i} failed:`, result.reason);
    }
  });

  // Optional minimum splash duration for branding purposes
  await new Promise(resolve => setTimeout(resolve, 800));
}

export default function App() {
  const [appIsReady, setAppIsReady] = useState(false);

  useEffect(() => {
    prepareApp().finally(() => setAppIsReady(true));
  }, []);

  const onLayoutRootView = useCallback(async () => {
    if (appIsReady) {
      await SplashScreen.hideAsync();
    }
  }, [appIsReady]);

  if (!appIsReady) return null;

  return (
    <View style={{ flex: 1 }} onLayout={onLayoutRootView}>
      <MainApp />
    </View>
  );
}
```

</details>

---

## Common Patterns

<details> <summary>Minimal setup — fonts only</summary>

```tsx
import { useEffect } from 'react';
import * as SplashScreen from 'expo-splash-screen';
import { useFonts } from 'expo-font';

SplashScreen.preventAutoHideAsync();

export default function App() {
  const [loaded, error] = useFonts({
    'Inter-Regular': require('./assets/fonts/Inter-Regular.ttf'),
  });

  useEffect(() => {
    if (loaded || error) {
      SplashScreen.hideAsync();
    }
  }, [loaded, error]);

  if (!loaded && !error) return null;

  return <MainApp />;
}
```

</details> <details> <summary>With fade transition (SDK 51+)</summary>

```tsx
import * as SplashScreen from 'expo-splash-screen';

SplashScreen.preventAutoHideAsync();
SplashScreen.setOptions({ duration: 500, fade: true });

export default function App() {
  // ... loading logic
  // hideAsync() now fades smoothly over 500ms instead of an abrupt cut
}
```

</details> <details> <summary>Minimum splash duration (branding requirement)</summary>

Sometimes you want the splash to show for at least N seconds even if loading finishes faster — common for branding/marketing requirements.

```tsx
async function prepare() {
  const minDuration = new Promise(resolve => setTimeout(resolve, 1500));
  const loadData = loadAppData();

  // Wait for both — whichever takes longer
  await Promise.all([minDuration, loadData]);
}

export default function App() {
  const [ready, setReady] = useState(false);

  useEffect(() => {
    prepare().finally(() => setReady(true));
  }, []);

  useEffect(() => {
    if (ready) SplashScreen.hideAsync();
  }, [ready]);

  if (!ready) return null;
  return <MainApp />;
}
```

</details> <details> <summary>Error-safe loading — always hide splash even on failure</summary>

```tsx
export default function App() {
  const [appIsReady, setAppIsReady] = useState(false);
  const [loadError, setLoadError] = useState<Error | null>(null);

  useEffect(() => {
    async function prepare() {
      try {
        await criticalSetupTask();
      } catch (e) {
        // Log error but still let the app continue —
        // show an error screen instead of being stuck on splash forever
        console.error('Setup failed:', e);
        setLoadError(e as Error);
      } finally {
        setAppIsReady(true);  // always set ready, even on error
      }
    }
    prepare();
  }, []);

  useEffect(() => {
    if (appIsReady) {
      SplashScreen.hideAsync();
    }
  }, [appIsReady]);

  if (!appIsReady) return null;

  if (loadError) {
    return <ErrorScreen error={loadError} />;
  }

  return <MainApp />;
}
```

> ⚠️ Never let a thrown error in your prepare function leave `hideAsync()` uncalled — this would trap the user on an infinite splash screen. Always use `try/finally`.

</details> <details> <summary>Onboarding-aware splash flow</summary>

```tsx
export default function App() {
  const [appIsReady, setAppIsReady] = useState(false);
  const [hasOnboarded, setHasOnboarded] = useState<boolean | null>(null);

  useEffect(() => {
    async function prepare() {
      const value = await AsyncStorage.getItem('onboarding_complete');
      setHasOnboarded(value === 'true');
      setAppIsReady(true);
    }
    prepare();
  }, []);

  useEffect(() => {
    if (appIsReady) SplashScreen.hideAsync();
  }, [appIsReady]);

  if (!appIsReady) return null;

  return hasOnboarded ? <MainApp /> : <OnboardingFlow />;
}
```

</details> <details> <summary>Testing splash logic in development</summary>

```tsx
// Force splash to stay longer for visual testing
const __DEBUG_SPLASH_DELAY__ = __DEV__ ? 3000 : 0;

async function prepare() {
  await loadFontsAsync();
  if (__DEBUG_SPLASH_DELAY__) {
    await new Promise(resolve => setTimeout(resolve, __DEBUG_SPLASH_DELAY__));
  }
}
```

</details>

---

## Troubleshooting

<details> <summary>Splash screen flashes white before showing custom splash</summary>

- Ensure `preventAutoHideAsync()` is called at **module level**, not inside a `useEffect`
- Make sure it's called before any other import side-effects that might render
- Check that your custom splash component's background color **exactly** matches `app.json`'s `backgroundColor`

</details> <details> <summary>Splash hides too early, showing blank/incomplete UI</summary>

- Make sure `hideAsync()` is only called after **all** loading tasks resolve — use `Promise.all` or `Promise.allSettled`
- Use `onLayout` on the root view instead of a plain `useEffect` to ensure the first frame is ready before hiding
- Check that font loading state (`fontsLoaded`) is actually included in your readiness check

</details> <details> <summary>App is stuck on splash screen forever</summary>

- Check for an unhandled promise rejection in your prepare function — wrap in `try/catch/finally`
- Verify `hideAsync()` is actually being called — add a `console.log` to confirm
- Check if a `useFonts` font path is wrong, causing the hook to never resolve

</details> <details> <summary>Changes to splash image don't appear after editing app.json</summary>

- Splash screen config requires a **native rebuild** — JS-only reload (`expo start --clear`) is not enough
- Run `npx expo prebuild --clean` for bare workflow, or trigger a new EAS build
- On managed workflow with Expo Go, some config plugin changes don't apply — use a development build instead

</details> <details> <summary>Dark mode splash not switching</summary>

- Verify `userInterfaceStyle: "automatic"` is set in `app.json`
- Confirm the `dark` object is nested correctly under the `expo-splash-screen` plugin config, not the legacy `splash` key
- Native rebuild required for dark mode splash config changes to take effect

</details>

---

## Quick-Reference Cheatsheet

|API / Config|Use case|
|---|---|
|`app.json` → `plugins: ["expo-splash-screen", {...}]`|Configure native splash image/colors|
|`image`|Splash logo/icon path|
|`imageWidth`|Control rendered size|
|`resizeMode: "contain"`|Recommended fit mode|
|`backgroundColor`|Background behind image|
|`dark: { image, backgroundColor }`|Dark mode variant|
|`SplashScreen.preventAutoHideAsync()`|Hold splash — call at module scope|
|`SplashScreen.hideAsync()`|Reveal app once ready|
|`SplashScreen.setOptions({ fade: true })`|Smooth fade-out transition|
|`onLayout` on root view|Hide splash only after first frame renders|
|`try/finally` around prepare logic|Prevent infinite splash on error|
|Custom JS splash component|Animated/Lottie intro matching native splash|
|`npx expo prebuild --clean`|Required after splash config changes|

---

_Reference based on `expo-splash-screen` SDK 50+. Always check the [official docs](https://docs.expo.dev/versions/latest/sdk/splash-screen/) for the latest updates._