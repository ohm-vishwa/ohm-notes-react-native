```jsx
<Stack.Screen 
  name="YourScreenName"

  options={{
    // --- Header Content & Text ---
    title: 'Screen Title', // Fallback string for header title and tab/drawer labels
    headerTitle: (props) => <CustomTitleView {...props} />, // String or custom React component for the title
    headerBackTitle: 'Back', // iOS only: Text displayed next to the back arrow
    headerBackTitleVisible: true, // Whether the back button text should be visible
    headerLeft: (props) => <CustomLeftButton {...props} />, // Custom React component for the left side of the header
    headerRight: (props) => <CustomRightButton {...props} />, // Custom React component for the right side of the header
    headerTitleAlign: 'center', // Aligns the header title text ('left' | 'center')

    // --- Header Styling & Visibility ---
    headerShown: true, // Whether to show or hide the header for this screen
    headerStyle: {
      backgroundColor: '#6200ee', // Background color of the header
    },
    headerTintColor: '#ffffff', // Color of the header title text and back button icon
    headerTitleStyle: {
      fontWeight: 'bold', // Typography styles for the header title
      fontSize: 18,
    },
    headerBackTitleStyle: {
      fontSize: 16, // Typography styles for the back button text
    },
    headerTransparent: false, // If true, header floats on top of the screen content with absolute positioning
    headerBackground: () => <CustomBackgroundComponent />, // Custom React element to render behind the header
    headerBlurEffect: 'light', // iOS only: Applies a native blur effect to a transparent header ('extraLight' | 'light' | 'dark' | 'regular' | 'prominent')
    headerShadowVisible: true, // Toggles the bottom border/shadow line beneath the header

    // --- Layout & Window Behavior ---
    presentation: 'card', // Presentation style ('card' | 'modal' | 'transparentModal' | 'containedModal' | 'containedTransparentModal' | 'fullScreenModal')
    orientation: 'default', // Restricts screen orientation ('default' | 'all' | 'portrait' | 'landscape' | 'landscape_left' | 'landscape_right')
    statusBarHidden: false, // Whether to completely hide the device status bar at the top
    statusBarStyle: 'auto', // Color scheme of status bar text/icons ('auto' | 'inverted' | 'light' | 'dark')
    statusBarAnimation: 'fade', // Status bar transition effect ('fade' | 'slide' | 'none')
    contentStyle: {
      backgroundColor: '#f5f5f5', // Styles applied directly to the view wrapper wrapping your screen component
    },

    // --- Animations & Touch Gestures ---
    animation: 'default', // Entry transition style ('default' | 'fade' | 'fade_from_bottom' | 'flip' | 'slide_from_bottom' | 'slide_from_right' | 'slide_from_left' | 'none')
    animationDuration: 350, // Duration of the transition animation in milliseconds
    animationTypeForReplace: 'push', // Animation behavior when replacing a screen in history ('push' | 'pop')
    gestureEnabled: true, // Toggles whether swipe gestures can dismiss the screen
    gestureDirection: 'horizontal', // Sets swipe axis direction (primarily used in JS-based stack: 'horizontal' | 'vertical')

    // --- Native Device Integrations ---
    autoHideHomeIndicator: false, // iOS only: Automatically hides the native home indicator bar at the bottom
    freezeOnBlur: true, // Prevents the screen from re-rendering or consuming memory when in the background
    headerLargeTitle: false, // iOS only: Enables native expanding/collapsing large title layout
    headerLargeTitleStyle: {
      fontSize: 34, // Styles applied explicitly to the iOS native large title
      color: '#000000',
    },
    navigationBarColor: '#000000', // Android only: Background color of Android's system navigation bar (bottom buttons)
    navigationBarHidden: false, // Android only: Whether to hide the bottom system navigation bar
  }} 
/>
```

# React Native Stack Navigator — Complete Reference

Covers both `@react-navigation/stack` (JS-based) and `@react-navigation/native-stack` (native-based).  
Includes Navigator props, Screen props, Screen Options, Navigation methods, and common patterns.

> **Install:**
> 
> ```bash
> npm install @react-navigation/native @react-navigation/native-stack
> # or JS stack:
> npm install @react-navigation/stack
> ```

---

## Table of Contents

