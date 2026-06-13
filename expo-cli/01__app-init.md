# Expo App Initialization — Complete Reference

A comprehensive reference for creating, configuring, and setting up Expo apps from scratch.  
Covers project creation, directory structure, `app.json` / `app.config.js`, environment variables, EAS, and essential first steps.

---

## Table of Contents

1. [Creating a New App](#creating-a-new-app)
2. [Project Templates](#project-templates)
3. [Project Structure](#project-structure)
4. [Running the App](#running-the-app)
5. [app.json — Full Config Reference](#appjson--full-config-reference)
6. [app.config.js — Dynamic Config](#appconfigjs--dynamic-config)
7. [Environment Variables](#environment-variables)
8. [package.json Scripts](#packagejson-scripts)
9. [Babel Config](#babel-config)
10. [Metro Config](#metro-config)
11. [TypeScript Config](#typescript-config)
12. [EAS — Expo Application Services](#eas--expo-application-services)
13. [eas.json — Build Profiles](#easjson--build-profiles)
14. [Installing Packages](#installing-packages)
15. [Expo SDK Upgrades](#expo-sdk-upgrades)
16. [Prebuild — Bare Workflow](#prebuild--bare-workflow)
17. [Common First Steps](#common-first-steps)

---

## Creating a New App

<details> <summary><code>create-expo-app</code> — the recommended way to start</summary>

The official CLI for bootstrapping a new Expo project. Always use this over `expo init` (deprecated).

```bash
# Latest (recommended)
npx create-expo-app@latest my-app

# With a specific template
npx create-expo-app@latest my-app --template blank-typescript

# With Expo Router (file-based routing)
npx create-expo-app@latest my-app --template expo-template-blank-typescript

# Then navigate in
cd my-app
npx expo start
```

</details> <details> <summary>Available flags for <code>create-expo-app</code></summary>

|Flag|Short|Description|
|---|---|---|
|`--template <name>`|`-t`|Use a specific template (see below)|
|`--no-install`|—|Skip `npm install`|
|`--yes`|`-y`|Accept all defaults, skip prompts|

```bash
# Skip install (useful in CI)
npx create-expo-app@latest my-app --no-install

# Accept all defaults
npx create-expo-app@latest my-app --yes

# Combine
npx create-expo-app@latest my-app -t blank-typescript --yes
```

</details>

---

## Project Templates

<details> <summary>Official templates</summary>

|Template|Description|
|---|---|
|`blank`|Minimal JS setup — single `App.js`, no navigation|
|`blank-typescript`|Minimal TypeScript setup — single `App.tsx`|
|`tabs`|Expo Router with tab navigation, TypeScript|
|`bare-minimum`|Bare React Native — no Expo modules preinstalled|

```bash
# Default (blank — asks during setup)
npx create-expo-app@latest my-app

# Blank TypeScript
npx create-expo-app@latest my-app -t blank-typescript

# Expo Router with tabs (most feature-rich starting point)
npx create-expo-app@latest my-app -t tabs

# Bare workflow
npx create-expo-app@latest my-app -t bare-minimum
```

</details> <details> <summary>Community & custom templates</summary>

```bash
# Any npm package named expo-template-* or a GitHub repo
npx create-expo-app@latest my-app -t <npm-package-or-github-url>

# Example community templates
npx create-expo-app@latest my-app -t expo-template-blank-typescript
npx create-expo-app@latest my-app -t https://github.com/myorg/my-expo-template
```

</details>

---

## Project Structure

<details> <summary>Managed workflow (default) — no native folders</summary>

```
my-app/
├── app/                      ← Expo Router screens (if using Router)
│   ├── _layout.tsx
│   ├── index.tsx
│   └── (tabs)/
│       ├── _layout.tsx
│       ├── index.tsx
│       └── explore.tsx
├── assets/                   ← Images, fonts, icons
│   ├── fonts/
│   │   └── SpaceMono-Regular.ttf
│   ├── images/
│   │   ├── adaptive-icon.png    ← Android adaptive icon foreground
│   │   ├── favicon.png          ← Web favicon
│   │   ├── icon.png             ← App icon (1024×1024)
│   │   └── splash-icon.png      ← Splash screen image
├── components/               ← Reusable components
├── constants/                ← Colors, sizes, etc.
├── hooks/                    ← Custom hooks
├── .env                      ← Environment variables (don't commit)
├── .env.example              ← Template for env vars (commit this)
├── .gitignore
├── app.json                  ← Expo config
├── babel.config.js           ← Babel config
├── expo-env.d.ts             ← Auto-generated env var types
├── package.json
├── tsconfig.json             ← TypeScript config
└── README.md
```

</details> <details> <summary>Bare workflow — after <code>expo prebuild</code></summary>

After running `npx expo prebuild`, native folders are generated:

```
my-app/
├── android/                  ← Android native project (Gradle)
│   ├── app/
│   │   ├── src/main/
│   │   │   ├── AndroidManifest.xml
│   │   │   └── java/com/myapp/
│   │   └── build.gradle
│   ├── build.gradle
│   └── gradle.properties
├── ios/                      ← iOS native project (Xcode)
│   ├── MyApp/
│   │   ├── AppDelegate.swift
│   │   ├── Info.plist
│   │   └── Images.xcassets/
│   ├── MyApp.xcodeproj/
│   ├── MyApp.xcworkspace/
│   └── Podfile
├── app/                      ← Same as managed
├── app.json
├── package.json
└── ...
```

</details> <details> <summary>Recommended <code>src/</code> directory layout for large apps</summary>

```
my-app/
├── app/                      ← Expo Router (screens only)
│   ├── _layout.tsx
│   └── (tabs)/
├── src/
│   ├── api/                  ← API clients, fetchers
│   │   ├── client.ts
│   │   └── endpoints/
│   ├── components/           ← Shared UI components
│   │   ├── ui/               ← Primitives (Button, Input, etc.)
│   │   └── features/         ← Feature-specific components
│   ├── constants/            ← Colors, sizes, config
│   │   ├── colors.ts
│   │   └── layout.ts
│   ├── hooks/                ← Custom hooks
│   ├── store/                ← State management (Zustand, Redux)
│   ├── types/                ← Global TypeScript types
│   ├── utils/                ← Helper functions
│   └── context/              ← React contexts
├── assets/
├── app.json
├── babel.config.js
├── tsconfig.json
└── package.json
```

</details>

---

## Running the App

<details> <summary><code>npx expo start</code> — start dev server</summary>

Starts the Metro bundler and opens the Expo developer tools.

```bash
npx expo start

# Clear cache (fix weird issues)
npx expo start --clear

# Start in tunnel mode (for physical devices on different networks)
npx expo start --tunnel

# Start in LAN mode (default)
npx expo start --lan

# Start for web
npx expo start --web
```

</details> <details> <summary>Running on specific platforms</summary>

```bash
# Open on iOS Simulator
npx expo run:ios

# Open on Android Emulator
npx expo run:android

# Open in Expo Go (scan QR code)
npx expo start   # then press 'i' (iOS) or 'a' (Android) or 'w' (web)

# Run web
npx expo start --web
```

</details> <details> <summary>Keyboard shortcuts in the dev server</summary>

|Key|Action|
|---|---|
|`i`|Open iOS simulator|
|`a`|Open Android emulator|
|`w`|Open web browser|
|`r`|Reload the app|
|`m`|Toggle menu|
|`j`|Open debugger|
|`o`|Open project in editor|
|`?`|Show all shortcuts|
|`Ctrl+C`|Stop server|

</details>

---

## app.json — Full Config Reference

<details> <summary>Top-level structure</summary>

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/images/icon.png",
    "scheme": "myapp",
    "userInterfaceStyle": "automatic",
    "newArchEnabled": true,
    "ios": { ... },
    "android": { ... },
    "web": { ... },
    "plugins": [ ... ],
    "experiments": { ... },
    "extra": { ... }
  }
}
```

</details> <details> <summary><code>name</code> — display name of the app</summary>

The name shown on the device home screen and in app stores.

|||
|---|---|
|**Type**|`string`|
|**Required**|✅|

```json
{
  "expo": {
    "name": "My Awesome App"
  }
}
```

</details> <details> <summary><code>slug</code> — URL-friendly identifier</summary>

Unique identifier used in Expo's services and deep links. Lowercase, hyphens only.

|||
|---|---|
|**Type**|`string`|
|**Required**|✅|

```json
{
  "expo": {
    "slug": "my-awesome-app"
  }
}
```

</details> <details> <summary><code>version</code> — app version string</summary>

Human-readable version shown in app stores. Follows semver convention.

|||
|---|---|
|**Type**|`string`|
|**Required**|✅|

```json
{
  "expo": {
    "version": "1.2.3"
  }
}
```

> `version` is the display version. For store submission, also set `ios.buildNumber` and `android.versionCode`.

</details> <details> <summary><code>orientation</code> — screen orientation lock</summary>

|||
|---|---|
|**Type**|`'default' \| 'portrait' \| 'landscape'`|
|**Default**|`'default'` (follows device)|

```json
{
  "expo": {
    "orientation": "portrait"
  }
}
```

</details> <details> <summary><code>icon</code> — app icon (1024×1024 PNG)</summary>

Path to the app icon. Must be a PNG, 1024×1024px, no transparency.

|||
|---|---|
|**Type**|`string` (local path)|

```json
{
  "expo": {
    "icon": "./assets/images/icon.png"
  }
}
```

</details> <details> <summary><code>scheme</code> — deep link URL scheme</summary>

The URL scheme used for deep links (`myapp://screen`). Must be unique in the App Store.

|||
|---|---|
|**Type**|`string`|

```json
{
  "expo": {
    "scheme": "myapp"
  }
}
```

```
# Deep link examples:
myapp://          → opens app
myapp://profile   → /profile screen
myapp://posts/123 → /posts/[id] screen
```

</details> <details> <summary><code>userInterfaceStyle</code> — light / dark / automatic</summary>

Controls whether the app supports dark mode.

|||
|---|---|
|**Type**|`'light' \| 'dark' \| 'automatic'`|
|**Default**|`'light'`|

```json
{
  "expo": {
    "userInterfaceStyle": "automatic"
  }
}
```

</details> <details> <summary><code>newArchEnabled</code> — React Native New Architecture</summary>

Enables the New Architecture (Fabric renderer + TurboModules) for both platforms.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true` (Expo SDK 52+)|

```json
{
  "expo": {
    "newArchEnabled": true
  }
}
```

</details> <details> <summary><code>splash</code> — splash screen config</summary>

Configures the native splash screen shown while the app loads.

|Field|Type|Description|
|---|---|---|
|`image`|`string`|Path to splash image (PNG)|
|`resizeMode`|`'contain' \| 'cover' \| 'native'`|How to fit the image|
|`backgroundColor`|`string`|Background color behind image|

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

</details> <details> <summary><code>assetBundlePatterns</code> — bundle assets at build time</summary>

Glob patterns for assets that should be bundled into the app binary (not loaded over the network).

|||
|---|---|
|**Type**|`string[]`|
|**Default**|`["**/*"]`|

```json
{
  "expo": {
    "assetBundlePatterns": [
      "**/*",
      "assets/**"
    ]
  }
}
```

</details> <details> <summary><code>ios</code> — iOS-specific configuration</summary>

|Field|Type|Description|
|---|---|---|
|`bundleIdentifier`|`string`|Unique ID (e.g. `com.company.app`)|
|`buildNumber`|`string`|Build number for App Store (e.g. `"42"`)|
|`icon`|`string`|iOS-specific icon override|
|`supportsTablet`|`boolean`|Whether app supports iPad|
|`requireFullScreen`|`boolean`|Require full screen on iPad|
|`infoPlist`|`object`|Extra Info.plist keys|
|`entitlements`|`object`|Xcode entitlements|
|`associatedDomains`|`string[]`|Universal links domains|
|`usesAppleSignIn`|`boolean`|Enable Sign in with Apple|
|`googleServicesFile`|`string`|Path to `GoogleService-Info.plist`|
|`config`|`object`|Third-party service configs|
|`privacyManifests`|`object`|Apple Privacy Manifests (SDK 51+)|

```json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.mycompany.myapp",
      "buildNumber": "1",
      "supportsTablet": false,
      "requireFullScreen": false,
      "infoPlist": {
        "NSCameraUsageDescription": "Used to scan QR codes",
        "NSLocationWhenInUseUsageDescription": "Used to show nearby places",
        "NSPhotoLibraryUsageDescription": "Used to upload photos",
        "UIBackgroundModes": ["fetch", "remote-notification"]
      },
      "entitlements": {
        "com.apple.developer.associated-domains": [
          "applinks:myapp.com"
        ]
      },
      "associatedDomains": ["applinks:myapp.com"],
      "usesAppleSignIn": true,
      "googleServicesFile": "./GoogleService-Info.plist"
    }
  }
}
```

</details> <details> <summary><code>android</code> — Android-specific configuration</summary>

|Field|Type|Description|
|---|---|---|
|`package`|`string`|Package name (e.g. `com.company.app`)|
|`versionCode`|`number`|Integer build number for Play Store|
|`icon`|`string`|Android-specific icon override|
|`adaptiveIcon`|`object`|Adaptive icon config|
|`permissions`|`string[]`|Required Android permissions|
|`blockedPermissions`|`string[]`|Permissions to explicitly remove|
|`googleServicesFile`|`string`|Path to `google-services.json`|
|`intentFilters`|`array`|Android intent filters (deep links)|
|`softwareKeyboardLayoutMode`|`string`|Keyboard resize mode|
|`allowBackup`|`boolean`|Allow ADB backup|
|`config`|`object`|Third-party service configs|

```json
{
  "expo": {
    "android": {
      "package": "com.mycompany.myapp",
      "versionCode": 1,
      "adaptiveIcon": {
        "foregroundImage": "./assets/images/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "permissions": [
        "android.permission.CAMERA",
        "android.permission.ACCESS_FINE_LOCATION",
        "android.permission.READ_MEDIA_IMAGES"
      ],
      "googleServicesFile": "./google-services.json",
      "intentFilters": [
        {
          "action": "VIEW",
          "autoVerify": true,
          "data": [
            {
              "scheme": "https",
              "host": "myapp.com",
              "pathPrefix": "/"
            }
          ],
          "category": ["BROWSABLE", "DEFAULT"]
        }
      ],
      "softwareKeyboardLayoutMode": "pan"
    }
  }
}
```

</details> <details> <summary><code>web</code> — web-specific configuration</summary>

|Field|Type|Description|
|---|---|---|
|`bundler`|`'metro' \| 'webpack'`|Web bundler (metro recommended)|
|`favicon`|`string`|Path to favicon|
|`name`|`string`|Web app name (PWA)|
|`shortName`|`string`|Short name for home screen|
|`lang`|`string`|HTML lang attribute|
|`themeColor`|`string`|Browser theme color|
|`backgroundColor`|`string`|PWA background color|
|`output`|`'static' \| 'server' \| 'single'`|Build output mode|

```json
{
  "expo": {
    "web": {
      "bundler": "metro",
      "output": "static",
      "favicon": "./assets/images/favicon.png",
      "name": "My App",
      "shortName": "MyApp",
      "lang": "en",
      "themeColor": "#6366f1",
      "backgroundColor": "#ffffff"
    }
  }
}
```

</details> <details> <summary><code>plugins</code> — Expo config plugins</summary>

Config plugins modify native project files during `expo prebuild`. Use for packages that require native setup.

```json
{
  "expo": {
    "plugins": [
      "expo-router",
      "expo-font",
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access your camera."
        }
      ],
      [
        "expo-location",
        {
          "locationWhenInUsePermission": "Allow $(PRODUCT_NAME) to use your location."
        }
      ],
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#ffffff",
          "sounds": ["./assets/notification.wav"]
        }
      ],
      [
        "react-native-google-mobile-ads",
        {
          "androidAppId": "ca-app-pub-xxx~xxx",
          "iosAppId": "ca-app-pub-xxx~xxx"
        }
      ]
    ]
  }
}
```

</details> <details> <summary><code>experiments</code> — experimental features</summary>

```json
{
  "expo": {
    "experiments": {
      "typedRoutes": true,        // TypeScript route autocomplete
      "reactCanary": false        // React canary builds
    }
  }
}
```

</details> <details> <summary><code>extra</code> — custom config values (app.config.js only)</summary>

Pass arbitrary values accessible at runtime via `expo-constants`. Better replaced by environment variables in modern apps.

```json
{
  "expo": {
    "extra": {
      "apiUrl": "https://api.myapp.com",
      "featureFlags": {
        "newDashboard": true
      },
      "eas": {
        "projectId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
      }
    }
  }
}
```

```ts
// Access at runtime
import Constants from 'expo-constants';
const apiUrl = Constants.expoConfig?.extra?.apiUrl;
```

</details> <details> <summary><code>owner</code> and <code>runtimeVersion</code></summary>

```json
{
  "expo": {
    "owner": "my-expo-account",           // Expo account username or org
    "runtimeVersion": {
      "policy": "appVersion"              // OTA update compatibility
    }
  }
}
```

**runtimeVersion policies:**

|Policy|Description|
|---|---|
|`appVersion`|New native build required when `version` changes|
|`nativeVersion`|Tracks native code changes more granularly|
|`sdkVersion`|Tied to Expo SDK version (not recommended)|
|`string`|Manual version string (full control)|

</details>

---

## app.config.js — Dynamic Config

<details> <summary>Why use <code>app.config.js</code> instead of <code>app.json</code></summary>

`app.config.js` is JavaScript — it can read environment variables, compute values, and use conditional logic. Use it when:

- You need different configs per environment (dev / staging / prod)
- Config values come from environment variables
- You want to reuse values (DRY)

```js
// app.config.js replaces app.json
module.exports = {
  expo: {
    name: 'My App',
    slug: 'my-app',
    version: '1.0.0',
    // ...
  }
};
```

> ⚠️ You can have **either** `app.json` or `app.config.js` — not both (`.js` takes precedence if both exist).

</details> <details> <summary>Reading environment variables in app.config.js</summary>

```js
// app.config.js
module.exports = ({ config }) => ({
  ...config,
  expo: {
    ...config.expo,
    name: process.env.APP_ENV === 'production' ? 'My App' : 'My App (Dev)',
    ios: {
      ...config.expo?.ios,
      bundleIdentifier: process.env.APP_ENV === 'production'
        ? 'com.mycompany.myapp'
        : 'com.mycompany.myapp.dev',
    },
    android: {
      ...config.expo?.android,
      package: process.env.APP_ENV === 'production'
        ? 'com.mycompany.myapp'
        : 'com.mycompany.myapp.dev',
    },
    extra: {
      apiUrl: process.env.EXPO_PUBLIC_API_URL,
      eas: {
        projectId: process.env.EAS_PROJECT_ID,
      },
    },
  },
});
```

</details> <details> <summary>Multi-environment config (dev / staging / production)</summary>

```js
// app.config.js
const ENV = process.env.APP_ENV ?? 'development';

const envConfig = {
  development: {
    name: 'My App (Dev)',
    bundleId: 'com.mycompany.myapp.dev',
    package: 'com.mycompany.myapp.dev',
    icon: './assets/icon-dev.png',
    apiUrl: 'http://localhost:3000',
  },
  staging: {
    name: 'My App (Staging)',
    bundleId: 'com.mycompany.myapp.staging',
    package: 'com.mycompany.myapp.staging',
    icon: './assets/icon-staging.png',
    apiUrl: 'https://staging-api.myapp.com',
  },
  production: {
    name: 'My App',
    bundleId: 'com.mycompany.myapp',
    package: 'com.mycompany.myapp',
    icon: './assets/icon.png',
    apiUrl: 'https://api.myapp.com',
  },
};

const current = envConfig[ENV] ?? envConfig.development;

module.exports = {
  expo: {
    name: current.name,
    slug: 'my-app',
    version: '1.0.0',
    icon: current.icon,
    ios: {
      bundleIdentifier: current.bundleId,
      buildNumber: '1',
    },
    android: {
      package: current.package,
      versionCode: 1,
    },
    extra: {
      appEnv: ENV,
      apiUrl: current.apiUrl,
    },
  },
};
```

</details>

---

## Environment Variables

<details> <summary>Public env vars — <code>EXPO_PUBLIC_</code> prefix</summary>

Variables prefixed with `EXPO_PUBLIC_` are automatically bundled into the JS bundle and accessible at runtime.

```bash
# .env
EXPO_PUBLIC_API_URL=https://api.myapp.com
EXPO_PUBLIC_STRIPE_KEY=pk_live_xxx
EXPO_PUBLIC_SENTRY_DSN=https://xxx@sentry.io/xxx
EXPO_PUBLIC_APP_ENV=production
```

```ts
// Access anywhere in your app
const apiUrl = process.env.EXPO_PUBLIC_API_URL;
const stripeKey = process.env.EXPO_PUBLIC_STRIPE_KEY;
```

> ⚠️ `EXPO_PUBLIC_` vars are public — bundled into the JS. Never put secrets (private API keys, tokens) here.

</details> <details> <summary>Private env vars — EAS Secrets</summary>

Secrets used at build time only (not in the JS bundle). Set via EAS CLI — not in `.env`.

```bash
# Set EAS secrets (build-time only — never in JS bundle)
eas secret:create --scope project --name SENTRY_AUTH_TOKEN --value xxx
eas secret:create --scope project --name GOOGLE_SERVICES_JSON --type file --value ./google-services.json

# List secrets
eas secret:list

# Delete a secret
eas secret:delete --id <id>
```

</details> <details> <summary>Environment files — <code>.env</code>, <code>.env.local</code>, <code>.env.production</code></summary>

Expo SDK 49+ uses `dotenv` automatically. Files are loaded in priority order:

|File|When loaded|Committed?|
|---|---|---|
|`.env`|Always|⚠️ Only if no secrets|
|`.env.local`|Local only|❌ No (in .gitignore)|
|`.env.development`|`expo start`|⚠️ Maybe|
|`.env.production`|Production builds|⚠️ Maybe|
|`.env.development.local`|Local dev only|❌ No|

```bash
# .env (base — commit safe values only)
EXPO_PUBLIC_APP_ENV=development
EXPO_PUBLIC_API_URL=https://api.myapp.com

# .env.local (local overrides — never commit)
EXPO_PUBLIC_API_URL=http://localhost:3000

# .env.production (production values)
EXPO_PUBLIC_APP_ENV=production
EXPO_PUBLIC_API_URL=https://api.myapp.com
```

```bash
# .gitignore
.env.local
.env.*.local
.env.production
```

</details> <details> <summary>TypeScript types for env vars — <code>expo-env.d.ts</code></summary>

Expo auto-generates `expo-env.d.ts` for public env vars. Add your own types here.

```ts
// expo-env.d.ts (auto-generated, extend as needed)
/// <reference types="expo/types/env.d.ts" />

// Extend with your env vars for autocomplete
declare global {
  namespace NodeJS {
    interface ProcessEnv {
      EXPO_PUBLIC_API_URL: string;
      EXPO_PUBLIC_APP_ENV: 'development' | 'staging' | 'production';
      EXPO_PUBLIC_STRIPE_KEY: string;
    }
  }
}
```

</details>

---

## package.json Scripts

<details> <summary>Default scripts and what they do</summary>

```json
{
  "scripts": {
    "start":    "expo start",
    "android":  "expo run:android",
    "ios":      "expo run:ios",
    "web":      "expo start --web",
    "test":     "jest --watchAll",
    "lint":     "eslint . --ext .js,.jsx,.ts,.tsx",
    "prebuild": "expo prebuild"
  }
}
```

</details> <details> <summary>Recommended additional scripts</summary>

```json
{
  "scripts": {
    "start":           "expo start",
    "start:clear":     "expo start --clear",
    "start:tunnel":    "expo start --tunnel",
    "android":         "expo run:android",
    "ios":             "expo run:ios",
    "web":             "expo start --web",

    "prebuild":        "expo prebuild --clean",
    "prebuild:ios":    "expo prebuild --platform ios --clean",
    "prebuild:android":"expo prebuild --platform android --clean",

    "build:dev":       "eas build --profile development",
    "build:preview":   "eas build --profile preview",
    "build:prod":      "eas build --profile production",

    "submit:ios":      "eas submit -p ios",
    "submit:android":  "eas submit -p android",

    "update":          "eas update",
    "update:prod":     "eas update --branch production",

    "lint":            "eslint . --ext .js,.jsx,.ts,.tsx",
    "type-check":      "tsc --noEmit",
    "test":            "jest --watchAll=false",
    "test:watch":      "jest --watchAll",
    "test:ci":         "jest --ci --coverage"
  }
}
```

</details>

---

## Babel Config

<details> <summary>Default <code>babel.config.js</code></summary>

```js
// babel.config.js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
  };
};
```

</details> <details> <summary>With Reanimated and other common plugins</summary>

```js
// babel.config.js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      // Required for react-native-reanimated — must be LAST
      'react-native-reanimated/plugin',
    ],
  };
};
```

> ⚠️ `react-native-reanimated/plugin` must always be **last** in the plugins array.

</details> <details> <summary>With module path aliases</summary>

```bash
npm install --save-dev babel-plugin-module-resolver
```

```js
// babel.config.js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      [
        'module-resolver',
        {
          root: ['./'],
          alias: {
            '@': './src',
            '@components': './src/components',
            '@hooks': './src/hooks',
            '@store': './src/store',
            '@utils': './src/utils',
            '@api': './src/api',
            '@assets': './assets',
          },
          extensions: ['.ios.js', '.android.js', '.js', '.ts', '.tsx', '.json'],
        },
      ],
      // Reanimated must be last
      'react-native-reanimated/plugin',
    ],
  };
};
```

```ts
// tsconfig.json — mirror the aliases for TypeScript
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@hooks/*": ["src/hooks/*"],
      "@store/*": ["src/store/*"],
      "@utils/*": ["src/utils/*"],
      "@api/*": ["src/api/*"],
      "@assets/*": ["assets/*"]
    }
  }
}
```

</details>

---

## Metro Config

<details> <summary>Default <code>metro.config.js</code></summary>

```js
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config');

const config = getDefaultConfig(__dirname);

module.exports = config;
```

</details> <details> <summary>Support SVG imports</summary>

```bash
npm install react-native-svg react-native-svg-transformer
```

```js
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config');

const config = getDefaultConfig(__dirname);

const { transformer, resolver } = config;

config.transformer = {
  ...transformer,
  babelTransformerPath: require.resolve('react-native-svg-transformer'),
};

config.resolver = {
  ...resolver,
  assetExts: resolver.assetExts.filter(ext => ext !== 'svg'),
  sourceExts: [...resolver.sourceExts, 'svg'],
};

module.exports = config;
```

```ts
// types/svg.d.ts — TypeScript support for SVG imports
declare module '*.svg' {
  import React from 'react';
  import { SvgProps } from 'react-native-svg';
  const content: React.FC<SvgProps>;
  export default content;
}
```

</details> <details> <summary>Enable CSS Modules / NativeWind</summary>

```bash
npm install nativewind
npm install --save-dev tailwindcss
npx tailwindcss init
```

```js
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config');
const { withNativeWind } = require('nativewind/metro');

const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, { input: './global.css' });
```

</details>

---

## TypeScript Config

<details> <summary>Default <code>tsconfig.json</code></summary>

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true
  },
  "include": [
    "**/*.ts",
    "**/*.tsx",
    ".expo/types/**/*.d.ts",
    "expo-env.d.ts"
  ]
}
```

</details> <details> <summary>Recommended extended config</summary>

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "exactOptionalPropertyTypes": false,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@hooks/*": ["src/hooks/*"],
      "@store/*": ["src/store/*"],
      "@utils/*": ["src/utils/*"],
      "@api/*": ["src/api/*"]
    }
  },
  "include": [
    "**/*.ts",
    "**/*.tsx",
    ".expo/types/**/*.d.ts",
    "expo-env.d.ts"
  ],
  "exclude": [
    "node_modules",
    "android",
    "ios",
    "dist"
  ]
}
```

