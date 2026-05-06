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