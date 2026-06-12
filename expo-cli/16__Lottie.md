# React Native Lottie — Complete Reference

A comprehensive reference for `lottie-react-native` — rendering After Effects animations in React Native.  
Covers all props, ref methods, source types, color filters, progress control, and common patterns.

> **Install:**
> 
> ```bash
> # Expo managed workflow
> npx expo install lottie-react-native
> 
> # Bare React Native
> npm install lottie-react-native
> cd ios && pod install
> ```

---

## Table of Contents

1. [Overview — How Lottie Works](#overview--how-lottie-works)
2. [Basic Usage](#basic-usage)
3. [Source Types](#source-types)
4. [Core Props](#core-props)
5. [Playback Props](#playback-props)
6. [Style & Layout Props](#style--layout-props)
7. [Color Filters](#color-filters)
8. [Text Filters](#text-filters)
9. [Platform-specific Props](#platform-specific-props)
10. [Event Callbacks](#event-callbacks)
11. [Ref Methods](#ref-methods)
12. [Progress Control — Animated API](#progress-control--animated-api)
13. [Progress Control — Reanimated](#progress-control--reanimated)
14. [Render Mode](#render-mode)
15. [Resize Mode](#resize-mode)
16. [DotLottie Format](#dotlottie-format)
17. [Common Patterns](#common-patterns)

---

## Overview — How Lottie Works

<details> <summary>What is Lottie and how animations are exported</summary>

Lottie renders animations exported from **Adobe After Effects** as JSON using the **Bodymovin** plugin. The JSON describes shapes, paths, colors, and keyframes — no videos or GIFs needed.

|Feature|Description|
|---|---|
|Format|JSON (`.json`) or DotLottie (`.lottie`)|
|Resolution|Fully vector — scales to any size without blurring|
|File size|Typically 5–100 KB vs MB for GIFs|
|Platforms|iOS, Android, Web|
|Renderer|Native on iOS (Lottie-iOS), Android (Lottie-Android)|
|Control|Play, pause, loop, scrub with progress value|
|Color override|Runtime color replacement via `colorFilters`|

**Animation sources:**

- [LottieFiles](https://lottiefiles.com) — free & premium animations
- [IconScout](https://iconscout.com/lottie-animations) — free animations
- Adobe After Effects + Bodymovin plugin — custom animations

</details>

---

## Basic Usage

<details> <summary>Minimal setup</summary>

```tsx
import LottieView from 'lottie-react-native';
import { View } from 'react-native';

export default function App() {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <LottieView
        source={require('./assets/animations/loading.json')}
        autoPlay
        loop
        style={{ width: 200, height: 200 }}
      />
    </View>
  );
}
```

> ⚠️ Always specify `width` and `height` in `style`. Without explicit dimensions the animation won't render.

</details>

---

## Source Types

<details> <summary>Local JSON file — <code>require('./animation.json')</code></summary>

The most common and performant source. Bundle the JSON with your app.

```tsx
<LottieView
  source={require('./assets/animations/success.json')}
  autoPlay
  loop={false}
  style={{ width: 150, height: 150 }}
/>
```

</details> <details> <summary>Inline JSON object</summary>

Pass a parsed JSON object directly. Useful when the animation data comes from an API or is generated dynamically.

```tsx
import animationData from './assets/data.json';

<LottieView
  source={animationData}
  autoPlay
  loop
  style={{ width: 200, height: 200 }}
/>

// Or from state/API
const [animation, setAnimation] = useState(null);

useEffect(() => {
  fetch('https://api.example.com/animation.json')
    .then(res => res.json())
    .then(setAnimation);
}, []);

{animation && (
  <LottieView
    source={animation}
    autoPlay
    loop
    style={{ width: 200, height: 200 }}
  />
)}
```

</details> <details> <summary>Remote URL string</summary>

Pass a URL string to load the animation from a remote server. Requires network access.

```tsx
<LottieView
  source={{ uri: 'https://assets.lottiefiles.com/packages/lf20_animation.json' }}
  autoPlay
  loop
  style={{ width: 200, height: 200 }}
/>
```

> ⚠️ Remote sources add a network fetch on render. Prefer bundling animations locally for critical UI. Use remote loading for large or user-specific animations.

</details> <details> <summary>DotLottie format — <code>.lottie</code> file</summary>

DotLottie (`.lottie`) is a newer compressed format — typically 2–3× smaller than JSON. Requires `lottie-react-native` 6.0+.

```tsx
<LottieView
  source={require('./assets/animation.lottie')}
  autoPlay
  loop
  style={{ width: 200, height: 200 }}
/>
```

See [DotLottie Format](#dotlottie-format) for full details.

</details>

---

## Core Props

<details> <summary><code>source</code> — <em>object | string | number</em> ⚠️ Required</summary>

The animation source. Accepts a `require()` result, an inline JSON object, or a URL string.

|||
|---|---|
|**Type**|`AnimationObject \| string \| { uri: string }`|
|**Required**|Yes|

```tsx
// Local file
source={require('./animation.json')}

// Inline object
source={animationData}

// Remote URL
source={{ uri: 'https://example.com/animation.json' }}

// DotLottie
source={require('./animation.lottie')}
```

</details> <details> <summary><code>style</code> — <em>StyleProp&lt;ViewStyle&gt;</em> ⚠️ Always set width/height</summary>

Style for the LottieView container. **Must include `width` and `height`** for the animation to render.

|||
|---|---|
|**Type**|`StyleProp<ViewStyle>`|

```tsx
// Fixed size
<LottieView style={{ width: 200, height: 200 }} ... />

// Responsive
<LottieView style={{ width: '100%', aspectRatio: 1 }} ... />

// Fill parent
<LottieView style={{ flex: 1 }} ... />
```

</details>

---

## Playback Props

<details> <summary><code>autoPlay</code> — <em>boolean</em></summary>

If `true`, the animation starts playing as soon as it loads.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```tsx
<LottieView source={source} autoPlay style={{ width: 200, height: 200 }} />
```

</details> <details> <summary><code>loop</code> — <em>boolean</em></summary>

If `true`, the animation repeats indefinitely. If `false`, it plays once and stops.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|

```tsx
// Loop forever (default)
<LottieView source={source} autoPlay loop style={{ width: 200, height: 200 }} />

// Play once
<LottieView source={source} autoPlay loop={false} style={{ width: 200, height: 200 }} />
```

</details> <details> <summary><code>speed</code> — <em>number</em></summary>

Playback speed multiplier. `1` = normal, `2` = double speed, `0.5` = half speed, `-1` = play in reverse.

|||
|---|---|
|**Type**|`number`|
|**Default**|`1`|

```tsx
// Fast forward
<LottieView source={source} autoPlay loop speed={2} style={{ width: 200, height: 200 }} />

// Slow motion
<LottieView source={source} autoPlay loop speed={0.3} style={{ width: 200, height: 200 }} />

// Reverse
<LottieView source={source} autoPlay loop speed={-1} style={{ width: 200, height: 200 }} />
```

</details> <details> <summary><code>duration</code> — <em>number</em></summary>

Override the animation duration in milliseconds. Calculated from the original animation's frame rate if not set.

|||
|---|---|
|**Type**|`number`|
|**Default**|Auto from animation|

```tsx
// Force 2 second duration regardless of original speed
<LottieView source={source} autoPlay loop duration={2000} style={{ width: 200, height: 200 }} />
```

</details> <details> <summary><code>progress</code> — <em>Animated.Value | number</em></summary>

Manually control the animation frame using a value from `0` (start) to `1` (end). When set, `autoPlay` is ignored — you control playback entirely.

|||
|---|---|
|**Type**|`Animated.Value \| SharedValue<number> \| number`|
|**Default**|`undefined`|

```tsx
import { Animated } from 'react-native';

const progress = useRef(new Animated.Value(0)).current;

<LottieView
  source={source}
  progress={progress}
  style={{ width: 200, height: 200 }}
/>

// Animate it:
Animated.timing(progress, {
  toValue: 1,
  duration: 2000,
  useNativeDriver: false, // must be false for Lottie progress
}).start();
```

See [Progress Control](#progress-control--animated-api) for full examples.

</details>

---

## Style & Layout Props

<details> <summary><code>resizeMode</code> — <em>enum</em></summary>

Controls how the animation is resized to fit its container.

|||
|---|---|
|**Type**|`'cover' \| 'contain' \| 'center'`|
|**Default**|`'contain'`|

```tsx
// Contain — fits the full animation inside bounds (default)
<LottieView resizeMode="contain" ... />

// Cover — fills container, may crop
<LottieView resizeMode="cover" ... />

// Center — original size, no scaling
<LottieView resizeMode="center" ... />
```

</details> <details> <summary><code>autoSize</code> — <em>boolean</em></summary>

If `true`, automatically sizes the view to the animation's intrinsic dimensions. Overrides `style` width/height.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```tsx
<LottieView source={source} autoPlay loop autoSize style={{}} />
```

> ⚠️ Use with caution — large animations can overflow the screen. Better to set explicit dimensions.

</details>

---

## Color Filters

<details> <summary><code>colorFilters</code> — <em>ColorFilter[]</em></summary>

Overrides colors in the animation at runtime. Each filter targets a named layer in the animation's JSON and replaces its color.

|||
|---|---|
|**Type**|`Array<{ keypath: string, color: string }>`|
|**Default**|`[]`|

```tsx
<LottieView
  source={require('./heart.json')}
  autoPlay
  loop
  colorFilters={[
    {
      keypath: 'Heart Fill',    // layer name from After Effects
      color: '#e11d48',         // replacement color
    },
    {
      keypath: 'Outline',
      color: '#fff',
    },
  ]}
  style={{ width: 100, height: 100 }}
/>
```

> 💡 To find layer names: open the JSON file and look for `"nm"` fields, or use [LottieFiles editor](https://lottiefiles.com/editor) to inspect layers.

</details> <details> <summary>Dynamic color filters — theme-aware</summary>

```tsx
import { useColorScheme } from 'react-native';

function ThemedAnimation() {
  const colorScheme = useColorScheme();
  const isDark = colorScheme === 'dark';

  return (
    <LottieView
      source={require('./icon.json')}
      autoPlay
      loop
      colorFilters={[
        { keypath: 'Background', color: isDark ? '#1f2937' : '#ffffff' },
        { keypath: 'Icon Fill',  color: isDark ? '#a5b4fc' : '#6366f1' },
        { keypath: 'Stroke',     color: isDark ? '#6b7280' : '#374151' },
      ]}
      style={{ width: 80, height: 80 }}
    />
  );
}
```

</details> <details> <summary>Wildcard keypath — target all matching layers</summary>

Lottie supports glob-style wildcards in keypaths to target multiple layers at once.

```tsx
colorFilters={[
  // Target all layers named "Fill" regardless of parent
  { keypath: '**.Fill', color: '#6366f1' },

  // Target all layers
  { keypath: '*', color: '#000000' },

  // Target a specific nested layer
  { keypath: 'Button.Background.Fill', color: '#f0f0f0' },
]}
```

</details>

---

## Text Filters

<details> <summary><code>textFiltersAndroid</code> — <em>TextFilter[]</em> · 🤖 Android</summary>

Replaces text content in Lottie text layers on Android at runtime.

|||
|---|---|
|**Type**|`Array<{ keypath: string, text: string }>`|
|**Platform**|Android|

```tsx
<LottieView
  source={require('./countdown.json')}
  autoPlay
  loop
  textFiltersAndroid={[
    { keypath: 'CounterText', text: '42' },
    { keypath: 'LabelText', text: 'Points' },
  ]}
  style={{ width: 200, height: 200 }}
/>
```

</details> <details> <summary><code>textFiltersIOS</code> — <em>TextFilter[]</em> · 🍎 iOS</summary>

Replaces text content in Lottie text layers on iOS.

|||
|---|---|
|**Type**|`Array<{ keypath: string, text: string }>`|
|**Platform**|iOS|

```tsx
<LottieView
  source={require('./badge.json')}
  autoPlay
  loop
  textFiltersIOS={[
    { keypath: 'BadgeCount', text: String(unreadCount) },
  ]}
  style={{ width: 60, height: 60 }}
/>
```

</details>

---

## Platform-specific Props

<details> <summary><code>enableMergePathsAndroidForKitKatAndAbove</code> — <em>boolean</em> · 🤖 Android</summary>

Enables merge paths on Android API 19+ (KitKat). Required for some animations that use merge path shapes. May impact performance on older devices.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|
|**Platform**|Android|

```tsx
<LottieView
  source={source}
  autoPlay
  loop
  enableMergePathsAndroidForKitKatAndAbove
  style={{ width: 200, height: 200 }}
/>
```

</details> <details> <summary><code>hardwareAccelerationAndroid</code> — <em>boolean</em> · 🤖 Android</summary>

Enables hardware acceleration for the Lottie view on Android. Can improve performance for complex animations but may cause issues with some effects.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|
|**Platform**|Android|

```tsx
<LottieView
  source={source}
  autoPlay
  loop
  hardwareAccelerationAndroid
  style={{ width: 200, height: 200 }}
/>
```

</details> <details> <summary><code>imageAssetsFolder</code> — <em>string</em> · 🤖 Android</summary>

Folder name (relative to `assets/`) where image assets referenced by the animation are stored. Required on Android when the animation uses external image files.

|||
|---|---|
|**Type**|`string`|
|**Platform**|Android|

```tsx
<LottieView
  source={require('./animation-with-images.json')}
  autoPlay
  loop
  imageAssetsFolder="lottie_images"  // assets/lottie_images/
  style={{ width: 200, height: 200 }}
/>
```

</details> <details> <summary><code>cacheComposition</code> — <em>boolean</em></summary>

If `true`, caches the parsed animation composition to avoid re-parsing on every render. Recommended for frequently-rendered animations.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|

```tsx
<LottieView
  source={source}
  autoPlay
  loop
  cacheComposition={true}
  style={{ width: 200, height: 200 }}
/>
```

</details> <details> <summary><code>webStyle</code> — <em>CSSProperties</em> · 🌐 Web</summary>

Additional CSS style applied to the underlying `<lottie-player>` element on web.

|||
|---|---|
|**Type**|`React.CSSProperties`|
|**Platform**|Web|

```tsx
<LottieView
  source={source}
  autoPlay
  loop
  webStyle={{ display: 'block', margin: '0 auto' }}
  style={{ width: 200, height: 200 }}
/>
```

</details>

---

## Event Callbacks

<details> <summary><code>onAnimationFinish</code> — <em>(isCancelled: boolean) =&gt; void</em></summary>

Called when the animation completes. For looping animations, called on each loop cycle end. `isCancelled` is `true` if the animation was stopped manually.

|||
|---|---|
|**Type**|`(isCancelled: boolean) => void`|

```tsx
<LottieView
  source={require('./success.json')}
  autoPlay
  loop={false}
  onAnimationFinish={(isCancelled) => {
    if (!isCancelled) {
      // Animation completed naturally — navigate to next screen
      router.push('/success');
    }
  }}
  style={{ width: 150, height: 150 }}
/>
```

</details> <details> <summary><code>onAnimationLoop</code> — <em>() =&gt; void</em></summary>

Called each time the animation loops back to the beginning. Only fires when `loop` is `true`.

|||
|---|---|
|**Type**|`() => void`|

```tsx
const loopCount = useRef(0);

<LottieView
  source={source}
  autoPlay
  loop
  onAnimationLoop={() => {
    loopCount.current += 1;
    if (loopCount.current >= 3) {
      // Stop after 3 loops
      lottieRef.current?.pause();
    }
  }}
  style={{ width: 200, height: 200 }}
/>
```

</details> <details> <summary><code>onAnimationFailure</code> — <em>(error: string) =&gt; void</em></summary>

Called when the animation fails to load or parse.

|||
|---|---|
|**Type**|`(error: string) => void`|

```tsx
<LottieView
  source={{ uri: remoteUrl }}
  autoPlay
  loop
  onAnimationFailure={(error) => {
    console.error('Lottie failed:', error);
    setShowFallback(true);
  }}
  style={{ width: 200, height: 200 }}
/>
```

</details> <details> <summary><code>onAnimationLoaded</code> — <em>() =&gt; void</em></summary>

Called when the animation JSON has been loaded and parsed successfully — before the first frame plays.

|||
|---|---|
|**Type**|`() => void`|

```tsx
<LottieView
  source={{ uri: remoteUrl }}
  autoPlay
  loop
  onAnimationLoaded={() => {
    setIsLoading(false);
  }}
  style={{ width: 200, height: 200 }}
/>
```

</details>

---

## Ref Methods

<details> <summary><code>lottieRef.current.play(startFrame?, endFrame?)</code></summary>

Starts or resumes playback. Optionally plays only a specific frame range.

|Param|Type|Description|
|---|---|---|
|`startFrame`|`number`|Frame to start from (optional)|
|`endFrame`|`number`|Frame to stop at (optional)|

```tsx
const lottieRef = useRef<LottieView>(null);

// Play the entire animation
lottieRef.current?.play();

// Play frames 0 to 60
lottieRef.current?.play(0, 60);

// Play frames 60 to 120 (segment)
lottieRef.current?.play(60, 120);
```

</details> <details> <summary><code>lottieRef.current.pause()</code></summary>

Pauses playback at the current frame.

```tsx
lottieRef.current?.pause();
```

</details> <details> <summary><code>lottieRef.current.resume()</code></summary>

Resumes a paused animation from where it stopped.

```tsx
lottieRef.current?.resume();
```

</details> <details> <summary><code>lottieRef.current.reset()</code></summary>

Stops the animation and resets it to the first frame.

```tsx
lottieRef.current?.reset();

// Reset then play from start
lottieRef.current?.reset();
lottieRef.current?.play();
```

</details>

---

## Progress Control — Animated API

<details> <summary>Scrub animation with <code>Animated.Value</code></summary>

Use the `progress` prop with an `Animated.Value` to drive the animation from JS. Enables scroll-linked, gesture-driven, or timer-driven animations.

> ⚠️ Progress animations with `Animated.Value` **must** use `useNativeDriver: false`.

```tsx
import { Animated, PanResponder, View } from 'react-native';
import LottieView from 'lottie-react-native';

function ScrubAnimation() {
  const progress = useRef(new Animated.Value(0)).current;

  // Play forward once
  const playForward = () => {
    progress.setValue(0);
    Animated.timing(progress, {
      toValue: 1,
      duration: 1500,
      useNativeDriver: false,
    }).start();
  };

  // Loop continuously
  const playLoop = () => {
    Animated.loop(
      Animated.timing(progress, {
        toValue: 1,
        duration: 1500,
        useNativeDriver: false,
      })
    ).start();
  };

  return (
    <>
      <LottieView
        source={require('./animation.json')}
        progress={progress}
        style={{ width: 200, height: 200 }}
      />
      <Button title="Play" onPress={playForward} />
    </>
  );
}
```

</details> <details> <summary>Scroll-linked animation with <code>Animated.event</code></summary>

```tsx
function ScrollAnimation() {
  const scrollY = useRef(new Animated.Value(0)).current;

  // Map scroll 0–300px to animation 0–1
  const progress = scrollY.interpolate({
    inputRange: [0, 300],
    outputRange: [0, 1],
    extrapolate: 'clamp',
  });

  return (
    <View style={{ flex: 1 }}>
      <Animated.ScrollView
        onScroll={Animated.event(
          [{ nativeEvent: { contentOffset: { y: scrollY } } }],
          { useNativeDriver: false }
        )}
        scrollEventThrottle={16}
      >
        <View style={{ height: 1000 }}>
          <Text>Scroll down to scrub the animation</Text>
        </View>
      </Animated.ScrollView>

      {/* Fixed overlay */}
      <LottieView
        source={require('./animation.json')}
        progress={progress}
        style={{ position: 'absolute', top: 20, right: 20, width: 100, height: 100 }}
      />
    </View>
  );
}
```

</details> <details> <summary>Gesture-driven progress with PanResponder</summary>

```tsx
function GestureAnimation() {
  const progress = useRef(new Animated.Value(0)).current;
  const lastProgress = useRef(0);

  const panResponder = useRef(
    PanResponder.create({
      onStartShouldSetPanResponder: () => true,
      onPanResponderMove: (_, { dx }) => {
        const delta = dx / 300; // 300px = full animation
        const next = Math.min(1, Math.max(0, lastProgress.current + delta));
        progress.setValue(next);
      },
      onPanResponderRelease: (_, { dx }) => {
        lastProgress.current = Math.min(1, Math.max(0, lastProgress.current + dx / 300));
      },
    })
  ).current;

  return (
    <View {...panResponder.panHandlers}>
      <LottieView
        source={require('./animation.json')}
        progress={progress}
        style={{ width: 250, height: 250 }}
      />
      <Text>← Drag to scrub →</Text>
    </View>
  );
}
```

</details>

---

## Progress Control — Reanimated

<details> <summary>Drive Lottie with a Reanimated <code>useSharedValue</code></summary>

```tsx
import Animated, {
  useSharedValue,
  withTiming,
  withRepeat,
  useAnimatedProps,
  Easing,
} from 'react-native-reanimated';
import LottieView from 'lottie-react-native';

const AnimatedLottie = Animated.createAnimatedComponent(LottieView);

function ReanimatedLottie() {
  const progress = useSharedValue(0);

  useEffect(() => {
    progress.value = withRepeat(
      withTiming(1, { duration: 2000, easing: Easing.linear }),
      -1,  // infinite
      false
    );
  }, []);

  const animatedProps = useAnimatedProps(() => ({
    progress: progress.value,
  }));

  return (
    <AnimatedLottie
      source={require('./animation.json')}
      animatedProps={animatedProps}
      style={{ width: 200, height: 200 }}
    />
  );
}
```

</details> <details> <summary>Scroll-driven with Reanimated <code>useScrollViewOffset</code></summary>

```tsx
import Animated, {
  useSharedValue,
  useAnimatedScrollHandler,
  useAnimatedProps,
  interpolate,
  Extrapolation,
} from 'react-native-reanimated';

const AnimatedLottie = Animated.createAnimatedComponent(LottieView);

function ScrollDrivenAnimation() {
  const scrollY = useSharedValue(0);

  const scrollHandler = useAnimatedScrollHandler((event) => {
    scrollY.value = event.contentOffset.y;
  });

  const animatedProps = useAnimatedProps(() => ({
    progress: interpolate(scrollY.value, [0, 400], [0, 1], Extrapolation.CLAMP),
  }));

  return (
    <View style={{ flex: 1 }}>
      <AnimatedLottie
        source={require('./animation.json')}
        animatedProps={animatedProps}
        style={{ width: 200, height: 200 }}
      />
      <Animated.ScrollView onScroll={scrollHandler} scrollEventThrottle={16}>
        <View style={{ height: 1200 }} />
      </Animated.ScrollView>
    </View>
  );
}
```

</details>

---

## Render Mode

<details> <summary><code>renderMode</code> — control rendering engine</summary>

Controls whether the animation is rendered via hardware or software.

|||
|---|---|
|**Type**|`'AUTOMATIC' \| 'HARDWARE' \| 'SOFTWARE'`|
|**Default**|`'AUTOMATIC'`|

```tsx
import LottieView, { AnimationObject } from 'lottie-react-native';

// Let Lottie decide (default — usually hardware when possible)
<LottieView renderMode="AUTOMATIC" ... />

// Force GPU hardware rendering (better performance)
<LottieView renderMode="HARDWARE" ... />

// Force CPU software rendering (better compatibility for some effects)
<LottieView renderMode="SOFTWARE" ... />
```

|Mode|When to use|
|---|---|
|`AUTOMATIC`|Default — Lottie picks based on animation content|
|`HARDWARE`|Complex animations, smooth 60fps priority|
|`SOFTWARE`|Animations with blend modes or mask effects that look wrong in hardware mode|

</details>

---

## Resize Mode

<details> <summary><code>resizeMode</code> — how animation fills its container</summary>

|Mode|Behavior|
|---|---|
|`'contain'`|Shows the full animation inside bounds, may leave empty space (default)|
|`'cover'`|Fills container, may crop edges of animation|
|`'center'`|Centers at intrinsic size without scaling|

```tsx
// Best for most use cases — full animation visible
<LottieView resizeMode="contain" style={{ width: 200, height: 200 }} ... />

// Fill a banner/card — may clip
<LottieView resizeMode="cover" style={{ width: '100%', height: 120 }} ... />

// Show animation at 1:1 pixel scale
<LottieView resizeMode="center" style={{ width: 200, height: 200 }} ... />
```

</details>

---

## DotLottie Format

<details> <summary>What is DotLottie and how to use it</summary>

DotLottie (`.lottie`) is a ZIP-based format that packages the JSON, images, and assets into a single binary file. Typically 2–3× smaller than the equivalent `.json`.

|Feature|JSON|DotLottie|
|---|---|---|
|Extension|`.json`|`.lottie`|
|Compression|None|ZIP compressed|
|Images|External files|Bundled inside|
|Multiple animations|❌|✅|
|Support|All versions|`lottie-react-native` 6.0+|

```tsx
// Use exactly like a JSON source
<LottieView
  source={require('./assets/animation.lottie')}
  autoPlay
  loop
  style={{ width: 200, height: 200 }}
/>
```

**Convert JSON to DotLottie:**

- Use the [LottieFiles DotLottie converter](https://lottiefiles.com/tools/dotlottie-converter)
- Or the `@dotlottie/dotlottie-js` npm package

</details>

---

## Common Patterns

<details> <summary>Looping loading spinner</summary>

```tsx
function LoadingSpinner({ size = 80 }) {
  return (
    <LottieView
      source={require('./assets/animations/spinner.json')}
      autoPlay
      loop
      style={{ width: size, height: size }}
    />
  );
}

// Usage
{isLoading && <LoadingSpinner size={60} />}
```

</details> <details> <summary>One-shot success / error animation with callback</summary>

```tsx
function ResultAnimation({ type, onComplete }) {
  const source = type === 'success'
    ? require('./assets/success.json')
    : require('./assets/error.json');

  return (
    <LottieView
      source={source}
      autoPlay
      loop={false}
      onAnimationFinish={(isCancelled) => {
        if (!isCancelled) onComplete();
      }}
      style={{ width: 160, height: 160 }}
    />
  );
}

// Usage
const [result, setResult] = useState(null); // 'success' | 'error' | null

{result && (
  <ResultAnimation
    type={result}
    onComplete={() => {
      setResult(null);
      router.replace('/home');
    }}
  />
)}
```

</details> <details> <summary>Play/pause toggle with ref</summary>

```tsx
function PlayPauseAnimation() {
  const lottieRef = useRef<LottieView>(null);
  const [isPlaying, setIsPlaying] = useState(true);

  const toggle = () => {
    if (isPlaying) {
      lottieRef.current?.pause();
    } else {
      lottieRef.current?.resume();
    }
    setIsPlaying(prev => !prev);
  };

  return (
    <View style={{ alignItems: 'center' }}>
      <LottieView
        ref={lottieRef}
        source={require('./animation.json')}
        autoPlay
        loop
        style={{ width: 200, height: 200 }}
      />
      <TouchableOpacity onPress={toggle} style={styles.button}>
        <Text>{isPlaying ? 'Pause' : 'Play'}</Text>
      </TouchableOpacity>
    </View>
  );
}
```

</details> <details> <summary>Play a specific frame segment on event</summary>

```tsx
function MultiSegmentAnimation() {
  const lottieRef = useRef<LottieView>(null);

  const playIdle = () => lottieRef.current?.play(0, 60);
  const playWalk = () => lottieRef.current?.play(61, 120);
  const playRun  = () => lottieRef.current?.play(121, 180);

  return (
    <View style={{ alignItems: 'center' }}>
      <LottieView
        ref={lottieRef}
        source={require('./character.json')}
        autoPlay={false}
        loop
        style={{ width: 200, height: 200 }}
      />
      <View style={{ flexDirection: 'row', gap: 12 }}>
        <Button title="Idle" onPress={playIdle} />
        <Button title="Walk" onPress={playWalk} />
        <Button title="Run"  onPress={playRun} />
      </View>
    </View>
  );
}
```

</details> <details> <summary>Splash screen with Lottie intro</summary>

```tsx
// app/_layout.tsx (Expo Router)
import { SplashScreen } from 'expo-router';
import LottieView from 'lottie-react-native';
import { useState } from 'react';

SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const [splashDone, setSplashDone] = useState(false);

  const handleAnimationFinish = () => {
    setSplashDone(true);
    SplashScreen.hideAsync();
  };

  if (!splashDone) {
    return (
      <View style={{ flex: 1, backgroundColor: '#6366f1', justifyContent: 'center', alignItems: 'center' }}>
        <LottieView
          source={require('./assets/splash-animation.json')}
          autoPlay
          loop={false}
          onAnimationFinish={handleAnimationFinish}
          style={{ width: 300, height: 300 }}
          resizeMode="contain"
        />
      </View>
    );
  }

  return <Stack />;
}
```

</details> <details> <summary>Like button — play on press, reverse on unlike</summary>

```tsx
function LikeButton({ liked, onToggle }) {
  const lottieRef = useRef<LottieView>(null);
  const wasLiked = useRef(liked);

  useEffect(() => {
    if (liked && !wasLiked.current) {
      // Liked — play forward (0 to 1)
      lottieRef.current?.play(0, 50);
    } else if (!liked && wasLiked.current) {
      // Unliked — play in reverse
      lottieRef.current?.play(50, 0);
    }
    wasLiked.current = liked;
  }, [liked]);

  return (
    <TouchableOpacity onPress={onToggle}>
      <LottieView
        ref={lottieRef}
        source={require('./assets/heart-like.json')}
        autoPlay={false}
        loop={false}
        style={{ width: 48, height: 48 }}
        colorFilters={[
          { keypath: 'Heart Fill', color: liked ? '#ef4444' : '#d1d5db' },
        ]}
      />
    </TouchableOpacity>
  );
}
```

</details> <details> <summary>Animated button press feedback</summary>

```tsx
function AnimatedButton({ onPress, label }) {
  const lottieRef = useRef<LottieView>(null);

  const handlePress = () => {
    lottieRef.current?.reset();
    lottieRef.current?.play();
    onPress();
  };

  return (
    <TouchableOpacity onPress={handlePress} style={styles.button}>
      <LottieView
        ref={lottieRef}
        source={require('./assets/ripple.json')}
        autoPlay={false}
        loop={false}
        style={StyleSheet.absoluteFillObject}
        resizeMode="cover"
      />
      <Text style={styles.label}>{label}</Text>
    </TouchableOpacity>
  );
}
```

</details> <details> <summary>Remote animation with loading and error fallback</summary>

```tsx
function RemoteLottie({ url, fallback }) {
  const [loaded, setLoaded] = useState(false);
  const [failed, setFailed] = useState(false);

  if (failed) {
    return fallback || <ActivityIndicator />;
  }

  return (
    <View>
      {!loaded && <ActivityIndicator style={StyleSheet.absoluteFillObject} />}
      <LottieView
        source={{ uri: url }}
        autoPlay
        loop
        onAnimationLoaded={() => setLoaded(true)}
        onAnimationFailure={() => setFailed(true)}
        style={{ width: 200, height: 200, opacity: loaded ? 1 : 0 }}
      />
    </View>
  );
}
```

</details> <details> <summary>Themed animation using colorFilters</summary>

```tsx
function BrandedAnimation({ primaryColor, secondaryColor }) {
  return (
    <LottieView
      source={require('./assets/branded.json')}
      autoPlay
      loop
      colorFilters={[
        { keypath: 'Primary.**', color: primaryColor },
        { keypath: 'Secondary.**', color: secondaryColor },
        { keypath: 'Background', color: 'transparent' },
      ]}
      style={{ width: 200, height: 200 }}
    />
  );
}

// Usage
<BrandedAnimation primaryColor="#6366f1" secondaryColor="#a5b4fc" />
```

</details> <details> <summary>Counter animation with text filter</summary>

```tsx
function AnimatedCounter({ value }) {
  return (
    <LottieView
      source={require('./assets/counter.json')}
      autoPlay
      loop={false}
      textFiltersAndroid={[{ keypath: 'NumberText', text: String(value) }]}
      textFiltersIOS={[{ keypath: 'NumberText', text: String(value) }]}
      style={{ width: 120, height: 80 }}
    />
  );
}
```

</details> <details> <summary>Visibility-triggered animation (play when in view)</summary>

```tsx
import { useIsFocused } from '@react-navigation/native';

function VisibilityAnimation() {
  const lottieRef = useRef<LottieView>(null);
  const isFocused = useIsFocused();

  useEffect(() => {
    if (isFocused) {
      lottieRef.current?.reset();
      lottieRef.current?.play();
    } else {
      lottieRef.current?.pause();
    }
  }, [isFocused]);

  return (
    <LottieView
      ref={lottieRef}
      source={require('./animation.json')}
      autoPlay={false}
      loop
      style={{ width: 200, height: 200 }}
    />
  );
}
```

</details>

---

## Quick-Reference Cheatsheet

|Prop / Method|Use case|
|---|---|
|`source={require(...)}`|Local bundled JSON or .lottie|
|`source={{ uri: url }}`|Remote animation URL|
|`autoPlay`|Start on mount|
|`loop`|Repeat forever|
|`loop={false}`|Play once|
|`speed={2}`|Double speed|
|`speed={-1}`|Play in reverse|
|`progress={animatedValue}`|Manual frame control|
|`resizeMode="contain"`|Fit full animation in bounds|
|`resizeMode="cover"`|Fill container, may clip|
|`colorFilters={[...]}`|Runtime color replacement|
|`textFiltersIOS/Android`|Runtime text replacement|
|`onAnimationFinish`|Callback when animation ends|
|`onAnimationLoop`|Callback on each loop|
|`onAnimationFailure`|Handle load errors|
|`onAnimationLoaded`|Animation ready to play|
|`ref.play()`|Imperatively start|
|`ref.play(0, 60)`|Play a frame range|
|`ref.pause()`|Freeze current frame|
|`ref.resume()`|Continue from paused|
|`ref.reset()`|Return to frame 0|
|`cacheComposition`|Cache parsed animation|
|`renderMode="HARDWARE"`|GPU acceleration|
|`renderMode="SOFTWARE"`|CPU rendering (blend modes)|
|`enableMergePathsAndroidForKitKatAndAbove`|Fix missing shapes on Android|
|`hardwareAccelerationAndroid`|Android GPU acceleration|
|`imageAssetsFolder`|Android external image assets|

---

## Troubleshooting

<details> <summary>Animation not showing / blank view</summary>

- ✅ Ensure `style` has explicit `width` and `height`
- ✅ Check `source` is a valid JSON or .lottie file
- ✅ Run `pod install` after installing on iOS bare workflow
- ✅ Clear Metro cache: `npx expo start --clear`

</details> <details> <summary>Animation looks wrong on Android</summary>

- Try `enableMergePathsAndroidForKitKatAndAbove` for missing shapes
- Try `renderMode="SOFTWARE"` for blend mode / mask issues
- Check that `imageAssetsFolder` is set if the animation uses external images
- Some After Effects effects are not supported on Android — test with Lottie's preview tool

</details> <details> <summary>Progress animation not smooth</summary>

- Ensure `useNativeDriver: false` with `Animated.Value` progress
- Use Reanimated (`useSharedValue`) for better performance
- Avoid re-rendering the parent component on every frame

</details> <details> <summary>Colors don't change with colorFilters</summary>

- Layer `keypath` must exactly match the layer name in the AE/JSON file
- Open the `.json` and search for `"nm"` fields to find layer names
- Use the LottieFiles editor to inspect and test keypaths
- Try wildcard: `{ keypath: '**', color: '#ff0000' }` to see if any color changes

</details>

---

_Reference based on `lottie-react-native` v6+. Always check the [official docs](https://github.com/lottie-community/lottie-react-native) for the latest updates._