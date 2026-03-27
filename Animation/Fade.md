```jsx
import { Animated, Button, StyleSheet, Text, View } from 'react-native'
import React, { useRef } from 'react'
import { SafeAreaView } from 'react-native-safe-area-context'

export default function Animation01() {
    const value = useRef(new Animated.Value(0)).current;
    const FadeIn = () => {
        Animated.timing(value, {
            toValue: 1,
            duration: 2000,
            useNativeDriver: false,

        }).start()
    }
    const FadeOut = () => {
        Animated.timing(value, {
            toValue: 0,
            duration: 2000,
            useNativeDriver: false,

        }).start()
    }
    return (
        <SafeAreaView style={{ flex: 1, backgroundColor: 'white', alignItems: 'center' }}>
            <Animated.View style={[styles.box, { opacity: value }]}></Animated.View>
            <View style={{ width: 100, alignSelf: 'center' }}>
                <Button title='fade In' onPress={FadeIn} />
            </View>
            <View style={{ marginTop: 30, alignSelf: 'center' }}>
                <Button title='fade out' onPress={FadeOut} />
            </View>
        </SafeAreaView>
    )
}



const styles = StyleSheet.create({
    box: {
        width: 50,
        height: 50,
        backgroundColor: 'red',
        marginTop: 20,
        borderRadius: 5,
        marginBottom: 30
    }
})
```