# React Native / Expo Game Development Libraries

A curated list of libraries useful for building games (brain training, casual, arcade, multiplayer) in React Native / Expo, with npm links.

---

## 🎮 Game Engines

|Library|Description|npm|
|---|---|---|
|react-native-game-engine|Lightweight 2D game engine with entity-component system, game loop, touch handling|https://www.npmjs.com/package/react-native-game-engine|
|matter-js|2D physics engine (gravity, collisions, constraints) — pairs well with game-engine|https://www.npmjs.com/package/matter-js|
|react-native-skia (@shopify/react-native-skia)|High-performance 2D graphics rendering (canvas-like), great for custom game visuals|https://www.npmjs.com/package/@shopify/react-native-skia|
|react-three-fiber + expo-three|3D rendering using Three.js inside React Native/Expo|https://www.npmjs.com/package/expo-three|
|react-native-gl-view / expo-gl|Raw OpenGL ES access for custom rendering engines|https://www.npmjs.com/package/expo-gl|
|pixi.js (via WebView)|2D WebGL renderer, usable inside a WebView for HTML5-style games|https://www.npmjs.com/package/pixi.js|

---

## 🧩 Animation & Gestures (core for any interactive game)

|Library|Description|npm|
|---|---|---|
|react-native-reanimated|High-performance animations, smooth 60fps UI thread animations|https://www.npmjs.com/package/react-native-reanimated|
|react-native-gesture-handler|Native-driven gesture recognition (swipe, drag, pinch, tap)|https://www.npmjs.com/package/react-native-gesture-handler|
|react-native-svg|SVG rendering — useful for vector-based game elements/icons|https://www.npmjs.com/package/react-native-svg|
|lottie-react-native|Play After Effects animations (great for win/lose screens, effects)|https://www.npmjs.com/package/lottie-react-native|
|react-native-animatable|Simple declarative animations (bounce, shake, fade)|https://www.npmjs.com/package/react-native-animatable|
|moti|Universal animation library built on Reanimated, simpler API|https://www.npmjs.com/package/moti|

---

## 🔊 Audio / Sound Effects

|Library|Description|npm|
|---|---|---|
|expo-av|Play sound effects, background music, haptics-synced audio|https://www.npmjs.com/package/expo-av|
|react-native-sound|Native sound playback (alternative to expo-av for bare RN)|https://www.npmjs.com/package/react-native-sound|
|expo-haptics|Vibration/haptic feedback for taps, wins, collisions|https://www.npmjs.com/package/expo-haptics|

---

## 🧠 Brain-Training / Puzzle Specific Helpers

|Library|Description|npm|
|---|---|---|
|react-native-draggable-flatlist|Drag-and-drop reordering — useful for sliding puzzles, sorting games|https://www.npmjs.com/package/react-native-draggable-flatlist|
|react-native-sortable-grid|Grid-based drag/sort, good for memory match or tile games|https://www.npmjs.com/package/react-native-sortable-grid|
|seedrandom|Seedable random number generator — for daily-challenge style deterministic puzzles|https://www.npmjs.com/package/seedrandom|
|dayjs|Lightweight date library — for daily streaks, daily puzzle resets|https://www.npmjs.com/package/dayjs|

---

## 🌐 Multiplayer / Backend / Realtime

|Library|Description|npm|
|---|---|---|
|socket.io-client|Realtime multiplayer communication (turn-based or live)|https://www.npmjs.com/package/socket.io-client|
|firebase|Realtime Database/Firestore for leaderboards, matchmaking, multiplayer state|https://www.npmjs.com/package/firebase|
|@supabase/supabase-js|Postgres-backed backend, realtime subscriptions, auth — good Firebase alternative|https://www.npmjs.com/package/@supabase/supabase-js|
|colyseus.js|Dedicated multiplayer game server framework client|https://www.npmjs.com/package/colyseus.js|

---

## 💾 Local Storage (offline games, high scores, progress)

|Library|Description|npm|
|---|---|---|
|@react-native-async-storage/async-storage|Simple key-value local storage|https://www.npmjs.com/package/@react-native-async-storage/async-storage|
|expo-sqlite|Local SQLite database — structured data (levels, scores, stats)|https://www.npmjs.com/package/expo-sqlite|
|react-native-mmkv|Extremely fast key-value storage, great for frequent score/state writes|https://www.npmjs.com/package/react-native-mmkv|
|redux-persist|Persist Redux game state across app restarts|https://www.npmjs.com/package/redux-persist|

---

## 🗂 State Management (managing game logic/state)

|Library|Description|npm|
|---|---|---|
|zustand|Minimal state management, popular for game state machines|https://www.npmjs.com/package/zustand|
|redux + @reduxjs/toolkit|Predictable state container for complex game logic|https://www.npmjs.com/package/@reduxjs/toolkit|
|xstate|Finite state machines — excellent for game states (menu/playing/paused/gameover)|https://www.npmjs.com/package/xstate|
|jotai|Atomic state management, lightweight alternative to Redux|https://www.npmjs.com/package/jotai|

---

## 💰 Monetization (ads, IAP — essential for earning on Play Store)

|Library|Description|npm|
|---|---|---|
|react-native-google-mobile-ads|AdMob banner/interstitial/rewarded ads (official Google library)|https://www.npmjs.com/package/react-native-google-mobile-ads|
|react-native-iap|In-app purchases (remove ads, unlock levels, pro features)|https://www.npmjs.com/package/react-native-iap|
|expo-in-app-purchases|Expo-managed IAP (works with EAS Build)|https://www.npmjs.com/package/expo-in-app-purchases|

---

## 📊 Analytics & Crash Reporting

|Library|Description|npm|
|---|---|---|
|@react-native-firebase/analytics|Track player behavior, retention, level drop-off|https://www.npmjs.com/package/@react-native-firebase/analytics|
|@sentry/react-native|Crash reporting — critical for catching game-breaking bugs post-launch|https://www.npmjs.com/package/@sentry/react-native|

---

## 🏆 Leaderboards / Social

|Library|Description|npm|
|---|---|---|
|react-native-game-center (iOS)|Apple Game Center integration (leaderboards/achievements)|https://www.npmjs.com/package/react-native-game-center|
|react-native-google-play-game-services|Google Play Games Services (leaderboards/achievements on Android)|https://www.npmjs.com/package/react-native-google-play-game-services|

---

## 🛠 Recommended Stack by Game Type

- **Memory/Match/Puzzle (offline):** react-native-reanimated + react-native-gesture-handler + expo-sqlite + react-native-google-mobile-ads
- **Endless Runner / Arcade:** react-native-game-engine + matter-js + expo-av + expo-haptics
- **Trivia/Daily Challenge (backend):** firebase or supabase-js + dayjs + react-native-iap
- **Multiplayer (Tic-Tac-Toe, Connect4):** socket.io-client or firebase realtime + zustand
- **Custom-rendered visual games:** @shopify/react-native-skia + react-native-reanimated

---

_Note: Always check Expo compatibility before installing — prefer Expo SDK modules (expo-av, expo-sqlite, expo-gl, expo-haptics) when using Expo managed workflow to avoid needing to eject/prebuild. For libraries requiring native code not in Expo Go, use `expo prebuild` or EAS Build (Expo's "development build")._