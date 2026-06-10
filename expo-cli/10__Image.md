# React Native Image — Complete Reference

A comprehensive reference for the `Image` component and `ImageBackground` in React Native.  
Covers all props, source object fields, static methods, resize modes, and common patterns.

---

## Table of Contents

1. [Core Props](#core-props)
2. [Source Object Fields](#source-object-fields)
3. [Resize Mode & Scaling](#resize-mode--scaling)
4. [Loading & Error Events](#loading--error-events)
5. [Style & Visual](#style--visual)
6. [Accessibility](#accessibility)
7. [iOS-Only Props](#ios-only-props)
8. [Android-Only Props](#android-only-props)
9. [Static Methods](#static-methods)
10. [ImageBackground](#imagebackground)
11. [Common Patterns](#common-patterns)

---

## Core Props

<details> <summary><code>source</code> — <em>ImageSource</em> ⚠️ Required</summary>

The image source. Accepts a local asset require, a URI object, an array of URIs, or `null`.

|||
|---|---|
|**Type**|`ImageSourcePropType`|
|**Required**|Yes|

```jsx
// Local asset
<Image source={require('./assets/logo.png')} />

// Remote URI
<Image source={{ uri: 'https://example.com/photo.jpg' }} />

// Array of URIs (picks best based on size)
<Image source={[
  { uri: 'https://example.com/small.jpg', width: 320, height: 240 },
  { uri: 'https://example.com/large.jpg', width: 640, height: 480 },
]} />
```

</details> <details> <summary><code>defaultSource</code> — <em>ImageSource</em></summary>

Displayed while the remote image is loading. Acts as a placeholder. Only supported with static local assets on Android.

|||
|---|---|
|**Type**|`ImageSourcePropType`|
|**Default**|`undefined`|

```jsx
<Image
  source={{ uri: 'https://example.com/photo.jpg' }}
  defaultSource={require('./assets/placeholder.png')}
/>
```

</details> <details> <summary><code>loadingIndicatorSource</code> — <em>ImageSource</em></summary>

Similar to `defaultSource` — shown while the main image loads. Must be a local asset.

|||
|---|---|
|**Type**|`ImageSourcePropType`|
|**Default**|`undefined`|

```jsx
<Image
  source={{ uri: remoteUrl }}
  loadingIndicatorSource={require('./spinner.gif')}
/>
```

</details> <details> <summary><code>style</code> — <em>StyleProp&lt;ImageStyle&gt;</em></summary>

Style for the image. **You must specify `width` and `height`** (or use `flex`) for the image to render.

|||
|---|---|
|**Type**|`StyleProp<ImageStyle>`|
|**Default**|`{}`|

```jsx
<Image
  source={{ uri: url }}
  style={{ width: 200, height: 150, borderRadius: 8 }}
/>
```

</details> <details> <summary><code>resizeMode</code> — <em>enum</em></summary>

How the image is resized to fit its container. See [Resize Mode & Scaling](#resize-mode--scaling) for visual explanation.

|||
|---|---|
|**Type**|`enum`|
|**Default**|`'cover'`|

**Values:** `cover`, `contain`, `stretch`, `repeat`, `center`

```jsx
<Image source={source} style={{ width: 200, height: 150 }} resizeMode="contain" />
```

</details> <details> <summary><code>alt</code> — <em>string</em></summary>

Alternative text for the image. Sets `accessibilityLabel` and marks the element as accessible. Preferred over `accessibilityLabel` for images.

|||
|---|---|
|**Type**|`string`|
|**Default**|`undefined`|

```jsx
<Image source={avatarSource} alt="User profile photo" />
```

</details> <details> <summary><code>tintColor</code> — <em>color</em></summary>

Tints all non-transparent pixels to the given color. Commonly used to colorize monochrome icons.

|||
|---|---|
|**Type**|`color`|
|**Default**|`undefined`|

```jsx
<Image
  source={require('./icons/heart.png')}
  tintColor="#e11d48"
  style={{ width: 24, height: 24 }}
/>
```

</details> <details> <summary><code>blurRadius</code> — <em>number</em></summary>

Adds a blur filter to the rendered image. Useful for background blurs or obscuring sensitive content.

|||
|---|---|
|**Type**|`number`|
|**Default**|`0`|

```jsx
<Image
  source={{ uri: backgroundUrl }}
  blurRadius={10}
  style={StyleSheet.absoluteFillObject}
/>
```

</details> <details> <summary><code>fadeDuration</code> — <em>number</em> · 🤖 Android</summary>

Duration of the fade-in animation when a remote image loads (in ms). Set to `0` to disable fading.

|||
|---|---|
|**Type**|`number`|
|**Default**|`300`|
|**Platform**|Android|

```jsx
<Image source={{ uri: url }} fadeDuration={0} />
```

</details> <details> <summary><code>testID</code> — <em>string</em></summary>

Used to locate this component in end-to-end tests.

|||
|---|---|
|**Type**|`string`|
|**Default**|`undefined`|

```jsx
<Image source={source} testID="profile-avatar" />
```

</details> <details> <summary><code>nativeID</code> — <em>string</em></summary>

Used to reference the component from native code.

|||
|---|---|
|**Type**|`string`|
|**Default**|`undefined`|

```jsx
<Image source={source} nativeID="heroImage" />
```

</details> <details> <summary><code>crossOrigin</code> — <em>enum</em></summary>

CORS policy for remote image requests (Web compatibility / Fabric).

|||
|---|---|
|**Type**|`'anonymous' \| 'use-credentials'`|
|**Default**|`undefined`|

```jsx
<Image source={{ uri: url }} crossOrigin="anonymous" />
```

</details> <details> <summary><code>referrerPolicy</code> — <em>enum</em></summary>

HTTP `Referrer-Policy` header when requesting the image.

|||
|---|---|
|**Type**|`'no-referrer' \| 'no-referrer-when-downgrade' \| 'origin' \| 'origin-when-cross-origin' \| 'same-origin' \| 'strict-origin' \| 'strict-origin-when-cross-origin' \| 'unsafe-url'`|
|**Default**|`'strict-origin-when-cross-origin'`|

```jsx
<Image source={{ uri: url }} referrerPolicy="no-referrer" />
```

</details>

---

## Source Object Fields

When using a URI-based source (`source={{ ... }}`), the following fields are available:

<details> <summary><code>uri</code> — <em>string</em> ⚠️ Required for remote images</summary>

The URL or file path of the image.

|||
|---|---|
|**Type**|`string`|

```jsx
// Remote URL
<Image source={{ uri: 'https://example.com/image.jpg' }} />

// Local file path
<Image source={{ uri: `file://${filePath}` }} />

// Data URI
<Image source={{ uri: `data:image/png;base64,${base64Data}` }} />
```

</details> <details> <summary><code>width</code> and <code>height</code> — <em>number</em></summary>

Intrinsic dimensions of the image at the specified URI. Used for source selection in an array of sources.

|||
|---|---|
|**Type**|`number`|

```jsx
<Image source={[
  { uri: 'https://example.com/image@1x.jpg', width: 320, height: 240 },
  { uri: 'https://example.com/image@2x.jpg', width: 640, height: 480 },
]} style={{ width: 320, height: 240 }} />
```

</details> <details> <summary><code>scale</code> — <em>number</em></summary>

The scale factor of the image. A `scale: 2` image will be drawn at half size, like `@2x` retina assets.

|||
|---|---|
|**Type**|`number`|
|**Default**|`1`|

```jsx
<Image source={{ uri: url, scale: 2 }} style={{ width: 100, height: 100 }} />
```

</details> <details> <summary><code>headers</code> — <em>object</em></summary>

Custom HTTP headers to send with the image request. Useful for authenticated image endpoints.

|||
|---|---|
|**Type**|`{ [key: string]: string }`|

```jsx
<Image
  source={{
    uri: 'https://api.example.com/private-image.jpg',
    headers: { Authorization: `Bearer ${token}` },
  }}
/>
```

</details> <details> <summary><code>body</code> — <em>string</em></summary>

HTTP body to send with the request. Used for POST-based image endpoints.

|||
|---|---|
|**Type**|`string`|

```jsx
<Image source={{ uri: url, method: 'POST', body: JSON.stringify({ id: 1 }) }} />
```

</details> <details> <summary><code>method</code> — <em>string</em></summary>

HTTP method for the image request. Defaults to `GET`.

|||
|---|---|
|**Type**|`string`|
|**Default**|`'GET'`|

```jsx
<Image source={{ uri: url, method: 'POST', headers: { 'Content-Type': 'application/json' } }} />
```

</details> <details> <summary><code>cache</code> — <em>enum</em> · 🍎 iOS</summary>

Controls the local caching strategy for the image request.

|||
|---|---|
|**Type**|`enum`|
|**Default**|`'default'`|
|**Platform**|iOS|

**Values:**

- `default` — Use the native platform cache policy
- `reload` — Always fetch from the network, ignoring cache
- `force-cache` — Use cached version if available, regardless of age
- `only-if-cached` — Only use cache; if not cached, fail silently

```jsx
<Image source={{ uri: url, cache: 'force-cache' }} />
```

</details>

---

## Resize Mode & Scaling

<details> <summary><code>cover</code> — scales to fill the frame, cropping excess</summary>

Scales the image up or down uniformly so that it covers the entire frame. The image may be cropped on the sides or top/bottom.

**Best for:** Profile avatars, hero images, thumbnails.

```jsx
<Image
  source={{ uri: url }}
  style={{ width: 200, height: 200 }}
  resizeMode="cover"
/>
```

```
Container: 200×200
Image: 300×200  →  fills width, crops nothing
Image: 300×500  →  fills height, crops left/right
```

</details> <details> <summary><code>contain</code> — fits the entire image inside the frame, with letterboxing</summary>

Scales the image so the entire image fits inside the frame without cropping. May leave empty space on sides or top/bottom.

**Best for:** Product images, logos, icons where full visibility matters.

```jsx
<Image
  source={{ uri: url }}
  style={{ width: 200, height: 200 }}
  resizeMode="contain"
/>
```

</details> <details> <summary><code>stretch</code> — stretches independently on each axis, may distort</summary>

Stretches the image to exactly fill the frame, ignoring aspect ratio.

**Best for:** Backgrounds that tile or when distortion is acceptable.

```jsx
<Image
  source={{ uri: url }}
  style={{ width: 200, height: 200 }}
  resizeMode="stretch"
/>
```

</details> <details> <summary><code>repeat</code> — tiles the image to fill the frame · 🍎 iOS</summary>

Tiles the image repeatedly to fill the frame. On Android, behaves like `stretch`.

**Best for:** Pattern/texture backgrounds.

```jsx
<Image
  source={require('./pattern.png')}
  style={{ width: '100%', height: 300 }}
  resizeMode="repeat"
/>
```

</details> <details> <summary><code>center</code> — centers the image without scaling, clips if larger</summary>

Centers the image without any scaling. If the image is larger than the container, it is clipped.

**Best for:** Small icons that shouldn't be scaled.

```jsx
<Image
  source={require('./icon.png')}
  style={{ width: 100, height: 100 }}
  resizeMode="center"
/>
```

</details> <details> <summary><code>resizeMethod</code> — <em>enum</em> · 🤖 Android</summary>

The algorithm used to resize the image when the displayed size differs from the image dimensions. Affects performance and quality.

|||
|---|---|
|**Type**|`enum`|
|**Default**|`'auto'`|
|**Platform**|Android|

**Values:**

- `auto` — Heuristic: uses `resize` for smaller targets, `scale` for larger
- `resize` — Software resize before decoding — use for large images to avoid OOM
- `scale` — Scales the canvas at draw time — faster but uses more memory

```jsx
<Image
  source={{ uri: largeImageUrl }}
  style={{ width: 100, height: 100 }}
  resizeMethod="resize"
/>
```

</details>

---

## Loading & Error Events

<details> <summary><code>onLoadStart</code> — <em>() =&gt; void</em></summary>

Called when the image starts loading. Use to show a spinner.

|||
|---|---|
|**Type**|`() => void`|

```jsx
<Image
  source={{ uri: url }}
  onLoadStart={() => setLoading(true)}
/>
```

</details> <details> <summary><code>onLoad</code> — <em>(event) =&gt; void</em></summary>

Called when the image successfully loads. The event includes the image's natural `width` and `height`.

|||
|---|---|
|**Type**|`(event: ImageLoadEvent) => void`|

```jsx
<Image
  source={{ uri: url }}
  onLoad={(e) => {
    const { width, height } = e.nativeEvent.source;
    console.log(`Loaded: ${width}×${height}`);
    setLoading(false);
  }}
/>
```

</details> <details> <summary><code>onLoadEnd</code> — <em>() =&gt; void</em></summary>

Called when the load completes — whether it succeeded or failed. Good for hiding spinners.

|||
|---|---|
|**Type**|`() => void`|

```jsx
<Image
  source={{ uri: url }}
  onLoadEnd={() => setLoading(false)}
/>
```

</details> <details> <summary><code>onError</code> — <em>(event) =&gt; void</em></summary>

Called when the image fails to load. The event includes an `error` message.

|||
|---|---|
|**Type**|`(event: { nativeEvent: { error: string } }) => void`|

```jsx
<Image
  source={{ uri: url }}
  onError={(e) => {
    console.warn('Image failed:', e.nativeEvent.error);
    setFailed(true);
  }}
/>
```

</details> <details> <summary><code>onProgress</code> — <em>(event) =&gt; void</em> · 🍎 iOS</summary>

Called repeatedly as the image downloads. Use to build a download progress indicator.

|||
|---|---|
|**Type**|`(event: { nativeEvent: { loaded: number, total: number } }) => void`|
|**Platform**|iOS|

```jsx
<Image
  source={{ uri: url }}
  onProgress={({ nativeEvent: { loaded, total } }) => {
    setProgress(loaded / total);
  }}
/>
```

</details> <details> <summary><code>onPartialLoad</code> — <em>() =&gt; void</em> · 🍎 iOS</summary>

Called when a partial image has been loaded. Fired during progressive JPEG loading on iOS.

|||
|---|---|
|**Type**|`() => void`|
|**Platform**|iOS|

```jsx
<Image source={{ uri: progressiveJpegUrl }} onPartialLoad={() => setPartialLoaded(true)} />
```

</details> <details> <summary><code>onLayout</code> — <em>(event) =&gt; void</em></summary>

Called when the component mounts or its layout changes.

|||
|---|---|
|**Type**|`(event: LayoutChangeEvent) => void`|

```jsx
<Image
  source={source}
  onLayout={(e) => {
    const { width, height } = e.nativeEvent.layout;
    console.log(`Image container: ${width}×${height}`);
  }}
/>
```

</details>

---

## Style & Visual

<details> <summary><code>borderRadius</code>, <code>borderTopLeftRadius</code>, etc. — via <code>style</code></summary>

All border radius variants work on Image. Use `overflow: 'hidden'` if the image clips unexpectedly on Android.

```jsx
<Image
  source={{ uri: url }}
  style={{
    width: 80,
    height: 80,
    borderRadius: 40,      // circle
    overflow: 'hidden',    // required on Android for clipping
  }}
/>
```

</details> <details> <summary><code>opacity</code> — via <code>style</code></summary>

Sets transparency of the image.

```jsx
<Image source={source} style={{ width: 200, height: 150, opacity: 0.6 }} />
```

</details> <details> <summary><code>tintColor</code> — via <code>style</code> or prop</summary>

Colors all non-transparent pixels. Works as both a direct prop and inside `style`. Prefer the prop for clarity.

```jsx
// As prop
<Image source={require('./icon.png')} tintColor="#6366f1" style={{ width: 24, height: 24 }} />

// In style (also works)
<Image source={require('./icon.png')} style={{ width: 24, height: 24, tintColor: '#6366f1' }} />
```

</details> <details> <summary><code>overlayColor</code> — <em>color</em> · 🤖 Android</summary>

When the image has rounded corners, Android may show white corners in the clipped areas. `overlayColor` fills them with the given color to match the background.

|||
|---|---|
|**Type**|`color`|
|**Default**|`undefined`|
|**Platform**|Android|

```jsx
<Image
  source={{ uri: url }}
  style={{ width: 80, height: 80, borderRadius: 40 }}
  overlayColor="#ffffff"
/>
```

</details>

---

## Accessibility

<details> <summary><code>accessible</code> — <em>boolean</em></summary>

If `true`, marks the image as an accessible element for screen readers.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```jsx
<Image source={source} accessible accessibilityLabel="Company logo" />
```

</details> <details> <summary><code>accessibilityLabel</code> — <em>string</em></summary>

Text read by screen readers when the image is focused. Prefer `alt` for images (simpler, same effect).

|||
|---|---|
|**Type**|`string`|

```jsx
<Image source={source} accessible accessibilityLabel="User's profile photo" />
```

</details> <details> <summary><code>alt</code> — <em>string</em></summary>

Shorthand that sets `accessibilityLabel` and enables `accessible` automatically. Recommended approach.

|||
|---|---|
|**Type**|`string`|

```jsx
<Image source={source} alt="Team photo from the 2024 offsite" />
```

</details>

---

## iOS-Only Props

<details> <summary><code>capInsets</code> — <em>{ top, bottom, left, right }</em> · 🍎 iOS</summary>

Defines the inset areas of a stretchable (9-slice) image. The corners remain fixed while the middle stretches.

|||
|---|---|
|**Type**|`{ top: number, left: number, bottom: number, right: number }`|
|**Default**|`undefined`|
|**Platform**|iOS|

```jsx
<Image
  source={require('./button-bg.png')}
  capInsets={{ top: 10, left: 10, bottom: 10, right: 10 }}
  style={{ width: 200, height: 50 }}
/>
```

</details>

---

## Android-Only Props

<details> <summary><code>progressiveRenderingEnabled</code> — <em>boolean</em> · 🤖 Android</summary>

Enables progressive rendering for JPEG images — displays a low-resolution version while the full image loads.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|
|**Platform**|Android|

```jsx
<Image
  source={{ uri: progressiveJpegUrl }}
  progressiveRenderingEnabled
  style={{ width: 300, height: 200 }}
/>
```

</details>

---

## Static Methods

<details> <summary><code>Image.getSize(uri, success, failure?)</code></summary>

Fetches the width and height of a remote image before rendering it. Useful for calculating dynamic layout dimensions.

```jsx
Image.getSize(
  'https://example.com/image.jpg',
  (width, height) => {
    console.log(`Image size: ${width}×${height}`);
  },
  (error) => {
    console.error('Failed to get size:', error);
  }
);
```

</details> <details> <summary><code>Image.getSizeWithHeaders(uri, headers, success, failure?)</code></summary>

Same as `getSize` but also sends custom HTTP headers — needed for authenticated image endpoints.

```jsx
Image.getSizeWithHeaders(
  'https://api.example.com/image.jpg',
  { Authorization: `Bearer ${token}` },
  (width, height) => setImageDimensions({ width, height }),
  (error) => console.error(error)
);
```

</details> <details> <summary><code>Image.prefetch(url)</code></summary>

Downloads and caches a remote image in advance. Returns a Promise that resolves to `true` on success.

```jsx
// Prefetch before navigating to the next screen
await Image.prefetch('https://example.com/hero.jpg');
navigation.navigate('Details');
```

</details> <details> <summary><code>Image.prefetchWithMetadata(url, queryRootName, rootTag)</code></summary>

Prefetches an image with additional metadata for Relay / React Native infrastructure.

```jsx
Image.prefetchWithMetadata(url, 'MyQueryName', 0);
```

</details> <details> <summary><code>Image.abortPrefetch(requestId)</code> · 🤖 Android</summary>

Cancels an active prefetch request by its ID (returned by `prefetch`). Android only.

```jsx
const requestId = await Image.prefetch(url);
// Later, if no longer needed:
Image.abortPrefetch(requestId);
```

</details> <details> <summary><code>Image.queryCache(urls)</code></summary>

Checks which of the given URLs are currently cached. Returns an object mapping `url → 'memory' | 'disk' | 'disk/memory'`.

```jsx
const cacheMap = await Image.queryCache([
  'https://example.com/a.jpg',
  'https://example.com/b.jpg',
]);
// { 'https://example.com/a.jpg': 'disk', 'https://example.com/b.jpg': 'memory' }
```

</details> <details> <summary><code>Image.resolveAssetSource(source)</code></summary>

Resolves a static asset `require(...)` into a `{ uri, width, height, scale }` object. Useful for passing static asset info to native code.

```jsx
const asset = Image.resolveAssetSource(require('./logo.png'));
// { uri: 'file:///...', width: 200, height: 100, scale: 2 }
```

</details>

---

## ImageBackground

<details> <summary><code>ImageBackground</code> — renders children on top of an image</summary>

Wraps `Image` and renders children on top of it. A convenient alternative to absolute positioning.

|Key Props||
|---|---|
|`source`|Same as `Image`|
|`style`|Style for the outer container — **must include `width` and `height`**|
|`imageStyle`|Style specifically for the image (e.g. `borderRadius`, `opacity`)|
|`resizeMode`|Same as `Image`|
|`imageRef`|Ref forwarded to the inner `Image`|

```jsx
import { ImageBackground } from 'react-native';

<ImageBackground
  source={{ uri: heroUrl }}
  style={{ width: '100%', height: 300, justifyContent: 'flex-end' }}
  imageStyle={{ borderRadius: 12 }}
  resizeMode="cover"
>
  <View style={styles.overlay}>
    <Text style={styles.title}>Hello World</Text>
  </View>
</ImageBackground>
```

</details>

---

## Common Patterns

<details> <summary>Local static asset</summary>

```jsx
// Bundled at build time — width/height inferred automatically
<Image
  source={require('./assets/logo.png')}
  style={{ width: 120, height: 40 }}
/>
```

</details> <details> <summary>Remote image with loading and error fallback</summary>

```jsx
function RemoteImage({ uri, style }) {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);

  if (error) {
    return (
      <Image
        source={require('./assets/broken-image.png')}
        style={style}
      />
    );
  }

  return (
    <View>
      {loading && <ActivityIndicator style={StyleSheet.absoluteFill} />}
      <Image
        source={{ uri }}
        style={style}
        onLoadEnd={() => setLoading(false)}
        onError={() => { setLoading(false); setError(true); }}
      />
    </View>
  );
}
```

</details> <details> <summary>Circular avatar</summary>

```jsx
<Image
  source={{ uri: user.avatarUrl }}
  style={{
    width: 56,
    height: 56,
    borderRadius: 28,
    overflow: 'hidden',      // Android border clip fix
  }}
  defaultSource={require('./assets/default-avatar.png')}
  alt={`${user.name}'s avatar`}
/>
```

</details> <details> <summary>Authenticated image (private endpoint)</summary>

```jsx
<Image
  source={{
    uri: 'https://api.example.com/private/photo.jpg',
    headers: { Authorization: `Bearer ${authToken}` },
    cache: 'force-cache',
  }}
  style={{ width: 300, height: 200 }}
/>
```

</details> <details> <summary>Tinting an icon to match theme color</summary>

```jsx
function ThemedIcon({ name, color, size = 24 }) {
  return (
    <Image
      source={icons[name]}
      style={{ width: size, height: size }}
      tintColor={color}
      resizeMode="contain"
    />
  );
}

// Usage
<ThemedIcon name="heart" color={theme.primary} size={20} />
```

</details> <details> <summary>Blurred background effect</summary>

```jsx
<View style={{ flex: 1 }}>
  {/* Blurred background */}
  <Image
    source={{ uri: photoUrl }}
    style={[StyleSheet.absoluteFillObject, { transform: [{ scale: 1.1 }] }]}
    blurRadius={20}
    resizeMode="cover"
  />
  {/* Frosted overlay */}
  <View style={[StyleSheet.absoluteFillObject, { backgroundColor: 'rgba(0,0,0,0.3)' }]} />
  {/* Content */}
  <View style={styles.content}>
    <Text>Overlay content here</Text>
  </View>
</View>
```

</details> <details> <summary>Prefetch images for the next screen</summary>

```jsx
// Prefetch hero images as the user browses the list
async function prefetchNextScreen(items) {
  await Promise.all(
    items.slice(0, 5).map(item => Image.prefetch(item.heroImageUrl))
  );
}

// Check cache before deciding to preload
const cached = await Image.queryCache(urls);
const toFetch = urls.filter(url => !cached[url]);
await Promise.all(toFetch.map(url => Image.prefetch(url)));
```

</details> <details> <summary>Maintain aspect ratio dynamically</summary>

```jsx
function AspectImage({ uri, aspectRatio = 16 / 9 }) {
  return (
    <View style={{ width: '100%', aspectRatio }}>
      <Image
        source={{ uri }}
        style={StyleSheet.absoluteFillObject}
        resizeMode="cover"
      />
    </View>
  );
}

// Usage
<AspectImage uri={photoUrl} aspectRatio={4 / 3} />
```

</details> <details> <summary>Get image dimensions before rendering</summary>

```jsx
function SmartImage({ uri }) {
  const [dimensions, setDimensions] = useState(null);
  const screenWidth = Dimensions.get('window').width;

  useEffect(() => {
    Image.getSize(uri, (width, height) => {
      const ratio = height / width;
      setDimensions({ width: screenWidth, height: screenWidth * ratio });
    });
  }, [uri]);

  if (!dimensions) return null;

  return (
    <Image
      source={{ uri }}
      style={dimensions}
      resizeMode="cover"
    />
  );
}
```

</details> <details> <summary>Base64 image (e.g. from camera roll)</summary>

```jsx
// After picking from camera/library
const result = await launchImageLibrary({ includeBase64: true });
const base64Uri = `data:image/jpeg;base64,${result.assets[0].base64}`;

<Image
  source={{ uri: base64Uri }}
  style={{ width: 200, height: 200 }}
  resizeMode="cover"
/>
```

</details> <details> <summary>Hero image with text overlay using ImageBackground</summary>

```jsx 
<ImageBackground
  source={{ uri: heroUrl }}
  style={{ width: '100%', height: 240 }}
  imageStyle={{ borderRadius: 16 }}
  resizeMode="cover"
>
  <LinearGradient
    colors={['transparent', 'rgba(0,0,0,0.8)']}
    style={[StyleSheet.absoluteFillObject, { borderRadius: 16 }]}
  />
  <View style={{ position: 'absolute', bottom: 16, left: 16, right: 16 }}>
    <Text style={{ color: '#fff', fontSize: 22, fontWeight: '700' }}>
      {title}
    </Text>
    <Text style={{ color: 'rgba(255,255,255,0.8)', marginTop: 4 }}>
      {subtitle}
    </Text>
  </View>
</ImageBackground>
```

</details>

---

## Quick-Reference Cheatsheet

|Prop / Method|Use case|
|---|---|
|`source={require(...)}`|Local bundled asset|
|`source={{ uri }}`|Remote URL|
|`source={{ uri, headers }}`|Authenticated remote image|
|`defaultSource`|Placeholder while loading|
|`resizeMode="cover"`|Fill container, crop excess|
|`resizeMode="contain"`|Show full image, letterbox|
|`tintColor`|Recolor icons|
|`blurRadius`|Blur effect / obscure content|
|`onError`|Handle broken images|
|`onProgress`|Download progress bar (iOS)|
|`capInsets`|9-slice stretchable images (iOS)|
|`resizeMethod="resize"`|Avoid OOM on large Android images|
|`overlayColor`|Fix Android rounded-corner artifacts|
|`Image.prefetch(url)`|Pre-warm image cache|
|`Image.getSize(url, cb)`|Get dimensions before render|
|`Image.queryCache(urls)`|Check what's already cached|
|`ImageBackground`|Content overlaid on an image|

---

_Reference based on React Native stable API. Always check the [official docs](https://reactnative.dev/docs/image) for the latest updates._