1. [Stack.Navigator Props](#stacknavigator-props)
2. [Stack.Screen Props](#stackscreen-props)
3. [Screen Options — Header](#screen-options--header)
4. [Screen Options — Presentation & Animation](#screen-options--presentation--animation)
5. [Screen Options — Gesture](#screen-options--gesture)
6. [Screen Options — Status Bar](#screen-options--status-bar)
7. [Screen Options — Layout & Style](#screen-options--layout--style)
8. [Navigation Object Methods](#navigation-object-methods)
9. [Route Object](#route-object)
10. [Hooks](#hooks)
11. [Common Patterns](#common-patterns)

---

## Stack.Navigator Props

<details> <summary><code>initialRouteName</code> — <em>string</em></summary>

The name of the route to show on first load. Defaults to the first screen defined.

|||
|---|---|
|**Type**|`string`|
|**Default**|First screen in the navigator|

```jsx
<Stack.Navigator initialRouteName="Home">
  <Stack.Screen name="Home" component={HomeScreen} />
  <Stack.Screen name="Details" component={DetailsScreen} />
</Stack.Navigator>
```

</details> <details> <summary><code>screenOptions</code> — <em>object | function</em></summary>

Default options applied to all screens in this navigator. Can be an object or a function receiving `{ route, navigation }`.

|||
|---|---|
|**Type**|`ScreenOptions \| ({ route, navigation }) => ScreenOptions`|
|**Default**|`{}`|

```jsx
<Stack.Navigator
  screenOptions={{
    headerStyle: { backgroundColor: '#6366f1' },
    headerTintColor: '#fff',
    headerTitleStyle: { fontWeight: 'bold' },
  }}
>
  ...
</Stack.Navigator>
```

</details> <details> <summary><code>detachInactiveScreens</code> — <em>boolean</em></summary>

Detaches screens that are not visible to save memory. Requires `react-native-screens`.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|

```jsx
<Stack.Navigator detachInactiveScreens={false}>
  ...
</Stack.Navigator>
```

</details> <details> <summary><code>keyboardHandlingEnabled</code> — <em>boolean</em> · JS Stack only</summary>

If `false`, the keyboard is not dismissed when navigating between screens.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|
|**Navigator**|`@react-navigation/stack` only|

```jsx
<Stack.Navigator keyboardHandlingEnabled={false}>
  ...
</Stack.Navigator>
```

</details> <details> <summary><code>id</code> — <em>string</em></summary>

A unique ID for the navigator. Used to reference it from `navigation.getParent(id)`.

|||
|---|---|
|**Type**|`string`|
|**Default**|`undefined`|

```jsx
<Stack.Navigator id="RootStack">
  ...
</Stack.Navigator>
```

</details>

---

## Stack.Screen Props

<details> <summary><code>name</code> — <em>string</em> ⚠️ Required</summary>

The unique name of the route. Used when navigating with `navigation.navigate('Name')`.

|||
|---|---|
|**Type**|`string`|
|**Required**|Yes|

```jsx
<Stack.Screen name="Profile" component={ProfileScreen} />
```

</details> <details> <summary><code>component</code> — <em>ComponentType</em></summary>

The React component to render for this screen. Receives `navigation` and `route` as props.

|||
|---|---|
|**Type**|`ComponentType`|

```jsx
<Stack.Screen name="Home" component={HomeScreen} />
```

</details> <details> <summary><code>children</code> — <em>function</em></summary>

A render function alternative to `component`. Useful for passing extra props.

|||
|---|---|
|**Type**|`(props) => ReactNode`|

```jsx
<Stack.Screen name="Home">
  {(props) => <HomeScreen {...props} extraData={data} />}
</Stack.Screen>
```

</details> <details> <summary><code>options</code> — <em>object | function</em></summary>

Screen-specific options that override `screenOptions`. Can be a function to access route params.

|||
|---|---|
|**Type**|`ScreenOptions \| ({ route, navigation }) => ScreenOptions`|

```jsx
<Stack.Screen
  name="Details"
  component={DetailsScreen}
  options={({ route }) => ({ title: route.params.title })}
/>
```

</details> <details> <summary><code>initialParams</code> — <em>object</em></summary>

Default params merged with any params passed on navigate. Useful for optional parameters.

|||
|---|---|
|**Type**|`object`|
|**Default**|`undefined`|

```jsx
<Stack.Screen
  name="Profile"
  component={ProfileScreen}
  initialParams={{ userId: 'guest' }}
/>
```

</details> <details> <summary><code>getId</code> — <em>({ params }) =&gt; string</em></summary>

Returns a unique ID for this screen instance. Prevents duplicate screens with the same name in the stack.

|||
|---|---|
|**Type**|`({ params }) => string \| undefined`|
|**Default**|`undefined`|

```jsx
<Stack.Screen
  name="Profile"
  component={ProfileScreen}
  getId={({ params }) => params.userId}
/>
```

</details>

---

## Screen Options — Header

<details> <summary><code>title</code> — <em>string</em></summary>

Sets the text shown in the header. Falls back to the route `name` if not set.

|||
|---|---|
|**Type**|`string`|
|**Default**|Route name|

```jsx
options={{ title: 'My Profile' }}
```

</details> <details> <summary><code>headerShown</code> — <em>boolean</em></summary>

Shows or hides the entire header bar.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|

```jsx
options={{ headerShown: false }}
```

</details> <details> <summary><code>headerTitle</code> — <em>string | function</em></summary>

Custom title for the header. Accepts a string or a component-returning function.

|||
|---|---|
|**Type**|`string \| ({ children, tintColor }) => ReactNode`|

```jsx
options={{
  headerTitle: () => (
    <Image source={logo} style={{ width: 120, height: 30 }} />
  ),
}}
```

</details> <details> <summary><code>headerTitleAlign</code> — <em>enum</em></summary>

Horizontal alignment of the header title.

|||
|---|---|
|**Type**|`enum`|
|**Default**|`'center'` (iOS), `'left'` (Android)|

**Values:** `left`, `center`

```jsx
options={{ headerTitleAlign: 'center' }}
```

</details> <details> <summary><code>headerStyle</code> — <em>StyleProp&lt;ViewStyle&gt;</em></summary>

Style for the header container.

|||
|---|---|
|**Type**|`StyleProp<ViewStyle>`|

```jsx
options={{ headerStyle: { backgroundColor: '#6366f1' } }}
```

</details> <details> <summary><code>headerTitleStyle</code> — <em>StyleProp&lt;TextStyle&gt;</em></summary>

Style for the header title text.

|||
|---|---|
|**Type**|`StyleProp<TextStyle>`|

```jsx
options={{ headerTitleStyle: { fontSize: 18, fontWeight: '700' } }}
```

</details> <details> <summary><code>headerTintColor</code> — <em>color</em></summary>

Color applied to the back button and header title.

|||
|---|---|
|**Type**|`color`|

```jsx
options={{ headerTintColor: '#fff' }}
```

</details> <details> <summary><code>headerLeft</code> — <em>function</em></summary>

Custom component rendered on the left side of the header. Replaces the default back button.

|||
|---|---|
|**Type**|`({ tintColor, canGoBack }) => ReactNode`|

```jsx
options={{
  headerLeft: () => (
    <TouchableOpacity onPress={() => navigation.goBack()}>
      <Text>Cancel</Text>
    </TouchableOpacity>
  ),
}}
```

</details> <details> <summary><code>headerRight</code> — <em>function</em></summary>

Custom component rendered on the right side of the header.

|||
|---|---|
|**Type**|`({ tintColor }) => ReactNode`|

```jsx
options={{
  headerRight: () => (
    <TouchableOpacity onPress={handleSave}>
      <Text style={{ color: '#fff' }}>Save</Text>
    </TouchableOpacity>
  ),
}}
```

</details> <details> <summary><code>headerBackground</code> — <em>function</em></summary>

Renders a custom background behind the header content. Useful for blur effects.

|||
|---|---|
|**Type**|`() => ReactNode`|

```jsx
options={{
  headerTransparent: true,
  headerBackground: () => (
    <BlurView intensity={80} style={StyleSheet.absoluteFill} />
  ),
}}
```

</details> <details> <summary><code>headerBackTitle</code> — <em>string</em> · 🍎 iOS</summary>

Label shown next to the back arrow on iOS. Set to `''` to show only the arrow.

|||
|---|---|
|**Type**|`string`|
|**Default**|Previous screen title|
|**Platform**|iOS|

```jsx
options={{ headerBackTitle: 'Back' }}
```

</details> <details> <summary><code>headerBackTitleVisible</code> — <em>boolean</em> · 🍎 iOS</summary>

Whether to show the back title text next to the back arrow on iOS.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|
|**Platform**|iOS|

```jsx
options={{ headerBackTitleVisible: false }}
```

</details> <details> <summary><code>headerBackTitleStyle</code> — <em>StyleProp&lt;TextStyle&gt;</em> · 🍎 iOS</summary>

Style for the back button title on iOS.

|||
|---|---|
|**Type**|`StyleProp<TextStyle>`|
|**Platform**|iOS|

```jsx
options={{ headerBackTitleStyle: { fontSize: 14, color: '#fff' } }}
```

</details> <details> <summary><code>headerBackVisible</code> — <em>boolean</em> · Native Stack</summary>

Whether the back button is visible in the header.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true` if there is a screen to go back to|
|**Navigator**|`@react-navigation/native-stack`|

```jsx
options={{ headerBackVisible: false }}
```

</details> <details> <summary><code>headerBackButtonMenuEnabled</code> — <em>boolean</em> · 🍎 iOS · Native Stack</summary>

Enables long-press on the back button to show a menu of previous screens.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|
|**Platform**|iOS|

```jsx
options={{ headerBackButtonMenuEnabled: false }}
```

</details> <details> <summary><code>headerBackImageSource</code> — <em>ImageSource</em> · JS Stack</summary>

Custom image for the back button arrow.

|||
|---|---|
|**Type**|`ImageSourcePropType`|
|**Navigator**|`@react-navigation/stack` only|

```jsx
options={{ headerBackImageSource: require('./back-arrow.png') }}
```

</details> <details> <summary><code>headerShadowVisible</code> — <em>boolean</em></summary>

Shows or hides the shadow/border at the bottom of the header.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|

```jsx
options={{ headerShadowVisible: false }}
```

</details> <details> <summary><code>headerTransparent</code> — <em>boolean</em></summary>

Makes the header background transparent so content scrolls underneath it.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```jsx
options={{ headerTransparent: true }}
```

</details> <details> <summary><code>headerBlurEffect</code> — <em>enum</em> · 🍎 iOS · Native Stack</summary>

Applies a native blur behind the transparent header. Requires `headerTransparent: true`.

|||
|---|---|
|**Type**|`enum`|
|**Platform**|iOS|
|**Navigator**|`@react-navigation/native-stack`|

**Values:** `none`, `extraLight`, `light`, `dark`, `regular`, `prominent`, `systemUltraThinMaterial`, `systemThinMaterial`, `systemMaterial`, `systemThickMaterial`, `systemChromeMaterial`

```jsx
options={{ headerTransparent: true, headerBlurEffect: 'systemMaterial' }}
```

</details> <details> <summary><code>headerLargeTitle</code> — <em>boolean</em> · 🍎 iOS · Native Stack</summary>

Enables iOS large title style (the big collapsing title that shrinks on scroll).

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|
|**Platform**|iOS|
|**Navigator**|`@react-navigation/native-stack`|

```jsx
options={{ headerLargeTitle: true, title: 'Inbox' }}
```

</details> <details> <summary><code>headerLargeTitleStyle</code> — <em>StyleProp&lt;TextStyle&gt;</em> · 🍎 iOS · Native Stack</summary>

Style for the large title text when `headerLargeTitle` is `true`.

|||
|---|---|
|**Type**|`StyleProp<TextStyle>`|
|**Platform**|iOS|

```jsx
options={{ headerLargeTitleStyle: { fontFamily: 'Inter-Bold' } }}
```

</details> <details> <summary><code>headerSearchBarOptions</code> — <em>object</em> · Native Stack</summary>

Adds a native search bar into the header. iOS uses `UISearchController`; Android uses the action bar search.

|||
|---|---|
|**Type**|`{ placeholder?, onChangeText?, onClose?, autoCapitalize?, ... }`|
|**Navigator**|`@react-navigation/native-stack`|

```jsx
options={{
  headerSearchBarOptions: {
    placeholder: 'Search…',
    onChangeText: (e) => setQuery(e.nativeEvent.text),
  },
}}
```

</details>

---

## Screen Options — Presentation & Animation

<details> <summary><code>presentation</code> — <em>enum</em> · Native Stack</summary>

Controls how the screen is presented.

|||
|---|---|
|**Type**|`enum`|
|**Default**|`'card'`|
|**Navigator**|`@react-navigation/native-stack`|

**Values:** `card`, `modal`, `transparentModal`, `containedModal` _(iOS)_, `containedTransparentModal` _(iOS)_, `fullScreenModal` _(iOS)_, `formSheet` _(iOS)_

```jsx
options={{ presentation: 'modal' }}
```

</details> <details> <summary><code>animation</code> — <em>enum</em> · Native Stack</summary>

Controls the transition animation when the screen mounts.

|||
|---|---|
|**Type**|`enum`|
|**Default**|`'default'`|
|**Navigator**|`@react-navigation/native-stack`|

**Values:** `default`, `fade`, `fade_from_bottom`, `flip` _(iOS)_, `simple_push` _(iOS)_, `slide_from_bottom`, `slide_from_right`, `slide_from_left`, `ios_from_right`, `ios_from_left`, `none`

```jsx
options={{ animation: 'slide_from_bottom' }}
```

</details> <details> <summary><code>animationDuration</code> — <em>number</em> · 🤖 Android · Native Stack</summary>

Duration of the screen transition in milliseconds on Android.

|||
|---|---|
|**Type**|`number`|
|**Default**|Platform default|
|**Platform**|Android|

```jsx
options={{ animationDuration: 300 }}
```

</details> <details> <summary><code>animationTypeForReplace</code> — <em>enum</em></summary>

Animation direction when using `navigation.replace()`.

|||
|---|---|
|**Type**|`'push' \| 'pop'`|
|**Default**|`'pop'`|

```jsx
options={{ animationTypeForReplace: 'push' }}
```

</details> <details> <summary><code>cardStyleInterpolator</code> — <em>function</em> · JS Stack</summary>

Custom interpolation for the card animation. React Navigation ships several presets.

|||
|---|---|
|**Type**|`(props: StackCardInterpolationProps) => StackCardInterpolatedStyle`|
|**Navigator**|`@react-navigation/stack` only|

```jsx
import { CardStyleInterpolators } from '@react-navigation/stack';

options={{ cardStyleInterpolator: CardStyleInterpolators.forModalPresentationIOS }}
```

</details> <details> <summary><code>transitionSpec</code> — <em>object</em> · JS Stack</summary>

Full control over the transition animation timing (open and close).

|||
|---|---|
|**Type**|`{ open: TransitionSpec, close: TransitionSpec }`|
|**Navigator**|`@react-navigation/stack` only|

```jsx
import { TransitionSpecs } from '@react-navigation/stack';

options={{
  transitionSpec: {
    open: TransitionSpecs.TransitionIOSSpec,
    close: TransitionSpecs.TransitionIOSSpec,
  },
}}
```

</details> <details> <summary><code>freezeOnBlur</code> — <em>boolean</em></summary>

Prevents screens from re-rendering while they are out of focus.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```jsx
options={{ freezeOnBlur: true }}
```

</details> <details> <summary><code>detachPreviousScreen</code> — <em>boolean</em> · JS Stack</summary>

Detaches the previous screen from the view hierarchy when not visible, saving memory.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true` for the last screen, `false` otherwise|
|**Navigator**|`@react-navigation/stack` only|

```jsx
options={{ detachPreviousScreen: false }}
```

</details>

---

## Screen Options — Gesture

<details> <summary><code>gestureEnabled</code> — <em>boolean</em></summary>

Enables or disables the swipe-back gesture to dismiss the screen.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true` (iOS), `false` (Android)|

```jsx
options={{ gestureEnabled: false }}
```

</details> <details> <summary><code>gestureDirection</code> — <em>enum</em> · JS Stack</summary>

Direction of the swipe-to-dismiss gesture.

|||
|---|---|
|**Type**|`enum`|
|**Default**|`'horizontal'`|
|**Navigator**|`@react-navigation/stack` only|

**Values:** `horizontal`, `horizontal-inverted`, `vertical`, `vertical-inverted`

```jsx
options={{ gestureDirection: 'vertical' }}
```

</details> <details> <summary><code>gestureResponseDistance</code> — <em>number</em></summary>

Edge area width/height in pixels from which the swipe gesture is recognized.

|||
|---|---|
|**Type**|`number`|
|**Default**|`25` (horizontal), `135` (vertical)|

```jsx
options={{ gestureResponseDistance: 50 }}
```

</details> <details> <summary><code>fullScreenGestureEnabled</code> — <em>boolean</em> · 🍎 iOS · Native Stack</summary>

Allows swiping from anywhere on the screen (not just the edge) to go back.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|
|**Platform**|iOS|
|**Navigator**|`@react-navigation/native-stack`|

```jsx
options={{ fullScreenGestureEnabled: true }}
```

</details> <details> <summary><code>customAnimationOnGesture</code> — <em>boolean</em> · JS Stack</summary>

If `true`, uses the custom `cardStyleInterpolator` animation even during gesture dismissal.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|
|**Navigator**|`@react-navigation/stack` only|

```jsx
options={{ customAnimationOnGesture: true }}
```

</details>

---

## Screen Options — Status Bar

<details> <summary><code>statusBarStyle</code> — <em>enum</em> · Native Stack</summary>

Style of the status bar text and icons for this screen.

|||
|---|---|
|**Type**|`enum`|
|**Default**|`'auto'`|
|**Navigator**|`@react-navigation/native-stack`|

**Values:** `auto`, `inverted`, `light`, `dark`

```jsx
options={{ statusBarStyle: 'light' }}
```

</details> <details> <summary><code>statusBarColor</code> — <em>color</em> · 🤖 Android · Native Stack</summary>

Background color of the status bar on Android.

|||
|---|---|
|**Type**|`color`|
|**Platform**|Android|

```jsx
options={{ statusBarColor: '#6366f1' }}
```

</details> <details> <summary><code>statusBarHidden</code> — <em>boolean</em> · Native Stack</summary>

Hides the status bar for this screen.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```jsx
options={{ statusBarHidden: true }}
```

</details> <details> <summary><code>statusBarAnimation</code> — <em>enum</em> · Native Stack</summary>

Animation when the status bar changes state.

|||
|---|---|
|**Type**|`'none' \| 'fade' \| 'slide'`|
|**Default**|`'fade'`|

```jsx
options={{ statusBarAnimation: 'slide' }}
```

</details> <details> <summary><code>statusBarTranslucent</code> — <em>boolean</em> · 🤖 Android · Native Stack</summary>

Makes the status bar translucent on Android — content renders behind it.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|
|**Platform**|Android|

```jsx
options={{ statusBarTranslucent: true }}
```

</details>

---

## Screen Options — Layout & Style

<details> <summary><code>cardStyle</code> — <em>StyleProp&lt;ViewStyle&gt;</em> · JS Stack</summary>

Style for the card (the screen container) — useful for background color.

|||
|---|---|
|**Type**|`StyleProp<ViewStyle>`|
|**Navigator**|`@react-navigation/stack` only|

```jsx
options={{ cardStyle: { backgroundColor: '#fff' } }}
```

</details> <details> <summary><code>contentStyle</code> — <em>StyleProp&lt;ViewStyle&gt;</em> · Native Stack</summary>

Style for the screen content area below the header.

|||
|---|---|
|**Type**|`StyleProp<ViewStyle>`|
|**Navigator**|`@react-navigation/native-stack`|

```jsx
options={{ contentStyle: { backgroundColor: '#f9f9f9' } }}
```

</details> <details> <summary><code>cardOverlayEnabled</code> — <em>boolean</em> · JS Stack</summary>

Shows a semi-transparent overlay on screens below the current one (used in modal presentation).

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false` (card), `true` (modal)|
|**Navigator**|`@react-navigation/stack` only|

```jsx
options={{ cardOverlayEnabled: true }}
```

</details> <details> <summary><code>cardShadowEnabled</code> — <em>boolean</em> · JS Stack</summary>

Enables the shadow behind the card during transition.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|
|**Navigator**|`@react-navigation/stack` only|

```jsx
options={{ cardShadowEnabled: false }}
```

</details> <details> <summary><code>orientation</code> — <em>enum</em> · Native Stack</summary>

Locks screen orientation for this specific screen.

|||
|---|---|
|**Type**|`enum`|
|**Default**|`'default'`|
|**Navigator**|`@react-navigation/native-stack`|

**Values:** `default`, `all`, `portrait`, `portrait_up`, `portrait_down`, `landscape`, `landscape_left`, `landscape_right`

```jsx
options={{ orientation: 'landscape' }}
```

</details>

---

## Navigation Object Methods

<details> <summary><code>navigation.navigate(name, params?)</code></summary>

Navigate to a route. If the screen already exists in the stack, navigates back to it. Use `push` to always add a new screen.

```jsx
// Simple
navigation.navigate('Details');

// With params
navigation.navigate('Details', { itemId: 42, title: 'My Item' });

// Nested navigator
navigation.navigate('HomeStack', { screen: 'Profile', params: { userId: '123' } });
```

</details> <details> <summary><code>navigation.push(name, params?)</code></summary>

Always pushes a new screen onto the stack, even if a screen with that name already exists.

```jsx
navigation.push('Details', { itemId: 99 });
```

</details> <details> <summary><code>navigation.pop(count?)</code></summary>

Goes back `count` screens in the stack. Defaults to 1.

```jsx
navigation.pop();      // go back 1
navigation.pop(3);     // go back 3 screens
```

</details> <details> <summary><code>navigation.popTo(name, params?)</code></summary>

Pops back to the first screen in the stack with the given name.

```jsx
navigation.popTo('Home');
```

</details> <details> <summary><code>navigation.popToTop()</code></summary>

Pops all screens in the stack and returns to the first screen.

```jsx
navigation.popToTop();
```

</details> <details> <summary><code>navigation.replace(name, params?)</code></summary>

Replaces the current screen with a new one (no back navigation to the replaced screen).

```jsx
navigation.replace('Home', { resetData: true });
```

</details> <details> <summary><code>navigation.reset(state)</code></summary>

Resets the entire navigation state to the given object. Useful for post-login redirect.

```jsx
navigation.reset({
  index: 0,
  routes: [{ name: 'Home' }],
});
```

</details> <details> <summary><code>navigation.goBack()</code></summary>

Goes back to the previous screen. Safe to call from any screen.

```jsx
navigation.goBack();
```

</details> <details> <summary><code>navigation.setOptions(options)</code></summary>

Dynamically updates the screen options from inside the component (e.g. update the title based on state).

```jsx
useEffect(() => {
  navigation.setOptions({ title: user.name });
}, [navigation, user.name]);
```

</details> <details> <summary><code>navigation.setParams(params)</code></summary>

Updates the params for the current route.

```jsx
navigation.setParams({ title: 'Updated Title', count: 5 });
```

</details> <details> <summary><code>navigation.dispatch(action)</code></summary>

Dispatches a navigation action object directly. Useful for advanced navigation logic.

```jsx
import { CommonActions } from '@react-navigation/native';

navigation.dispatch(
  CommonActions.reset({ index: 0, routes: [{ name: 'Home' }] })
);
```

</details> <details> <summary><code>navigation.addListener(event, callback)</code></summary>

Subscribes to navigation lifecycle events on the current screen.

```jsx
useEffect(() => {
  const unsubscribe = navigation.addListener('focus', () => {
    fetchData();
  });
  return unsubscribe;
}, [navigation]);
```

**Events:** `focus`, `blur`, `beforeRemove`, `state`, `transitionStart`, `transitionEnd`, `gestureStart`, `gestureEnd`, `gestureCancel`

</details> <details> <summary><code>navigation.isFocused()</code></summary>

Returns `true` if the current screen is focused. Use the `useFocusEffect` hook for reactive behavior instead.

```jsx
if (navigation.isFocused()) {
  doSomething();
}
```

</details> <details> <summary><code>navigation.canGoBack()</code></summary>

Returns `true` if there is a screen to go back to in the stack.

```jsx
if (navigation.canGoBack()) {
  navigation.goBack();
}
```

</details> <details> <summary><code>navigation.getState()</code></summary>

Returns the current navigation state of the navigator containing this screen.

```jsx
const state = navigation.getState();
console.log(state.routes, state.index);
```

</details> <details> <summary><code>navigation.getParent(id?)</code></summary>

Returns the navigation object of the parent navigator. Pass an `id` to target a specific ancestor.

```jsx
navigation.getParent()?.navigate('Modal');
navigation.getParent('RootStack')?.popToTop();
```

</details>

---

## Route Object

<details> <summary><code>route.name</code> — <em>string</em></summary>

The name of the current screen/route as defined in `Stack.Screen name=`.

```jsx
function MyScreen({ route }) {
  console.log(route.name); // e.g. "Details"
}
```

</details> <details> <summary><code>route.key</code> — <em>string</em></summary>

A unique key auto-generated by React Navigation for this screen instance.

```jsx
console.log(route.key); // e.g. "Details-abc123"
```

</details> <details> <summary><code>route.params</code> — <em>object</em></summary>

Parameters passed to this screen via `navigation.navigate('Name', params)`.

```jsx
function DetailsScreen({ route }) {
  const { itemId, title } = route.params;
  return <Text>{title}</Text>;
}
```

</details>

---

## Hooks

<details> <summary><code>useNavigation()</code></summary>

Returns the `navigation` object from any component without needing to pass it as a prop.

```jsx
import { useNavigation } from '@react-navigation/native';

function MyButton() {
  const navigation = useNavigation();
  return <Button title="Go" onPress={() => navigation.navigate('Home')} />;
}
```

</details> <details> <summary><code>useRoute()</code></summary>

Returns the `route` object from any component without prop drilling.

```jsx
import { useRoute } from '@react-navigation/native';

function MyComponent() {
  const route = useRoute();
  return <Text>{route.params?.title}</Text>;
}
```

</details> <details> <summary><code>useFocusEffect(callback)</code></summary>

Runs a side effect when the screen comes into focus. Cleanup runs on blur. Replaces `useEffect` + focus listener.

```jsx
import { useFocusEffect } from '@react-navigation/native';

useFocusEffect(
  useCallback(() => {
    fetchData();
    return () => cancelRequests();
  }, [])
);
```

</details> <details> <summary><code>useIsFocused()</code></summary>

Returns a boolean that is `true` when the screen is focused. Triggers re-render on focus/blur.

```jsx
import { useIsFocused } from '@react-navigation/native';

const isFocused = useIsFocused();
return <Text>{isFocused ? 'Focused' : 'Not focused'}</Text>;
```

</details> <details> <summary><code>useNavigationState(selector)</code></summary>

Subscribes to navigation state changes and returns the selected slice. Avoids full re-renders.

```jsx
import { useNavigationState } from '@react-navigation/native';

const routeCount = useNavigationState(state => state.routes.length);
```

</details> <details> <summary><code>usePreventRemove(shouldPrevent, callback)</code></summary>

Prevents the user from leaving the screen (e.g. unsaved form). Replaces the `beforeRemove` listener pattern.

```jsx
import { usePreventRemove } from '@react-navigation/native';

usePreventRemove(hasUnsavedChanges, ({ data }) => {
  Alert.alert('Discard changes?', '', [
    { text: 'Cancel' },
    { text: 'Discard', onPress: () => navigation.dispatch(data.action) },
  ]);
});
```

</details>

---

## Common Patterns

<details> <summary>Basic stack setup</summary>

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

</details> <details> <summary>Pass and read params between screens</summary>

```jsx
// Sender
navigation.navigate('Details', { itemId: 42, title: 'Item Name' });

// Receiver
function DetailsScreen({ route }) {
  const { itemId, title } = route.params;
  return <Text>{title} — ID: {itemId}</Text>;
}
```

</details> <details> <summary>Update header title dynamically from inside a screen</summary>

```jsx
function ProfileScreen({ navigation, route }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    loadUser().then(u => {
      setUser(u);
      navigation.setOptions({ title: u.name });
    });
  }, [navigation]);

  return <Text>{user?.bio}</Text>;
}
```

</details> <details> <summary>Add a button to the header from inside a screen</summary>

```jsx
function EditScreen({ navigation }) {
  const handleSave = () => { /* ... */ };

  useLayoutEffect(() => {
    navigation.setOptions({
      headerRight: () => (
        <TouchableOpacity onPress={handleSave}>
          <Text style={{ color: '#6366f1', fontWeight: '600' }}>Save</Text>
        </TouchableOpacity>
      ),
    });
  }, [navigation]);

  return <View>...</View>;
}
```

</details> <details> <summary>Modal screen (slides up from bottom)</summary>

```jsx
// Native Stack
<Stack.Screen
  name="CreatePost"
  component={CreatePostScreen}
  options={{ presentation: 'modal' }}
/>

// JS Stack — wrap in a group
<Stack.Group screenOptions={{ presentation: 'modal' }}>
  <Stack.Screen name="CreatePost" component={CreatePostScreen} />
  <Stack.Screen name="Filters" component={FiltersScreen} />
</Stack.Group>
```

</details> <details> <summary>Prevent leaving a screen with unsaved changes</summary>

```jsx
function EditScreen({ navigation }) {
  const [hasChanges, setHasChanges] = useState(false);

  usePreventRemove(hasChanges, ({ data }) => {
    Alert.alert(
      'Discard changes?',
      'You have unsaved changes.',
      [
        { text: 'Stay', style: 'cancel' },
        {
          text: 'Discard',
          style: 'destructive',
          onPress: () => navigation.dispatch(data.action),
        },
      ]
    );
  });

  return <TextInput onChangeText={() => setHasChanges(true)} />;
}
```

</details> <details> <summary>Authentication flow — reset stack after login</summary>

```jsx
function LoginScreen({ navigation }) {
  const handleLogin = async () => {
    await signIn();
    navigation.reset({
      index: 0,
      routes: [{ name: 'Home' }],
    });
  };

  return <Button title="Login" onPress={handleLogin} />;
}
```

</details> <details> <summary>Transparent / blurred header with large title (iOS)</summary>

```jsx
<Stack.Screen
  name="Feed"
  component={FeedScreen}
  options={{
    headerLargeTitle: true,
    headerTransparent: true,
    headerBlurEffect: 'systemMaterial',
    headerShadowVisible: false,
  }}
/>
```

</details> <details> <summary>Deep link to a nested screen</summary>

```jsx
// linking config
const linking = {
  prefixes: ['myapp://'],
  config: {
    screens: {
      Home: 'home',
      Details: 'details/:itemId',
      Profile: {
        path: 'user/:userId',
        screens: {
          Posts: 'posts',
        },
      },
    },
  },
};

<NavigationContainer linking={linking}>
  ...
</NavigationContainer>
```

</details> <details> <summary>Full custom header</summary>

```jsx
<Stack.Screen
  name="Home"
  component={HomeScreen}
  options={{
    headerShown: true,
    header: ({ navigation, route, options, back }) => (
      <View style={styles.customHeader}>
        {back && (
          <TouchableOpacity onPress={navigation.goBack}>
            <Ionicons name="arrow-back" size={24} />
          </TouchableOpacity>
        )}
        <Text style={styles.title}>{options.title ?? route.name}</Text>
      </View>
    ),
  }}
/>
```

</details>

---

## JS Stack vs Native Stack — Quick Comparison

|Feature|`@react-navigation/stack`|`@react-navigation/native-stack`|
|---|---|---|
|Implementation|JS (Reanimated)|Native (UINavigationController / Fragment)|
|Performance|Good|Better (native thread)|
|Custom animations|✅ Full control|⚠️ Limited|
|`presentation: 'modal'`|✅|✅|
|`headerLargeTitle`|❌|✅ iOS|
|`headerBlurEffect`|❌|✅ iOS|
|`fullScreenGestureEnabled`|❌|✅ iOS|
|`cardStyleInterpolator`|✅|❌|
|Recommended for|Custom transitions|Most production apps|

---

_Reference based on React Navigation v6/v7. Always check the [official docs](https://reactnavigation.org/docs/stack-navigator) for the latest updates._