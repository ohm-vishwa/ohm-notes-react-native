# expo-camera — Complete Reference

A comprehensive reference for `expo-camera` v16+ (latest).  
Covers permissions, CameraView component, all props, ref methods, scanning, recording, and common patterns.

> **Install:**
> 
> ```bash
> npx expo install expo-camera
> ```
> 
> **Plugin config in app.json:**
> 
> ```json
> {
>   "expo": {
>     "plugins": [
>       [
>         "expo-camera",
>         {
>           "cameraPermission": "Allow $(PRODUCT_NAME) to access your camera.",
>           "microphonePermission": "Allow $(PRODUCT_NAME) to access your microphone.",
>           "recordAudioAndroid": true
>         }
>       ]
>     ]
>   }
> }
> ```

---

## Table of Contents

1. [Overview — What Changed in v14+](#overview--what-changed-in-v14)
2. [Permissions](#permissions)
3. [CameraView — Core Component](#cameraview--core-component)
4. [Core Props](#core-props)
5. [Photo Props](#photo-props)
6. [Video / Recording Props](#video--recording-props)
7. [Barcode / QR Scanning Props](#barcode--qr-scanning-props)
8. [Focus & Zoom Props](#focus--zoom-props)
9. [Flash & Torch Props](#flash--torch-props)
10. [Mirror & Orientation Props](#mirror--orientation-props)
11. [Style Props](#style-props)
12. [Event Callbacks](#event-callbacks)
13. [CameraView Ref Methods](#cameraview-ref-methods)
14. [takePictureAsync Options](#takepictureasync-options)
15. [recordAsync Options](#recordasync-options)
16. [Scan Result Object](#scan-result-object)
17. [Camera Constants & Enums](#camera-constants--enums)
18. [useCameraPermissions Hook](#usecamerapermissions-hook)
19. [useMicrophonePermissions Hook](#usemicrophonepermissions-hook)
20. [Common Patterns](#common-patterns)

---

## Overview — What Changed in v14+

<details> <summary>Breaking changes from older expo-camera API</summary>

`expo-camera` v14+ (SDK 50+) introduced `CameraView` as the new primary component, replacing the old `Camera` component.

|Old API (deprecated)|New API (v14+)|
|---|---|
|`<Camera>`|`<CameraView>`|
|`type="front"/"back"`|`facing="front"/"back"`|
|`Camera.requestCameraPermissionsAsync()`|`useCameraPermissions()` hook|
|`Camera.getCameraPermissionsAsync()`|`useCameraPermissions()` hook|
|`barCodeScannerSettings`|`barcodeScannerSettings`|
|`onBarCodeScanned`|`onBarcodeScanned`|
|`Camera.Constants.Type`|`"front" \| "back"` string|
|`Camera.Constants.FlashMode`|`"on" \| "off" \| "auto" \| "torch"`|
|`Camera.Constants.WhiteBalance`|`WhiteBalance` enum|

```tsx
// ❌ Old (deprecated)
import { Camera } from 'expo-camera';
<Camera type={Camera.Constants.Type.back} />

// ✅ New (v14+)
import { CameraView } from 'expo-camera';
<CameraView facing="back" />
```

</details>

---

## Permissions

<details> <summary><code>useCameraPermissions()</code> — request camera permission</summary>

Returns the current camera permission status and a function to request it. The recommended way to handle permissions.

|||
|---|---|
|**Returns**|`[PermissionResponse \| null, requestPermission, getPermission]`|

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';
import { View, Text, Button } from 'react-native';

export default function App() {
  const [permission, requestPermission] = useCameraPermissions();

  if (!permission) {
    // Permission status still loading
    return <View />;
  }

  if (!permission.granted) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text>Camera permission is required.</Text>
        <Button title="Grant Permission" onPress={requestPermission} />
      </View>
    );
  }

  return (
    <CameraView style={{ flex: 1 }} facing="back" />
  );
}
```

</details> <details> <summary><code>PermissionResponse</code> — permission status object</summary>

|Field|Type|Description|
|---|---|---|
|`granted`|`boolean`|Whether permission is granted|
|`status`|`PermissionStatus`|`'granted' \| 'denied' \| 'undetermined'`|
|`expires`|`'never' \| number`|When permission expires|
|`canAskAgain`|`boolean`|Whether the OS allows re-prompting|

```tsx
const [permission, requestPermission] = useCameraPermissions();

// Check status
if (permission?.status === 'denied' && !permission.canAskAgain) {
  // User permanently denied — open settings
  Linking.openSettings();
} else if (!permission?.granted) {
  await requestPermission();
}
```

</details> <details> <summary><code>useMicrophonePermissions()</code> — request microphone permission</summary>

Required when recording video with audio. Same API as `useCameraPermissions`.

```tsx
import { useCameraPermissions, useMicrophonePermissions } from 'expo-camera';

export default function VideoScreen() {
  const [cameraPermission, requestCameraPermission] = useCameraPermissions();
  const [micPermission, requestMicPermission] = useMicrophonePermissions();

  const hasAllPermissions = cameraPermission?.granted && micPermission?.granted;

  const requestAll = async () => {
    await requestCameraPermission();
    await requestMicPermission();
  };

  if (!hasAllPermissions) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', padding: 24 }}>
        <Text>Camera and microphone access required for video recording.</Text>
        <Button title="Grant Permissions" onPress={requestAll} />
      </View>
    );
  }

  return <CameraView style={{ flex: 1 }} mode="video" />;
}
```

</details> <details> <summary>Static permission methods (non-hook)</summary>

Use these outside of React components (e.g. in utility functions or before rendering).

```tsx
import { Camera } from 'expo-camera';

// Get current status without requesting
const cameraStatus = await Camera.getCameraPermissionsAsync();
const micStatus    = await Camera.getMicrophonePermissionsAsync();

// Request
const cameraResult = await Camera.requestCameraPermissionsAsync();
const micResult    = await Camera.requestMicrophonePermissionsAsync();

console.log(cameraResult.granted); // true | false
```

</details>

---

## CameraView — Core Component

<details> <summary>Basic setup</summary>

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';
import { useRef } from 'react';
import { View, TouchableOpacity, Text } from 'react-native';

export default function CameraScreen() {
  const cameraRef = useRef<CameraView>(null);
  const [permission, requestPermission] = useCameraPermissions();

  if (!permission?.granted) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text>No camera permission</Text>
        <TouchableOpacity onPress={requestPermission}>
          <Text>Grant Access</Text>
        </TouchableOpacity>
      </View>
    );
  }

  const takePicture = async () => {
    const photo = await cameraRef.current?.takePictureAsync();
    console.log('Photo URI:', photo?.uri);
  };

  return (
    <View style={{ flex: 1 }}>
      <CameraView
        ref={cameraRef}
        style={{ flex: 1 }}
        facing="back"
      />
      <TouchableOpacity
        style={{ position: 'absolute', bottom: 40, alignSelf: 'center' }}
        onPress={takePicture}
      >
        <View style={{ width: 70, height: 70, borderRadius: 35, backgroundColor: '#fff' }} />
      </TouchableOpacity>
    </View>
  );
}
```

</details>

---

## Core Props

<details> <summary><code>facing</code> — <em>'front' | 'back'</em></summary>

Which camera to use. Switches between front and rear camera.

|||
|---|---|
|**Type**|`'front' \| 'back'`|
|**Default**|`'back'`|

```tsx
// Rear camera (default)
<CameraView facing="back" style={{ flex: 1 }} />

// Front camera (selfie)
<CameraView facing="front" style={{ flex: 1 }} />

// Toggle state
const [facing, setFacing] = useState<'front' | 'back'>('back');
const flip = () => setFacing(f => f === 'back' ? 'front' : 'back');

<CameraView facing={facing} style={{ flex: 1 }} />
```

</details> <details> <summary><code>mode</code> — <em>'picture' | 'video'</em></summary>

Switches the camera between photo capture mode and video recording mode.

|||
|---|---|
|**Type**|`'picture' \| 'video'`|
|**Default**|`'picture'`|

```tsx
// Photo mode (default)
<CameraView mode="picture" style={{ flex: 1 }} />

// Video recording mode
<CameraView mode="video" style={{ flex: 1 }} />
```

> ⚠️ Set `mode="video"` before calling `recordAsync()`. Switching mode while recording is not supported.

</details> <details> <summary><code>active</code> — <em>boolean</em></summary>

Controls whether the camera is actively previewing. Set to `false` to pause the camera stream without unmounting the component — useful for performance when switching tabs.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|

```tsx
const isFocused = useIsFocused();

<CameraView
  active={isFocused}  // pause camera when screen is not focused
  style={{ flex: 1 }}
/>
```

</details> <details> <summary><code>animateShutter</code> — <em>boolean</em></summary>

Enables or disables the native shutter animation when taking a photo.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|

```tsx
<CameraView animateShutter={false} style={{ flex: 1 }} />
```

</details>

---

## Photo Props

<details> <summary><code>pictureSize</code> — <em>string</em></summary>

Sets the resolution of captured photos. Must be one of the sizes returned by `getAvailablePictureSizesAsync()`.

|||
|---|---|
|**Type**|`string`|
|**Default**|Highest available|

```tsx
<CameraView
  ref={cameraRef}
  pictureSize="1920x1080"
  style={{ flex: 1 }}
/>

// Get available sizes
const sizes = await cameraRef.current?.getAvailablePictureSizesAsync();
// ['4032x3024', '3264x2448', '1920x1080', '1280x720', ...]
```

</details>

---

## Video / Recording Props

<details> <summary><code>videoQuality</code> — <em>VideoQuality enum</em></summary>

Sets the quality of recorded video.

|||
|---|---|
|**Type**|`VideoQuality`|
|**Default**|`VideoQuality['1080p']`|

```tsx
import { CameraView, VideoQuality } from 'expo-camera';

<CameraView
  mode="video"
  videoQuality={VideoQuality['720p']}
  style={{ flex: 1 }}
/>
```

**VideoQuality values:**

|Value|Resolution|
|---|---|
|`VideoQuality['2160p']`|4K (3840×2160)|
|`VideoQuality['1080p']`|Full HD (1920×1080)|
|`VideoQuality['720p']`|HD (1280×720)|
|`VideoQuality['480p']`|SD (640×480)|
|`VideoQuality['4:3']`|640×480 (4:3 ratio)|

</details> <details> <summary><code>videoStabilizationMode</code> — <em>VideoStabilization enum</em> · 🍎 iOS</summary>

Enables video stabilization on iOS.

|||
|---|---|
|**Type**|`VideoStabilization`|
|**Platform**|iOS|

```tsx
import { CameraView, VideoStabilization } from 'expo-camera';

<CameraView
  mode="video"
  videoStabilizationMode={VideoStabilization.auto}
  style={{ flex: 1 }}
/>
```

**VideoStabilization values:**

|Value|Description|
|---|---|
|`VideoStabilization.off`|No stabilization|
|`VideoStabilization.standard`|Standard stabilization|
|`VideoStabilization.cinematic`|Cinematic stabilization|
|`VideoStabilization.auto`|System chooses best mode|

</details> <details> <summary><code>mute</code> — <em>boolean</em></summary>

Mutes audio during video recording.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```tsx
<CameraView mode="video" mute={true} style={{ flex: 1 }} />
```

</details>

---

## Barcode / QR Scanning Props

<details> <summary><code>barcodeScannerSettings</code> — configure what to scan</summary>

Configures the barcode scanner. Define which barcode types to detect.

|Field|Type|Description|
|---|---|---|
|`barcodeTypes`|`BarcodeType[]`|Array of barcode formats to scan|
|`interval`|`number`|Minimum ms between scan events (Android)|

```tsx
import { CameraView, BarcodeType } from 'expo-camera';

<CameraView
  barcodeScannerSettings={{
    barcodeTypes: ['qr', 'pdf417', 'ean13', 'code128'],
  }}
  onBarcodeScanned={handleScan}
  style={{ flex: 1 }}
/>
```

**All BarcodeType values:**

|Value|Description|
|---|---|
|`'aztec'`|Aztec 2D barcode|
|`'code128'`|Code 128 linear barcode|
|`'code39'`|Code 39 linear barcode|
|`'code93'`|Code 93 linear barcode|
|`'codabar'`|Codabar linear barcode|
|`'datamatrix'`|Data Matrix 2D barcode|
|`'ean13'`|EAN-13 barcode (retail)|
|`'ean8'`|EAN-8 barcode|
|`'itf14'`|ITF-14 barcode|
|`'pdf417'`|PDF417 2D barcode|
|`'qr'`|QR Code|
|`'upc_a'`|UPC-A barcode|
|`'upc_e'`|UPC-E barcode|
|`'databar'`|GS1 DataBar · iOS|
|`'databarexpanded'`|GS1 DataBar Expanded · iOS|
|`'maxicode'`|MaxiCode · iOS|
|`'microqr'`|Micro QR Code · iOS|
|`'micropdf417'`|Micro PDF417 · iOS|

</details> <details> <summary><code>onBarcodeScanned</code> — <em>callback</em></summary>

Called whenever a barcode is detected in the camera frame.

|||
|---|---|
|**Type**|`(result: BarcodeScanningResult) => void`|

```tsx
<CameraView
  barcodeScannerSettings={{ barcodeTypes: ['qr'] }}
  onBarcodeScanned={({ data, type, bounds, cornerPoints }) => {
    console.log('Scanned:', type, data);
    // data         → scanned content string
    // type         → barcode type (e.g. 'qr')
    // bounds       → { origin: {x,y}, size: {width,height} }
    // cornerPoints → [{ x, y }, ...] (4 corners)
  }}
  style={{ flex: 1 }}
/>
```

</details>

---

## Focus & Zoom Props

<details> <summary><code>autofocus</code> — <em>'on' | 'off'</em></summary>

Enables or disables continuous autofocus.

|||
|---|---|
|**Type**|`'on' \| 'off'`|
|**Default**|`'off'`|

```tsx
<CameraView autofocus="on" style={{ flex: 1 }} />
```

</details> <details> <summary><code>focusable</code> — <em>boolean</em> · 🤖 Android</summary>

Whether the camera view can receive tap-to-focus events on Android.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`true`|
|**Platform**|Android|

```tsx
<CameraView focusable={false} style={{ flex: 1 }} />
```

</details> <details> <summary><code>zoom</code> — <em>number</em></summary>

Zoom level from `0` (no zoom) to `1` (maximum zoom). Use with pinch gestures for smooth zooming.

|||
|---|---|
|**Type**|`number`|
|**Range**|`0` to `1`|
|**Default**|`0`|

```tsx
const [zoom, setZoom] = useState(0);

<CameraView zoom={zoom} style={{ flex: 1 }} />

// Pinch gesture to zoom
const pinchGesture = Gesture.Pinch()
  .onUpdate((e) => {
    const newZoom = Math.min(1, Math.max(0, zoom + (e.scale - 1) * 0.1));
    runOnJS(setZoom)(newZoom);
  });
```

</details>

---

## Flash & Torch Props

<details> <summary><code>flash</code> — <em>FlashMode</em></summary>

Controls the camera flash for photo capture.

|||
|---|---|
|**Type**|`'off' \| 'on' \| 'auto'`|
|**Default**|`'off'`|

```tsx
import { CameraView } from 'expo-camera';

// Flash off (default)
<CameraView flash="off" style={{ flex: 1 }} />

// Always on
<CameraView flash="on" style={{ flex: 1 }} />

// Auto (device decides)
<CameraView flash="auto" style={{ flex: 1 }} />
```

</details> <details> <summary><code>enableTorch</code> — <em>boolean</em></summary>

Enables the flashlight/torch independently of flash mode. Works in both photo and video modes.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```tsx
const [torchOn, setTorchOn] = useState(false);

<CameraView enableTorch={torchOn} style={{ flex: 1 }} />
<Button title={torchOn ? 'Torch Off' : 'Torch On'} onPress={() => setTorchOn(t => !t)} />
```

</details>

---

## Mirror & Orientation Props

<details> <summary><code>mirror</code> — <em>boolean</em></summary>

Mirrors the camera output horizontally. Applied only to the preview, not the captured photo/video.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```tsx
// Mirror front camera preview (selfie mode)
<CameraView facing="front" mirror={true} style={{ flex: 1 }} />
```

</details> <details> <summary><code>videoMirrorMode</code> — <em>boolean</em> · 🍎 iOS</summary>

Mirrors the recorded video output on iOS. Applies to the saved file, not just the preview.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|
|**Platform**|iOS|

```tsx
<CameraView
  facing="front"
  mirror={true}
  videoMirrorMode={true}
  mode="video"
  style={{ flex: 1 }}
/>
```

</details> <details> <summary><code>responsiveOrientationWhenOrientationLocked</code> — <em>boolean</em></summary>

When enabled, the camera rotates its output to match device orientation even if the app's orientation is locked.

|||
|---|---|
|**Type**|`boolean`|
|**Default**|`false`|

```tsx
<CameraView
  responsiveOrientationWhenOrientationLocked={true}
  style={{ flex: 1 }}
/>
```

</details>

---

## Style Props

<details> <summary><code>style</code> — <em>StyleProp&lt;ViewStyle&gt;</em></summary>

Style for the camera container view. Always set `flex: 1` or explicit dimensions.

```tsx
// Full screen
<CameraView style={{ flex: 1 }} />

// Fixed size
<CameraView style={{ width: 300, height: 400, borderRadius: 12, overflow: 'hidden' }} />

// With border
<CameraView style={{ flex: 1, margin: 16, borderRadius: 20, overflow: 'hidden' }} />
```

</details>

---

## Event Callbacks

<details> <summary><code>onCameraReady</code> — <em>() =&gt; void</em></summary>

Called when the camera finishes initializing and the preview is ready. Safe to call `takePictureAsync` after this fires.

```tsx
<CameraView
  onCameraReady={() => {
    console.log('Camera is ready');
    setIsReady(true);
  }}
  style={{ flex: 1 }}
/>
```

</details> <details> <summary><code>onMountError</code> — <em>(error) =&gt; void</em></summary>

Called when the camera fails to mount. Provides an error object with a `message` field.

```tsx
<CameraView
  onMountError={({ message }) => {
    console.error('Camera mount error:', message);
    setError(message);
  }}
  style={{ flex: 1 }}
/>
```

</details> <details> <summary><code>onBarcodeScanned</code> — <em>(result: BarcodeScanningResult) =&gt; void</em></summary>

Called when a barcode or QR code is detected in the camera frame.

```tsx
<CameraView
  barcodeScannerSettings={{ barcodeTypes: ['qr', 'ean13'] }}
  onBarcodeScanned={(result) => {
    // result.data         — scanned string value
    // result.type         — barcode format string
    // result.bounds       — screen bounding box
    // result.cornerPoints — 4 corner coordinates
    handleScan(result.data);
  }}
  style={{ flex: 1 }}
/>
```

</details> <details> <summary><code>onResponsiveOrientationChanged</code> — <em>(event) =&gt; void</em></summary>

Called when the camera's responsive orientation changes (when `responsiveOrientationWhenOrientationLocked` is enabled).

```tsx
<CameraView
  responsiveOrientationWhenOrientationLocked
  onResponsiveOrientationChanged={({ orientation }) => {
    console.log('New orientation:', orientation);
    // 'portrait' | 'portraitUpsideDown' | 'landscapeLeft' | 'landscapeRight'
  }}
  style={{ flex: 1 }}
/>
```

</details>

---

## CameraView Ref Methods

<details> <summary><code>cameraRef.current?.takePictureAsync(options?)</code></summary>

Captures a photo. Returns a `CameraCapturedPicture` object.

**Returns:** `Promise<CameraCapturedPicture>`

|Result field|Type|Description|
|---|---|---|
|`uri`|`string`|Local file URI of the photo|
|`width`|`number`|Photo width in pixels|
|`height`|`number`|Photo height in pixels|
|`base64`|`string \| undefined`|Base64 encoded image (if requested)|
|`exif`|`object \| undefined`|EXIF metadata (if requested)|

```tsx
const cameraRef = useRef<CameraView>(null);

const takePicture = async () => {
  const photo = await cameraRef.current?.takePictureAsync({
    quality: 0.8,
    base64: false,
    exif: false,
  });

  if (photo) {
    console.log('URI:', photo.uri);
    console.log('Size:', photo.width, '×', photo.height);
    navigate('/preview', { uri: photo.uri });
  }
};
```

</details> <details> <summary><code>cameraRef.current?.recordAsync(options?)</code></summary>

Starts recording video. Returns a promise that resolves when recording stops. Must set `mode="video"` on the `CameraView`.

**Returns:** `Promise<{ uri: string }>`

```tsx
const startRecording = async () => {
  setIsRecording(true);
  const video = await cameraRef.current?.recordAsync({
    maxDuration: 60,           // max 60 seconds
    maxFileSize: 50 * 1024 * 1024, // max 50 MB
    mute: false,
  });

  if (video) {
    console.log('Video URI:', video.uri);
    setVideoUri(video.uri);
  }
  setIsRecording(false);
};
```

</details> <details> <summary><code>cameraRef.current?.stopRecording()</code></summary>

Stops an active video recording. The `recordAsync` promise resolves with the video URI.

```tsx
const stopRecording = () => {
  cameraRef.current?.stopRecording();
};

// Usage
<TouchableOpacity
  onPress={isRecording ? stopRecording : startRecording}
>
  <Text>{isRecording ? '⏹ Stop' : '⏺ Record'}</Text>
</TouchableOpacity>
```

</details> <details> <summary><code>cameraRef.current?.getAvailablePictureSizesAsync()</code></summary>

Returns an array of supported photo resolution strings for the current camera and device.

**Returns:** `Promise<string[]>`

```tsx
const sizes = await cameraRef.current?.getAvailablePictureSizesAsync();
// ['4032x3024', '3264x2448', '1920x1080', '1280x720', '640x480']

// Set the best quality
setPictureSize(sizes?.[0] ?? '1920x1080');
```

</details> <details> <summary><code>cameraRef.current?.pausePreview()</code></summary>

Pauses the live camera preview. The last frame is frozen on screen. Useful to "freeze" the frame before processing.

```tsx
cameraRef.current?.pausePreview();
```

</details> <details> <summary><code>cameraRef.current?.resumePreview()</code></summary>

Resumes the live camera preview after `pausePreview()`.

```tsx
cameraRef.current?.resumePreview();
```

</details>

---

## takePictureAsync Options

<details> <summary>All <code>takePictureAsync</code> options</summary>

|Option|Type|Default|Description|
|---|---|---|---|
|`quality`|`number`|`1`|JPEG quality from `0` (lowest) to `1` (highest)|
|`base64`|`boolean`|`false`|Include base64 string in result|
|`exif`|`boolean`|`false`|Include EXIF metadata in result|
|`imageType`|`'jpg' \| 'png'`|`'jpg'`|Image format|
|`scale`|`number`|`1`|Scale factor (0–1) to downsample|
|`skipProcessing`|`boolean`|`false`|Skip post-capture processing (faster, Android only)|
|`shutterSound`|`boolean`|`true`|Play shutter sound · iOS|
|`additionalExif`|`object`|—|Extra EXIF data to embed · iOS|
|`mirror`|`boolean`|`false`|Mirror the output image (front camera)|
|`fastMode`|`boolean`|`false`|Resolve promise immediately without waiting · Android|

```tsx
const photo = await cameraRef.current?.takePictureAsync({
  quality: 0.7,          // balance quality vs file size
  base64: false,         // set true only if you need base64
  exif: true,            // include GPS, timestamp, etc.
  imageType: 'jpg',
  mirror: facing === 'front',  // mirror selfies
  shutterSound: true,          // iOS shutter click
});
```

</details>

---

## recordAsync Options

<details> <summary>All <code>recordAsync</code> options</summary>

|Option|Type|Default|Description|
|---|---|---|---|
|`maxDuration`|`number`|unlimited|Max recording duration in seconds|
|`maxFileSize`|`number`|unlimited|Max file size in bytes|
|`mute`|`boolean`|`false`|Record without audio|
|`mirror`|`boolean`|`false`|Mirror video output (front camera)|
|`codec`|`VideoCodec`|—|Video codec · iOS|

```tsx
const video = await cameraRef.current?.recordAsync({
  maxDuration: 30,
  maxFileSize: 25 * 1024 * 1024,  // 25 MB
  mute: false,
  mirror: facing === 'front',
});
```

**VideoCodec values (iOS):**

```tsx
import { VideoCodec } from 'expo-camera';

VideoCodec.H264
VideoCodec.HEVC     // better compression, iOS 11+
VideoCodec.JPEG
VideoCodec.AppleProRes4444
VideoCodec.AppleProRes422
```

</details>

---

## Scan Result Object

<details> <summary><code>BarcodeScanningResult</code> — full structure</summary>

Returned by `onBarcodeScanned`.

|Field|Type|Description|
|---|---|---|
|`data`|`string`|The decoded content of the barcode|
|`type`|`BarcodeType`|The format of the detected barcode|
|`bounds`|`{ origin: Point, size: Size }`|Bounding box in screen coordinates|
|`cornerPoints`|`Point[]`|Array of 4 `{x, y}` corner coordinates|

```tsx
onBarcodeScanned={({ data, type, bounds, cornerPoints }) => {
  // data = 'https://example.com'
  // type = 'qr'
  // bounds = {
  //   origin: { x: 100, y: 200 },
  //   size: { width: 150, height: 150 }
  // }
  // cornerPoints = [
  //   { x: 100, y: 200 },
  //   { x: 250, y: 200 },
  //   { x: 250, y: 350 },
  //   { x: 100, y: 350 },
  // ]
}}
```

</details>

---

## Camera Constants & Enums

<details> <summary><code>VideoQuality</code></summary>

```tsx
import { VideoQuality } from 'expo-camera';

VideoQuality['2160p']   // 4K
VideoQuality['1080p']   // Full HD (default)
VideoQuality['720p']    // HD
VideoQuality['480p']    // SD
VideoQuality['4:3']     // 640×480
```

</details> <details> <summary><code>VideoStabilization</code> · 🍎 iOS</summary>

```tsx
import { VideoStabilization } from 'expo-camera';

VideoStabilization.off
VideoStabilization.standard
VideoStabilization.cinematic
VideoStabilization.cinematicExtended   // iOS 13+
VideoStabilization.auto
```

</details> <details> <summary><code>VideoCodec</code> · 🍎 iOS</summary>

```tsx
import { VideoCodec } from 'expo-camera';

VideoCodec.H264
VideoCodec.HEVC
VideoCodec.JPEG
VideoCodec.AppleProRes4444
VideoCodec.AppleProRes422
```

</details> <details> <summary><code>PermissionStatus</code></summary>

```tsx
import { PermissionStatus } from 'expo-camera';

PermissionStatus.GRANTED       // 'granted'
PermissionStatus.DENIED        // 'denied'
PermissionStatus.UNDETERMINED  // 'undetermined'
```

</details>

---

## useCameraPermissions Hook

<details> <summary>Full API</summary>

```tsx
import { useCameraPermissions } from 'expo-camera';

const [permission, requestPermission, getPermission] = useCameraPermissions();

// permission — PermissionResponse | null
// permission.granted — boolean
// permission.status — 'granted' | 'denied' | 'undetermined'
// permission.canAskAgain — boolean
// permission.expires — 'never' | number

// requestPermission() — shows system dialog, returns PermissionResponse
// getPermission()     — re-fetches current status without showing dialog

// Typical guard
if (!permission) return <Loading />;

if (!permission.granted) {
  return (
    <View>
      {permission.canAskAgain
        ? <Button title="Allow Camera" onPress={requestPermission} />
        : <Button title="Open Settings" onPress={() => Linking.openSettings()} />
      }
    </View>
  );
}
```

</details>

---

## useMicrophonePermissions Hook

<details> <summary>Full API</summary>

```tsx
import { useMicrophonePermissions } from 'expo-camera';

const [permission, requestPermission, getPermission] = useMicrophonePermissions();

// Same API as useCameraPermissions
if (!permission?.granted) {
  return <Button title="Allow Microphone" onPress={requestPermission} />;
}
```

</details>

---

## Common Patterns

<details> <summary>Photo capture with preview</summary>

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';
import { useRef, useState } from 'react';
import { View, Image, TouchableOpacity, Text, StyleSheet } from 'react-native';

export default function PhotoCaptureScreen() {
  const cameraRef = useRef<CameraView>(null);
  const [permission, requestPermission] = useCameraPermissions();
  const [photoUri, setPhotoUri] = useState<string | null>(null);
  const [facing, setFacing] = useState<'front' | 'back'>('back');
  const [flash, setFlash] = useState<'off' | 'on' | 'auto'>('off');

  if (!permission?.granted) {
    return (
      <View style={styles.center}>
        <Text style={styles.message}>Camera access required</Text>
        <TouchableOpacity style={styles.button} onPress={requestPermission}>
          <Text style={styles.buttonText}>Allow Camera</Text>
        </TouchableOpacity>
      </View>
    );
  }

  if (photoUri) {
    return (
      <View style={{ flex: 1 }}>
        <Image source={{ uri: photoUri }} style={{ flex: 1 }} resizeMode="contain" />
        <View style={styles.row}>
          <TouchableOpacity style={styles.button} onPress={() => setPhotoUri(null)}>
            <Text style={styles.buttonText}>Retake</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.button} onPress={() => savePhoto(photoUri)}>
            <Text style={styles.buttonText}>Save</Text>
          </TouchableOpacity>
        </View>
      </View>
    );
  }

  const takePicture = async () => {
    const photo = await cameraRef.current?.takePictureAsync({ quality: 0.8 });
    if (photo) setPhotoUri(photo.uri);
  };

  return (
    <View style={{ flex: 1 }}>
      <CameraView
        ref={cameraRef}
        style={{ flex: 1 }}
        facing={facing}
        flash={flash}
      />

      {/* Controls */}
      <View style={[styles.row, { position: 'absolute', bottom: 40, width: '100%' }]}>
        {/* Flash toggle */}
        <TouchableOpacity onPress={() => setFlash(f => f === 'off' ? 'on' : 'off')}>
          <Text style={{ color: '#fff', fontSize: 24 }}>
            {flash === 'on' ? '⚡' : '🔦'}
          </Text>
        </TouchableOpacity>

        {/* Shutter */}
        <TouchableOpacity onPress={takePicture}>
          <View style={styles.shutter} />
        </TouchableOpacity>

        {/* Flip camera */}
        <TouchableOpacity onPress={() => setFacing(f => f === 'back' ? 'front' : 'back')}>
          <Text style={{ color: '#fff', fontSize: 24 }}>🔄</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  center:      { flex: 1, justifyContent: 'center', alignItems: 'center' },
  message:     { fontSize: 16, marginBottom: 16, textAlign: 'center' },
  row:         { flexDirection: 'row', justifyContent: 'space-around', alignItems: 'center', paddingHorizontal: 24 },
  button:      { backgroundColor: '#6366f1', padding: 14, borderRadius: 10 },
  buttonText:  { color: '#fff', fontWeight: '600' },
  shutter:     { width: 72, height: 72, borderRadius: 36, backgroundColor: '#fff', borderWidth: 4, borderColor: '#6366f1' },
});
```

</details> <details> <summary>Video recording with timer</summary>

```tsx
import { CameraView, useCameraPermissions, useMicrophonePermissions } from 'expo-camera';
import { useRef, useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

export default function VideoScreen() {
  const cameraRef = useRef<CameraView>(null);
  const [cameraPermission, requestCameraPermission] = useCameraPermissions();
  const [micPermission, requestMicPermission] = useMicrophonePermissions();
  const [isRecording, setIsRecording] = useState(false);
  const [videoUri, setVideoUri] = useState<string | null>(null);
  const [duration, setDuration] = useState(0);
  const timerRef = useRef<ReturnType<typeof setInterval> | null>(null);

  const hasPermissions = cameraPermission?.granted && micPermission?.granted;

  useEffect(() => {
    if (isRecording) {
      timerRef.current = setInterval(() => setDuration(d => d + 1), 1000);
    } else {
      if (timerRef.current) clearInterval(timerRef.current);
      setDuration(0);
    }
    return () => { if (timerRef.current) clearInterval(timerRef.current); };
  }, [isRecording]);

  if (!hasPermissions) {
    return (
      <View style={styles.center}>
        <Text>Camera & microphone permissions required</Text>
        <TouchableOpacity onPress={async () => {
          await requestCameraPermission();
          await requestMicPermission();
        }}>
          <Text>Grant Permissions</Text>
        </TouchableOpacity>
      </View>
    );
  }

  const startRecording = async () => {
    setIsRecording(true);
    const result = await cameraRef.current?.recordAsync({ maxDuration: 60 });
    if (result) setVideoUri(result.uri);
    setIsRecording(false);
  };

  const stopRecording = () => {
    cameraRef.current?.stopRecording();
  };

  const formatDuration = (s: number) =>
    `${String(Math.floor(s / 60)).padStart(2, '0')}:${String(s % 60).padStart(2, '0')}`;

  return (
    <View style={{ flex: 1 }}>
      <CameraView ref={cameraRef} style={{ flex: 1 }} mode="video" facing="back" />

      {isRecording && (
        <View style={styles.timer}>
          <View style={styles.redDot} />
          <Text style={styles.timerText}>{formatDuration(duration)}</Text>
        </View>
      )}

      <TouchableOpacity
        style={[styles.recordButton, isRecording && styles.recording]}
        onPress={isRecording ? stopRecording : startRecording}
      >
        <Text style={{ color: '#fff', fontWeight: '700' }}>
          {isRecording ? '⏹ Stop' : '⏺ Record'}
        </Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  center:       { flex: 1, justifyContent: 'center', alignItems: 'center' },
  timer:        { position: 'absolute', top: 50, alignSelf: 'center', flexDirection: 'row', alignItems: 'center', backgroundColor: 'rgba(0,0,0,0.5)', paddingHorizontal: 12, paddingVertical: 6, borderRadius: 20 },
  redDot:       { width: 8, height: 8, borderRadius: 4, backgroundColor: '#ef4444', marginRight: 8 },
  timerText:    { color: '#fff', fontWeight: '700', fontSize: 16, fontVariant: ['tabular-nums'] },
  recordButton: { position: 'absolute', bottom: 40, alignSelf: 'center', backgroundColor: '#6366f1', paddingHorizontal: 32, paddingVertical: 16, borderRadius: 30 },
  recording:    { backgroundColor: '#ef4444' },
});
```

</details> <details> <summary>QR code scanner with scan lock</summary>

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';
import { useRef, useState } from 'react';
import { View, Text, StyleSheet, Linking, Alert } from 'react-native';

export default function QRScannerScreen() {
  const [permission, requestPermission] = useCameraPermissions();
  const [scanned, setScanned] = useState(false);
  const scannedRef = useRef(false);  // prevent double-fire

  if (!permission?.granted) {
    return (
      <View style={styles.center}>
        <Text>Camera access required to scan QR codes.</Text>
      </View>
    );
  }

  const handleScan = ({ data, type }: { data: string; type: string }) => {
    if (scannedRef.current) return;  // debounce
    scannedRef.current = true;
    setScanned(true);

    // Handle URL QR codes
    if (data.startsWith('http')) {
      Alert.alert(
        'Open URL?',
        data,
        [
          { text: 'Cancel', onPress: () => resetScanner() },
          { text: 'Open', onPress: () => { Linking.openURL(data); resetScanner(); } },
        ]
      );
    } else {
      Alert.alert('Scanned', `${type}: ${data}`, [
        { text: 'OK', onPress: resetScanner },
      ]);
    }
  };

  const resetScanner = () => {
    scannedRef.current = false;
    setScanned(false);
  };

  return (
    <View style={{ flex: 1 }}>
      <CameraView
        style={{ flex: 1 }}
        barcodeScannerSettings={{ barcodeTypes: ['qr'] }}
        onBarcodeScanned={scanned ? undefined : handleScan}
      />

      {/* Scan overlay */}
      <View style={styles.overlay}>
        <View style={styles.frame} />
        <Text style={styles.hint}>Point your camera at a QR code</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  center:  { flex: 1, justifyContent: 'center', alignItems: 'center' },
  overlay: { ...StyleSheet.absoluteFillObject, justifyContent: 'center', alignItems: 'center' },
  frame:   { width: 240, height: 240, borderWidth: 3, borderColor: '#6366f1', borderRadius: 12, backgroundColor: 'transparent' },
  hint:    { position: 'absolute', bottom: 80, color: '#fff', fontSize: 15, backgroundColor: 'rgba(0,0,0,0.5)', paddingHorizontal: 16, paddingVertical: 8, borderRadius: 8 },
});
```

</details> <details> <summary>Multi-barcode scanner (scan multiple types)</summary>

```tsx
import { CameraView, useCameraPermissions, BarcodeScanningResult } from 'expo-camera';
import { useState } from 'react';
import { View, Text, FlatList, StyleSheet } from 'react-native';

export default function MultiBarcodeScanner() {
  const [permission, requestPermission] = useCameraPermissions();
  const [results, setResults] = useState<BarcodeScanningResult[]>([]);

  const handleScan = (result: BarcodeScanningResult) => {
    setResults(prev => {
      const exists = prev.some(r => r.data === result.data);
      if (exists) return prev;
      return [result, ...prev].slice(0, 10); // keep last 10
    });
  };

  if (!permission?.granted) {
    return <View style={styles.center}><Text>No permission</Text></View>;
  }

  return (
    <View style={{ flex: 1 }}>
      <CameraView
        style={{ flex: 2 }}
        barcodeScannerSettings={{
          barcodeTypes: ['qr', 'ean13', 'code128', 'upc_a', 'pdf417'],
        }}
        onBarcodeScanned={handleScan}
      />
      <FlatList
        style={{ flex: 1, backgroundColor: '#f9f9f9' }}
        data={results}
        keyExtractor={(_, i) => String(i)}
        renderItem={({ item }) => (
          <View style={styles.result}>
            <Text style={styles.type}>{item.type}</Text>
            <Text style={styles.data} numberOfLines={1}>{item.data}</Text>
          </View>
        )}
        ListEmptyComponent={
          <Text style={styles.empty}>Scan barcodes to see results</Text>
        }
      />
    </View>
  );
}

const styles = StyleSheet.create({
  center: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  result: { flexDirection: 'row', padding: 12, borderBottomWidth: 1, borderColor: '#e5e7eb' },
  type:   { width: 80, fontWeight: '600', color: '#6366f1', fontSize: 12, textTransform: 'uppercase' },
  data:   { flex: 1, color: '#111' },
  empty:  { textAlign: 'center', color: '#9ca3af', padding: 24 },
});
```

</details> <details> <summary>Camera with pinch-to-zoom (Reanimated + Gesture Handler)</summary>

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import { useSharedValue, runOnJS } from 'react-native-reanimated';
import { useRef, useState } from 'react';
import { View } from 'react-native';

export default function ZoomCamera() {
  const cameraRef = useRef<CameraView>(null);
  const [permission] = useCameraPermissions();
  const [zoom, setZoom] = useState(0);
  const startZoom = useSharedValue(0);

  const pinchGesture = Gesture.Pinch()
    .onStart(() => {
      startZoom.value = zoom;
    })
    .onUpdate((e) => {
      const next = Math.min(1, Math.max(0, startZoom.value + (e.scale - 1) * 0.5));
      runOnJS(setZoom)(next);
    });

  if (!permission?.granted) return null;

  return (
    <GestureDetector gesture={pinchGesture}>
      <View style={{ flex: 1 }}>
        <CameraView
          ref={cameraRef}
          style={{ flex: 1 }}
          zoom={zoom}
          autofocus="on"
        />
      </View>
    </GestureDetector>
  );
}
```

</details> <details> <summary>Save photo to camera roll</summary>

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';
import * as MediaLibrary from 'expo-media-library';
import { useRef } from 'react';
import { Alert, TouchableOpacity, Text, View } from 'react-native';

export default function SaveToGallery() {
  const cameraRef = useRef<CameraView>(null);
  const [cameraPermission, requestCameraPermission] = useCameraPermissions();
  const [mediaPermission, requestMediaPermission] = MediaLibrary.usePermissions();

  const captureAndSave = async () => {
    // Ensure media library permission
    if (!mediaPermission?.granted) {
      const result = await requestMediaPermission();
      if (!result.granted) {
        Alert.alert('Permission denied', 'Cannot save to camera roll.');
        return;
      }
    }

    const photo = await cameraRef.current?.takePictureAsync({ quality: 0.9 });
    if (!photo) return;

    const asset = await MediaLibrary.createAssetAsync(photo.uri);
    await MediaLibrary.createAlbumAsync('My App', asset, false);
    Alert.alert('Saved!', 'Photo saved to camera roll.');
  };

  if (!cameraPermission?.granted) return null;

  return (
    <View style={{ flex: 1 }}>
      <CameraView ref={cameraRef} style={{ flex: 1 }} />
      <TouchableOpacity
        style={{ position: 'absolute', bottom: 40, alignSelf: 'center' }}
        onPress={captureAndSave}
      >
        <Text style={{ color: '#fff', backgroundColor: '#6366f1', padding: 16, borderRadius: 30 }}>
          📸 Capture & Save
        </Text>
      </TouchableOpacity>
    </View>
  );
}
```

</details> <details> <summary>Pause camera when screen loses focus</summary>

```tsx
import { CameraView } from 'expo-camera';
import { useIsFocused } from '@react-navigation/native';
import { useFocusEffect } from 'expo-router';

// With React Navigation's useIsFocused
function CameraScreen() {
  const isFocused = useIsFocused();

  return (
    <CameraView
      active={isFocused}  // pauses preview when navigating away
      style={{ flex: 1 }}
    />
  );
}

// With Expo Router's useFocusEffect
function CameraScreenRouter() {
  const [isActive, setIsActive] = useState(false);

  useFocusEffect(
    useCallback(() => {
      setIsActive(true);
      return () => setIsActive(false);
    }, [])
  );

  return (
    <CameraView active={isActive} style={{ flex: 1 }} />
  );
}
```

</details> <details> <summary>Document scanner — capture with edge overlay</summary>

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';
import { useRef } from 'react';
import { View, StyleSheet, TouchableOpacity, Text, Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');
const PADDING = 24;

export default function DocumentScanner() {
  const cameraRef = useRef<CameraView>(null);
  const [permission, requestPermission] = useCameraPermissions();

  const capture = async () => {
    const photo = await cameraRef.current?.takePictureAsync({
      quality: 1,
      exif: true,
    });
    if (photo) {
      // Process document — pass to your doc scanner/crop API
      processDocument(photo.uri);
    }
  };

  if (!permission?.granted) {
    return (
      <View style={styles.center}>
        <Text>Camera access required</Text>
        <TouchableOpacity onPress={requestPermission}>
          <Text>Grant Permission</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={{ flex: 1, backgroundColor: '#000' }}>
      <CameraView
        ref={cameraRef}
        style={StyleSheet.absoluteFillObject}
        facing="back"
        autofocus="on"
      />

      {/* Document frame guide */}
      <View style={styles.frame}>
        {/* Corner markers */}
        {['TL', 'TR', 'BL', 'BR'].map((corner) => (
          <View key={corner} style={[styles.corner, styles[corner as keyof typeof styles]]} />
        ))}
      </View>

      <Text style={styles.hint}>Align document within the frame</Text>

      <TouchableOpacity style={styles.captureBtn} onPress={capture}>
        <Text style={{ color: '#fff', fontWeight: '700' }}>Scan Document</Text>
      </TouchableOpacity>
    </View>
  );
}

const FRAME_W = width  - PADDING * 2;
const FRAME_H = FRAME_W * 1.414;  // A4 ratio
const CORNER  = 24;
const BORDER  = 3;

const styles = StyleSheet.create({
  center:     { flex: 1, justifyContent: 'center', alignItems: 'center' },
  frame:      { position: 'absolute', top: (height - FRAME_H) / 2, left: PADDING, width: FRAME_W, height: FRAME_H },
  corner:     { position: 'absolute', width: CORNER, height: CORNER, borderColor: '#6366f1' },
  TL:         { top: 0, left: 0, borderTopWidth: BORDER, borderLeftWidth: BORDER },
  TR:         { top: 0, right: 0, borderTopWidth: BORDER, borderRightWidth: BORDER },
  BL:         { bottom: 0, left: 0, borderBottomWidth: BORDER, borderLeftWidth: BORDER },
  BR:         { bottom: 0, right: 0, borderBottomWidth: BORDER, borderRightWidth: BORDER },
  hint:       { position: 'absolute', top: (height - FRAME_H) / 2 - 40, alignSelf: 'center', color: '#fff', backgroundColor: 'rgba(0,0,0,0.5)', paddingHorizontal: 12, paddingVertical: 6, borderRadius: 8 },
  captureBtn: { position: 'absolute', bottom: 48, alignSelf: 'center', backgroundColor: '#6366f1', paddingHorizontal: 36, paddingVertical: 16, borderRadius: 30 },
});
```

</details>

---

## Quick-Reference Cheatsheet

|API|Use case|
|---|---|
|`useCameraPermissions()`|Request & check camera permission|
|`useMicrophonePermissions()`|Request & check mic permission|
|`<CameraView facing="back">`|Rear camera|
|`<CameraView facing="front">`|Front camera|
|`<CameraView mode="picture">`|Photo mode (default)|
|`<CameraView mode="video">`|Video mode|
|`<CameraView active={isFocused}>`|Pause when screen unfocused|
|`<CameraView flash="on/off/auto">`|Flash control|
|`<CameraView enableTorch={true}>`|Torch / flashlight|
|`<CameraView zoom={0–1}>`|Zoom level|
|`<CameraView autofocus="on">`|Continuous autofocus|
|`<CameraView mirror={true}>`|Mirror preview|
|`<CameraView barcodeScannerSettings={...}>`|Enable barcode scanning|
|`onBarcodeScanned={fn}`|Barcode scan callback|
|`ref.takePictureAsync(opts)`|Capture photo|
|`ref.recordAsync(opts)`|Start video recording|
|`ref.stopRecording()`|Stop video recording|
|`ref.getAvailablePictureSizesAsync()`|List supported resolutions|
|`ref.pausePreview()`|Freeze camera frame|
|`ref.resumePreview()`|Resume live preview|
|`quality: 0–1`|Photo quality (takePicture)|
|`base64: true`|Include base64 in photo result|
|`maxDuration`|Max recording seconds|
|`VideoQuality['1080p']`|Video quality enum|
|`onCameraReady`|Camera init complete|
|`onMountError`|Camera failed to open|

---

_Reference based on `expo-camera` v16 / SDK 52+. Always check the [official docs](https://docs.expo.dev/versions/latest/sdk/camera/) for the latest updates._