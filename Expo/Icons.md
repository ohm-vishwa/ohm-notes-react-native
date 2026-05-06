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