</details>

---

## EAS — Expo Application Services

<details> <summary>Setup EAS for your project</summary>

```bash
# Install EAS CLI globally
npm install -g eas-cli

# Login to Expo account
eas login

# Initialize EAS in your project (creates eas.json)
eas init

# Link to existing EAS project or create new
eas project:init
```

</details> <details> <summary>Key EAS commands</summary>

|Command|Description|
|---|---|
|`eas build`|Build app binary|
|`eas build -p ios`|Build for iOS only|
|`eas build -p android`|Build for Android only|
|`eas build --profile development`|Build with dev profile|
|`eas submit`|Submit to App Store / Play Store|
|`eas update`|Send OTA JavaScript update|
|`eas update --branch production`|Update production channel|
|`eas credentials`|Manage signing certs and keys|
|`eas secret:create`|Add a build secret|
|`eas secret:list`|List all secrets|
|`eas build:list`|List recent builds|
|`eas build:cancel`|Cancel a running build|

</details>

---

## eas.json — Build Profiles

<details> <summary>Full <code>eas.json</code> reference</summary>

```json
{
  "cli": {
    "version": ">= 10.0.0",
    "appVersionSource": "remote"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "simulator": true
      },
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleDebug"
      },
      "env": {
        "APP_ENV": "development",
        "EXPO_PUBLIC_API_URL": "http://localhost:3000"
      }
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "simulator": false
      },
      "android": {
        "buildType": "apk"
      },
      "env": {
        "APP_ENV": "staging",
        "EXPO_PUBLIC_API_URL": "https://staging-api.myapp.com"
      }
    },
    "production": {
      "autoIncrement": true,
      "ios": {
        "resourceClass": "m-medium"
      },
      "android": {
        "buildType": "app-bundle"
      },
      "env": {
        "APP_ENV": "production",
        "EXPO_PUBLIC_API_URL": "https://api.myapp.com"
      }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "dev@mycompany.com",
        "ascAppId": "1234567890",
        "appleTeamId": "XXXXXXXXXX"
      },
      "android": {
        "serviceAccountKeyPath": "./play-store-key.json",
        "track": "internal"
      }
    }
  }
}
```

