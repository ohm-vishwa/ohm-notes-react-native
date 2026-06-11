# React Native SafeAreaView — Complete Reference

A comprehensive reference for handling safe areas in React Native.  
Covers the built-in `SafeAreaView`, the recommended `react-native-safe-area-context` library, all hooks, context APIs, and common patterns.

---

## Table of Contents

1. [Overview — Built-in vs Library](#overview--built-in-vs-library)
2. [Built-in SafeAreaView (core RN)](#built-in-safeareaview-core-rn)
3. [Installation — react-native-safe-area-context](#installation--react-native-safe-area-context)
4. [SafeAreaProvider](#safeareaprovider)
5. [SafeAreaView (library)](#safeareaview-library)
6. [useSafeAreaInsets](#usesafeareainsets)
7. [useSafeAreaFrame](#usesafeareaframe)
8. [SafeAreaInsetsContext](#safeareainsetscontext)
9. [SafeAreaFrameContext](#safeareaframecontext)
10. [SafeAreaConsumer](#safeareaconsumer)
11. [withSafeAreaInsets HOC](#withsafeareainsets-hoc)
12. [initialWindowMetrics](#initialwindowmetrics)
13. [Edges Prop](#edges-prop)
14. [Insets Object](#insets-object)
15. [Common Patterns](#common-patterns)

---

## Overview — Built-in vs Library

<details> <summary>When to use built-in <code>SafeAreaView</code> vs <code>react-native-safe-area-context</code></summary>

React Native ships a basic `SafeAreaView` component, but it has significant limitations. The community library `react-native-safe-area-context` is the recommended solution for most apps.

|Feature|Core RN `SafeAreaView`|`react-native-safe-area-context`|
|---|---|---|
|Basic notch/home bar avoidance|✅|✅|
|Android support|❌ Poor|✅ Full|
|Insets as values (hooks)|❌|✅|
|Control per-edge|✅ (RN 0.69+)|✅|
|Works inside modals|❌|✅|
|Works with `react-navigation`|⚠️|✅ Built-in|
|Frame/window dimensions|❌|✅|
|Custom inset overrides|❌|✅|
|Expo support|✅ Included|✅ `npx expo install`|

> ✅ **Recommendation:** Use `react-native-safe-area-context` for all production apps. React Navigation ships it as a dependency, so you likely already have it.

</details> <details> <summary>What are safe area insets?</summary>

Safe area insets are the distances from the screen edges to the area where content is safe to display — avoiding notches, Dynamic Island, home indicator bars, status bars, and rounded screen corners.

```
┌──────────────────────────┐  ─── top inset (e.g. 59px on iPhone 14 Pro)
│  ▐ Dynamic Island ▌      │
│                          │
│   Safe content area      │
│                          │
│──────────────────────────│  ─── bottom inset (e.g. 34px on iPhone 14)
│   ───  home indicator    │
└──────────────────────────┘
```

Insets vary by:

- Device model (notch, Dynamic Island, punch-hole camera)
- Orientation (portrait vs landscape — left/right insets appear)
- Platform (iOS insets are larger; Android varies by manufacturer)
- OS version

</details>

---

## Built-in SafeAreaView (core RN)

<details> <summary><code>SafeAreaView</code> — basic usage</summary>

The core React Native `SafeAreaView` renders a `View` that automatically adds padding to avoid the status bar, notch, and home indicator on **iOS only**.

```jsx
import { SafeAreaView, StyleSheet } from 'react-native';

export default function App() {
  return (
    <SafeAreaView style={styles.container}>
      <Text>Content safely inside screen bounds</Text>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
});
```

> ⚠️ Limitations:
> 
> - **Android**: No support before RN 0.64; even then, behavior is inconsistent across devices
> - Does not expose inset values — you can't use them in custom layouts
> - Does not work correctly inside modals

</details> <details> <summary><code>edges</code> prop — control which edges to pad (RN 0.69+)</summary>

Specifies which edges of the screen to apply safe area insets to. Defaults to all four edges.

|||
|---|---|
|**Type**|`Array<'top' \| 'right' \| 'bottom' \| 'left'>`|
|**Default**|`['top', 'right', 'bottom', 'left']`|

```jsx
// Only pad top and bottom (ignore left/right in landscape)
<SafeAreaView edges={['top', 'bottom']}>
  ...
</SafeAreaView>

// Only pad top (e.g. screen with a custom bottom bar)
<SafeAreaView edges={['top']}>
  ...
</SafeAreaView>
```

</details> <details> <summary><code>style</code> prop</summary>

Standard `ViewStyle`. Always set `flex: 1` and a `backgroundColor` so the safe area padding matches the screen background.

|||
|---|---|
|**Type**|`StyleProp<ViewStyle>`|

```jsx
<SafeAreaView style={{ flex: 1, backgroundColor: '#f9f9f9' }}>
  ...
</SafeAreaView>
```

> ⚠️ Without `flex: 1`, the view may not fill the screen. Without `backgroundColor`, the padded area at top/bottom may appear as a different color.

</details>

---

## Installation — react-native-safe-area-context

<details> <summary>Install and configure</summary>

```bash
# Expo (managed or bare workflow)
npx expo install react-native-safe-area-context

# Bare React Native
npm install react-native-safe-area-context
cd ios && pod install
```

**Required: wrap your entire app in `SafeAreaProvider`**

```jsx
// App.js or index.js
import { SafeAreaProvider } from 'react-native-safe-area-context';

export default function App() {
  return (
    <SafeAreaProvider>
      {/* rest of your app */}
    </SafeAreaProvider>
  );
}
```

> If you use **React Navigation**, `NavigationContainer` includes `SafeAreaProvider` automatically — you don't need to add it again.

</details>

---

## SafeAreaProvider

<details> <summary><code>SafeAreaProvider</code> — required root wrapper</summary>

Provides inset values to all descendant components via React Context. Must wrap your entire app (or at minimum every component that uses safe area hooks).

|Prop|Type|Description|
|---|---|---|
|`initialMetrics`|`Metrics \| null`|Initial inset values to avoid flicker on first render|
|`children`|`ReactNode`|Your app content|

```jsx
import { SafeAreaProvider, initialWindowMetrics } from 'react-native-safe-area-context';

export default function App() {
  return (
    // Pass initialWindowMetrics to avoid layout flicker on first render
    <SafeAreaProvider initialMetrics={initialWindowMetrics}>
      <RootNavigator />
    </SafeAreaProvider>
  );
}
```

> ⚠️ You can nest multiple `SafeAreaProvider`s (e.g. inside a Modal) if you need isolated inset contexts. Each provider measures its own container.

</details>

---

## SafeAreaView (library)

<details> <summary><code>SafeAreaView</code> — from react-native-safe-area-context</summary>

A drop-in replacement for the core `SafeAreaView` with full Android support, `edges` control, and hook access.

|Prop|Type|Default|Description|
|---|---|---|---|
|`style`|`StyleProp<ViewStyle>`|—|Style for the view|
|`edges`|`Edge[] \| EdgeInsets`|all edges|Which edges to apply insets to|
|`mode`|`'padding' \| 'margin'`|`'padding'`|Whether insets are applied as padding or margin|
|`children`|`ReactNode`|—|Content|

```jsx
import { SafeAreaView } from 'react-native-safe-area-context';

function Screen() {
  return (
    <SafeAreaView style={{ flex: 1, backgroundColor: '#fff' }}>
      <Text>Safe content</Text>
    </SafeAreaView>
  );
}
```

</details> <details> <summary><code>edges</code> prop — control which edges are padded</summary>

An array specifying which screen edges to apply safe area insets to.

|||
|---|---|
|**Type**|`Array<'top' \| 'right' \| 'bottom' \| 'left'>` or `{ top?, right?, bottom?, left? }`|
|**Default**|All four edges|

```jsx
// All edges (default)
<SafeAreaView edges={['top', 'right', 'bottom', 'left']}>

// Top only — e.g. screen content has its own bottom bar
<SafeAreaView edges={['top']}>

// Bottom only — e.g. full-bleed header, safe bottom
<SafeAreaView edges={['bottom']}>

// Skip top — e.g. when inside a navigator that handles the header
<SafeAreaView edges={['bottom', 'left', 'right']}>

// Object syntax with per-edge modes (v4.x+)
<SafeAreaView edges={{ top: 'additive', bottom: 'off', left: 'maximum', right: 'maximum' }}>
```

</details> <details> <summary><code>mode</code> prop — <code>'padding'</code> vs <code>'margin'</code></summary>

Controls whether safe area insets are applied as padding or margin on the outer view.

|||
|---|---|
|**Type**|`'padding' \| 'margin'`|
|**Default**|`'padding'`|

```jsx
// Padding mode (default) — safe area is inside the view's background
<SafeAreaView mode="padding" style={{ backgroundColor: 'blue' }}>
  {/* Background color extends to edges, content is padded inward */}
</SafeAreaView>

// Margin mode — safe area is outside the view
<SafeAreaView mode="margin">
  {/* The view itself starts below the notch */}
</SafeAreaView>
```

> ✅ Use `'padding'` when you want the background color to fill the notch/status bar area.  
> Use `'margin'` when you want the view to start below the notch.

</details>

---

## useSafeAreaInsets

<details> <summary><code>useSafeAreaInsets()</code> — access inset values in any component</summary>

Returns the current safe area insets as a plain object. Updates automatically when orientation changes. Must be used inside a `SafeAreaProvider`.

|||
|---|---|
|**Returns**|`{ top: number, right: number, bottom: number, left: number }`|

```jsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function MyScreen() {
  const insets = useSafeAreaInsets();

  return (
    <View
      style={{
        flex: 1,
        paddingTop: insets.top,
        paddingBottom: insets.bottom,
        paddingLeft: insets.left,
        paddingRight: insets.right,
      }}
    >
      <Text>Insets — top: {insets.top}, bottom: {insets.bottom}</Text>
    </View>
  );
}
```

</details> <details> <summary>Using insets for precise custom layouts</summary>

```jsx
function ChatScreen() {
  const insets = useSafeAreaInsets();

  return (
    <View style={{ flex: 1 }}>
      {/* Custom header that respects status bar */}
      <View style={{
        height: 56 + insets.top,
        paddingTop: insets.top,
        backgroundColor: '#6366f1',
        justifyContent: 'flex-end',
        paddingHorizontal: 16,
        paddingBottom: 10,
      }}>
        <Text style={{ color: '#fff', fontSize: 18 }}>Messages</Text>
      </View>

      {/* Scrollable content */}
      <FlatList data={messages} renderItem={renderMessage} style={{ flex: 1 }} />

      {/* Input bar above home indicator */}
      <View style={{
        paddingBottom: insets.bottom,
        backgroundColor: '#fff',
        borderTopWidth: 1,
        borderColor: '#e5e7eb',
        paddingHorizontal: 16,
        paddingTop: 10,
      }}>
        <TextInput placeholder="Message..." />
      </View>
    </View>
  );
}
```

</details>

---

## useSafeAreaFrame

<details> <summary><code>useSafeAreaFrame()</code> — get the window frame dimensions</summary>

Returns the dimensions of the safe area provider's frame (effectively the window/screen size). Useful as an alternative to `Dimensions.get('window')` since it updates correctly inside modals and split-screen modes.

|||
|---|---|
|**Returns**|`{ x: number, y: number, width: number, height: number }`|

```jsx
import { useSafeAreaFrame } from 'react-native-safe-area-context';

function ResponsiveLayout() {
  const frame = useSafeAreaFrame();
  const isWide = frame.width > 600;

  return (
    <View style={{
      flexDirection: isWide ? 'row' : 'column',
    }}>
      {isWide && <Sidebar />}
      <MainContent />
    </View>
  );
}
```

</details>

---

## SafeAreaInsetsContext

<details> <summary><code>SafeAreaInsetsContext</code> — React Context for insets</summary>

The raw React Context used by `useSafeAreaInsets`. Useful in class components or when you need to consume the context directly.

|||
|---|---|
|**Type**|`React.Context<EdgeInsets \| null>`|

```jsx
import { SafeAreaInsetsContext } from 'react-native-safe-area-context';

// In a class component
class MyScreen extends React.Component {
  render() {
    return (
      <SafeAreaInsetsContext.Consumer>
        {(insets) => (
          <View style={{ paddingTop: insets?.top ?? 0 }}>
            <Text>Hello</Text>
          </View>
        )}
      </SafeAreaInsetsContext.Consumer>
    );
  }
}

// Or with useContext in a function component (same as useSafeAreaInsets)
const insets = useContext(SafeAreaInsetsContext);
```

</details>

---

## SafeAreaFrameContext

<details> <summary><code>SafeAreaFrameContext</code> — React Context for frame</summary>

The raw React Context used by `useSafeAreaFrame`.

|||
|---|---|
|**Type**|`React.Context<Rect \| null>`|

```jsx
import { SafeAreaFrameContext } from 'react-native-safe-area-context';

// Class component usage
<SafeAreaFrameContext.Consumer>
  {(frame) => (
    <Text>Width: {frame?.width ?? 0}</Text>
  )}
</SafeAreaFrameContext.Consumer>

// useContext equivalent
const frame = useContext(SafeAreaFrameContext);
```

</details>

---

## SafeAreaConsumer

<details> <summary><code>SafeAreaConsumer</code> — render prop API for insets</summary>

A convenience component that wraps `SafeAreaInsetsContext.Consumer`. Useful for class components.

```jsx
import { SafeAreaConsumer } from 'react-native-safe-area-context';

function MyComponent() {
  return (
    <SafeAreaConsumer>
      {(insets) => (
        <View style={{ paddingTop: insets.top, paddingBottom: insets.bottom }}>
          <Text>Content</Text>
        </View>
      )}
    </SafeAreaConsumer>
  );
}
```

</details>

---

## withSafeAreaInsets HOC

<details> <summary><code>withSafeAreaInsets(Component)</code> — higher-order component</summary>

Wraps a class component and injects `insets` as a prop. Prefer `useSafeAreaInsets()` in function components.

|Injected prop|Type|Description|
|---|---|---|
|`insets`|`EdgeInsets`|Current safe area insets|

```jsx
import { withSafeAreaInsets } from 'react-native-safe-area-context';

class MyHeader extends React.Component {
  render() {
    const { insets, title } = this.props;
    return (
      <View style={{ paddingTop: insets.top, height: 56 + insets.top }}>
        <Text>{title}</Text>
      </View>
    );
  }
}

export default withSafeAreaInsets(MyHeader);
```

</details>

---

## initialWindowMetrics

<details> <summary><code>initialWindowMetrics</code> — avoid layout flicker on first render</summary>

A synchronously available snapshot of the current window's safe area insets and frame. Pass this to `SafeAreaProvider`'s `initialMetrics` prop to avoid a brief flicker caused by the async inset measurement on first render.

|||
|---|---|
|**Type**|`Metrics \| null`|
|**Shape**|`{ insets: EdgeInsets, frame: Rect }`|

```jsx
import { SafeAreaProvider, initialWindowMetrics } from 'react-native-safe-area-context';

export default function App() {
  return (
    <SafeAreaProvider initialMetrics={initialWindowMetrics}>
      <AppNavigator />
    </SafeAreaProvider>
  );
}
```

> ✅ Always pass `initialMetrics={initialWindowMetrics}` in production apps to eliminate the one-frame layout shift on startup.

</details>

---

## Edges Prop

<details> <summary>Edge values — <code>'top'</code>, <code>'bottom'</code>, <code>'left'</code>, <code>'right'</code></summary>

Controls which screen edges the `SafeAreaView` applies insets for.

```jsx
// All edges (default — most screens)
<SafeAreaView edges={['top', 'bottom', 'left', 'right']}>

// Top only (e.g. custom bottom navigation handled separately)
<SafeAreaView edges={['top']}>

// Bottom only (e.g. full-bleed image header, need to avoid home indicator)
<SafeAreaView edges={['bottom']}>

// Horizontal only (landscape mode sidebar)
<SafeAreaView edges={['left', 'right']}>

// Skip top when a navigator header already handles it
<SafeAreaView edges={['bottom', 'left', 'right']}>
```

</details> <details> <summary>Edge mode values (v4.x) — <code>'additive'</code>, <code>'maximum'</code>, <code>'off'</code></summary>

In v4+, you can pass an object with per-edge mode values instead of an array.

|Mode|Behavior|
|---|---|
|`'additive'`|Adds inset on top of existing padding|
|`'maximum'`|Uses whichever is greater — existing padding or inset|
|`'off'`|Ignores this edge entirely|

```jsx
<SafeAreaView
  edges={{
    top: 'additive',    // add inset on top of any existing top padding
    bottom: 'maximum',  // use max of inset vs existing padding
    left: 'off',        // ignore left edge
    right: 'off',       // ignore right edge
  }}
>
  ...
</SafeAreaView>
```

</details>

---

## Insets Object

<details> <summary>Inset values — <code>top</code>, <code>bottom</code>, <code>left</code>, <code>right</code></summary>

All safe area APIs return an insets object with four numeric values in device-independent pixels (dp/pt).

|Property|Description|Example values|
|---|---|---|
|`top`|Distance from top edge to safe area|`59` (iPhone 14 Pro), `44` (iPhone X), `20` (no notch), `0` (Android without status bar)|
|`bottom`|Distance from bottom edge to safe area|`34` (iPhones with home indicator), `0` (older iPhones, most Androids)|
|`left`|Distance from left edge (landscape)|`44–59` in landscape on notched iPhones, `0` in portrait|
|`right`|Distance from right edge (landscape)|`44–59` in landscape on notched iPhones, `0` in portrait|

```jsx
const insets = useSafeAreaInsets();
// Portrait iPhone 14 Pro:   { top: 59, bottom: 34, left: 0, right: 0 }
// Landscape iPhone 14 Pro:  { top: 0,  bottom: 21, left: 59, right: 59 }
// Android Pixel 7:          { top: 28, bottom: 0,  left: 0,  right: 0 }
// Android with gesture nav: { top: 28, bottom: 16, left: 0,  right: 0 }
```

</details>

---

## Common Patterns

<details> <summary>Basic full-screen layout</summary>

```jsx
import { SafeAreaView } from 'react-native-safe-area-context';

function HomeScreen() {
  return (
    <SafeAreaView style={{ flex: 1, backgroundColor: '#fff' }}>
      <ScrollView>
        <Text>Your content here</Text>
      </ScrollView>
    </SafeAreaView>
  );
}
```

</details> <details> <summary>Custom header respecting status bar</summary>

```jsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const HEADER_HEIGHT = 56;

function AppHeader({ title, onBack }) {
  const insets = useSafeAreaInsets();

  return (
    <View
      style={{
        height: HEADER_HEIGHT + insets.top,
        paddingTop: insets.top,
        paddingHorizontal: 16,
        backgroundColor: '#6366f1',
        flexDirection: 'row',
        alignItems: 'flex-end',
        paddingBottom: 12,
      }}
    >
      {onBack && (
        <TouchableOpacity onPress={onBack} style={{ marginRight: 12 }}>
          <Ionicons name="arrow-back" size={24} color="#fff" />
        </TouchableOpacity>
      )}
      <Text style={{ color: '#fff', fontSize: 18, fontWeight: '600' }}>{title}</Text>
    </View>
  );
}
```

</details> <details> <summary>Bottom tab bar that avoids home indicator</summary>

```jsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const TAB_BAR_HEIGHT = 56;

function CustomTabBar({ state, navigation }) {
  const insets = useSafeAreaInsets();

  return (
    <View
      style={{
        height: TAB_BAR_HEIGHT + insets.bottom,
        paddingBottom: insets.bottom,
        flexDirection: 'row',
        backgroundColor: '#fff',
        borderTopWidth: 1,
        borderColor: '#e5e7eb',
      }}
    >
      {state.routes.map((route, index) => {
        const isFocused = state.index === index;
        return (
          <TouchableOpacity
            key={route.key}
            style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}
            onPress={() => navigation.navigate(route.name)}
          >
            <TabIcon name={route.name} focused={isFocused} />
          </TouchableOpacity>
        );
      })}
    </View>
  );
}
```

</details> <details> <summary>FlatList with safe area padding</summary>

```jsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function FeedScreen() {
  const insets = useSafeAreaInsets();

  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={(item) => item.id}
      // Pads the scroll content, not the FlatList container
      contentContainerStyle={{
        paddingTop: insets.top,
        paddingBottom: insets.bottom + 16,
        paddingHorizontal: Math.max(insets.left, insets.right),
      }}
      // Optional: show scroll indicators inside safe area
      scrollIndicatorInsets={{ right: 1 }}
    />
  );
}
```

</details> <details> <summary>Modal with its own SafeAreaProvider</summary>

```jsx
import { Modal } from 'react-native';
import { SafeAreaProvider, SafeAreaView } from 'react-native-safe-area-context';

function BottomSheet({ visible, onClose, children }) {
  return (
    <Modal visible={visible} transparent animationType="slide" onRequestClose={onClose}>
      {/* Modals are outside the root SafeAreaProvider — wrap in a new one */}
      <SafeAreaProvider>
        <View style={{ flex: 1, justifyContent: 'flex-end' }}>
          <SafeAreaView
            edges={['bottom']}
            style={{ backgroundColor: '#fff', borderTopLeftRadius: 20, borderTopRightRadius: 20 }}
          >
            <View style={{ padding: 24 }}>{children}</View>
          </SafeAreaView>
        </View>
      </SafeAreaProvider>
    </Modal>
  );
}
```

</details> <details> <summary>Scroll view with transparent header (content behind header)</summary>

```jsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const HEADER_HEIGHT = 56;

function TransparentHeaderScreen() {
  const insets = useSafeAreaInsets();
  const totalHeaderHeight = HEADER_HEIGHT + insets.top;

  return (
    <View style={{ flex: 1 }}>
      {/* Transparent floating header */}
      <View
        style={{
          position: 'absolute',
          top: 0, left: 0, right: 0,
          height: totalHeaderHeight,
          paddingTop: insets.top,
          zIndex: 10,
          backgroundColor: 'rgba(255,255,255,0.9)',
        }}
      >
        <Text style={{ paddingHorizontal: 16, fontSize: 18 }}>Header</Text>
      </View>

      {/* Scroll content starts behind header */}
      <ScrollView
        contentContainerStyle={{
          paddingTop: totalHeaderHeight,
          paddingBottom: insets.bottom + 16,
        }}
      >
        <Image source={heroImage} style={{ width: '100%', height: 300 }} />
        <View style={{ padding: 16 }}>
          <Text>Scrollable content...</Text>
        </View>
      </ScrollView>
    </View>
  );
}
```

</details> <details> <summary>Keyboard avoiding view with safe area bottom</summary>

```jsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import { KeyboardAvoidingView, Platform } from 'react-native';

function LoginScreen() {
  const insets = useSafeAreaInsets();

  return (
    <KeyboardAvoidingView
      style={{ flex: 1 }}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      keyboardVerticalOffset={insets.top}
    >
      <SafeAreaView style={{ flex: 1 }} edges={['top']}>
        <ScrollView contentContainerStyle={{ flexGrow: 1, justifyContent: 'center', padding: 24 }}>
          <TextInput placeholder="Email" style={styles.input} />
          <TextInput placeholder="Password" secureTextEntry style={styles.input} />
          <View style={{ paddingBottom: insets.bottom }}>
            <Button title="Login" onPress={handleLogin} />
          </View>
        </ScrollView>
      </SafeAreaView>
    </KeyboardAvoidingView>
  );
}
```

</details> <details> <summary>React Navigation — recommended setup</summary>

React Navigation's `NavigationContainer` handles `SafeAreaProvider` automatically. However you should still manage edges on individual screens.

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

// App.js — NavigationContainer includes SafeAreaProvider
export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        {/* Screens with built-in header — skip top edge */}
        <Stack.Screen name="Home" component={HomeScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

// Screen that uses navigator's header — don't double-pad the top
function HomeScreen() {
  return (
    <SafeAreaView
      edges={['bottom', 'left', 'right']}  // navigator handles top
      style={{ flex: 1 }}
    >
      <ScrollView>...</ScrollView>
    </SafeAreaView>
  );
}

// Screen with headerShown: false — handle all edges manually
function SplashScreen() {
  return (
    <SafeAreaView edges={['top', 'bottom']} style={{ flex: 1 }}>
      <Text>Welcome</Text>
    </SafeAreaView>
  );
}
```

</details> <details> <summary>Full-bleed hero image with safe area overlay</summary>

```jsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function HeroCard({ imageUri, title, onBack }) {
  const insets = useSafeAreaInsets();

  return (
    <View style={{ flex: 1 }}>
      {/* Full-bleed image */}
      <Image source={{ uri: imageUri }} style={StyleSheet.absoluteFillObject} resizeMode="cover" />

      {/* Gradient overlay */}
      <LinearGradient
        colors={['rgba(0,0,0,0.6)', 'transparent', 'rgba(0,0,0,0.8)']}
        style={StyleSheet.absoluteFillObject}
      />

      {/* Back button at top — respects status bar */}
      <TouchableOpacity
        style={{
          position: 'absolute',
          top: insets.top + 8,
          left: 16,
          padding: 8,
          borderRadius: 20,
          backgroundColor: 'rgba(0,0,0,0.4)',
        }}
        onPress={onBack}
      >
        <Ionicons name="arrow-back" size={20} color="#fff" />
      </TouchableOpacity>

      {/* Title at bottom — respects home indicator */}
      <View
        style={{
          position: 'absolute',
          bottom: insets.bottom + 16,
          left: 16, right: 16,
        }}
      >
        <Text style={{ color: '#fff', fontSize: 28, fontWeight: '700' }}>{title}</Text>
      </View>
    </View>
  );
}
```

</details> <details> <summary>Testing — mock insets in Jest</summary>

```jsx
// jest.setup.js
jest.mock('react-native-safe-area-context', () => {
  const insets = { top: 44, bottom: 34, left: 0, right: 0 };
  const frame = { x: 0, y: 0, width: 390, height: 844 };
  return {
    SafeAreaProvider: ({ children }) => children,
    SafeAreaView: ({ children, style }) => (
      <View style={style}>{children}</View>
    ),
    useSafeAreaInsets: () => insets,
    useSafeAreaFrame: () => frame,
    SafeAreaInsetsContext: {
      Consumer: ({ children }) => children(insets),
    },
    initialWindowMetrics: { insets, frame },
  };
});
```

</details> <details> <summary>Dynamic landscape insets — handling orientation change</summary>

```jsx
function LandscapeAwareLayout({ children }) {
  const insets = useSafeAreaInsets();
  // insets.left and insets.right are non-zero in landscape on notched iPhones

  return (
    <View
      style={{
        flex: 1,
        paddingTop: insets.top,
        paddingBottom: insets.bottom,
        paddingLeft: insets.left,
        paddingRight: insets.right,
      }}
    >
      {children}
    </View>
  );
}
```

</details>

---

## Quick-Reference Cheatsheet

|API|Use case|
|---|---|
|`<SafeAreaProvider>`|Wrap entire app once at the root|
|`<SafeAreaView>`|Screen wrapper — handles all edges|
|`<SafeAreaView edges={['top']}>`|Only pad specific edges|
|`<SafeAreaView mode="margin">`|Inset as margin, not padding|
|`useSafeAreaInsets()`|Get inset numbers for custom layouts|
|`useSafeAreaFrame()`|Get window dimensions (modal-safe)|
|`SafeAreaInsetsContext`|Consume insets in class components|
|`SafeAreaConsumer`|Render prop API for insets|
|`withSafeAreaInsets(Comp)`|HOC for class components|
|`initialWindowMetrics`|Pass to `initialMetrics` to avoid flicker|
|`edges={['bottom']}`|Skip top when navigator handles header|
|`insets.top`|Padding below status bar / notch|
|`insets.bottom`|Padding above home indicator|
|`insets.left/right`|Padding in landscape on notched phones|

---

## Built-in vs Library — Final Recommendation

```jsx
// ❌ Avoid — limited Android support, no hook access, no edge control
import { SafeAreaView } from 'react-native';

// ✅ Use this — full platform support, hooks, edge control
import { SafeAreaProvider, SafeAreaView, useSafeAreaInsets }
  from 'react-native-safe-area-context';
```

---

_Reference based on `react-native-safe-area-context` v4.x. Always check the [official docs](https://docs.expo.dev/versions/latest/sdk/safe-area-context/) for the latest updates._