# React Native Animated — Complete Reference

A comprehensive reference for the `Animated` API in React Native.  
Covers Animated Values, animation types, composition, interpolation, Easing, and common patterns.

---

## Table of Contents

1. [Animated Values](#animated-values)
2. [Animated.Value Methods](#animatedvalue-methods)
3. [Animated.ValueXY Methods](#animatedvaluexy-methods)
4. [Animation Types](#animation-types)
5. [Timing Config](#timing-config)
6. [Spring Config](#spring-config)
7. [Decay Config](#decay-config)
8. [Animation Control](#animation-control)
9. [Composing Animations](#composing-animations)
10. [Combining Values](#combining-values)
11. [Interpolation](#interpolation)
12. [Easing](#easing)
13. [Animated Components](#animated-components)
14. [Hooks](#hooks)
15. [Common Patterns](#common-patterns)

---

## Animated Values

<details>
<summary><code>Animated.Value(initialValue)</code></summary>

The core building block. A single numeric value that can be animated over time. All animation drivers target an `Animated.Value`.

| | |
|---|---|
| **Type** | `number` |

```jsx
import { Animated } from 'react-native';

// Declaration — always outside render (or use useRef)
const opacity = useRef(new Animated.Value(0)).current;
const translateY = useRef(new Animated.Value(-50)).current;

// Use in style
<Animated.View style={{ opacity, transform: [{ translateY }] }} />
```

> ⚠️ Always declare with `useRef` (hooks) or as a class field — **never** inside the render function or `useState`. Creating a new `Animated.Value` on every render breaks animations.

</details>

<details>
<summary><code>Animated.ValueXY({ x, y })</code></summary>

Holds two `Animated.Value`s (`x` and `y`) in a single object. Ideal for tracking 2D position (dragging, panning).

| | |
|---|---|
| **Type** | `{ x: number, y: number }` |

```jsx
const position = useRef(new Animated.ValueXY({ x: 0, y: 0 })).current;

// Use as transform
<Animated.View style={{ transform: position.getTranslateTransform() }} />

// Use as layout (top/left)
<Animated.View style={position.getLayout()} />
```

</details>

<details>
<summary><code>Animated.Color(value)</code></summary>

Holds an animated color value. Introduced in React Native 0.72+. Supports interpolation between color strings.

| | |
|---|---|
| **Type** | `string` (CSS color) |

```jsx
const bgColor = useRef(new Animated.Color('#ffffff')).current;

Animated.timing(bgColor, {
  toValue: '#6366f1',
  duration: 400,
  useNativeDriver: false, // color animations can't use native driver
}).start();

<Animated.View style={{ backgroundColor: bgColor }} />
```

</details>

---

## Animated.Value Methods

<details>
<summary><code>.setValue(value)</code></summary>

Immediately sets the value without animation. Useful for resetting after an animation completes.

```jsx
opacity.setValue(0);        // snap to 0
translateX.setValue(-300);  // snap back to start
```

</details>

<details>
<summary><code>.setOffset(offset)</code></summary>

Sets an offset that is applied on top of any animated value. Useful to "lock in" a current drag position before starting a new gesture.

```jsx
position.setOffset(50);
// Now the value 0 renders as 50 on screen
```

</details>

<details>
<summary><code>.flattenOffset()</code></summary>

Merges the current offset into the base value and resets the offset to `0`. Call this after `setOffset` to clean up.

```jsx
// After gesture ends:
position.flattenOffset();
// offset is now 0, base value includes the prior offset
```

</details>

<details>
<summary><code>.extractOffset()</code></summary>

Sets the offset to the current value and resets the base value to `0`. Useful at the start of a new gesture so that movement is relative.

```jsx
// Before new gesture starts:
position.extractOffset();
```

</details>

<details>
<summary><code>.addListener(callback) → id</code></summary>

Subscribes to value changes. Returns a string ID used to remove the listener. Runs on the JS thread even when `useNativeDriver` is `true`.

```jsx
const id = opacity.addListener(({ value }) => {
  console.log('Current opacity:', value);
});
```

</details>

<details>
<summary><code>.removeListener(id)</code></summary>

Removes a specific listener by its ID string.

```jsx
opacity.removeListener(id);
```

</details>

<details>
<summary><code>.removeAllListeners()</code></summary>

Removes all registered listeners for this value.

```jsx
opacity.removeAllListeners();
```

</details>

<details>
<summary><code>.stopAnimation(callback?)</code></summary>

Stops any running animation on this value. Calls `callback` with the final value when stopped.

```jsx
opacity.stopAnimation((finalValue) => {
  console.log('Stopped at:', finalValue);
});
```

</details>

<details>
<summary><code>.resetAnimation(callback?)</code></summary>

Stops the animation and resets the value to its initial value. Calls `callback` with the reset value.

```jsx
opacity.resetAnimation((value) => {
  console.log('Reset to:', value); // back to initial
});
```

</details>

<details>
<summary><code>.interpolate(config)</code></summary>

Returns a new derived value that maps the input range to an output range. Does not modify the original value.  
See [Interpolation](#interpolation) section for full config details.

```jsx
const rotate = opacity.interpolate({
  inputRange: [0, 1],
  outputRange: ['0deg', '360deg'],
});
```

</details>

---

## Animated.ValueXY Methods

<details>
<summary><code>.setValue({ x, y })</code></summary>

Immediately sets both `x` and `y` values.

```jsx
position.setValue({ x: 0, y: 0 });
```

</details>

<details>
<summary><code>.setOffset({ x, y })</code></summary>

Sets the offset for both axes simultaneously.

```jsx
position.setOffset({ x: 10, y: 20 });
```

</details>

<details>
<summary><code>.flattenOffset()</code></summary>

Merges offsets into base values and resets offsets to `{ x: 0, y: 0 }`.

```jsx
position.flattenOffset();
```

</details>

<details>
<summary><code>.extractOffset()</code></summary>

Moves current values into the offset and resets base values to `{ x: 0, y: 0 }`. Use at the start of each gesture.

```jsx
position.extractOffset(); // call in onPanResponderGrant
```

</details>

<details>
<summary><code>.getLayout()</code></summary>

Returns a style object `{ left: Animated.Value, top: Animated.Value }`. Use for absolute positioning.

```jsx
<Animated.View style={[styles.box, position.getLayout()]} />
```

</details>

<details>
<summary><code>.getTranslateTransform()</code></summary>

Returns a transform array `[{ translateX }, { translateY }]` suitable for the `transform` style prop.

```jsx
<Animated.View style={{ transform: position.getTranslateTransform() }} />
```

</details>

<details>
<summary><code>.addListener(callback) → id</code></summary>

Subscribes to changes on both `x` and `y`. Callback receives `{ x, y }`.

```jsx
const id = position.addListener(({ x, y }) => {
  console.log(x, y);
});
```

</details>

<details>
<summary><code>.stopAnimation(callback?)</code></summary>

Stops animations on both `x` and `y`. Callback receives `{ x, y }`.

```jsx
position.stopAnimation(({ x, y }) => console.log('Stopped at', x, y));
```

</details>

---

## Animation Types

<details>
<summary><code>Animated.timing(value, config)</code></summary>

Animates a value along a timed easing curve. The most commonly used animation type.

```jsx
Animated.timing(opacity, {
  toValue: 1,
  duration: 300,
  easing: Easing.out(Easing.ease),
  useNativeDriver: true,
}).start();
```

See [Timing Config](#timing-config) for all options.

</details>

<details>
<summary><code>Animated.spring(value, config)</code></summary>

Animates a value using a spring physics model. Creates naturally bouncy motion without specifying a duration.

```jsx
Animated.spring(scale, {
  toValue: 1,
  damping: 10,
  mass: 1,
  stiffness: 100,
  useNativeDriver: true,
}).start();
```

See [Spring Config](#spring-config) for all options.

</details>

<details>
<summary><code>Animated.decay(value, config)</code></summary>

Animates a value from an initial velocity, gradually decelerating to zero. Used for momentum/fling-style animations.

```jsx
Animated.decay(position, {
  velocity: { x: 10, y: 5 },
  deceleration: 0.997,
  useNativeDriver: true,
}).start();
```

See [Decay Config](#decay-config) for all options.

</details>

---

## Timing Config

<details>
<summary><code>toValue</code> — <em>number | Animated.Value</em> ⚠️ Required</summary>

The target value to animate to.

| | |
|---|---|
| **Type** | `number \| AnimatedValue` |

```jsx
Animated.timing(opacity, { toValue: 1, duration: 300, useNativeDriver: true }).start();
```

</details>

<details>
<summary><code>duration</code> — <em>number</em></summary>

Length of the animation in milliseconds.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `500` |

```jsx
Animated.timing(opacity, { toValue: 1, duration: 200, useNativeDriver: true }).start();
```

</details>

<details>
<summary><code>easing</code> — <em>function</em></summary>

Easing function that defines the animation curve. Import from `Easing`.

| | |
|---|---|
| **Type** | `(t: number) => number` |
| **Default** | `Easing.inOut(Easing.ease)` |

```jsx
import { Easing } from 'react-native';

Animated.timing(translateY, {
  toValue: 0,
  duration: 400,
  easing: Easing.out(Easing.back(1.5)),
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>delay</code> — <em>number</em></summary>

Delay in milliseconds before the animation starts.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `0` |

```jsx
Animated.timing(opacity, {
  toValue: 1,
  duration: 300,
  delay: 500,
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>useNativeDriver</code> — <em>boolean</em> ⚠️ Important</summary>

Offloads the animation to the native thread, resulting in smoother 60fps performance — **especially critical on Android**. Set to `true` whenever possible.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` (required to be explicitly set) |

**Supports native driver:** `opacity`, `transform` (`translateX`, `translateY`, `scale`, `rotate`, etc.)  
**Does NOT support:** `width`, `height`, `top`, `left`, `backgroundColor`, `borderRadius`, and other layout/color props.

```jsx
// ✅ Works with native driver
Animated.timing(opacity, { toValue: 1, duration: 300, useNativeDriver: true }).start();

// ❌ Must use useNativeDriver: false
Animated.timing(height, { toValue: 200, duration: 300, useNativeDriver: false }).start();
```

</details>

<details>
<summary><code>isInteraction</code> — <em>boolean</em></summary>

If `true`, this animation registers with the `InteractionManager` — it will delay tasks queued with `InteractionManager.runAfterInteractions` until the animation completes.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |

```jsx
Animated.timing(opacity, {
  toValue: 1,
  duration: 300,
  isInteraction: false, // don't block runAfterInteractions
  useNativeDriver: true,
}).start();
```

</details>

---

## Spring Config

<details>
<summary><code>toValue</code> — <em>number | Animated.Value</em> ⚠️ Required</summary>

The target value (the "resting position" of the spring).

```jsx
Animated.spring(scale, { toValue: 1, useNativeDriver: true }).start();
```

</details>

<details>
<summary><code>damping</code> — <em>number</em></summary>

Friction force opposing the spring. Higher = less oscillation. Use with `mass` and `stiffness`.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `10` |

```jsx
Animated.spring(scale, { toValue: 1, damping: 20, stiffness: 150, useNativeDriver: true }).start();
```

</details>

<details>
<summary><code>mass</code> — <em>number</em></summary>

Mass of the object on the spring. Higher = slower to accelerate and slower to stop.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `1` |

```jsx
Animated.spring(scale, { toValue: 1, mass: 2, damping: 20, useNativeDriver: true }).start();
```

</details>

<details>
<summary><code>stiffness</code> — <em>number</em></summary>

Spring stiffness constant. Higher = snappier spring.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `100` |

```jsx
Animated.spring(scale, { toValue: 1, stiffness: 200, damping: 15, useNativeDriver: true }).start();
```

</details>

<details>
<summary><code>overshootClamping</code> — <em>boolean</em></summary>

If `true`, prevents the spring from bouncing past the `toValue`.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
Animated.spring(opacity, {
  toValue: 1,
  overshootClamping: true,
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>restDisplacementThreshold</code> — <em>number</em></summary>

Distance (in units) below which the spring snaps to `toValue` and stops.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `0.001` |

```jsx
Animated.spring(scale, {
  toValue: 1,
  restDisplacementThreshold: 0.01,
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>restSpeedThreshold</code> — <em>number</em></summary>

Speed (in units/s) below which the spring snaps to `toValue` and stops.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `0.001` |

```jsx
Animated.spring(scale, {
  toValue: 1,
  restSpeedThreshold: 0.01,
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>velocity</code> — <em>number | { x, y }</em></summary>

Initial velocity of the spring. Matches the velocity of a gesture for seamless hand-off.

| | |
|---|---|
| **Type** | `number \| { x: number, y: number }` |
| **Default** | `0` |

```jsx
// After a gesture ends, hand off velocity to spring
Animated.spring(position, {
  toValue: { x: 0, y: 0 },
  velocity: gestureState.vx,
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>bounciness</code> and <code>speed</code> — <em>number</em></summary>

A simpler spring model (alternative to `damping` / `stiffness` / `mass`). `bounciness` controls oscillation amount; `speed` controls how fast it settles. **Cannot be combined with `damping`, `stiffness`, or `mass`.**

| | |
|---|---|
| **bounciness default** | `8` |
| **speed default** | `12` |

```jsx
Animated.spring(scale, {
  toValue: 1.2,
  bounciness: 15,
  speed: 8,
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>tension</code> and <code>friction</code> — <em>number</em></summary>

Another legacy spring model (from the Origami design tool). **Cannot be combined with `damping`, `stiffness`, `mass`, `bounciness`, or `speed`.**

| | |
|---|---|
| **tension default** | `40` |
| **friction default** | `7` |

```jsx
Animated.spring(scale, {
  toValue: 1,
  tension: 60,
  friction: 5,
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>delay</code> and <code>isInteraction</code> — <em>number / boolean</em></summary>

Same as in `timing`. `delay` defers start; `isInteraction` registers with `InteractionManager`.

```jsx
Animated.spring(scale, {
  toValue: 1,
  delay: 200,
  isInteraction: false,
  useNativeDriver: true,
}).start();
```

</details>

---

## Decay Config

<details>
<summary><code>velocity</code> — <em>number | { x, y }</em> ⚠️ Required</summary>

Initial velocity. Should come from a gesture's `vx` / `vy` for realistic fling behavior.

| | |
|---|---|
| **Type** | `number \| { x: number, y: number }` |

```jsx
Animated.decay(position, {
  velocity: { x: gestureState.vx, y: gestureState.vy },
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>deceleration</code> — <em>number</em></summary>

Rate at which velocity decreases per frame. Values closer to `1` = slower deceleration (longer glide).

| | |
|---|---|
| **Type** | `number` |
| **Default** | `0.997` |

```jsx
Animated.decay(pan, {
  velocity: { x: 2, y: 0 },
  deceleration: 0.990,
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>useNativeDriver</code> and <code>isInteraction</code></summary>

Same semantics as in `timing`.

```jsx
Animated.decay(position, {
  velocity: { x: 5, y: 3 },
  deceleration: 0.997,
  useNativeDriver: true,
  isInteraction: false,
}).start();
```

</details>

---

## Animation Control

<details>
<summary><code>.start(callback?)</code></summary>

Starts the animation. The optional callback receives `{ finished: boolean }` — `true` if the animation ran to completion, `false` if it was stopped early.

```jsx
Animated.timing(opacity, { toValue: 1, duration: 300, useNativeDriver: true })
  .start(({ finished }) => {
    if (finished) console.log('Animation completed');
    else console.log('Animation was interrupted');
  });
```

</details>

<details>
<summary><code>.stop()</code></summary>

Stops a running animation mid-way. The value stays at its current position.

```jsx
const anim = Animated.timing(translateX, { toValue: 300, duration: 1000, useNativeDriver: true });
anim.start();

// Stop after 200ms
setTimeout(() => anim.stop(), 200);
```

</details>

<details>
<summary><code>.reset()</code></summary>

Stops the animation and resets the value back to its initial state (as if `.start()` was never called).

```jsx
const anim = Animated.timing(opacity, { toValue: 1, duration: 500, useNativeDriver: true });
anim.start();
anim.reset(); // stops and resets to 0
```

</details>

---

## Composing Animations

<details>
<summary><code>Animated.sequence(animations)</code></summary>

Runs animations one after another. Each animation starts only after the previous one finishes. If any animation is stopped, subsequent ones do not run.

```jsx
Animated.sequence([
  Animated.timing(opacity, { toValue: 1, duration: 300, useNativeDriver: true }),
  Animated.timing(translateY, { toValue: 0, duration: 400, useNativeDriver: true }),
  Animated.spring(scale, { toValue: 1, useNativeDriver: true }),
]).start();
```

</details>

<details>
<summary><code>Animated.parallel(animations, config?)</code></summary>

Runs multiple animations simultaneously. By default, if one animation stops, all others stop too. Set `stopTogether: false` to allow independent stopping.

| Config | Default |
|---|---|
| `stopTogether` | `true` |

```jsx
Animated.parallel([
  Animated.timing(opacity, { toValue: 1, duration: 400, useNativeDriver: true }),
  Animated.timing(translateY, { toValue: 0, duration: 400, useNativeDriver: true }),
  Animated.spring(scale, { toValue: 1, useNativeDriver: true }),
], { stopTogether: false }).start();
```

</details>

<details>
<summary><code>Animated.stagger(delay, animations)</code></summary>

Starts animations with a fixed time offset between each. The next animation starts `delay` ms after the previous one starts (not after it ends).

```jsx
const items = [opacity1, opacity2, opacity3, opacity4];

Animated.stagger(100,
  items.map(val =>
    Animated.timing(val, { toValue: 1, duration: 400, useNativeDriver: true })
  )
).start();
```

</details>

<details>
<summary><code>Animated.delay(time)</code></summary>

Creates a delay animation — an animation that does nothing for `time` ms. Useful inside `sequence` to add pauses.

```jsx
Animated.sequence([
  Animated.timing(opacity, { toValue: 1, duration: 300, useNativeDriver: true }),
  Animated.delay(500),   // wait 500ms
  Animated.timing(opacity, { toValue: 0, duration: 300, useNativeDriver: true }),
]).start();
```

</details>

<details>
<summary><code>Animated.loop(animation, config?)</code></summary>

Loops an animation indefinitely (or a set number of times). Resets and repeats.

| Config | Default |
|---|---|
| `iterations` | `-1` (infinite) |
| `resetBeforeIteration` | `false` |

```jsx
const spin = useRef(new Animated.Value(0)).current;

Animated.loop(
  Animated.timing(spin, {
    toValue: 1,
    duration: 1000,
    easing: Easing.linear,
    useNativeDriver: true,
  }),
  { iterations: -1 }  // infinite
).start();

const rotate = spin.interpolate({
  inputRange: [0, 1],
  outputRange: ['0deg', '360deg'],
});
```

</details>

---

## Combining Values

<details>
<summary><code>Animated.add(a, b)</code></summary>

Creates a new value that is the sum of `a` and `b`. Either or both can be `Animated.Value`.

```jsx
const combinedY = Animated.add(scrollY, headerHeight);
<Animated.View style={{ transform: [{ translateY: combinedY }] }} />
```

</details>

<details>
<summary><code>Animated.subtract(a, b)</code></summary>

Creates a value equal to `a - b`.

```jsx
const remaining = Animated.subtract(totalHeight, scrollY);
```

</details>

<details>
<summary><code>Animated.multiply(a, b)</code></summary>

Creates a value equal to `a * b`. Useful for inverting or scaling another animated value.

```jsx
// Invert a scroll value
const invertedScroll = Animated.multiply(scrollY, -1);

// Scale another animation
const scaledOpacity = Animated.multiply(progress, 0.5);
```

</details>

<details>
<summary><code>Animated.divide(a, b)</code></summary>

Creates a value equal to `a / b`. Useful for normalizing a value (e.g. dividing by screen width).

```jsx
const normalized = Animated.divide(scrollX, SCREEN_WIDTH);
// 0 to 1 as user scrolls across one screen width
```

</details>

<details>
<summary><code>Animated.modulo(a, modulus)</code></summary>

Creates a value that is `a % modulus` — keeps the value within `[0, modulus)`. Useful for repeating animations.

```jsx
const loopedValue = Animated.modulo(scrollX, ITEM_WIDTH);
```

</details>

<details>
<summary><code>Animated.diffClamp(value, min, max)</code></summary>

Creates a value clamped between `min` and `max` based on the **difference** of the input value — not its absolute position. This makes it ideal for hiding/showing a header as the user scrolls: the header hides as they scroll down, reveals as they scroll up, regardless of total scroll position.

```jsx
const clampedScroll = Animated.diffClamp(scrollY, 0, HEADER_HEIGHT);

const headerTranslate = clampedScroll.interpolate({
  inputRange: [0, HEADER_HEIGHT],
  outputRange: [0, -HEADER_HEIGHT],
});
```

</details>

---

## Interpolation

<details>
<summary><code>.interpolate(config)</code> — map one range to another</summary>

Maps input values to output values using a linear (or custom) mapping. Can output numbers, strings (colors, degrees, px), or any CSS-like values.

| Config field | Description |
|---|---|
| `inputRange` | Array of input numbers (must be ascending) |
| `outputRange` | Array of output values (same length as inputRange) |
| `easing` | Optional easing function per segment |
| `extrapolate` | Behavior beyond the input range: `'extend'`, `'clamp'`, `'identity'` |
| `extrapolateLeft` | Overrides `extrapolate` for values below the input range |
| `extrapolateRight` | Overrides `extrapolate` for values above the input range |

```jsx
const opacity = useRef(new Animated.Value(0)).current;

// Map 0→1 to 0%→100% opacity
const animatedOpacity = opacity.interpolate({
  inputRange: [0, 1],
  outputRange: [0, 1],
});

// Map scroll position to header opacity
const headerOpacity = scrollY.interpolate({
  inputRange: [0, 100],
  outputRange: [1, 0],
  extrapolate: 'clamp',   // don't go below 0
});

// Map 0→1 to a rotation angle
const spin = progress.interpolate({
  inputRange: [0, 1],
  outputRange: ['0deg', '360deg'],
});

// Multi-segment interpolation
const color = progress.interpolate({
  inputRange: [0, 0.5, 1],
  outputRange: ['rgb(255,0,0)', 'rgb(255,165,0)', 'rgb(0,128,0)'],
});
```

</details>

<details>
<summary><code>extrapolate: 'clamp'</code> — stop at range boundaries</summary>

The output value is clamped to the first or last output value when the input goes beyond the range. Most common choice.

```jsx
const opacity = scrollY.interpolate({
  inputRange: [0, 200],
  outputRange: [1, 0],
  extrapolate: 'clamp',  // stays 0 even when scrollY > 200
});
```

</details>

<details>
<summary><code>extrapolate: 'extend'</code> — continue the slope beyond range</summary>

The output continues linearly past the boundaries. Default behavior.

```jsx
const translateX = progress.interpolate({
  inputRange: [0, 1],
  outputRange: [0, 100],
  extrapolate: 'extend',  // continues to 200 when progress is 2
});
```

</details>

<details>
<summary><code>extrapolate: 'identity'</code> — return input value beyond range</summary>

Past the boundaries, the interpolated value equals the input value directly.

```jsx
const value = animated.interpolate({
  inputRange: [0, 10],
  outputRange: [0, 100],
  extrapolate: 'identity',
});
```

</details>

---

## Easing

<details>
<summary><code>Easing.linear</code> — constant speed</summary>

No acceleration or deceleration — constant speed throughout.

```jsx
import { Easing } from 'react-native';

Animated.timing(opacity, {
  toValue: 1, duration: 300,
  easing: Easing.linear,
  useNativeDriver: true,
}).start();
```

</details>

<details>
<summary><code>Easing.ease</code> — default ease-in-out</summary>

Starts slow, speeds up, then slows down. The classic ease curve. Used as the default for `Easing.inOut(Easing.ease)`.

```jsx
easing: Easing.ease
```

</details>

<details>
<summary><code>Easing.quad</code>, <code>Easing.cubic</code> — polynomial curves</summary>

Power-based curves. `quad` = x², `cubic` = x³. Combine with `Easing.in` / `Easing.out` / `Easing.inOut`.

```jsx
easing: Easing.inOut(Easing.quad)
easing: Easing.out(Easing.cubic)
```

</details>

<details>
<summary><code>Easing.sin</code>, <code>Easing.circle</code>, <code>Easing.exp</code></summary>

Trigonometric and exponential curves for more dramatic effects.

```jsx
easing: Easing.out(Easing.sin)
easing: Easing.in(Easing.exp)
easing: Easing.inOut(Easing.circle)
```

</details>

<details>
<summary><code>Easing.elastic(bounciness)</code></summary>

Spring-like oscillation at the end of the animation. `bounciness` defaults to `1`.

```jsx
easing: Easing.elastic(1.5)  // more bounce
```

</details>

<details>
<summary><code>Easing.back(s)</code></summary>

Briefly overshoots in the opposite direction before moving to the target. `s` controls overshoot amount (default `1.70158`).

```jsx
easing: Easing.out(Easing.back(2))
```

</details>

<details>
<summary><code>Easing.bounce</code></summary>

Simulates a bouncing ball hitting the floor.

```jsx
easing: Easing.bounce
```

</details>

<details>
<summary><code>Easing.bezier(x1, y1, x2, y2)</code></summary>

Defines a cubic Bézier curve — same as CSS `cubic-bezier()`. Use [cubic-bezier.com](https://cubic-bezier.com) to design curves visually.

```jsx
easing: Easing.bezier(0.25, 0.1, 0.25, 1)   // ease
easing: Easing.bezier(0.42, 0, 1, 1)         // ease-in
easing: Easing.bezier(0, 0, 0.58, 1)         // ease-out
easing: Easing.bezier(0.42, 0, 0.58, 1)      // ease-in-out
```

</details>

<details>
<summary><code>Easing.in(easing)</code>, <code>Easing.out(easing)</code>, <code>Easing.inOut(easing)</code></summary>

Higher-order functions that wrap any easing to apply it at the start, end, or both.

```jsx
Easing.in(Easing.quad)        // accelerates in
Easing.out(Easing.quad)       // decelerates out
Easing.inOut(Easing.quad)     // slow start, fast middle, slow end
Easing.in(Easing.elastic(2))  // elastic on entry
```

</details>

---

## Animated Components

<details>
<summary>Built-in Animated components</summary>

React Native ships these out of the box. They accept `Animated.Value` in their style props.

```jsx
import { Animated } from 'react-native';

<Animated.View style={{ opacity: fadeAnim }} />
<Animated.Text style={{ transform: [{ scale: scaleAnim }] }} />
<Animated.Image source={source} style={{ opacity: fadeAnim }} />
<Animated.ScrollView onScroll={Animated.event([...])} />
<Animated.FlatList data={data} renderItem={renderItem} />
<Animated.SectionList sections={sections} renderItem={renderItem} />
```

</details>

<details>
<summary><code>Animated.createAnimatedComponent(Component)</code></summary>

Wraps any custom component to make it animatable. The component must forward its ref and pass `style` through to a native element.

```jsx
const AnimatedPressable = Animated.createAnimatedComponent(Pressable);
const AnimatedLinearGradient = Animated.createAnimatedComponent(LinearGradient);

<AnimatedPressable style={{ opacity: fadeAnim }} onPress={handlePress} />
```

</details>

<details>
<summary><code>Animated.event(argMapping, config?)</code></summary>

Creates an event handler that directly maps native event values to `Animated.Value`s without going through JS. Use with `onScroll`, `onPanResponderMove`, etc.

| Config | Default |
|---|---|
| `useNativeDriver` | `false` |
| `listener` | `undefined` |

```jsx
const scrollY = useRef(new Animated.Value(0)).current;

<Animated.ScrollView
  onScroll={Animated.event(
    [{ nativeEvent: { contentOffset: { y: scrollY } } }],
    {
      useNativeDriver: true,
      listener: (event) => {
        // Optional JS listener alongside native mapping
        console.log(event.nativeEvent.contentOffset.y);
      },
    }
  )}
  scrollEventThrottle={16}
/>
```

</details>

---

## Hooks

<details>
<summary><code>useAnimatedValue(initialValue)</code></summary>

A convenience hook that creates an `Animated.Value` and stores it in a ref. Equivalent to `useRef(new Animated.Value(n)).current` but cleaner.

| | |
|---|---|
| **Available from** | React Native 0.71+ |

```jsx
import { useAnimatedValue } from 'react-native';

// Before (verbose)
const opacity = useRef(new Animated.Value(0)).current;

// After (cleaner)
const opacity = useAnimatedValue(0);
const translateY = useAnimatedValue(-50);
```

</details>

---

## Common Patterns

<details>
<summary>Fade in on mount</summary>

```jsx
function FadeIn({ children, duration = 400 }) {
  const opacity = useAnimatedValue(0);

  useEffect(() => {
    Animated.timing(opacity, {
      toValue: 1,
      duration,
      easing: Easing.out(Easing.ease),
      useNativeDriver: true,
    }).start();
  }, []);

  return <Animated.View style={{ opacity }}>{children}</Animated.View>;
}
```

</details>

<details>
<summary>Slide in from bottom</summary>

```jsx
function SlideUp({ children }) {
  const translateY = useAnimatedValue(60);
  const opacity = useAnimatedValue(0);

  useEffect(() => {
    Animated.parallel([
      Animated.timing(opacity, { toValue: 1, duration: 350, useNativeDriver: true }),
      Animated.spring(translateY, {
        toValue: 0,
        damping: 18,
        stiffness: 180,
        useNativeDriver: true,
      }),
    ]).start();
  }, []);

  return (
    <Animated.View style={{ opacity, transform: [{ translateY }] }}>
      {children}
    </Animated.View>
  );
}
```

</details>

<details>
<summary>Press scale (button feedback)</summary>

```jsx
function ScalePressable({ onPress, children }) {
  const scale = useAnimatedValue(1);

  const onPressIn = () =>
    Animated.spring(scale, {
      toValue: 0.93,
      useNativeDriver: true,
      damping: 15,
      stiffness: 300,
    }).start();

  const onPressOut = () =>
    Animated.spring(scale, {
      toValue: 1,
      useNativeDriver: true,
      damping: 15,
      stiffness: 300,
    }).start();

  return (
    <Pressable onPress={onPress} onPressIn={onPressIn} onPressOut={onPressOut}>
      <Animated.View style={{ transform: [{ scale }] }}>
        {children}
      </Animated.View>
    </Pressable>
  );
}
```

</details>

<details>
<summary>Infinite spinner / rotation loop</summary>

```jsx
function Spinner({ size = 32, color = '#6366f1' }) {
  const rotation = useAnimatedValue(0);

  useEffect(() => {
    Animated.loop(
      Animated.timing(rotation, {
        toValue: 1,
        duration: 900,
        easing: Easing.linear,
        useNativeDriver: true,
      })
    ).start();
  }, []);

  const spin = rotation.interpolate({
    inputRange: [0, 1],
    outputRange: ['0deg', '360deg'],
  });

  return (
    <Animated.View
      style={{
        width: size, height: size,
        borderRadius: size / 2,
        borderWidth: 3,
        borderColor: color,
        borderTopColor: 'transparent',
        transform: [{ rotate: spin }],
      }}
    />
  );
}
```

</details>

<details>
<summary>Stagger entrance animation for a list</summary>

```jsx
function StaggerList({ items }) {
  const anims = useRef(items.map(() => new Animated.Value(0))).current;

  useEffect(() => {
    Animated.stagger(
      60,
      anims.map(anim =>
        Animated.spring(anim, {
          toValue: 1,
          damping: 16,
          stiffness: 150,
          useNativeDriver: true,
        })
      )
    ).start();
  }, []);

  return (
    <View>
      {items.map((item, i) => (
        <Animated.View
          key={item.id}
          style={{
            opacity: anims[i],
            transform: [{
              translateY: anims[i].interpolate({
                inputRange: [0, 1],
                outputRange: [20, 0],
              }),
            }],
          }}
        >
          <ItemCard item={item} />
        </Animated.View>
      ))}
    </View>
  );
}
```

</details>

<details>
<summary>Collapsible / accordion height animation</summary>

```jsx
function Accordion({ isOpen, children, collapsedHeight = 0, expandedHeight = 200 }) {
  const height = useAnimatedValue(isOpen ? expandedHeight : collapsedHeight);

  useEffect(() => {
    Animated.timing(height, {
      toValue: isOpen ? expandedHeight : collapsedHeight,
      duration: 300,
      easing: Easing.inOut(Easing.ease),
      useNativeDriver: false, // height can't use native driver
    }).start();
  }, [isOpen]);

  return (
    <Animated.View style={{ height, overflow: 'hidden' }}>
      {children}
    </Animated.View>
  );
}
```

</details>

<details>
<summary>Animated progress bar</summary>

```jsx
function ProgressBar({ progress }) { // progress: 0 to 1
  const width = useAnimatedValue(0);

  useEffect(() => {
    Animated.timing(width, {
      toValue: progress,
      duration: 400,
      easing: Easing.out(Easing.ease),
      useNativeDriver: false, // width can't use native driver
    }).start();
  }, [progress]);

  const barWidth = width.interpolate({
    inputRange: [0, 1],
    outputRange: ['0%', '100%'],
    extrapolate: 'clamp',
  });

  return (
    <View style={{ height: 6, backgroundColor: '#e5e7eb', borderRadius: 3 }}>
      <Animated.View
        style={{
          height: '100%',
          width: barWidth,
          backgroundColor: '#6366f1',
          borderRadius: 3,
        }}
      />
    </View>
  );
}
```

</details>

<details>
<summary>Draggable element with PanResponder</summary>

```jsx
function Draggable({ children }) {
  const position = useRef(new Animated.ValueXY({ x: 0, y: 0 })).current;

  const panResponder = useRef(
    PanResponder.create({
      onStartShouldSetPanResponder: () => true,
      onPanResponderGrant: () => {
        position.extractOffset(); // lock in current position
      },
      onPanResponderMove: Animated.event(
        [null, { dx: position.x, dy: position.y }],
        { useNativeDriver: false }
      ),
      onPanResponderRelease: () => {
        position.flattenOffset(); // merge offset into value
        // Optionally spring back to origin:
        // Animated.spring(position, { toValue: { x: 0, y: 0 }, useNativeDriver: false }).start();
      },
    })
  ).current;

  return (
    <Animated.View
      {...panResponder.panHandlers}
      style={{ transform: position.getTranslateTransform() }}
    >
      {children}
    </Animated.View>
  );
}
```

</details>

<details>
<summary>Hide header on scroll down, show on scroll up (diffClamp)</summary>

```jsx
const scrollY = useRef(new Animated.Value(0)).current;
const HEADER_HEIGHT = 56;

const clampedScroll = Animated.diffClamp(scrollY, 0, HEADER_HEIGHT);

const headerTranslateY = clampedScroll.interpolate({
  inputRange: [0, HEADER_HEIGHT],
  outputRange: [0, -HEADER_HEIGHT],
  extrapolate: 'clamp',
});

const headerOpacity = clampedScroll.interpolate({
  inputRange: [0, HEADER_HEIGHT],
  outputRange: [1, 0],
  extrapolate: 'clamp',
});

// Header
<Animated.View style={{
  transform: [{ translateY: headerTranslateY }],
  opacity: headerOpacity,
}}>
  <AppHeader />
</Animated.View>

// List
<Animated.ScrollView
  onScroll={Animated.event(
    [{ nativeEvent: { contentOffset: { y: scrollY } } }],
    { useNativeDriver: true }
  )}
  scrollEventThrottle={16}
/>
```

</details>

<details>
<summary>Shimmer / skeleton loading effect</summary>

```jsx
function Shimmer({ width = '100%', height = 20, borderRadius = 4 }) {
  const shimmer = useAnimatedValue(0);

  useEffect(() => {
    Animated.loop(
      Animated.sequence([
        Animated.timing(shimmer, {
          toValue: 1,
          duration: 1000,
          easing: Easing.inOut(Easing.ease),
          useNativeDriver: false,
        }),
        Animated.timing(shimmer, {
          toValue: 0,
          duration: 1000,
          easing: Easing.inOut(Easing.ease),
          useNativeDriver: false,
        }),
      ])
    ).start();
  }, []);

  const backgroundColor = shimmer.interpolate({
    inputRange: [0, 1],
    outputRange: ['#e5e7eb', '#f3f4f6'],
  });

  return (
    <Animated.View style={{ width, height, borderRadius, backgroundColor }} />
  );
}
```

</details>

---

## Quick-Reference Cheatsheet

| API | Use case |
|---|---|
| `Animated.Value(n)` | Single numeric animated value |
| `Animated.ValueXY({ x, y })` | 2D position tracking |
| `Animated.timing` | Duration-based smooth animation |
| `Animated.spring` | Physics-based bouncy animation |
| `Animated.decay` | Momentum / fling deceleration |
| `Animated.sequence` | Run animations one after another |
| `Animated.parallel` | Run animations simultaneously |
| `Animated.stagger` | Offset-delayed start for a group |
| `Animated.loop` | Repeat an animation forever |
| `Animated.delay` | Pause inside a sequence |
| `.interpolate()` | Map value to color, degrees, % etc |
| `Animated.diffClamp` | Scroll-aware header show/hide |
| `Animated.add/multiply` | Derive new values from existing ones |
| `Animated.event` | Map native events directly to values |
| `useNativeDriver: true` | ✅ Always use for `transform` & `opacity` |
| `useNativeDriver: false` | Required for layout props, colors |
| `Easing.bezier` | CSS cubic-bezier-style curves |
| `Easing.elastic` | Overshoot bouncy ending |
| `Easing.back` | Brief recoil before moving |

---

*Reference based on React Native stable API. Always check the [official docs](https://reactnative.dev/docs/animated) for the latest updates.*