</details> <details> <summary>Build profile fields reference</summary>

|Field|Type|Description|
|---|---|---|
|`developmentClient`|`boolean`|Build a development client (expo-dev-client)|
|`distribution`|`'store' \| 'internal'`|Where the build is distributed|
|`autoIncrement`|`boolean \| 'version' \| 'buildNumber'`|Auto-increment version/build number|
|`env`|`object`|Environment variables injected at build time|
|`extends`|`string`|Inherit from another profile|
|`channel`|`string`|OTA update channel name|
|`releaseChannel`|`string`|Classic update channel (deprecated)|
|`withoutCredentials`|`boolean`|Skip signing (simulator builds)|
|`credentialsSource`|`'local' \| 'remote'`|Where to get signing credentials|
|`cache.key`|`string`|Custom cache key for build cache|
|`ios.simulator`|`boolean`|Build for iOS simulator|
|`ios.resourceClass`|`string`|Build machine tier|
|`android.buildType`|`'apk' \| 'app-bundle'`|Output format|
|`android.gradleCommand`|`string`|Custom Gradle task|

```json
{
  "build": {
    "staging": {
      "extends": "production",
      "distribution": "internal",
      "env": {
        "EXPO_PUBLIC_API_URL": "https://staging-api.myapp.com"
      }
    }
  }
}
```

