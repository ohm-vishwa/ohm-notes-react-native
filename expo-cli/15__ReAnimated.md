# React Native Reanimated — Complete Reference

A comprehensive reference for `react-native-reanimated` v3.  
Covers shared values, animation functions, hooks, interpolation, layout animations, worklets, and common patterns.

> **Install:**
> 
> ```bash
> npm install react-native-reanimated
> # Add to babel.config.js plugins:
> plugins: ['react-native-reanimated/plugin']
> ```

---

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Shared Values](#shared-values)
3. [useAnimatedStyle](#useanimatedstyle)
4. [useAnimatedProps](#useanimatedprops)
5. [Animation Functions](#animation-functions)
6. [Timing Config](#timing-config)
7. [Spring Config](#spring-config)
8. [Decay Config](#decay-config)
9. [Animation Modifiers](#animation-modifiers)
10. [Cancelling Animations](#cancelling-animations)
11. [Derived Values](#derived-values)
12. [Animated Reactions](#animated-reactions)
13. [Interpolation](#interpolation)
14. [Easing](#easing)
15. [ReduceMotion](#reducemotion)
16. [Worklets](#worklets)
17. [Animated Components](#animated-components)
18. [Scroll Hooks](#scroll-hooks)
19. [Keyboard Hooks](#keyboard-hooks)
20. [Sensor Hooks](#sensor-hooks)
21. [Ref & Measurement Hooks](#ref--measurement-hooks)
22. [Frame Callback](#frame-callback)
23. [Layout Animations — Entering](#layout-animations--entering)
24. [Layout Animations — Exiting](#layout-animations--exiting)
25. [Layout Animations — Layout Transitions](#layout-animations--layout-transitions)
26. [Keyframe Animations](#keyframe-animations)
27. [Gesture Handler Integration](#gesture-handler-integration)
28. [Common Patterns](#common-patterns)

---

## Core Concepts

<details> <summary>UI thread vs JS thread — why Reanimated is faster</summary>

React Native's standard `Animated` API runs on the JS thread. Any JS work (renders, API calls) can block or drop animation frames.

Reanimated runs animations entirely on the **UI thread** using JSI (JavaScript Interface), enabling true 60/120fps animations that are never blocked by JS work.

||Standard Animated|Reanimated|
|---|---|---|
|Thread|JS (bridge)|UI (JSI)|
|Smooth under JS load|❌|✅|
|Layout animations|❌|✅|
|Shared element transitions|❌|✅|
|Gesture integration|Limited|Native-thread gestures|
|API style|Imperative|Declarative hooks|

```jsx
// The key mental model:
// Code inside useAnimatedStyle, useDerivedValue, worklets → runs on UI thread
// Code outside (React render, useEffect, event handlers) → runs on JS thread
```

</details> <details> <summary>Worklet — a function that runs on the UI thread</summary>

Any function marked with `'worklet'` directive can be called from the UI thread. Hooks like `useAnimatedStyle` automatically run their callbacks as worklets.

```jsx
// Explicitly mark a function as a worklet
function clamp(value, min, max) {
  'worklet';
  return Math.min(Math.max(value, min), max);
}

// Use in animated style (runs on UI thread)
const animatedStyle = useAnimatedStyle(() => {
  return { opacity: clamp(progress.value, 0, 1) };
});
```

> ⚠️ You cannot access most React state or props inside a worklet — only shared values and other worklets.

</details>

---

## Shared Values

<details> <summary><code>useSharedValue(initialValue)</code></summary>

Creates a shared value — a value that lives on the UI thread and can be read/written from both threads. The fundamental building block of Reanimated.

|||
|---|---|
|**Type**|`SharedValue<T>`|
|**Access**|`.value` property|

```jsx
import Animated, { useSharedValue } from 'react-native-reanimated';

const opacity = useSharedValue(0);
const position = useSharedValue({ x: 0, y: 0 });
const color = useSharedValue('#6366f1');
const isVisible = useSharedValue(false);

// Read the value
console.log(opacity.value); // 0

// Write the value (instant, no animation)
opacity.value = 1;

// Write with animation
opacity.value = withTiming(1, { duration: 300 });
```

> ⚠️ Unlike `useState`, updating `.value` does NOT cause a React re-render. It only updates the UI thread directly.

</details> <details> <summary>Reading and writing <code>.value</code></summary>

The `.value` property is how you interact with shared values from the JS thread.

```jsx
const progress = useSharedValue(0);

// Instant update (no animation)
progress.value = 0.5;

// Animated update — assign an animation function directly
progress.value = withTiming(1, { duration: 500 });
progress.value = withSpring(1, { damping: 10 });

// Read current value (JS thread snapshot)
const current = progress.value;

// Read inside a worklet (UI thread — always up to date)
const style = useAnimatedStyle(() => ({
  opacity: progress.value,   // always the live UI-thread value
}));
```

</details>

---

## useAnimatedStyle

<details> <summary><code>useAnimatedStyle(styleWorklet, dependencies?)</code></summary>

Returns an animated style object. The callback runs on the UI thread whenever any shared value it reads changes.

|||
|---|---|
|**Returns**|`AnimatedStyle`|
|**Thread**|UI thread (worklet)|

```jsx
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';

function Box() {
  const offset = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: offset.value }],
    opacity: offset.value > 50 ? 0.5 : 1,
  }));

  return (
    <>
      <Animated.View style={[styles.box, animatedStyle]} />
      <Button onPress={() => { offset.value = withSpring(100); }} title="Move" />
    </>
  );
}
```

> ✅ Always combine with static styles using an array: `style={[styles.base, animatedStyle]}`  
> ⚠️ Do not return new object references unnecessarily — use conditional expressions, not if/else blocks returning different objects.

</details>

---

## useAnimatedProps

<details> <summary><code>useAnimatedProps(propsWorklet, dependencies?)</code></summary>

Like `useAnimatedStyle` but for non-style props — path data on SVGs, `value` on inputs, etc.

|||
|---|---|
|**Returns**|`AnimatedProps`|
|**Thread**|UI thread (worklet)|

```jsx
import Animated, { useAnimatedProps, useSharedValue, withTiming } from 'react-native-reanimated';
import { TextInput } from 'react-native';
import Svg, { Circle } from 'react-native-svg';

// Animated text value (counter)
const AnimatedTextInput = Animated.createAnimatedComponent(TextInput);

function Counter() {
  const count = useSharedValue(0);

  const animatedProps = useAnimatedProps(() => ({
    value: String(Math.round(count.value)),
  }));

  return <AnimatedTextInput animatedProps={animatedProps} editable={false} />;
}

// Animated SVG circle radius
const AnimatedCircle = Animated.createAnimatedComponent(Circle);

function AnimatedCircleDemo() {
  const radius = useSharedValue(50);

  const circleProps = useAnimatedProps(() => ({
    r: radius.value,
  }));

  return (
    <Svg>
      <AnimatedCircle cx="100" cy="100" animatedProps={circleProps} fill="#6366f1" />
    </Svg>
  );
}
```

</details>

---

## Animation Functions

<details> <summary><code>withTiming(toValue, config?, callback?)</code></summary>

Animates to a target value over a fixed duration with an easing curve.

|||
|---|---|
|**Returns**|`AnimationObject`|

```jsx
import { withTiming } from 'react-native-reanimated';

// Basic
opacity.value = withTiming(1);

// With config
opacity.value = withTiming(1, { duration: 400, easing: Easing.out(Easing.ease) });

// With completion callback (runs on JS thread)
opacity.value = withTiming(1, { duration: 300 }, (finished) => {
  if (finished) runOnJS(setVisible)(true);
});
```

</details> <details> <summary><code>withSpring(toValue, config?, callback?)</code></summary>

Animates using spring physics. No fixed duration — settles naturally.

|||
|---|---|
|**Returns**|`AnimationObject`|

```jsx
import { withSpring } from 'react-native-reanimated';

// Basic
scale.value = withSpring(1.2);

// With physics config
scale.value = withSpring(1, {
  damping: 10,
  stiffness: 100,
  mass: 1,
});

// With callback
scale.value = withSpring(1, {}, (finished) => {
  'worklet';
  if (finished) console.log('Spring settled');
});
```

</details> <details> <summary><code>withDecay(config, callback?)</code></summary>

Starts at a given velocity and gradually decelerates to a stop. Perfect for fling/swipe momentum.

|||
|---|---|
|**Returns**|`AnimationObject`|

```jsx
import { withDecay } from 'react-native-reanimated';

// After gesture release
translateX.value = withDecay({
  velocity: gesture.velocityX,
  clamp: [0, SCREEN_WIDTH],   // optional boundaries
});
```

</details>

---

## Timing Config

<details> <summary><code>duration</code> — <em>number</em></summary>

Length of the animation in milliseconds.

|||
|---|---|
|**Type**|`number`|
|**Default**|`300`|

```jsx
opacity.value = withTiming(1, { duration: 500 });
```

</details> <details> <summary><code>easing</code> — <em>EasingFunction</em></summary>

Easing function that shapes the animation curve. Import `Easing` from Reanimated.

|||
|---|---|
|**Type**|`(t: number) => number`|
|**Default**|`Easing.inOut(Easing.quad)`|

```jsx
import { Easing } from 'react-native-reanimated';

opacity.value = withTiming(1, {
  duration: 400,
  easing: Easing.bezier(0.25, 0.1, 0.25, 1),
});
```

</details> <details> <summary><code>reduceMotion</code> — <em>ReduceMotion</em></summary>

Controls behavior when the device's "Reduce Motion" accessibility setting is on.

|||
|---|---|
|**Type**|`ReduceMotion`|
|**Default**|`ReduceMotion.System`|

**Values:** `ReduceMotion.System`, `ReduceMotion.Always`, `ReduceMotion.Never`

```jsx
import { ReduceMotion } from 'react-native-reanimated';

opacity.value = withTiming(1, {
  duration: 400,
  reduceMotion: ReduceMotion.System, // respects OS setting
});
```

</details>

---

## Spring Config

<details> <summary><code>damping</code> — <em>number</em></summary>

Resistance opposing the spring. Higher = less oscillation, settles faster.

|||
|---|---|
|**Type**|`number`|
|**Default**|`10`|

```jsx
scale.value = withSpring(1, { damping: 20, stiffness: 200 });
```

</details> <details> <summary><code>mass</code> — <em>number</em></summary>

Mass of the object. Higher = moves more slowly and overshoots more.

|||
|---|---|
|**Type**|`number`|
|**Default**|`1`|

```jsx
scale.value = withSpring(1, { mass: 0.5, damping: 8 });
```

</details> <details> <summary><code>stiffness</code> — <em>number</em></summary>

Spring constant. Higher = snappier, faster to reach target.

|||
|---|---|
|**Type**|`number`|
|**Default**|`100`|

```jsx
scale.value = withSpring(1, { stiffness: 300, damping: 20 });
```

</details> <details> <summary><code>duration</code> and <code>dampingRatio</code> — duration-based spring</summary>

Alternative spring config. Set `duration` (ms) and `dampingRatio` (0 = no damping, 1 = critically damped) instead of `damping/mass/stiffness`. Easier to reason about timing.

|||
|---|---|
|**duration default**|`2000`|
|**dampingRatio default**|`0.5`|

```jsx
scale.value = withSpring(1, {
  duration: 600,
  dampingRatio: 0.8, // 1 = no overshoot
});
```

</details> <details> <summary><code>overshootClamping</code> — <em>boolean</em></summary>

If `true`, stops the animation exactly at `toValue` without bouncing past it.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```jsx
opacity.value = withSpring(1, { overshootClamping: true });
```

</details> <details> <summary><code>restDisplacementThreshold</code> and <code>restSpeedThreshold</code> — <em>number</em></summary>

Thresholds below which the spring snaps to `toValue` and stops. Lower values = more precise but slower to finish.

|||
|---|---|
|**restDisplacementThreshold default**|`0.001`|
|**restSpeedThreshold default**|`2`|

```jsx
scale.value = withSpring(1, {
  restDisplacementThreshold: 0.01,
  restSpeedThreshold: 0.01,
});
```

</details> <details> <summary><code>velocity</code> — <em>number</em></summary>

Initial velocity to kick-start the spring. Use to seamlessly hand off from a gesture.

|||
|---|---|
|**Type**|`number`|
|**Default**|`0`|

```jsx
// After gesture ends, continue with spring from current gesture velocity
translateX.value = withSpring(0, { velocity: gesture.velocityX });
```

</details>

---

## Decay Config

<details> <summary><code>velocity</code> — <em>number | { x, y }</em> ⚠️ Required</summary>

The initial velocity. Should come from gesture `velocityX` / `velocityY`.

```jsx
translateX.value = withDecay({ velocity: gesture.velocityX });
```

</details> <details> <summary><code>deceleration</code> — <em>number</em></summary>

Rate of velocity decrease per frame. Closer to `1` = slower stop (longer glide).

|||
|---|---|
|**Type**|`number`|
|**Default**|`0.998`|

```jsx
translateX.value = withDecay({ velocity: 500, deceleration: 0.990 });
```

</details> <details> <summary><code>clamp</code> — <em>[min, max]</em></summary>

Boundaries for the decay. The animation stops at these limits instead of overshooting.

|||
|---|---|
|**Type**|`[number, number]`|
|**Default**|`undefined`|

```jsx
translateX.value = withDecay({
  velocity: gesture.velocityX,
  clamp: [0, SCREEN_WIDTH - CARD_WIDTH],
});
```

</details> <details> <summary><code>velocityFactor</code> — <em>number</em></summary>

Multiplies the initial velocity. Useful to scale down fast gesture velocities.

|||
|---|---|
|**Type**|`number`|
|**Default**|`1`|

```jsx
translateX.value = withDecay({ velocity: gesture.velocityX, velocityFactor: 0.5 });
```

</details> <details> <summary><code>rubberBandEffect</code> and <code>rubberBandFactor</code></summary>

Adds an elastic resistance effect when the value reaches the `clamp` boundaries — like rubber pulling back. `rubberBandFactor` controls how "stretchy" it is.

|||
|---|---|
|**rubberBandEffect default**|`false`|
|**rubberBandFactor default**|`0.6`|

```jsx
translateY.value = withDecay({
  velocity: gesture.velocityY,
  clamp: [0, MAX_SCROLL],
  rubberBandEffect: true,
  rubberBandFactor: 0.5,
});
```

</details>

---

## Animation Modifiers

<details> <summary><code>withDelay(delayMs, animation)</code></summary>

Waits `delayMs` milliseconds before starting the wrapped animation.

```jsx
import { withDelay, withTiming } from 'react-native-reanimated';

opacity.value = withDelay(500, withTiming(1, { duration: 300 }));
```

</details> <details> <summary><code>withSequence(...animations)</code></summary>

Runs animations one after another on the same value.

```jsx
import { withSequence, withTiming, withSpring } from 'react-native-reanimated';

// Shake animation
translateX.value = withSequence(
  withTiming(-10, { duration: 50 }),
  withTiming(10, { duration: 50 }),
  withTiming(-10, { duration: 50 }),
  withTiming(0, { duration: 50 }),
);
```

</details> <details> <summary><code>withRepeat(animation, count?, reverse?, callback?)</code></summary>

Repeats an animation. `count = -1` = infinite. `reverse = true` = ping-pong (forward then backward).

|Param|Type|Default|
|---|---|---|
|`count`|`number`|`2`|
|`reverse`|`boolean`|`false`|

```jsx
import { withRepeat, withTiming } from 'react-native-reanimated';

// Infinite pulse
scale.value = withRepeat(
  withTiming(1.1, { duration: 800 }),
  -1,    // infinite
  true   // reverse: ping-pong
);

// 3 times, no reverse
opacity.value = withRepeat(withTiming(0, { duration: 300 }), 3);
```

</details>

---

## Cancelling Animations

<details> <summary><code>cancelAnimation(sharedValue)</code></summary>

Immediately stops any running animation on the given shared value. The value stays at its current position.

```jsx
import { cancelAnimation } from 'react-native-reanimated';

// Stop animation mid-way
cancelAnimation(translateX);

// Then optionally snap to a position
translateX.value = 0;
```

</details>

---

## Derived Values

<details> <summary><code>useDerivedValue(worklet, dependencies?)</code></summary>

Creates a new shared value that is automatically computed from other shared values. Updates on the UI thread whenever its dependencies change. Useful for creating computed/transformed values without `useAnimatedStyle`.

|||
|---|---|
|**Returns**|`SharedValue<T>`|
|**Thread**|UI thread (worklet)|

```jsx
import { useDerivedValue, useSharedValue, withTiming } from 'react-native-reanimated';

const progress = useSharedValue(0);

// Derived values update automatically
const opacity = useDerivedValue(() => progress.value * 0.8);
const translateY = useDerivedValue(() => (1 - progress.value) * 100);
const color = useDerivedValue(() =>
  interpolateColor(progress.value, [0, 1], ['#fff', '#6366f1'])
);

// Use in animated style
const style = useAnimatedStyle(() => ({
  opacity: opacity.value,
  transform: [{ translateY: translateY.value }],
  backgroundColor: color.value,
}));
```

</details>

---

## Animated Reactions

<details> <summary><code>useAnimatedReaction(prepare, react, dependencies?)</code></summary>

Runs a side-effect worklet on the UI thread whenever a shared value changes. Use instead of `useEffect` for reacting to animated value changes without returning to JS.

|||
|---|---|
|**Thread**|UI thread|

```jsx
import { useAnimatedReaction, runOnJS } from 'react-native-reanimated';

const scrollY = useSharedValue(0);

// Trigger a JS function when scroll crosses a threshold
useAnimatedReaction(
  () => scrollY.value > 100,     // prepare: what to watch
  (isScrolled, prev) => {        // react: what to do
    if (isScrolled !== prev) {
      runOnJS(setHeaderVisible)(!isScrolled);
    }
  }
);
```

</details>

---

## Interpolation

<details> <summary><code>interpolate(value, inputRange, outputRange, extrapolation?)</code></summary>

Maps a value from an input range to an output range. Used inside worklets / `useAnimatedStyle`. Note: this is a function, not a method on the value.

|||
|---|---|
|**Thread**|UI thread (worklet)|

```jsx
import { interpolate, Extrapolation } from 'react-native-reanimated';

const style = useAnimatedStyle(() => ({
  opacity: interpolate(
    scrollY.value,
    [0, 200],
    [1, 0],
    Extrapolation.CLAMP
  ),
  transform: [{
    translateY: interpolate(
      scrollY.value,
      [0, 100],
      [0, -50],
      Extrapolation.CLAMP
    ),
  }],
}));
```

</details> <details> <summary><code>interpolateColor(value, inputRange, colors, colorSpace?, extrapolation?)</code></summary>

Maps a value to a color from an array of colors. Supports RGB and HSV color spaces.

|||
|---|---|
|**colorSpace**|`'RGB'` (default) or `'HSV'`|

```jsx
import { interpolateColor } from 'react-native-reanimated';

const style = useAnimatedStyle(() => ({
  backgroundColor: interpolateColor(
    progress.value,
    [0, 0.5, 1],
    ['#ef4444', '#f59e0b', '#22c55e']
  ),
}));

// HSV space (avoids muddy mid-transition colors)
const color = interpolateColor(
  progress.value,
  [0, 1],
  ['#ff0000', '#0000ff'],
  'HSV'
);
```

</details> <details> <summary><code>Extrapolation</code> — enum for out-of-range behavior</summary>

Controls what `interpolate` returns when the input is outside `inputRange`.

|Value|Behavior|
|---|---|
|`Extrapolation.CLAMP`|Clamps to the first/last output value|
|`Extrapolation.EXTEND`|Continues the slope linearly beyond range|
|`Extrapolation.IDENTITY`|Returns the raw input value|

```jsx
import { Extrapolation } from 'react-native-reanimated';

interpolate(value, [0, 1], [0, 100], Extrapolation.CLAMP);

// Per-side extrapolation
interpolate(value, [0, 1], [0, 100], {
  extrapolateLeft: Extrapolation.CLAMP,
  extrapolateRight: Extrapolation.EXTEND,
});
```

</details>

---

## Easing

<details> <summary>All Easing functions — from <code>react-native-reanimated</code></summary>

Import `Easing` from `react-native-reanimated` (not from `react-native`) for use with Reanimated animations.

```jsx
import { Easing } from 'react-native-reanimated';
```

|Function|Description|
|---|---|
|`Easing.linear`|Constant speed|
|`Easing.ease`|Smooth ease-in-out|
|`Easing.quad`|Quadratic (x²)|
|`Easing.cubic`|Cubic (x³)|
|`Easing.sin`|Sinusoidal|
|`Easing.circle`|Circular arc|
|`Easing.exp`|Exponential|
|`Easing.elastic(f?)`|Spring overshoot|
|`Easing.back(s?)`|Brief recoil before target|
|`Easing.bounce`|Bouncing ball|
|`Easing.bezier(x1,y1,x2,y2)`|CSS cubic-bezier|
|`Easing.bezierFn(x1,y1,x2,y2)`|Memoized bezier factory|
|`Easing.steps(n, start?)`|Stepped animation|
|`Easing.in(fn)`|Apply at the start|
|`Easing.out(fn)`|Apply at the end|
|`Easing.inOut(fn)`|Apply at both ends|

```jsx
opacity.value = withTiming(1, {
  duration: 400,
  easing: Easing.out(Easing.back(1.5)),
});
```

</details>

---

## ReduceMotion

<details> <summary><code>ReduceMotion</code> — respect accessibility motion settings</summary>

All animation functions accept a `reduceMotion` config option. When `ReduceMotion.System` is used and the OS "Reduce Motion" setting is on, the animation is replaced with an instant value change.

|Value|Behavior|
|---|---|
|`ReduceMotion.System`|Respects OS accessibility setting (recommended)|
|`ReduceMotion.Always`|Always reduces motion (testing/debug)|
|`ReduceMotion.Never`|Always animate regardless of OS setting|

```jsx
import { ReduceMotion, withTiming, withSpring } from 'react-native-reanimated';

// Per-animation
opacity.value = withTiming(1, {
  duration: 400,
  reduceMotion: ReduceMotion.System,
});

// Set default globally in your app entry
import { setDefaultAnimationConfig } from 'react-native-reanimated';
setDefaultAnimationConfig({ reduceMotion: ReduceMotion.System });
```

</details>

---

## Worklets

<details> <summary><code>'worklet'</code> directive — run a function on the UI thread</summary>

Adding `'worklet'` as the first statement in a function allows it to be called from the UI thread (e.g. inside `useAnimatedStyle`, gesture handlers).

```jsx
// A utility worklet — reusable in animated callbacks
function clamp(value, min, max) {
  'worklet';
  return Math.min(Math.max(value, min), max);
}

function lerp(a, b, t) {
  'worklet';
  return a + (b - a) * t;
}

const animatedStyle = useAnimatedStyle(() => ({
  opacity: clamp(progress.value, 0, 1),
  transform: [{ scale: lerp(0.8, 1, progress.value) }],
}));
```

</details> <details> <summary><code>runOnJS(fn)(...args)</code></summary>

Calls a JS-thread function from a worklet (UI thread). Required whenever you need to call `setState`, `navigate`, or any React/JS API from inside a worklet.

```jsx
import { runOnJS } from 'react-native-reanimated';

function onAnimationEnd() {
  setVisible(false);   // JS thread function
}

opacity.value = withTiming(0, { duration: 300 }, (finished) => {
  'worklet';
  if (finished) runOnJS(onAnimationEnd)();
});
```

</details> <details> <summary><code>runOnUI(fn)(...args)</code></summary>

Schedules a worklet to run on the UI thread from the JS thread. Use when you need to imperatively trigger UI-thread logic.

```jsx
import { runOnUI } from 'react-native-reanimated';

const triggerAnimation = () => {
  runOnUI(() => {
    'worklet';
    opacity.value = withSpring(1);
    scale.value = withSpring(1);
  })();
};
```

</details>

---

## Animated Components

<details> <summary>Built-in animated components</summary>

Reanimated exports animated versions of core RN components. Pass `useAnimatedStyle` results to their `style` prop.

```jsx
import Animated from 'react-native-reanimated';

<Animated.View style={animatedStyle} />
<Animated.Text style={animatedStyle} />
<Animated.Image source={source} style={animatedStyle} />
<Animated.ScrollView onScroll={scrollHandler} scrollEventThrottle={16} />
<Animated.FlatList data={data} renderItem={renderItem} onScroll={scrollHandler} />
```

</details> <details> <summary><code>Animated.createAnimatedComponent(Component)</code></summary>

Wraps any third-party component to make it compatible with Reanimated. The component must accept a `style` prop and forward refs.

```jsx
import Animated from 'react-native-reanimated';
import { BlurView } from '@react-native-community/blur';
import Svg, { Path } from 'react-native-svg';

const AnimatedBlurView = Animated.createAnimatedComponent(BlurView);
const AnimatedPath = Animated.createAnimatedComponent(Path);

// Now use with useAnimatedProps or useAnimatedStyle
<AnimatedBlurView style={animatedStyle} blurType="light" />
```

</details>

---

## Scroll Hooks

<details> <summary><code>useAnimatedScrollHandler(handlers, dependencies?)</code></summary>

Creates an optimized scroll event handler that runs entirely on the UI thread. Replaces `Animated.event` for scroll tracking.

```jsx
import { useAnimatedScrollHandler, useSharedValue } from 'react-native-reanimated';

const scrollY = useSharedValue(0);

const scrollHandler = useAnimatedScrollHandler({
  onScroll: (event) => {
    'worklet';
    scrollY.value = event.contentOffset.y;
  },
  onBeginDrag: (event) => {
    'worklet';
    // handle drag start
  },
  onEndDrag: (event) => {
    'worklet';
    // handle drag end
  },
  onMomentumBegin: (event) => {
    'worklet';
  },
  onMomentumEnd: (event) => {
    'worklet';
  },
});

<Animated.ScrollView onScroll={scrollHandler} scrollEventThrottle={16} />
```

</details> <details> <summary><code>useScrollViewOffset(animatedRef)</code></summary>

Returns a shared value tracking the scroll offset of an `Animated.ScrollView`. Handles both horizontal and vertical automatically.

```jsx
import { useScrollViewOffset, useAnimatedRef } from 'react-native-reanimated';

const scrollRef = useAnimatedRef();
const scrollOffset = useScrollViewOffset(scrollRef);

const headerStyle = useAnimatedStyle(() => ({
  opacity: interpolate(scrollOffset.value, [0, 100], [1, 0], Extrapolation.CLAMP),
}));

<Animated.ScrollView ref={scrollRef}>
  ...
</Animated.ScrollView>
```

</details>

---

## Keyboard Hooks

<details> <summary><code>useAnimatedKeyboard(options?)</code></summary>

Returns the animated keyboard height and state. Animates in sync with the keyboard show/hide transition on the UI thread.

|Returned field|Type|Description|
|---|---|---|
|`height`|`SharedValue<number>`|Current keyboard height (0 when hidden)|
|`state`|`SharedValue<KeyboardState>`|`UNKNOWN`, `OPENING`, `OPEN`, `CLOSING`, `CLOSED`|

```jsx
import { useAnimatedKeyboard, useAnimatedStyle } from 'react-native-reanimated';

function KeyboardAvoidingView({ children }) {
  const keyboard = useAnimatedKeyboard();

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: -keyboard.height.value }],
  }));

  return (
    <Animated.View style={[styles.container, animatedStyle]}>
      {children}
    </Animated.View>
  );
}
```

</details> <details> <summary><code>useKeyboardHandler(handlers, dependencies?)</code></summary>

Provides fine-grained keyboard animation lifecycle hooks that run on the UI thread, matching the keyboard animation curve exactly.

```jsx
import { useKeyboardHandler } from 'react-native-reanimated';

const keyboardHeight = useSharedValue(0);

useKeyboardHandler({
  onStart: (event) => {
    'worklet';
    // event.height, event.duration, event.progress
  },
  onMove: (event) => {
    'worklet';
    keyboardHeight.value = event.height;
  },
  onEnd: (event) => {
    'worklet';
    keyboardHeight.value = event.height;
  },
}, []);
```

</details>

---

## Sensor Hooks

<details> <summary><code>useAnimatedSensor(sensorType, config?)</code></summary>

Subscribes to device sensor data (accelerometer, gyroscope, etc.) as shared values on the UI thread. Useful for tilt/parallax effects.

|||
|---|---|
|**Returns**|`{ sensor: SharedValue<SensorData>, unregister: () => void }`|

```jsx
import { useAnimatedSensor, SensorType } from 'react-native-reanimated';

const gyroscope = useAnimatedSensor(SensorType.GYROSCOPE, { interval: 16 });
const accelerometer = useAnimatedSensor(SensorType.ACCELEROMETER);

const tiltStyle = useAnimatedStyle(() => ({
  transform: [
    { rotateX: `${gyroscope.sensor.value.y * 20}deg` },
    { rotateY: `${gyroscope.sensor.value.x * 20}deg` },
  ],
}));
```

**Sensor types:** `SensorType.ACCELEROMETER`, `SensorType.GYROSCOPE`, `SensorType.GRAVITY`, `SensorType.MAGNETIC_FIELD`, `SensorType.ROTATION`

</details>

---

## Ref & Measurement Hooks

<details> <summary><code>useAnimatedRef()</code></summary>

Creates a ref compatible with both React Native and Reanimated. Required for `measure`, `scrollTo`, and `useScrollViewOffset`.

```jsx
import { useAnimatedRef } from 'react-native-reanimated';

const ref = useAnimatedRef();

<Animated.View ref={ref} />
```

</details> <details> <summary><code>measure(animatedRef)</code></summary>

Measures the on-screen dimensions and position of a component. Must be called inside a worklet.

|Returns|Description|
|---|---|
|`x, y`|Position relative to parent|
|`width, height`|Component dimensions|
|`pageX, pageY`|Position relative to screen|

```jsx
import { measure, runOnUI } from 'react-native-reanimated';

const ref = useAnimatedRef();

const handlePress = () => {
  runOnUI(() => {
    'worklet';
    const layout = measure(ref);
    if (layout) {
      console.log(layout.width, layout.height, layout.pageX, layout.pageY);
    }
  })();
};
```

</details> <details> <summary><code>scrollTo(animatedRef, x, y, animated)</code></summary>

Imperatively scrolls a scroll view from a worklet on the UI thread.

```jsx
import { scrollTo, useAnimatedRef } from 'react-native-reanimated';

const scrollRef = useAnimatedRef();

const scrollToTop = () => {
  runOnUI(() => {
    'worklet';
    scrollTo(scrollRef, 0, 0, true);
  })();
};
```

</details>

---

## Frame Callback

<details> <summary><code>useFrameCallback(callback, autoStart?)</code></summary>

Calls a worklet on every UI frame (every ~16ms). Use for custom per-frame animation logic. Returns `{ isActive, callOnce, start, stop }`.

|||
|---|---|
|**autoStart**|`true`|
|**Thread**|UI thread|

```jsx
import { useFrameCallback, useSharedValue } from 'react-native-reanimated';

const progress = useSharedValue(0);

useFrameCallback((frameInfo) => {
  'worklet';
  // frameInfo.timestamp — current time in ms
  // frameInfo.timeSincePreviousFrame — delta time
  progress.value = (progress.value + 0.01) % 1;
});

const style = useAnimatedStyle(() => ({
  transform: [{ rotate: `${progress.value * 360}deg` }],
}));
```

</details>

---

## Layout Animations — Entering

<details> <summary>Fade entering presets</summary>

Apply with the `entering` prop on any Animated component. The animation triggers when the component mounts.

```jsx
import { FadeIn, FadeInUp, FadeInDown, FadeInLeft, FadeInRight } from 'react-native-reanimated';

<Animated.View entering={FadeIn} />
<Animated.View entering={FadeInUp.duration(400)} />
<Animated.View entering={FadeInDown.delay(200).duration(500)} />
<Animated.View entering={FadeInLeft.springify()} />
<Animated.View entering={FadeInRight.damping(15).stiffness(150)} />
```

</details> <details> <summary>Slide entering presets</summary>

```jsx
import {
  SlideInUp, SlideInDown, SlideInLeft, SlideInRight
} from 'react-native-reanimated';

<Animated.View entering={SlideInLeft.duration(300)} />
<Animated.View entering={SlideInUp.springify().damping(15)} />
```

</details> <details> <summary>Zoom / Scale entering presets</summary>

```jsx
import {
  ZoomIn, ZoomInDown, ZoomInUp, ZoomInLeft, ZoomInRight,
  ZoomInEasyDown, ZoomInEasyUp
} from 'react-native-reanimated';

<Animated.View entering={ZoomIn.duration(350)} />
<Animated.View entering={ZoomInDown.springify()} />
```

</details> <details> <summary>Bounce entering presets</summary>

```jsx
import {
  BounceIn, BounceInUp, BounceInDown, BounceInLeft, BounceInRight
} from 'react-native-reanimated';

<Animated.View entering={BounceIn} />
<Animated.View entering={BounceInUp.duration(600)} />
```

</details> <details> <summary>Flip entering presets</summary>

```jsx
import {
  FlipInEasyX, FlipInEasyY, FlipInXDown, FlipInXUp,
  FlipInYLeft, FlipInYRight
} from 'react-native-reanimated';

<Animated.View entering={FlipInEasyX.duration(400)} />
<Animated.View entering={FlipInYLeft.springify()} />
```

</details> <details> <summary>LightSpeed entering presets</summary>

```jsx
import { LightSpeedInLeft, LightSpeedInRight } from 'react-native-reanimated';

<Animated.View entering={LightSpeedInLeft.duration(400)} />
```

</details> <details> <summary>Pinwheel, Roll, Rotate entering presets</summary>

```jsx
import { PinwheelIn, RollInLeft, RollInRight, RotateInDownLeft } from 'react-native-reanimated';

<Animated.View entering={PinwheelIn.duration(500)} />
<Animated.View entering={RollInLeft} />
```

</details> <details> <summary>Modifier methods on entering presets</summary>

All entering animations support a fluent chain of modifier methods:

|Method|Description|
|---|---|
|`.duration(ms)`|Override animation duration|
|`.delay(ms)`|Delay before starting|
|`.springify()`|Switch from timing to spring|
|`.damping(n)`|Spring damping (use after `.springify()`)|
|`.stiffness(n)`|Spring stiffness|
|`.mass(n)`|Spring mass|
|`.withInitialValues(style)`|Override starting style values|
|`.withCallback(cb)`|Run callback when animation finishes|
|`.reduceMotion(mode)`|Override reduce motion behavior|
|`.randomDelay()`|Randomize delay (useful for stagger)|

```jsx
<Animated.View
  entering={FadeInUp
    .duration(500)
    .delay(200)
    .springify()
    .damping(15)
    .withCallback((finished) => {
      'worklet';
      if (finished) runOnJS(onEntered)();
    })
  }
/>
```

</details>

---

## Layout Animations — Exiting

<details> <summary>Exiting presets — mirror of entering</summary>

Apply with the `exiting` prop. Triggers when the component is removed from the tree.

```jsx
import {
  FadeOut, FadeOutDown, FadeOutUp, FadeOutLeft, FadeOutRight,
  SlideOutDown, SlideOutUp, SlideOutLeft, SlideOutRight,
  ZoomOut, ZoomOutDown,
  BounceOut, BounceOutDown,
  FlipOutEasyX, FlipOutEasyY,
  LightSpeedOutLeft,
  PinwheelOut, RollOutLeft,
} from 'react-native-reanimated';

<Animated.View exiting={FadeOut.duration(300)} />
<Animated.View exiting={SlideOutDown.springify()} />
<Animated.View exiting={ZoomOut.duration(250)} />
```

> ⚠️ For `exiting` to work, the parent must not be unmounted before the child animation completes. Wrap list items and conditional renders carefully.

</details>

---

## Layout Animations — Layout Transitions

<details> <summary><code>LinearTransition</code> — smooth layout changes</summary>

Animates when a component's size or position changes due to sibling additions/removals or state changes.

```jsx
import { LinearTransition } from 'react-native-reanimated';

<Animated.View layout={LinearTransition} />
<Animated.View layout={LinearTransition.duration(300).delay(100)} />

// Short alias
import { Layout } from 'react-native-reanimated';
<Animated.View layout={Layout} />
```

</details> <details> <summary><code>SequencedTransition</code> — new position first or old position first</summary>

Two-stage layout change: move first, then resize (or vice versa).

```jsx
import { SequencedTransition } from 'react-native-reanimated';

<Animated.View layout={SequencedTransition.duration(400)} />
<Animated.View layout={SequencedTransition.reverse()} /> // resize first
```

</details> <details> <summary><code>FadingTransition</code> — fade out then fade in at new position</summary>

```jsx
import { FadingTransition } from 'react-native-reanimated';
<Animated.View layout={FadingTransition.duration(350)} />
```

</details> <details> <summary><code>JumpingTransition</code> — jumps up then lands at new position</summary>

```jsx
import { JumpingTransition } from 'react-native-reanimated';
<Animated.View layout={JumpingTransition} />
```

</details> <details> <summary><code>CurvedTransition</code> — follows a curved path between positions</summary>

```jsx
import { CurvedTransition, Easing } from 'react-native-reanimated';

<Animated.View layout={CurvedTransition.easingX(Easing.ease).easingY(Easing.bounce)} />
```

</details> <details> <summary><code>EntryExitTransition</code> — different animations for entering/exiting elements in a list</summary>

```jsx
import { EntryExitTransition, FadeIn, FadeOut } from 'react-native-reanimated';

<Animated.View layout={EntryExitTransition.entering(FadeIn).exiting(FadeOut)} />
```

</details> <details> <summary><code>LayoutAnimationConfig</code> — configure layout animation for a subtree</summary>

Wraps children to enable/disable layout animations or skip the animation on first mount.

```jsx
import { LayoutAnimationConfig } from 'react-native-reanimated';

// Skip entering animation on initial render
<LayoutAnimationConfig skipEntering>
  {items.map(item => (
    <Animated.View key={item.id} entering={FadeIn} exiting={FadeOut} layout={LinearTransition}>
      <ItemCard item={item} />
    </Animated.View>
  ))}
</LayoutAnimationConfig>
```

</details>

---

## Keyframe Animations

<details> <summary><code>Keyframe</code> — CSS-style keyframe animations</summary>

Define multi-step animations using percentage-based keyframes. Use as `entering` or `exiting` props.

```jsx
import { Keyframe } from 'react-native-reanimated';

const enteringAnimation = new Keyframe({
  0: {
    opacity: 0,
    transform: [{ scale: 0.8 }, { translateY: 20 }],
    easing: Easing.out(Easing.ease),
  },
  60: {
    opacity: 1,
    transform: [{ scale: 1.05 }, { translateY: -5 }],
  },
  100: {
    opacity: 1,
    transform: [{ scale: 1 }, { translateY: 0 }],
  },
}).duration(500);

const exitingAnimation = new Keyframe({
  0: { opacity: 1, transform: [{ scale: 1 }] },
  100: { opacity: 0, transform: [{ scale: 0.6 }] },
}).duration(300);

<Animated.View entering={enteringAnimation} exiting={exitingAnimation} />
```

</details>

---

## Gesture Handler Integration

<details> <summary>Gesture Handler v2 + Reanimated (recommended)</summary>

Use `react-native-gesture-handler` v2 with Reanimated for gesture-driven animations that run fully on the UI thread.

```jsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, {
  useSharedValue, useAnimatedStyle,
  withSpring, withDecay, runOnJS,
} from 'react-native-reanimated';

function DraggableCard() {
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);
  const context = useSharedValue({ x: 0, y: 0 });
  const scale = useSharedValue(1);

  const panGesture = Gesture.Pan()
    .onStart(() => {
      context.value = { x: translateX.value, y: translateY.value };
      scale.value = withSpring(1.05);
    })
    .onUpdate((event) => {
      translateX.value = event.translationX + context.value.x;
      translateY.value = event.translationY + context.value.y;
    })
    .onEnd((event) => {
      scale.value = withSpring(1);
      translateX.value = withDecay({ velocity: event.velocityX, clamp: [-150, 150] });
      translateY.value = withDecay({ velocity: event.velocityY, clamp: [-300, 300] });
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { translateY: translateY.value },
      { scale: scale.value },
    ],
  }));

  return (
    <GestureDetector gesture={panGesture}>
      <Animated.View style={[styles.card, animatedStyle]} />
    </GestureDetector>
  );
}
```

</details> <details> <summary>Tap gesture with spring feedback</summary>

```jsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';

function SpringButton({ onPress, children }) {
  const scale = useSharedValue(1);

  const tapGesture = Gesture.Tap()
    .onBegin(() => {
      scale.value = withSpring(0.92, { damping: 15, stiffness: 300 });
    })
    .onFinalize(() => {
      scale.value = withSpring(1, { damping: 15, stiffness: 300 });
    })
    .onEnd(() => {
      runOnJS(onPress)();
    });

  const style = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return (
    <GestureDetector gesture={tapGesture}>
      <Animated.View style={[styles.button, style]}>
        {children}
      </Animated.View>
    </GestureDetector>
  );
}
```

</details> <details> <summary>Pinch to zoom</summary>

```jsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';

function PinchableImage({ source }) {
  const scale = useSharedValue(1);
  const savedScale = useSharedValue(1);

  const pinchGesture = Gesture.Pinch()
    .onUpdate((event) => {
      scale.value = savedScale.value * event.scale;
    })
    .onEnd(() => {
      if (scale.value < 1) {
        scale.value = withSpring(1);
        savedScale.value = 1;
      } else {
        savedScale.value = scale.value;
      }
    });

  const style = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return (
    <GestureDetector gesture={pinchGesture}>
      <Animated.Image source={source} style={[styles.image, style]} />
    </GestureDetector>
  );
}
```

</details>

---

## Common Patterns

<details> <summary>Fade in on mount</summary>

```jsx
function FadeIn({ children, delay = 0 }) {
  const opacity = useSharedValue(0);

  useEffect(() => {
    opacity.value = withDelay(delay, withTiming(1, { duration: 400 }));
  }, []);

  const style = useAnimatedStyle(() => ({ opacity: opacity.value }));

  return <Animated.View style={style}>{children}</Animated.View>;
}
```

</details> <details> <summary>Slide in from bottom with spring</summary>

```jsx
function SlideUp({ children }) {
  const translateY = useSharedValue(60);
  const opacity = useSharedValue(0);

  useEffect(() => {
    translateY.value = withSpring(0, { damping: 18, stiffness: 180 });
    opacity.value = withTiming(1, { duration: 350 });
  }, []);

  const style = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [{ translateY: translateY.value }],
  }));

  return <Animated.View style={style}>{children}</Animated.View>;
}
```

</details> <details> <summary>Stagger list entrance with layout animations</summary>

```jsx
import { FadeInDown, LinearTransition, LayoutAnimationConfig } from 'react-native-reanimated';

function AnimatedList({ items }) {
  return (
    <LayoutAnimationConfig skipEntering={false}>
      {items.map((item, index) => (
        <Animated.View
          key={item.id}
          entering={FadeInDown.delay(index * 60).duration(400).springify()}
          exiting={FadeOut.duration(200)}
          layout={LinearTransition}
        >
          <ItemRow item={item} />
        </Animated.View>
      ))}
    </LayoutAnimationConfig>
  );
}
```

</details> <details> <summary>Scroll-based parallax header</summary>

```jsx
const HEADER_HEIGHT = 280;

function ParallaxScreen() {
  const scrollRef = useAnimatedRef();
  const scrollOffset = useScrollViewOffset(scrollRef);

  const headerStyle = useAnimatedStyle(() => ({
    transform: [{
      translateY: interpolate(
        scrollOffset.value,
        [-HEADER_HEIGHT, 0, HEADER_HEIGHT],
        [-HEADER_HEIGHT / 2, 0, HEADER_HEIGHT * 0.75],
        Extrapolation.CLAMP
      ),
    }],
  }));

  const headerOpacity = useAnimatedStyle(() => ({
    opacity: interpolate(
      scrollOffset.value,
      [0, HEADER_HEIGHT / 2],
      [1, 0],
      Extrapolation.CLAMP
    ),
  }));

  return (
    <Animated.ScrollView ref={scrollRef} scrollEventThrottle={16}>
      <Animated.View style={[styles.header, headerStyle]}>
        <Animated.Image source={heroImage} style={[StyleSheet.absoluteFillObject, headerOpacity]} />
      </Animated.View>
      <View style={styles.content}>...</View>
    </Animated.ScrollView>
  );
}
```

</details> <details> <summary>Animated tab bar indicator</summary>

```jsx
function TabBar({ tabs, activeIndex, onTabPress }) {
  const indicatorX = useSharedValue(0);
  const TAB_WIDTH = SCREEN_WIDTH / tabs.length;

  useEffect(() => {
    indicatorX.value = withSpring(activeIndex * TAB_WIDTH, {
      damping: 20,
      stiffness: 200,
    });
  }, [activeIndex]);

  const indicatorStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: indicatorX.value }],
  }));

  return (
    <View style={styles.tabBar}>
      {tabs.map((tab, i) => (
        <TouchableOpacity key={tab} style={styles.tab} onPress={() => onTabPress(i)}>
          <Text>{tab}</Text>
        </TouchableOpacity>
      ))}
      <Animated.View style={[styles.indicator, indicatorStyle, { width: TAB_WIDTH }]} />
    </View>
  );
}
```

</details> <details> <summary>Swipeable card with snap-back or dismiss</summary>

```jsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

const DISMISS_THRESHOLD = 120;

function SwipeableCard({ onDismiss, children }) {
  const translateX = useSharedValue(0);
  const opacity = useSharedValue(1);

  const panGesture = Gesture.Pan()
    .onUpdate((event) => {
      translateX.value = event.translationX;
      opacity.value = interpolate(
        Math.abs(event.translationX),
        [0, DISMISS_THRESHOLD],
        [1, 0.4],
        Extrapolation.CLAMP
      );
    })
    .onEnd((event) => {
      if (Math.abs(event.translationX) > DISMISS_THRESHOLD) {
        const direction = event.translationX > 0 ? 1 : -1;
        translateX.value = withTiming(direction * SCREEN_WIDTH, {}, () => {
          runOnJS(onDismiss)();
        });
      } else {
        translateX.value = withSpring(0);
        opacity.value = withTiming(1);
      }
    });

  const style = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { rotate: `${interpolate(translateX.value, [-200, 200], [-15, 15])}deg` },
    ],
    opacity: opacity.value,
  }));

  return (
    <GestureDetector gesture={panGesture}>
      <Animated.View style={[styles.card, style]}>{children}</Animated.View>
    </GestureDetector>
  );
}
```

</details> <details> <summary>Keyboard-aware animated view</summary>

```jsx
function KeyboardAware({ children }) {
  const keyboard = useAnimatedKeyboard();

  const containerStyle = useAnimatedStyle(() => ({
    transform: [{
      translateY: interpolate(
        keyboard.height.value,
        [0, keyboard.height.value],
        [0, -keyboard.height.value],
        Extrapolation.CLAMP
      ),
    }],
  }));

  return (
    <Animated.View style={[styles.flex, containerStyle]}>
      {children}
    </Animated.View>
  );
}
```

</details> <details> <summary>Infinite shimmer / skeleton loading</summary>

```jsx
function Shimmer({ width = '100%', height = 16, borderRadius = 4 }) {
  const progress = useSharedValue(0);

  useEffect(() => {
    progress.value = withRepeat(
      withTiming(1, { duration: 1200, easing: Easing.inOut(Easing.ease) }),
      -1,
      true
    );
  }, []);

  const style = useAnimatedStyle(() => ({
    backgroundColor: interpolateColor(
      progress.value,
      [0, 1],
      ['#e5e7eb', '#f3f4f6']
    ),
  }));

  return <Animated.View style={[{ height, borderRadius }, { width }, style]} />;
}
```

</details>

---

## Reanimated vs Animated — Quick Comparison

|Feature|`Animated` (core)|`Reanimated`|
|---|---|---|
|Thread|JS thread|UI thread (JSI)|
|Blocked by heavy JS|Yes|No|
|`useNativeDriver`|Required flag|Always native|
|Layout animations|❌|✅|
|Shared element transitions|❌|✅|
|Gesture integration|Limited|Full (with RNGH v2)|
|Keyboard animation sync|❌|✅|
|Sensor hooks|❌|✅|
|API style|Imperative chains|Declarative hooks|
|`interpolate` syntax|Method on value|Standalone function|
|Bundle size|Included in RN|Additional dependency|

---

## Quick-Reference Cheatsheet

|API|Use case|
|---|---|
|`useSharedValue(n)`|Create animated value|
|`useAnimatedStyle(() => ({}))`|Drive styles from shared values|
|`useAnimatedProps(() => ({}))`|Drive non-style props (SVG, inputs)|
|`withTiming(v, config)`|Duration-based smooth animation|
|`withSpring(v, config)`|Physics spring animation|
|`withDecay(config)`|Fling / momentum animation|
|`withDelay(ms, anim)`|Delay before animation|
|`withSequence(...anims)`|Chain animations on one value|
|`withRepeat(anim, n, reverse)`|Loop or ping-pong an animation|
|`cancelAnimation(sv)`|Stop a running animation|
|`useDerivedValue(() => ...)`|Computed shared value|
|`useAnimatedReaction`|UI-thread side effects|
|`interpolate(v, in, out)`|Map value to output range|
|`interpolateColor(v, in, colors)`|Animate between colors|
|`runOnJS(fn)(args)`|Call JS from a worklet|
|`runOnUI(fn)(args)`|Call worklet from JS|
|`useAnimatedScrollHandler`|Track scroll on UI thread|
|`useScrollViewOffset(ref)`|Simple scroll offset tracking|
|`useAnimatedKeyboard()`|Keyboard-synced animations|
|`entering={FadeIn}`|Mount animation|
|`exiting={FadeOut}`|Unmount animation|
|`layout={LinearTransition}`|Animate layout changes|
|`new Keyframe({...})`|CSS-style keyframes|

---

_Reference based on `react-native-reanimated` v3. Always check the [official docs](https://docs.swmansion.com/react-native-reanimated/) for the latest updates._