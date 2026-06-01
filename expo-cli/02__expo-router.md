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