</details>

---

## Installing Packages

<details> <summary>Always use <code>npx expo install</code> — not npm/yarn directly</summary>

`npx expo install` resolves the **correct compatible version** of a package for your current Expo SDK. Using `npm install` can install incompatible versions.

```bash
# ✅ Correct — resolves version compatible with your SDK
npx expo install expo-camera
npx expo install react-native-safe-area-context
npx expo install expo-file-system lottie-react-native

# ❌ Avoid for Expo packages — may install wrong version
npm install expo-camera
```

</details> <details> <summary>Installing non-Expo packages</summary>

For packages not in the Expo ecosystem (Zustand, Axios, etc.), use your package manager directly.

```bash
# Third-party packages — npm/yarn/bun is fine
npm install zustand axios react-query zod
npm install --save-dev @types/node typescript
```

</details> <details> <summary>Check for outdated or incompatible packages</summary>

```bash
# Check for version mismatches
npx expo install --check

# Fix all incompatible versions automatically
npx expo install --fix
```

</details>

---

## Expo SDK Upgrades

<details> <summary>Upgrade to a new Expo SDK version</summary>

```bash
# Upgrade to the latest SDK
npx expo install expo@latest

# Upgrade all Expo packages to the new SDK versions
npx expo install --fix

# Or use the upgrade command
npx expo upgrade
```

