```jsx
import { Animated, Button, StyleSheet, Text, View } from 'react-native'
import React, { useRef } from 'react'
import { SafeAreaView } from 'react-native-safe-area-context'

export default function Animation01() {
    const value = useRef(new Animated.Value(20)).current;
    const moveBoxR = () => {
        Animated.timing(value, {
            toValue: 300,
            duration: 1000,
            useNativeDriver: false, //false JS thread, true UI thread

        }).start()
    }
    const moveBoxL = () => {
        Animated.timing(value, {
            toValue: 20,
            duration: 1000,
            useNativeDriver: false, 

        }).start()
    }
    return (
        <SafeAreaView style={{ flex: 1, backgroundColor: 'white' }}>
            <Animated.View style={[styles.box, { left: value }]}></Animated.View>
            <View style={{ width: 100, alignSelf: 'center' }}>
                <Button title='Move Right' onPress={moveBoxR} />
            </View>
            <View style={{ marginTop: 30, alignSelf: 'center' }}>
                <Button title='Move Left' onPress={moveBoxL} />
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
        left: 20,
        borderRadius: 5,
        marginBottom: 30
    },
    button: {

    }
})
```