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