</details> <details> <summary>Step-by-step upgrade process</summary>

```bash
# 1. Check current SDK version
cat app.json | grep sdkVersion

# 2. Read the migration guide at https://expo.dev/changelog

# 3. Upgrade the expo package
npx expo install expo@latest

# 4. Upgrade all related packages
npx expo install --fix

# 5. Clear all caches
npx expo start --clear

# 6. If using bare workflow — regenerate native files
npx expo prebuild --clean

# 7. iOS — reinstall pods
cd ios && pod install && cd ..

# 8. Test on both platforms
npx expo run:ios
npx expo run:android
```

</details>

---

## Prebuild — Bare Workflow

<details> <summary><code>npx expo prebuild</code> — generate native folders</summary>

Generates the `ios/` and `android/` folders from your `app.json` config and installed plugins. Replaces the old "eject" workflow.

```bash
# Generate both platforms
npx expo prebuild

# Clean existing native folders first (recommended)
npx expo prebuild --clean

# Generate for one platform only
npx expo prebuild --platform ios
npx expo prebuild --platform android
```

> ⚠️ After prebuild, you must run `pod install` in the `ios/` folder.

</details> <details> <summary>When to run prebuild</summary>

- Adding a native module that requires manual native setup
- After changing `plugins` in `app.json`
- After changing `bundleIdentifier` or `package` name
- Setting up CI/CD with native builds
- Debugging native issues

