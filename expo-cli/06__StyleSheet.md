# React Native StyleSheet — Complete Reference

A comprehensive reference for React Native's `StyleSheet` API.  
Covers all style properties organized by category, StyleSheet methods, units, inheritance rules, and common patterns.

---

## Table of Contents

1. [StyleSheet API Methods](#stylesheet-api-methods)
2. [Layout — Flexbox](#layout--flexbox)
3. [Layout — Dimensions & Position](#layout--dimensions--position)
4. [Layout — Spacing](#layout--spacing)
5. [Typography](#typography)
6. [Colors & Background](#colors--background)
7. [Borders](#borders)
8. [Shadow & Elevation](#shadow--elevation)
9. [Transforms](#transforms)
10. [Opacity & Visibility](#opacity--visibility)
11. [Image Styles](#image-styles)
12. [Overflow & Clipping](#overflow--clipping)
13. [View-Specific Styles](#view-specific-styles)
14. [Units & Values](#units--values)
15. [Platform-Specific Styles](#platform-specific-styles)
16. [Inheritance Rules](#inheritance-rules)
17. [Common Patterns](#common-patterns)

---

## StyleSheet API Methods

<details> <summary><code>StyleSheet.create(styles)</code> — define styles with validation</summary>

Creates a stylesheet object. In development, validates all style properties. In production, optimizes styles for performance by sending them to the native layer once.

|||
|---|---|
|**Type**|`(styles: NamedStyles<T>) => T`|
|**Returns**|Frozen style object with numeric IDs|

```tsx
import { StyleSheet, View, Text } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: '700',
    color: '#111',
  },
  button: {
    backgroundColor: '#6366f1',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 8,
    alignItems: 'center',
  },
});

// Usage
<View style={styles.container}>
  <Text style={styles.title}>Hello</Text>
</View>
```

> ✅ Always prefer `StyleSheet.create` over inline objects — it validates props, gives better DevTools support, and prevents recreating style objects on every render.

</details> <details> <summary><code>StyleSheet.flatten(style)</code> — merge style array into one object</summary>

Flattens an array of style objects or style IDs into a single plain object. Useful for reading computed style values.

|||
|---|---|
|**Type**|`(style: StyleProp<T>) => T`|

```tsx
const base  = { color: '#111', fontSize: 16 };
const bold  = { fontWeight: '700' };
const large = { fontSize: 24 };

// Flatten array into one object
const merged = StyleSheet.flatten([base, bold, large]);
// { color: '#111', fontSize: 24, fontWeight: '700' }

// Read a style value from a style ID
const flat = StyleSheet.flatten(styles.button);
console.log(flat.backgroundColor); // '#6366f1'

// Flatten conditional styles
const resolved = StyleSheet.flatten([
  styles.base,
  isActive && styles.active,
  isDisabled && styles.disabled,
]);
```

</details> <details> <summary><code>StyleSheet.compose(style1, style2)</code> — combine two styles</summary>

Combines two styles into one. Returns `null` if both are null/undefined. More performant than array syntax for exactly two styles.

|||
|---|---|
|**Type**|`(style1, style2) => StyleProp<T>`|

```tsx
const composed = StyleSheet.compose(styles.base, styles.active);
<View style={composed} />

// Equivalent to array syntax:
<View style={[styles.base, styles.active]} />
```

</details> <details> <summary><code>StyleSheet.hairlineWidth</code> — thinnest visible line</summary>

The width of a single physical pixel on the device screen. Typically `0.333...` or `0.5` depending on screen density. Use for thin dividers and borders.

|||
|---|---|
|**Type**|`number`|

```tsx
const styles = StyleSheet.create({
  divider: {
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#e5e7eb',
  },
  thinBorder: {
    borderWidth: StyleSheet.hairlineWidth,
    borderColor: '#d1d5db',
  },
});
```

</details> <details> <summary><code>StyleSheet.absoluteFill</code> and <code>StyleSheet.absoluteFillObject</code></summary>

Predefined styles for absolute positioning that fills the parent container entirely.

|||
|---|---|
|`absoluteFill`|Style ID (number) — use directly in `style` prop|
|`absoluteFillObject`|Plain object — use to spread or extend|

```tsx
// absoluteFill — most common
<View style={StyleSheet.absoluteFill}>
  <BlurView />
</View>

// absoluteFillObject — when you need to extend
<View style={[StyleSheet.absoluteFillObject, { backgroundColor: 'rgba(0,0,0,0.5)' }]}>
  <Overlay />
</View>

// What it equals:
// { position: 'absolute', top: 0, right: 0, bottom: 0, left: 0 }
```

</details>

---

## Layout — Flexbox

<details> <summary><code>flex</code> — grow/shrink to fill available space</summary>

The shorthand for how a component grows or shrinks in the main axis. In React Native, `flex` is a simplified version of CSS flex (only supports a single number).

|||
|---|---|
|**Type**|`number`|
|**Default**|`undefined`|

```tsx
// flex: 1 — take all available space
<View style={{ flex: 1 }}>...</View>

// flex: 2 — take twice as much space as flex: 1 siblings
<View style={{ flex: 2 }}>...</View>

// Common pattern — split screen
<View style={{ flex: 1, flexDirection: 'row' }}>
  <View style={{ flex: 1 }}>{/* left column */}</View>
  <View style={{ flex: 2 }}>{/* right column — twice as wide */}</View>
</View>
```

</details> <details> <summary><code>flexDirection</code> — main axis direction</summary>

Sets the direction children are laid out.

|||
|---|---|
|**Type**|`'column' \| 'row' \| 'column-reverse' \| 'row-reverse'`|
|**Default**|`'column'` ← React Native differs from web (CSS default is `'row'`)|

```tsx
// Column (default) — children stack top to bottom
<View style={{ flexDirection: 'column' }}>
  <Text>First</Text>
  <Text>Second</Text>
</View>

// Row — children stack left to right
<View style={{ flexDirection: 'row' }}>
  <Icon />
  <Text>Label</Text>
</View>

// Reverse
<View style={{ flexDirection: 'row-reverse' }} />
<View style={{ flexDirection: 'column-reverse' }} />
```

</details> <details> <summary><code>justifyContent</code> — main axis alignment</summary>

Aligns children along the **main axis** (defined by `flexDirection`).

|||
|---|---|
|**Type**|`'flex-start' \| 'flex-end' \| 'center' \| 'space-between' \| 'space-around' \| 'space-evenly'`|
|**Default**|`'flex-start'`|

```tsx
// Center vertically in a column
<View style={{ flex: 1, justifyContent: 'center' }}>
  <Text>Centered</Text>
</View>

// Space items evenly in a row
<View style={{ flexDirection: 'row', justifyContent: 'space-between' }}>
  <Icon name="home" />
  <Icon name="search" />
  <Icon name="profile" />
</View>

// Push content to the bottom
<View style={{ flex: 1, justifyContent: 'flex-end' }}>
  <Button />
</View>
```

</details> <details> <summary><code>alignItems</code> — cross axis alignment</summary>

Aligns children along the **cross axis** (perpendicular to `flexDirection`).

|||
|---|---|
|**Type**|`'flex-start' \| 'flex-end' \| 'center' \| 'stretch' \| 'baseline'`|
|**Default**|`'stretch'`|

```tsx
// Center children horizontally in a column
<View style={{ flex: 1, alignItems: 'center' }}>
  <Text>Centered horizontally</Text>
</View>

// Center both horizontally and vertically
<View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
  <Text>Dead center</Text>
</View>

// Align to start (left in column)
<View style={{ alignItems: 'flex-start' }}>
  <Text>Left aligned</Text>
</View>
```

</details> <details> <summary><code>alignSelf</code> — override parent's <code>alignItems</code> for one child</summary>

|||
|---|---|
|**Type**|`'auto' \| 'flex-start' \| 'flex-end' \| 'center' \| 'stretch' \| 'baseline'`|
|**Default**|`'auto'` (inherits from parent's `alignItems`)|

```tsx
<View style={{ alignItems: 'center' }}>
  <Text>Centered by parent</Text>
  <Text style={{ alignSelf: 'flex-end' }}>I'm right-aligned</Text>
  <Text style={{ alignSelf: 'stretch' }}>I'm full width</Text>
</View>
```

</details> <details> <summary><code>alignContent</code> — multi-line cross axis alignment</summary>

Controls alignment of lines in a wrapping flex container (`flexWrap: 'wrap'`).

|||
|---|---|
|**Type**|`'flex-start' \| 'flex-end' \| 'center' \| 'stretch' \| 'space-between' \| 'space-around'`|
|**Default**|`'flex-start'`|

```tsx
<View style={{ flexWrap: 'wrap', alignContent: 'space-between', height: 200 }}>
  {items.map(item => <Chip key={item.id} label={item.name} />)}
</View>
```

</details> <details> <summary><code>flexWrap</code> — wrap children to next line</summary>

|||
|---|---|
|**Type**|`'nowrap' \| 'wrap' \| 'wrap-reverse'`|
|**Default**|`'nowrap'`|

```tsx
// Tag cloud / chip list that wraps
<View style={{ flexDirection: 'row', flexWrap: 'wrap', gap: 8 }}>
  {tags.map(tag => <Chip key={tag} label={tag} />)}
</View>
```

</details> <details> <summary><code>flexGrow</code>, <code>flexShrink</code>, <code>flexBasis</code></summary>

More granular flex control. These are the longhand properties that `flex` combines.

|Property|Default|Description|
|---|---|---|
|`flexGrow`|`0`|How much a child grows to fill extra space|
|`flexShrink`|`1`|How much a child shrinks when space is tight|
|`flexBasis`|`'auto'`|Initial size before growing/shrinking|

```tsx
// Equivalent: flex: 1 = flexGrow: 1, flexShrink: 1, flexBasis: 0
<View style={{ flexGrow: 1, flexShrink: 1, flexBasis: 0 }} />

// Input takes all remaining space, button stays fixed
<View style={{ flexDirection: 'row' }}>
  <TextInput style={{ flexGrow: 1, flexShrink: 1 }} />
  <Button title="Send" />
</View>
```

</details> <details> <summary><code>gap</code>, <code>rowGap</code>, <code>columnGap</code> — spacing between children</summary>

Adds spacing between flex children without using margins. React Native 0.71+.

|||
|---|---|
|**Type**|`number`|
|**Default**|`0`|

```tsx
// Same gap on all axes
<View style={{ flexDirection: 'row', flexWrap: 'wrap', gap: 12 }}>
  {items.map(item => <Card key={item.id} />)}
</View>

// Different row and column gaps
<View style={{ flexDirection: 'row', flexWrap: 'wrap', rowGap: 16, columnGap: 8 }}>
  {items.map(item => <Card key={item.id} />)}
</View>
```

</details>

---

## Layout — Dimensions & Position

<details> <summary><code>width</code> and <code>height</code></summary>

Sets the dimensions of a component. Accepts numbers (dp), percentages, or `'auto'`.

|||
|---|---|
|**Type**|`number \| string \| 'auto'`|

```tsx
// Fixed size (density-independent pixels)
<View style={{ width: 200, height: 100 }} />

// Percentage of parent
<View style={{ width: '100%', height: '50%' }} />

// Auto (size to content)
<View style={{ width: 'auto' }} />

// Aspect ratio (set one dimension)
<View style={{ width: '100%', aspectRatio: 16 / 9 }} />

// Square
<View style={{ width: 80, height: 80, borderRadius: 40 }} />
```

</details> <details> <summary><code>minWidth</code>, <code>maxWidth</code>, <code>minHeight</code>, <code>maxHeight</code></summary>

Constrains dimensions to a range.

```tsx
// Button with minimum width
<View style={{ minWidth: 100, paddingHorizontal: 20 }}>
  <Text>OK</Text>
</View>

// Content area with max width (web-like centered layout)
<View style={{ width: '100%', maxWidth: 600, alignSelf: 'center' }}>
  <Content />
</View>

// Expandable text area with height limits
<TextInput
  multiline
  style={{ minHeight: 40, maxHeight: 200 }}
/>
```

</details> <details> <summary><code>position</code> — layout positioning mode</summary>

|||
|---|---|
|**Type**|`'relative' \| 'absolute' \| 'static'`|
|**Default**|`'relative'`|

```tsx
// Relative (default) — participates in flex layout
<View style={{ position: 'relative' }} />

// Absolute — positioned relative to nearest positioned ancestor
<View style={{ position: 'absolute', top: 0, right: 0, bottom: 0, left: 0 }} />

// Static — similar to relative, ignores top/right/bottom/left
<View style={{ position: 'static' }} />
```

</details> <details> <summary><code>top</code>, <code>right</code>, <code>bottom</code>, <code>left</code> — offset from edges</summary>

Offsets the component from each edge. For `position: 'absolute'`, positions relative to parent. For `position: 'relative'`, offsets from normal position.

```tsx
// Absolute positioned badge
<View style={{ position: 'relative' }}>
  <Icon name="bell" />
  <View style={{
    position: 'absolute',
    top: -4,
    right: -4,
    width: 16,
    height: 16,
    borderRadius: 8,
    backgroundColor: '#ef4444',
  }} />
</View>

// Sticky footer
<View style={{ position: 'absolute', bottom: 0, left: 0, right: 0 }}>
  <Footer />
</View>
```

</details> <details> <summary><code>zIndex</code> — stacking order</summary>

Controls which component renders on top when components overlap.

|||
|---|---|
|**Type**|`number`|
|**Default**|`0`|

```tsx
// Modal overlay above everything
<View style={{ position: 'absolute', zIndex: 1000, ...StyleSheet.absoluteFillObject }}>
  <Modal />
</View>

// Toast notification
<View style={{ position: 'absolute', bottom: 80, zIndex: 999, alignSelf: 'center' }}>
  <Toast message="Saved!" />
</View>
```

</details> <details> <summary><code>aspectRatio</code> — maintain proportional dimensions</summary>

Keeps width/height in a specific ratio. Set one dimension and `aspectRatio` calculates the other.

|||
|---|---|
|**Type**|`number`|

```tsx
// Widescreen video container
<View style={{ width: '100%', aspectRatio: 16 / 9, backgroundColor: '#000' }}>
  <Video />
</View>

// Square image
<Image style={{ width: '100%', aspectRatio: 1 }} source={source} />

// Portrait card
<View style={{ width: 160, aspectRatio: 3 / 4 }}>
  <Card />
</View>
```

</details>

---

## Layout — Spacing

<details> <summary><code>margin</code>, <code>marginHorizontal</code>, <code>marginVertical</code></summary>

Outer spacing between this component and its neighbors.

|Property|Axes|
|---|---|
|`margin`|All four sides|
|`marginTop`|Top only|
|`marginRight`|Right only|
|`marginBottom`|Bottom only|
|`marginLeft`|Left only|
|`marginHorizontal`|Left + Right|
|`marginVertical`|Top + Bottom|
|`marginStart`|Start (RTL-aware left)|
|`marginEnd`|End (RTL-aware right)|

```tsx
// All sides
<View style={{ margin: 16 }} />

// Horizontal and vertical separately
<View style={{ marginHorizontal: 24, marginVertical: 12 }} />

// Individual sides
<View style={{ marginTop: 8, marginBottom: 16 }} />

// Auto margin — center horizontally
<View style={{ width: 200, marginHorizontal: 'auto' }} />

// Negative margin (pull element closer)
<View style={{ marginTop: -8 }} />
```

</details> <details> <summary><code>padding</code>, <code>paddingHorizontal</code>, <code>paddingVertical</code></summary>

Inner spacing between the component's border and its children.

|Property|Axes|
|---|---|
|`padding`|All four sides|
|`paddingTop`|Top only|
|`paddingRight`|Right only|
|`paddingBottom`|Bottom only|
|`paddingLeft`|Left only|
|`paddingHorizontal`|Left + Right|
|`paddingVertical`|Top + Bottom|
|`paddingStart`|Start (RTL-aware left)|
|`paddingEnd`|End (RTL-aware right)|

```tsx
// Card with comfortable padding
<View style={{ padding: 16, backgroundColor: '#fff', borderRadius: 12 }}>
  <Content />
</View>

// Button with horizontal pill padding
<TouchableOpacity style={{ paddingHorizontal: 24, paddingVertical: 12, borderRadius: 24 }}>
  <Text>Click Me</Text>
</TouchableOpacity>

// Safe area padding
<View style={{ paddingTop: insets.top, paddingBottom: insets.bottom }}>
  <Screen />
</View>
```

</details>

---

## Typography

<details> <summary><code>fontSize</code></summary>

Size of the text in density-independent pixels.

|||
|---|---|
|**Type**|`number`|
|**Default**|`14`|

```tsx
// Common scale
<Text style={{ fontSize: 12 }}>Caption</Text>
<Text style={{ fontSize: 14 }}>Body (default)</Text>
<Text style={{ fontSize: 16 }}>Body Large</Text>
<Text style={{ fontSize: 18 }}>Subtitle</Text>
<Text style={{ fontSize: 22 }}>Title</Text>
<Text style={{ fontSize: 28 }}>Heading</Text>
<Text style={{ fontSize: 36 }}>Display</Text>
```

</details> <details> <summary><code>fontWeight</code></summary>

Thickness of the font strokes.

|||
|---|---|
|**Type**|`'normal' \| 'bold' \| '100' \| '200' \| '300' \| '400' \| '500' \| '600' \| '700' \| '800' \| '900'`|
|**Default**|`'normal'` (= `'400'`)|

```tsx
<Text style={{ fontWeight: '400' }}>Regular</Text>
<Text style={{ fontWeight: '500' }}>Medium</Text>
<Text style={{ fontWeight: '600' }}>SemiBold</Text>
<Text style={{ fontWeight: '700' }}>Bold</Text>
<Text style={{ fontWeight: '800' }}>ExtraBold</Text>
<Text style={{ fontWeight: '900' }}>Black</Text>
```

</details> <details> <summary><code>fontFamily</code></summary>

Font to use. Must be loaded first (with `expo-font` or bundled system font).

|||
|---|---|
|**Type**|`string`|
|**Default**|System default|

```tsx
// System fonts (no loading needed)
<Text style={{ fontFamily: 'System' }}>System default</Text>

// iOS system fonts
<Text style={{ fontFamily: '-apple-system' }}>SF Pro</Text>

// Custom loaded font
<Text style={{ fontFamily: 'Inter-Bold' }}>Inter Bold</Text>
<Text style={{ fontFamily: 'SpaceMono-Regular' }}>Space Mono</Text>
```

</details> <details> <summary><code>fontStyle</code></summary>

|||
|---|---|
|**Type**|`'normal' \| 'italic'`|
|**Default**|`'normal'`|

```tsx
<Text style={{ fontStyle: 'italic' }}>Italic text</Text>
```

</details> <details> <summary><code>lineHeight</code></summary>

Height of each line of text. Set to ~1.4–1.6× `fontSize` for comfortable reading.

|||
|---|---|
|**Type**|`number`|

```tsx
// Body text
<Text style={{ fontSize: 16, lineHeight: 24 }}>
  Long body text with comfortable line height
</Text>

// Tight headline
<Text style={{ fontSize: 28, lineHeight: 34, fontWeight: '700' }}>
  Headline
</Text>
```

</details> <details> <summary><code>letterSpacing</code></summary>

Space between characters in pixels.

|||
|---|---|
|**Type**|`number`|
|**Default**|`0`|

```tsx
// Tight
<Text style={{ letterSpacing: -0.5 }}>Tight headline</Text>

// Wide (ALL CAPS labels)
<Text style={{ letterSpacing: 2, textTransform: 'uppercase', fontSize: 11 }}>
  LABEL
</Text>
```

</details> <details> <summary><code>textAlign</code></summary>

|||
|---|---|
|**Type**|`'auto' \| 'left' \| 'right' \| 'center' \| 'justify'`|
|**Default**|`'auto'`|

```tsx
<Text style={{ textAlign: 'center' }}>Centered</Text>
<Text style={{ textAlign: 'right' }}>Right aligned</Text>
<Text style={{ textAlign: 'justify' }}>Justified paragraph text</Text>
```

</details> <details> <summary><code>textAlignVertical</code> · 🤖 Android</summary>

Vertical alignment within the text component on Android.

|||
|---|---|
|**Type**|`'auto' \| 'top' \| 'bottom' \| 'center'`|
|**Platform**|Android|

```tsx
<Text style={{ height: 80, textAlignVertical: 'center' }}>
  Vertically centered (Android)
</Text>
```

</details> <details> <summary><code>textDecorationLine</code></summary>

|||
|---|---|
|**Type**|`'none' \| 'underline' \| 'line-through' \| 'underline line-through'`|
|**Default**|`'none'`|

```tsx
<Text style={{ textDecorationLine: 'underline' }}>Underlined</Text>
<Text style={{ textDecorationLine: 'line-through' }}>Strikethrough</Text>
<Text style={{ textDecorationLine: 'underline line-through' }}>Both</Text>
```

</details> <details> <summary><code>textDecorationStyle</code> and <code>textDecorationColor</code> · 🍎 iOS</summary>

```tsx
<Text style={{
  textDecorationLine: 'underline',
  textDecorationStyle: 'dashed',   // 'solid' | 'double' | 'dotted' | 'dashed'
  textDecorationColor: '#ef4444',
}}>
  Colored dashed underline (iOS)
</Text>
```

</details> <details> <summary><code>textTransform</code></summary>

|||
|---|---|
|**Type**|`'none' \| 'uppercase' \| 'lowercase' \| 'capitalize'`|
|**Default**|`'none'`|

```tsx
<Text style={{ textTransform: 'uppercase' }}>UPPERCASE</Text>
<Text style={{ textTransform: 'lowercase' }}>lowercase</Text>
<Text style={{ textTransform: 'capitalize' }}>Title Case</Text>
```

</details> <details> <summary><code>textShadowColor</code>, <code>textShadowOffset</code>, <code>textShadowRadius</code></summary>

Adds a shadow behind text.

```tsx
<Text style={{
  fontSize: 28,
  fontWeight: '700',
  color: '#fff',
  textShadowColor: 'rgba(0,0,0,0.4)',
  textShadowOffset: { width: 0, height: 2 },
  textShadowRadius: 4,
}}>
  Text with shadow
</Text>
```

</details> <details> <summary><code>includeFontPadding</code> · 🤖 Android</summary>

Android adds extra padding around text for ascenders/descenders. Set to `false` to remove it for tighter layout.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|
|**Platform**|Android|

```tsx
<Text style={{ includeFontPadding: false, fontSize: 14 }}>
  No extra Android padding
</Text>
```

</details> <details> <summary><code>numberOfLines</code> and <code>ellipsizeMode</code></summary>

Truncate text after a maximum number of lines.

```tsx
// Single line with ellipsis
<Text numberOfLines={1} style={{ fontSize: 16 }}>
  Very long text that will be truncated...
</Text>

// Two lines max
<Text numberOfLines={2} ellipsizeMode="tail">
  Long description that wraps but never goes beyond two lines...
</Text>
```

**ellipsizeMode values:**

|Value|Truncation position|
|---|---|
|`'tail'`|End: `"Long text…"` (default)|
|`'head'`|Start: `"…long text"`|
|`'middle'`|Middle: `"Long…text"`|
|`'clip'`|No ellipsis — clips sharply|

</details>

---

## Colors & Background

<details> <summary><code>color</code> — text color</summary>

|||
|---|---|
|**Type**|`color`|
|**Default**|`'black'`|

```tsx
// Hex
<Text style={{ color: '#6366f1' }} />

// RGB / RGBA
<Text style={{ color: 'rgba(99, 102, 241, 0.8)' }} />

// HSL
<Text style={{ color: 'hsl(239, 84%, 67%)' }} />

// Named colors
<Text style={{ color: 'transparent' }} />
<Text style={{ color: 'white' }} />
<Text style={{ color: 'black' }} />

// Platform-aware (PlatformColor)
import { PlatformColor } from 'react-native';
<Text style={{ color: PlatformColor('label', '#111') }} />
```

</details> <details> <summary><code>backgroundColor</code></summary>

|||
|---|---|
|**Type**|`color`|
|**Default**|`'transparent'`|

```tsx
<View style={{ backgroundColor: '#6366f1' }} />
<View style={{ backgroundColor: 'rgba(0,0,0,0.5)' }} />
<View style={{ backgroundColor: 'transparent' }} />
```

</details> <details> <summary><code>opacity</code></summary>

Transparency of the entire component including its children.

|||
|---|---|
|**Type**|`number`|
|**Range**|`0` (invisible) to `1` (fully visible)|
|**Default**|`1`|

```tsx
<View style={{ opacity: 0.5 }}>...</View>

// Disabled state
<View style={{ opacity: disabled ? 0.4 : 1 }}>
  <Button />
</View>
```

> ⚠️ `opacity` affects all children. Use `backgroundColor: 'rgba(...)'` if you only want a semi-transparent background without affecting children.

</details> <details> <summary><code>PlatformColor</code> — system semantic colors</summary>

Maps to native semantic colors that automatically adapt to light/dark mode.

```tsx
import { PlatformColor, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  label: {
    // iOS: uses system label color (black/white auto)
    // Android: fallback to #000
    color: PlatformColor('label', '#000000'),
  },
  secondaryLabel: {
    color: PlatformColor('secondaryLabel', '#6b7280'),
  },
  background: {
    backgroundColor: PlatformColor('systemBackground', '#ffffff'),
  },
  separator: {
    backgroundColor: PlatformColor('separator', '#e5e7eb'),
  },
});
```

**Common iOS semantic colors:**

|Name|Usage|
|---|---|
|`label`|Primary text|
|`secondaryLabel`|Secondary text|
|`tertiaryLabel`|Tertiary text|
|`systemBackground`|Main background|
|`secondarySystemBackground`|Grouped background|
|`separator`|Divider lines|
|`systemBlue`|Blue accent|
|`systemRed`|Danger / error|
|`systemGreen`|Success|

</details> <details> <summary><code>DynamicColorIOS</code> — light/dark color pair · 🍎 iOS</summary>

Specify different colors for light and dark mode on iOS.

```tsx
import { DynamicColorIOS, Platform, StyleSheet } from 'react-native';

const dynamicColor = Platform.OS === 'ios'
  ? DynamicColorIOS({ light: '#000000', dark: '#ffffff' })
  : '#000000';

const styles = StyleSheet.create({
  text: {
    color: dynamicColor,
  },
  card: {
    backgroundColor: Platform.OS === 'ios'
      ? DynamicColorIOS({ light: '#ffffff', dark: '#1c1c1e' })
      : '#ffffff',
  },
});
```

</details>

---

## Borders

<details> <summary><code>borderWidth</code>, <code>borderTopWidth</code>, etc.</summary>

|Property|Description|
|---|---|
|`borderWidth`|All four sides|
|`borderTopWidth`|Top only|
|`borderRightWidth`|Right only|
|`borderBottomWidth`|Bottom only|
|`borderLeftWidth`|Left only|
|`borderStartWidth`|RTL-aware start|
|`borderEndWidth`|RTL-aware end|

```tsx
// Full border
<View style={{ borderWidth: 1, borderColor: '#e5e7eb' }} />

// Bottom border only (divider)
<View style={{ borderBottomWidth: StyleSheet.hairlineWidth, borderBottomColor: '#e5e7eb' }} />

// Top border only
<View style={{ borderTopWidth: 2, borderTopColor: '#6366f1' }} />
```

</details> <details> <summary><code>borderColor</code>, <code>borderTopColor</code>, etc.</summary>

|Property|Description|
|---|---|
|`borderColor`|All four sides|
|`borderTopColor`|Top only|
|`borderRightColor`|Right only|
|`borderBottomColor`|Bottom only|
|`borderLeftColor`|Left only|

```tsx
// Single color for all sides
<View style={{ borderWidth: 2, borderColor: '#6366f1', borderRadius: 8 }} />

// Different colors per side
<View style={{
  borderWidth: 2,
  borderTopColor: '#ef4444',
  borderRightColor: '#f59e0b',
  borderBottomColor: '#22c55e',
  borderLeftColor: '#6366f1',
}} />
```

</details> <details> <summary><code>borderRadius</code> and corner variants</summary>

|Property|Corner|
|---|---|
|`borderRadius`|All four corners|
|`borderTopLeftRadius`|Top-left|
|`borderTopRightRadius`|Top-right|
|`borderBottomLeftRadius`|Bottom-left|
|`borderBottomRightRadius`|Bottom-right|
|`borderTopStartRadius`|RTL-aware top-start|
|`borderTopEndRadius`|RTL-aware top-end|
|`borderBottomStartRadius`|RTL-aware bottom-start|
|`borderBottomEndRadius`|RTL-aware bottom-end|

```tsx
// Card with uniform radius
<View style={{ borderRadius: 12 }} />

// Pill / capsule
<View style={{ borderRadius: 999 }} />

// Circle (must be half of width/height)
<View style={{ width: 60, height: 60, borderRadius: 30 }} />

// Top rounded only
<View style={{ borderTopLeftRadius: 16, borderTopRightRadius: 16 }} />

// Mixed corners
<View style={{
  borderTopLeftRadius: 16,
  borderTopRightRadius: 4,
  borderBottomLeftRadius: 4,
  borderBottomRightRadius: 16,
}} />
```

</details> <details> <summary><code>borderStyle</code></summary>

|||
|---|---|
|**Type**|`'solid' \| 'dotted' \| 'dashed'`|
|**Default**|`'solid'`|

```tsx
<View style={{ borderWidth: 2, borderStyle: 'dashed', borderColor: '#6366f1' }} />
<View style={{ borderWidth: 2, borderStyle: 'dotted', borderColor: '#ef4444' }} />
```

</details> <details> <summary><code>outlineWidth</code>, <code>outlineColor</code>, <code>outlineStyle</code> — focus ring (web / RN 0.73+)</summary>

Renders an outline around the element. Unlike `border`, `outline` does not affect layout.

```tsx
<TextInput style={{
  outlineWidth: 2,
  outlineColor: '#6366f1',
  outlineStyle: 'solid',
}} />
```

</details>

---

## Shadow & Elevation

<details> <summary>iOS shadow — <code>shadowColor</code>, <code>shadowOffset</code>, <code>shadowOpacity</code>, <code>shadowRadius</code></summary>

iOS shadows require all four properties to be set together.

```tsx
// Card shadow — iOS
const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.12,
    shadowRadius: 8,
  },
});
```

|Property|Type|Description|
|---|---|---|
|`shadowColor`|`color`|Shadow color|
|`shadowOffset`|`{ width, height }`|Shadow displacement in px|
|`shadowOpacity`|`number` (0–1)|Shadow transparency|
|`shadowRadius`|`number`|Blur radius|

</details> <details> <summary>Android shadow — <code>elevation</code></summary>

Android uses Material Design elevation. Accepts a number — higher = more shadow.

|||
|---|---|
|**Type**|`number`|
|**Platform**|Android|

```tsx
// Card shadow — Android
<View style={{ elevation: 4, backgroundColor: '#fff', borderRadius: 12 }}>
  <Content />
</View>
```

</details> <details> <summary>Cross-platform shadow pattern</summary>

```tsx
const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 12,
    // iOS
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    // Android
    elevation: 4,
  },
  // Stronger shadow
  cardElevated: {
    backgroundColor: '#fff',
    borderRadius: 16,
    shadowColor: '#6366f1',
    shadowOffset: { width: 0, height: 6 },
    shadowOpacity: 0.2,
    shadowRadius: 16,
    elevation: 8,
  },
});
```

</details>

---

## Transforms

<details> <summary><code>transform</code> — array of transform operations</summary>

Applies geometric transformations. Pass an array of transform objects.

|Transform|Type|Description|
|---|---|---|
|`translateX`|`number`|Horizontal move in px|
|`translateY`|`number`|Vertical move in px|
|`scale`|`number`|Uniform scale|
|`scaleX`|`number`|Horizontal scale|
|`scaleY`|`number`|Vertical scale|
|`rotate`|`string`|Rotation (e.g. `'45deg'`)|
|`rotateX`|`string`|3D rotation around X axis|
|`rotateY`|`string`|3D rotation around Y axis|
|`rotateZ`|`string`|Same as `rotate`|
|`skewX`|`string`|Horizontal skew|
|`skewY`|`string`|Vertical skew|
|`perspective`|`number`|3D perspective depth|
|`matrix`|`number[]`|Raw 4×4 matrix|

```tsx
// Move
<View style={{ transform: [{ translateX: 50 }, { translateY: 20 }] }} />

// Scale
<View style={{ transform: [{ scale: 1.2 }] }} />

// Rotate
<View style={{ transform: [{ rotate: '45deg' }] }} />

// Flip horizontal
<View style={{ transform: [{ scaleX: -1 }] }} />

// Flip vertical
<View style={{ transform: [{ scaleY: -1 }] }} />

// Combined
<View style={{
  transform: [
    { translateY: -10 },
    { scale: 0.95 },
    { rotate: '5deg' },
  ]
}} />

// 3D card flip
<View style={{
  transform: [
    { perspective: 1000 },
    { rotateY: '45deg' },
  ]
}} />
```

</details> <details> <summary><code>transformOrigin</code> — pivot point for transforms (RN 0.73+)</summary>

Sets the origin point for rotations and scales. Defaults to center.

|||
|---|---|
|**Type**|`string \| [string \| number, string \| number]`|
|**Default**|`'50% 50%'` (center)|

```tsx
// Rotate from top-left corner
<View style={{ transform: [{ rotate: '15deg' }], transformOrigin: 'top left' }} />

// Scale from top center
<View style={{ transform: [{ scale: 0.8 }], transformOrigin: 'top center' }} />

// Precise pixel origin
<View style={{ transform: [{ rotate: '10deg' }], transformOrigin: [0, 0] }} />
```

</details>

---

## Opacity & Visibility

<details> <summary><code>opacity</code> — full component transparency</summary>

```tsx
// Semi-transparent
<View style={{ opacity: 0.5 }} />

// Completely hidden (but still takes up space)
<View style={{ opacity: 0 }} />

// Conditional visibility
<View style={{ opacity: isVisible ? 1 : 0 }} />
```

</details> <details> <summary><code>display</code> — show or hide without affecting layout</summary>

|||
|---|---|
|**Type**|`'flex' \| 'none'`|
|**Default**|`'flex'`|

```tsx
// Hidden — removes from layout (no space taken)
<View style={{ display: 'none' }}>
  <Hidden />
</View>

// Show/hide based on state
<View style={{ display: isOpen ? 'flex' : 'none' }}>
  <Dropdown />
</View>
```

> `display: 'none'` removes the component from layout (no space). `opacity: 0` hides it but keeps its space.

</details> <details> <summary><code>pointerEvents</code> — touch event handling</summary>

|||
|---|---|
|**Type**|`'box-none' \| 'none' \| 'box-only' \| 'auto'`|
|**Default**|`'auto'`|

|Value|Description|
|---|---|
|`'auto'`|Normal touch behavior|
|`'none'`|This view and all children ignore touches|
|`'box-none'`|This view ignores touches, children don't|
|`'box-only'`|This view captures touches, children don't|

```tsx
// Pass touches through overlay to content below
<View style={[StyleSheet.absoluteFillObject, { pointerEvents: 'box-none' }]}>
  <CornerButton />  {/* This still receives touches */}
</View>

// Disable interaction on disabled button
<View style={{ pointerEvents: isDisabled ? 'none' : 'auto', opacity: isDisabled ? 0.4 : 1 }}>
  <Button />
</View>
```

</details>

---

## Image Styles

<details> <summary><code>resizeMode</code> — image scaling</summary>

|||
|---|---|
|**Type**|`'cover' \| 'contain' \| 'stretch' \| 'repeat' \| 'center'`|
|**Default**|`'cover'`|

```tsx
// Fill container, crop excess
<Image style={{ width: 200, height: 200, resizeMode: 'cover' }} source={src} />

// Show full image, letterbox
<Image style={{ width: 200, height: 200, resizeMode: 'contain' }} source={src} />

// Stretch to fill (may distort)
<Image style={{ width: 200, height: 200, resizeMode: 'stretch' }} source={src} />

// Center at original size
<Image style={{ width: 200, height: 200, resizeMode: 'center' }} source={src} />
```

</details> <details> <summary><code>tintColor</code> — colorize image</summary>

Tints all non-transparent pixels to the given color. Great for icon theming.

```tsx
// Monochrome icon with brand color
<Image
  source={require('./icons/heart.png')}
  style={{ width: 24, height: 24, tintColor: '#ef4444' }}
/>

// Dynamic theme color
<Image
  source={tabIcon}
  style={{ tintColor: isFocused ? '#6366f1' : '#9ca3af' }}
/>
```

</details> <details> <summary><code>overlayColor</code> — rounded corner fix on Android · 🤖</summary>

Fixes Android's white corner artifacts when using `borderRadius` on images.

```tsx
<Image
  source={avatarUri}
  style={{
    width: 60, height: 60,
    borderRadius: 30,
    overlayColor: '#ffffff',   // match background color
  }}
/>
```

</details> <details> <summary><code>objectFit</code> — modern resizeMode alternative (RN 0.73+)</summary>

|||
|---|---|
|**Type**|`'cover' \| 'contain' \| 'fill' \| 'scale-down'`|

```tsx
<Image style={{ width: 200, height: 200, objectFit: 'cover' }} source={src} />
```

</details>

---

## Overflow & Clipping

<details> <summary><code>overflow</code> — clip content outside bounds</summary>

|||
|---|---|
|**Type**|`'visible' \| 'hidden' \| 'scroll'`|
|**Default**|`'hidden'` on Android, `'visible'` on iOS|

```tsx
// Clip children to rounded border (Android border-radius fix)
<View style={{ borderRadius: 16, overflow: 'hidden' }}>
  <Image style={{ width: '100%', height: 200 }} source={src} />
</View>

// Show content outside bounds (e.g. tooltips, badges)
<View style={{ overflow: 'visible' }}>
  <Badge style={{ position: 'absolute', top: -8, right: -8 }} />
</View>
```

</details>

---

## View-Specific Styles

<details> <summary><code>backfaceVisibility</code> — 3D flip visibility</summary>

|||
|---|---|
|**Type**|`'visible' \| 'hidden'`|
|**Default**|`'visible'`|

```tsx
// Hide back face during 3D card flip
<Animated.View style={[styles.card, { backfaceVisibility: 'hidden', transform: [{ rotateY: flipAnim }] }]}>
  <Front />
</Animated.View>
```

</details> <details> <summary><code>cursor</code> — pointer cursor on web (RN 0.71+)</summary>

|||
|---|---|
|**Type**|`'auto' \| 'default' \| 'pointer' \| 'grab' \| 'grabbing' \| ...`|

```tsx
<TouchableOpacity style={{ cursor: 'pointer' }}>
  <Text>Clickable on Web</Text>
</TouchableOpacity>
```

</details> <details> <summary><code>userSelect</code> — text selection on web</summary>

|||
|---|---|
|**Type**|`'auto' \| 'none' \| 'text' \| 'all' \| 'contain'`|

```tsx
// Prevent text selection
<Text style={{ userSelect: 'none' }}>Not selectable</Text>

// Allow text selection
<Text style={{ userSelect: 'text' }}>Selectable text</Text>
```

</details>

---

## Units & Values

<details> <summary>Density-independent pixels (dp / pt)</summary>

All numeric values in React Native are in **density-independent pixels** (dp on Android, pt on iOS). They scale automatically with screen density.

```tsx
// 16 here means 16dp — not physical pixels
<View style={{ padding: 16, margin: 8 }} />

// Get physical pixel ratio
import { PixelRatio } from 'react-native';
const ratio = PixelRatio.get();  // 1, 2, 3, or 3.5
const physicalPixels = 16 * ratio;  // actual pixels on screen

// Convert dp to physical pixels
const px = PixelRatio.roundToNearestPixel(16);

// Get font scale (accessibility)
const fontScale = PixelRatio.getFontScale();
```

</details> <details> <summary>Percentages</summary>

Some layout properties accept percentage strings relative to the parent.

```tsx
// Supported with percentages
<View style={{ width: '100%', height: '50%' }} />
<View style={{ minWidth: '20%', maxWidth: '80%' }} />
<View style={{ marginHorizontal: '5%' }} />
<View style={{ paddingVertical: '2%' }} />

// NOT supported with percentages
// fontSize, lineHeight, borderWidth, shadowRadius → numbers only
```

</details> <details> <summary>Getting screen dimensions</summary>

```tsx
import { Dimensions, useWindowDimensions } from 'react-native';

// Static (does not update on rotation)
const { width, height } = Dimensions.get('window');
const { width: screenW } = Dimensions.get('screen');

// Reactive hook (updates on rotation / split screen)
function MyComponent() {
  const { width, height, scale, fontScale } = useWindowDimensions();

  return (
    <View style={{ width: width * 0.9, maxWidth: 600 }}>
      <Content />
    </View>
  );
}
```

</details> <details> <summary>Color formats</summary>

React Native supports these color formats:

```tsx
// Hex
color: '#rgb'           // shorthand
color: '#rrggbb'        // standard
color: '#rrggbbaa'      // with alpha
color: '#rgba'          // shorthand with alpha

// RGB / RGBA
color: 'rgb(255, 99, 71)'
color: 'rgba(255, 99, 71, 0.5)'

// HSL / HSLA
color: 'hsl(9, 100%, 64%)'
color: 'hsla(9, 100%, 64%, 0.5)'

// Named colors (CSS Level 1–4)
color: 'red'
color: 'transparent'

// HWB (RN 0.72+)
color: 'hwb(360, 0%, 0%)'

// Lab / LCH / OKLab / OKLCH (RN 0.72+)
color: 'lab(50% 40 -20)'
color: 'oklch(70% 0.2 200)'
```

</details>

---

## Platform-Specific Styles

<details> <summary><code>Platform.select</code> — different styles per platform</summary>

```tsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  shadow: {
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.15,
        shadowRadius: 8,
      },
      android: {
        elevation: 4,
      },
      web: {
        boxShadow: '0 2px 8px rgba(0,0,0,0.15)',
      },
    }),
  },
  font: {
    ...Platform.select({
      ios: { fontFamily: '-apple-system' },
      android: { fontFamily: 'Roboto' },
      default: { fontFamily: 'System' },
    }),
  },
});
```

</details> <details> <summary>Platform-specific style files</summary>

React Native automatically picks up `.ios.ts` and `.android.ts` files.

```
components/
├── Card.tsx           ← shared code
├── Card.ios.tsx       ← iOS-specific (auto-selected on iOS)
└── Card.android.tsx   ← Android-specific (auto-selected on Android)
```

```tsx
// Card.ios.tsx — iOS card with native shadow
export const CardStyles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 16,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.12,
    shadowRadius: 12,
  },
});

// Card.android.tsx — Android card with elevation
export const CardStyles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 16,
    elevation: 6,
  },
});
```

</details>

---

## Inheritance Rules

<details> <summary>What styles do and don't inherit</summary>

React Native does NOT have global CSS cascade. Styles do not inherit from parent to child across component boundaries — with one important exception.

**What DOES inherit (within `<Text>` nesting only):**

```tsx
// Text styles propagate to nested Text components
<Text style={{ fontSize: 18, color: '#6366f1', fontFamily: 'Inter-Bold' }}>
  Parent text
  <Text>  {/* inherits fontSize, color, fontFamily */}
    Nested text (inherits all parent text styles)
  </Text>
  <Text style={{ fontSize: 12 }}>
    Override only fontSize — color and fontFamily still inherited
  </Text>
</Text>
```

**What does NOT inherit:**

```tsx
// View styles do NOT propagate to children
<View style={{ backgroundColor: 'red', padding: 16 }}>
  <Text>This text is NOT red — View color doesn't inherit</Text>
  <View>This nested View is NOT red</View>
</View>

// Text styles do NOT cross View boundaries
<Text style={{ color: 'blue', fontSize: 18 }}>
  <View>
    <Text>This Text does NOT inherit blue color — crossed a View boundary</Text>
  </View>
</Text>
```

</details>

---

## Common Patterns

<details> <summary>Full screen layout</summary>

```tsx
const styles = StyleSheet.create({
  screen: {
    flex: 1,
    backgroundColor: '#fff',
  },
  safeArea: {
    flex: 1,
  },
  content: {
    flex: 1,
    paddingHorizontal: 16,
  },
});
```

</details> <details> <summary>Centered content</summary>

```tsx
const styles = StyleSheet.create({
  // Absolute center
  center: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  // Horizontally centered in a row
  hCenter: {
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
  },
  // Centered with max width (web-like)
  container: {
    width: '100%',
    maxWidth: 600,
    alignSelf: 'center',
    paddingHorizontal: 16,
  },
});
```

</details> <details> <summary>Card component</summary>

```tsx
const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 16,
    padding: 16,
    // iOS shadow
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.08,
    shadowRadius: 12,
    // Android shadow
    elevation: 4,
  },
  cardHeader: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 12,
  },
  cardTitle: {
    fontSize: 18,
    fontWeight: '700',
    color: '#111',
    flex: 1,
  },
  cardBody: {
    fontSize: 14,
    lineHeight: 22,
    color: '#6b7280',
  },
});
```

</details> <details> <summary>Button variants</summary>

```tsx
const styles = StyleSheet.create({
  // Base button
  btn: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 10,
    gap: 8,
  },
  // Primary
  btnPrimary: {
    backgroundColor: '#6366f1',
  },
  // Secondary / outline
  btnOutline: {
    backgroundColor: 'transparent',
    borderWidth: 1.5,
    borderColor: '#6366f1',
  },
  // Ghost
  btnGhost: {
    backgroundColor: 'transparent',
  },
  // Danger
  btnDanger: {
    backgroundColor: '#ef4444',
  },
  // Pill shape
  btnPill: {
    borderRadius: 999,
    paddingHorizontal: 28,
  },
  // Full width
  btnBlock: {
    width: '100%',
  },
  // Disabled
  btnDisabled: {
    opacity: 0.4,
  },
  // Text styles
  btnText:        { color: '#fff', fontWeight: '600', fontSize: 15 },
  btnTextOutline: { color: '#6366f1', fontWeight: '600', fontSize: 15 },
  btnTextGhost:   { color: '#374151', fontWeight: '600', fontSize: 15 },
});
```

</details> <details> <summary>Typography scale</summary>

```tsx
const typography = StyleSheet.create({
  display:   { fontSize: 36, fontWeight: '800', lineHeight: 42, letterSpacing: -0.5 },
  h1:        { fontSize: 28, fontWeight: '700', lineHeight: 34, letterSpacing: -0.3 },
  h2:        { fontSize: 22, fontWeight: '700', lineHeight: 28 },
  h3:        { fontSize: 18, fontWeight: '600', lineHeight: 24 },
  subtitle:  { fontSize: 16, fontWeight: '500', lineHeight: 22 },
  body:      { fontSize: 15, fontWeight: '400', lineHeight: 23 },
  bodySmall: { fontSize: 13, fontWeight: '400', lineHeight: 20 },
  caption:   { fontSize: 11, fontWeight: '400', lineHeight: 16 },
  label:     { fontSize: 11, fontWeight: '600', letterSpacing: 0.8, textTransform: 'uppercase' },
});
```

</details> <details> <summary>Responsive layout with <code>useWindowDimensions</code></summary>

```tsx
import { useWindowDimensions, StyleSheet, View } from 'react-native';

function ResponsiveGrid() {
  const { width } = useWindowDimensions();
  const numColumns = width > 768 ? 3 : width > 480 ? 2 : 1;
  const itemWidth = (width - 16 * (numColumns + 1)) / numColumns;

  return (
    <View style={styles.grid}>
      {items.map(item => (
        <View key={item.id} style={[styles.item, { width: itemWidth }]}>
          <Card item={item} />
        </View>
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  grid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    padding: 16,
    gap: 16,
  },
  item: {
    // width set dynamically
  },
});
```

</details> <details> <summary>Absolute overlay patterns</summary>

```tsx
const styles = StyleSheet.create({
  // Full-screen overlay
  overlay: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: 'rgba(0,0,0,0.5)',
  },
  // Bottom sheet backdrop
  backdrop: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: 'rgba(0,0,0,0.4)',
    justifyContent: 'flex-end',
  },
  // Floating action button (FAB)
  fab: {
    position: 'absolute',
    bottom: 24,
    right: 24,
    width: 56,
    height: 56,
    borderRadius: 28,
    backgroundColor: '#6366f1',
    alignItems: 'center',
    justifyContent: 'center',
    shadowColor: '#6366f1',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.4,
    shadowRadius: 8,
    elevation: 8,
  },
  // Notification badge
  badge: {
    position: 'absolute',
    top: -6,
    right: -6,
    minWidth: 18,
    height: 18,
    borderRadius: 9,
    backgroundColor: '#ef4444',
    alignItems: 'center',
    justifyContent: 'center',
    paddingHorizontal: 4,
  },
  badgeText: {
    color: '#fff',
    fontSize: 10,
    fontWeight: '700',
  },
});
```

</details> <details> <summary>Dark mode with <code>useColorScheme</code></summary>

```tsx
import { useColorScheme, StyleSheet } from 'react-native';

const lightColors = {
  background: '#ffffff',
  card:       '#f9fafb',
  text:       '#111827',
  textMuted:  '#6b7280',
  border:     '#e5e7eb',
  primary:    '#6366f1',
};

const darkColors = {
  background: '#0f0f0f',
  card:       '#1f2937',
  text:       '#f9fafb',
  textMuted:  '#9ca3af',
  border:     '#374151',
  primary:    '#818cf8',
};

function useTheme() {
  const scheme = useColorScheme();
  return scheme === 'dark' ? darkColors : lightColors;
}

function MyComponent() {
  const colors = useTheme();

  const styles = StyleSheet.create({
    container: {
      flex: 1,
      backgroundColor: colors.background,
    },
    card: {
      backgroundColor: colors.card,
      borderColor: colors.border,
      borderWidth: StyleSheet.hairlineWidth,
      borderRadius: 12,
      padding: 16,
    },
    title: {
      color: colors.text,
      fontSize: 18,
      fontWeight: '700',
    },
    subtitle: {
      color: colors.textMuted,
      fontSize: 14,
    },
  });

  return (
    <View style={styles.container}>
      <View style={styles.card}>
        <Text style={styles.title}>Card Title</Text>
        <Text style={styles.subtitle}>Subtitle</Text>
      </View>
    </View>
  );
}
```

</details> <details> <summary>Style composition — base + variant pattern</summary>

```tsx
const styles = StyleSheet.create({
  // Base text
  text: {
    fontSize: 14,
    color: '#374151',
    fontFamily: 'Inter-Regular',
  },
  // Variants — compose with base
  textBold:    { fontWeight: '700', fontFamily: 'Inter-Bold' },
  textSmall:   { fontSize: 12 },
  textLarge:   { fontSize: 18 },
  textMuted:   { color: '#9ca3af' },
  textPrimary: { color: '#6366f1' },
  textDanger:  { color: '#ef4444' },
  textCenter:  { textAlign: 'center' },
});

// Usage — compose array
<Text style={[styles.text, styles.textBold, styles.textPrimary]}>
  Bold primary text
</Text>

<Text style={[styles.text, styles.textSmall, styles.textMuted]}>
  Small muted text
</Text>

// Conditional variant
<Text style={[styles.text, isError && styles.textDanger]}>
  {isError ? 'Error message' : 'Normal text'}
</Text>
```

</details>

---

## Quick-Reference Cheatsheet

|Property|Values|Use case|
|---|---|---|
|`flex: 1`|number|Grow to fill space|
|`flexDirection`|`row` \| `column`|Layout axis|
|`justifyContent`|`center` \| `space-between` ...|Main axis alignment|
|`alignItems`|`center` \| `stretch` ...|Cross axis alignment|
|`gap`|number|Space between children|
|`position: 'absolute'`|—|Remove from flow|
|`StyleSheet.absoluteFillObject`|—|Cover parent entirely|
|`zIndex`|number|Stacking order|
|`aspectRatio`|number|Proportional sizing|
|`borderRadius`|number|Rounded corners|
|`overflow: 'hidden'`|—|Clip children to bounds|
|`StyleSheet.hairlineWidth`|`~0.33`|Thin 1px border|
|`shadowColor + elevation`|—|Cross-platform shadow|
|`transform`|array|Move, scale, rotate|
|`opacity`|0–1|Transparency|
|`display: 'none'`|—|Hide without taking space|
|`tintColor`|color|Recolor icon image|
|`PlatformColor`|—|Semantic system colors|
|`Platform.select`|object|Per-platform styles|
|`useWindowDimensions`|hook|Reactive screen size|

---

_Reference based on React Native stable API (0.73+). Always check the [official docs](https://reactnative.dev/docs/stylesheet) for the latest updates._