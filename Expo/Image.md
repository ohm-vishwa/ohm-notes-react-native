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