```bash
# After adding new plugins in app.json
npx expo prebuild --clean
cd ios && pod install && cd ..
```

</details>

---

## Common First Steps

<details> <summary>1 — Set up ESLint and Prettier</summary>

```bash
npx expo install --save-dev eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-plugin-react eslint-plugin-react-hooks prettier eslint-config-prettier
```

```js
// .eslintrc.js
module.exports = {
  root: true,
  extends: [
    'expo',
    '@typescript-eslint/recommended',
    'prettier',
  ],
  plugins: ['@typescript-eslint', 'react-hooks'],
  rules: {
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    '@typescript-eslint/no-unused-vars': 'warn',
    '@typescript-eslint/no-explicit-any': 'warn',
  },
};
```

```json
// .prettierrc
{
  "singleQuote": true,
  "trailingComma": "all",
  "semi": true,
  "tabWidth": 2,
  "printWidth": 100,
  "bracketSpacing": true
}
```

</details> <details> <summary>2 — Set up absolute imports with path aliases</summary>

```bash
npm install --save-dev babel-plugin-module-resolver
```

```js
// babel.config.js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      ['module-resolver', {
        root: ['./'],
        alias: { '@': './src' },
      }],
      'react-native-reanimated/plugin', // last
    ],
  };
};
```

```json
// tsconfig.json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```

