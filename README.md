# Expo
## 01__app-init

```sh
npx create-expo-app@latest AppName --template 
```
best way to install packages or libraries in expo
```
npm install package ❌
```
installs tested version for expo app
```
npx expo install package ✅
```
---


---


---

## 02__expo-router

https://docs.expo.dev/router/installation/
```bash
npx expo install expo-router react-native-safe-area-context react-native-screens expo-linking expo-constants expo-status-bar
```

`package.json`
```bash
{
  "main": "expo-router/entry"
}
```

`app.json`
```bash
{
  "expo": {
    "scheme": "ohmscodeIg",
    "experiments": {
      "typedRoutes": true
    }
  }
}
```
---
for web
```sh
npx expo install react-native-web react-dom
```
`app.json`
```sh
{
  "expo": {
    "web": {
      "bundler": "metro"
    }
  }
}
```
---
`tsconfig.json`
```sh
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts"]
}
```

```sh
npx expo start --clear
```



---


---


---

## 03__nativewind

https://www.nativewind.dev/docs/getting-started/installation

## sdk 54
```sh
npm install nativewind react-native-reanimated react-native-safe-area-context
npm install --dev tailwindcss@^3.4.17 prettier-plugin-tailwindcss@^0.5.11 babel-preset-expo
```

`tailwind.config.js`
```sh
/** @type {import('tailwindcss').Config} */
module.exports = {
  // NOTE: Update this to include the paths to all files that contain Nativewind classes.
  content: ["./src/**/*.{js,jsx,ts,tsx}"],
  presets: [require("nativewind/preset")],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

`global.css`
```sh
@tailwind base;
@tailwind components;
@tailwind utilities;
```
`babel.config.js`
```sh
module.exports = function (api) {
  api.cache(true);
  return {
    presets: [
      ["babel-preset-expo", { jsxImportSource: "nativewind" }],
      "nativewind/babel",
    ],
  };
};
```

```sh
npx expo customize metro.config.js
```
`metro.config.js`
```sh
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require('nativewind/metro');
 
const config = getDefaultConfig(__dirname)
 
module.exports = withNativeWind(config, { input: './global.css' })
```
`src/app/_layout.tsx`
```jsx
import "./global.css";
```

```sh
npx expo start -c
```

`nativewind-env.d.ts` typescript
```jsx
/// <reference types="nativewind/types" />
```
---


---


---

## 04__SafeAreaView

### example
```jsx
import { Text, View, } from 'react-native';
import { SafeAreaProvider, SafeAreaView, useSafeAreaInsets } from 'react-native-safe-area-context';

const DisplayInsets = () => {
  const insets = useSafeAreaInsets()
  return (
    <Text>Insests = bottom: {insets.bottom}, top: {insets.top}</Text>
  )
}

export default function App() {
  return (
    <SafeAreaProvider>
      <SafeAreaView edges={['top']} style={{ backgroundColor: 'red' }}>
        <View style={{
          backgroundColor: 'palegreen',
          height: '100%',
          alignItems: 'center'
        }}>
          <Text style={{ fontSize: 100 }}>Header</Text>
          <DisplayInsets />
          <Text style={{ marginTop: 'auto' }}>Footer</Text>
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  )
}
```
---


---


---

## 05__Button

### example
```jsx
import { StyleSheet, View,  Button, Linking } from 'react-native';

export default function App() {

  const contactMe = () => {
    Linking.openURL('mailto:ohm.dto@gmail.com')
  }

  return (
      <View style={styles.container}>
        <Button title='Contact me' onPress={() => contactMe()} />
      </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
  },
});
```

### ways to use `callback` function
```jsx
onPress={() => contactMe()} ✅
```

```jsx
onPress={contactMe} ✅
```

```jsx
onPress={contactMe()} ❌
```
---


---


---

## 06__Icons

https://icons.expo.fyi/Index

### example
```jsx
import { StyleSheet, View } from 'react-native';
import FontAwesome6 from '@expo/vector-icons/FontAwesome6';

