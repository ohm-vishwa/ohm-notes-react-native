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