</details> <details> <summary>3 — Install essential packages for most apps</summary>

```bash
# Navigation + safe area
npx expo install react-native-safe-area-context react-native-screens

# Expo Router (if not already set up)
npx expo install expo-router

# Fonts
npx expo install expo-font @expo-google-fonts/inter

# Storage
npx expo install react-native-mmkv

# Gestures + animations
npx expo install react-native-gesture-handler react-native-reanimated

# Image handling
npx expo install expo-image

# Status bar
npx expo install expo-status-bar

# State management
npm install zustand

# API / data fetching
npm install @tanstack/react-query axios

# Form validation
npm install react-hook-form zod @hookform/resolvers
```

</details> <details> <summary>4 — Configure splash screen and app icon</summary>

```
assets/
├── images/
│   ├── icon.png              1024×1024 PNG — no transparency
│   ├── adaptive-icon.png     1024×1024 PNG — Android foreground
│   ├── splash-icon.png       Your logo for splash screen
│   └── favicon.png           32×32 PNG — web
```

```json
// app.json
{
  "expo": {
    "icon": "./assets/images/icon.png",
    "splash": {
      "image": "./assets/images/splash-icon.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/images/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      }
    },
    "web": {
      "favicon": "./assets/images/favicon.png"
    }
  }
}
```