export default function App() {
  return (
    <View style={{ flexDirection: 'row', marginVertical: 10, gap: 10 }}>
	    <FontAwesome6 name="github" size={24} color="black" />
        <FontAwesome6 name="x-twitter" size={24} color="black" />
        <FontAwesome6 name="at" size={24} color="black" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```
---


---


---

## 07__Image

`<Image/>` component can understand local image size but, 
```jsx
<Image source={require('./path/to/jpg')} />
```

it can't understand size of remote image `uri`
```jsx
<Image source={{uri:'https://remote-image.jpg'}} style={{width:'100%', height:100}} /> 
```
---
using a ratio is much more flexible for modern, multi-screen development.
```jsx
<Image source={{ uri: 'https://remote-image.jpg' }} 
style={{ width: '100%', aspectRatio: 16 / 9 }} />
```

---


---


---

## 08__expo-camera

https://docs.expo.dev/versions/latest/sdk/camera/

```sh
npx expo install expo-camera
```

`app.json`
```json
{
  "expo": {
    "plugins": [
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access your camera",
          "microphonePermission": "Allow $(PRODUCT_NAME) to access your microphone",
          "recordAudioAndroid": true,
          "barcodeScannerEnabled": true
        }
      ]
    ]
  }
}

```

`Usage`
```jsx
const CameraScreen = () => {
    const [permission, requestPermission] = useCameraPermissions()
    const [facing, setFacing] = useState<CameraType>('back')
    const [picture, setPicture] = useState<CameraCapturedPicture>()

    const camera = useRef<CameraView>(null)

    useEffect(() => {
        if (permission && !permission.granted && permission.canAskAgain) {
            requestPermission()
        }
    }, [permission])


    const toggleCameraFacing = () => {
        setFacing((current) => (current === 'back' ? 'front' : 'back'));
    }

    if (!permission?.granted) {
        return <ActivityIndicator size={30} />
    }

    const takePicture = async () => {
        if (!camera.current) return;

        try {
            const options = { quality: 1.0, skipProcessing: false };
            const res = await camera.current.takePictureAsync(options);

            if (res && res.uri) {
                console.log("Picture saved to cache:", res.uri);

                setTimeout(() => {
                    setPicture(res);
                }, 100);

            }
        } catch (error) {
            console.error("Capture error:", error);
        }
    };

    if (picture) {
        return (
            <View style={{ flex: 1 }}>
                <Image
                    key={picture.uri}
                    source={{ uri: picture.uri }}
                    style={StyleSheet.absoluteFill}
                />
                <AntDesign
                    style={styles.close}
                    name="close" size={24}
                    color="black"
                    onPress={() => {
                        setPicture(undefined)
                    }}
                />
            </View>
        )
    }

    return (
        <View>
            <CameraView ref={camera} style={styles.camera} facing={facing} mirror={true}>

                <View style={styles.footer}>
                    <View style={{ width: 25 }} />
                    <Pressable style={styles.recordButton} onPress={takePicture} />

                    <Ionicons
                        name="camera-reverse-sharp"
                        size={24}
                        color="white"
                        onPress={toggleCameraFacing}
                    />
                </View>
            </CameraView>
            <AntDesign
                style={styles.close}
                name="close" size={24}
                color="white"
                onPress={() => router.back()}
            />

            <Link href={'/'}>Home</Link>
        </View>
    )
}

export default CameraScreen
const styles = StyleSheet.create({
    camera: {
        width: '100%',
        height: "100%"
    },
    close: {
        position: 'absolute',
        top: 45,
        left: 30
    },
    footer: {
        marginTop: 'auto',
        padding: 20,
        paddingBottom: 50,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        backgroundColor: '#00000099'
    },
    recordButton: {
        width: 50,
        height: 50,
        backgroundColor: 'white',
        borderRadius: 50
    }
})
```
---


---


---

## 09__expo-file-system

```jsx
	import * as FS from 'expo-file-system';

    const saveFile = async (uri: string) => {
        //save file
        const filename = uri.split('/').pop() || 'photo.jpg'
        const source = new FS.File(uri)
        const destination = new FS.File(FS.Paths.document, filename)

        await source.copy(destination)
    }
```

```jsx
    const loadFile = async () => {
        if (!FS.Paths.document) {
            return;
        }
        const res = await FS.Paths.document.list()    
    }
```

```jsx
	const { name } = useLocalSearchParams<{ name: string }>()
    const fullUri = (FS.Paths.document.uri || '') + (name || '')
    
    const onDelete = async () => {
        const targetFile = new FS.File(fullUri)
        await targetFile.delete()
        router.back()
    }
```
---


---


---

## 10__useFocusEffect

`useFocusEffect` is used to run side effects **every time a screen comes into focus**

`useEffect`, it only runs when the component mounts. In a tab or stack navigator, screens stay mounted in the background, so `useEffect` won't trigger again when you navigate back to them
```jsx
import React, { useState, useCallback, useRef } from 'react';
import { Video } from 'expo-av'; // Or react-native-video
import { useFocusEffect } from '@react-navigation/native';

export default function VideoScreen() {
  const videoRef = useRef(null);
  const [shouldPlay, setShouldPlay] = useState(false);

  useFocusEffect(
    useCallback(() => {
      // Screen comes into view: Play video
      setShouldPlay(true);

      return () => {
        // Screen goes out of view: Pause video
        setShouldPlay(false);
      };
    }, [])
  );

  return (
    <Video
      ref={videoRef}
      source={{ uri: 'https://example.com/video.mp4' }}
      shouldPlay={shouldPlay}
      style={{ width: '100%', height: 300 }}
      resizeMode="contain"
    />
  );
}
```
---


---


---

## 11__Stack

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
---


---


---

## 12_Flatlist

```jsx
<FlatList
  // --- Core Data & Rendering ---
  data={dataArray} // Array of items to render
  renderItem={({ item, index, separators }) => (
    <CustomItem item={item} index={index} />
  )} // Function that renders each item
  keyExtractor={(item, index) => item.id.toString()} // Unique key for each item

  // --- Layout & Orientation ---
  horizontal={false} // Render items horizontally instead of vertically
  numColumns={1} // Number of columns for grid layout
  columnWrapperStyle={{
    justifyContent: 'space-between',
  }} // Style for each row when using numColumns
  inverted={false} // Reverses scroll direction
  contentContainerStyle={{
    padding: 16,
  }} // Styling for inner content container
  style={{
    flex: 1,
  }} // Style for FlatList itself

  // --- Header & Footer ---
  ListHeaderComponent={<Header />} // Component rendered at top
  ListHeaderComponentStyle={{
    marginBottom: 10,
  }} // Style for header
  ListFooterComponent={<Footer />} // Component rendered at bottom
  ListFooterComponentStyle={{
    marginTop: 10,
  }} // Style for footer

  // --- Empty State ---
  ListEmptyComponent={<Empty />} // Rendered when data is empty

  // --- Separators ---
  ItemSeparatorComponent={() => (
    <View style={{ height: 10 }} />
  )} // Separator between items

  // --- Pull To Refresh ---
  refreshing={loading} // Controls refresh spinner
  onRefresh={handleRefresh} // Pull-to-refresh callback

  // --- Infinite Scrolling ---
  onEndReached={loadMore} // Triggered when near end
  onEndReachedThreshold={0.5} // Distance from end before triggering

  // --- Scrolling Behavior ---
  showsVerticalScrollIndicator={false} // Hide vertical scrollbar
  showsHorizontalScrollIndicator={false} // Hide horizontal scrollbar
  pagingEnabled={false} // Snap scrolling page-by-page
  snapToAlignment="start" // Snap alignment
  decelerationRate="fast" // Scroll speed behavior
  snapToInterval={300} // Snap every X pixels

  // --- Initial Rendering & Performance ---
  initialNumToRender={10} // Initial items rendered
  maxToRenderPerBatch={10} // Items rendered per batch
  windowSize={21} // Number of screens rendered outside viewport
  removeClippedSubviews={true} // Unmount offscreen views
  updateCellsBatchingPeriod={50} // Delay between render batches
  getItemLayout={(data, index) => ({
    length: 80,
    offset: 80 * index,
    index,
  })} // Optimization for fixed-height items

  // --- Re-render Control ---
  extraData={selectedId} // Forces FlatList re-render on external state change

  // --- Scroll Position ---
  initialScrollIndex={0} // Start from specific item
  scrollEnabled={true} // Enable/disable scrolling

  // --- Keyboard Handling ---
  keyboardShouldPersistTaps="handled" // Keyboard behavior on tap

  // --- Viewability Tracking ---
  onViewableItemsChanged={({ viewableItems, changed }) => {
    console.log(viewableItems);
  }} // Detect visible items

  viewabilityConfig={{
    itemVisiblePercentThreshold: 50,
  }} // Visibility rules

  // --- Multi Touch & Interaction ---
  disableVirtualization={false} // Disable virtualization (not recommended)
  bounces={true} // iOS bounce effect
  alwaysBounceVertical={false}
  alwaysBounceHorizontal={false}

  // --- Scroll Events ---
  onScroll={(event) => {
    console.log(event.nativeEvent.contentOffset.y);
  }} // Scroll listener

  scrollEventThrottle={16} // Scroll event frequency

  // --- Sticky Headers ---
  stickyHeaderIndices={[0]} // Sticky header indexes

  // --- Custom Refresh Control ---
  refreshControl={
    <RefreshControl
      refreshing={loading}
      onRefresh={handleRefresh}
    />
  }

  // --- Accessibility ---
  accessible={true}
  accessibilityLabel="User list"

  // --- Debugging ---
  debug={false} // Debug virtualization

/>
```
---


---


---