</details> <details> <summary>5 — Initialize EAS and set up build profiles</summary>

```bash
# Install EAS CLI
npm install -g eas-cli

# Login
eas login

# Initialize (creates eas.json, links project)
eas init

# Configure credentials
eas credentials

# First dev build (creates expo-dev-client binary)
eas build --profile development --platform ios
eas build --profile development --platform android
```

</details> <details> <summary>6 — Set up OTA updates</summary>

```bash
# Configure update channel
eas update:configure

# Send an update to production
eas update --branch production --message "Fix login crash"

# Send to staging
eas update --branch staging --message "Test new feature"
```

```json
// app.json — enable OTA updates
{
  "expo": {
    "updates": {
      "url": "https://u.expo.dev/your-project-id",
      "enabled": true,
      "checkAutomatically": "ON_LOAD",
      "fallbackToCacheTimeout": 0
    },
    "runtimeVersion": {
      "policy": "appVersion"
    }
  }
}
```

</details> <details> <summary>7 — Configure <code>.gitignore</code></summary>

```bash
# .gitignore

# Expo
.expo/
dist/
web-build/

# Environment
.env.local
.env.*.local
.env.production

# Native (if managed workflow — no ios/android folders)
ios/
android/

# Dependencies
node_modules/

# Secrets
*.pem
*.p12
*.keystore
google-services.json
GoogleService-Info.plist
play-store-key.json

# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/
```

> ⚠️ If using **bare workflow** (ios/ and android/ are committed), remove them from .gitignore.

</details>

---

## Quick-Reference Cheatsheet

|Command / File|Use case|
|---|---|
|`npx create-expo-app@latest`|Create a new project|
|`npx expo start`|Start dev server|
|`npx expo start --clear`|Start with cleared cache|
|`npx expo run:ios`|Run on iOS|
|`npx expo run:android`|Run on Android|
|`npx expo install <pkg>`|Install SDK-compatible package|
|`npx expo install --fix`|Fix incompatible package versions|
|`npx expo prebuild --clean`|Generate/regenerate native folders|
|`npx expo upgrade`|Upgrade to new SDK|
|`eas init`|Initialize EAS for project|
|`eas build --profile development`|Build dev client|
|`eas build --profile production`|Build for store|
|`eas submit`|Submit to App Store / Play Store|
|`eas update`|Send OTA update|
|`eas secret:create`|Add a build secret|
|`app.json`|Static Expo config|
|`app.config.js`|Dynamic config with env vars|
|`eas.json`|Build and submit profiles|
|`EXPO_PUBLIC_` prefix|Public runtime env vars|
|`expo-env.d.ts`|TypeScript types for env vars|
|`.env.local`|Local env overrides (never commit)|
|`babel-preset-expo`|Required Babel preset|
|`reanimated/plugin`|Must be last Babel plugin|

---

_Reference based on Expo SDK 52+ and EAS CLI 10+. Always check the [official docs](https://docs.expo.dev/) for the latest updates._