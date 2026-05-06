```sh
npx create-expo-app@latest BusinessCard --template 
```

---

best way to install packages or libraries in expo
```
npm install react-native-safe-area-context ❌
```
---
installs tested version for expo app
```
npx expo install react-native-safe-area-context ✅
```

---

`jsconfig.json`
```sh
 {
    "compilerOptions": {
        "module": "ESNext",
        "moduleResolution": "Bundler",
        "target": "ES2024",
        "jsx": "react-jsx",
        "allowImportingTsExtensions": true,
        "checkJs": false,
        "experimentalDecorators": false,
        "strictNullChecks": true,
        "strictFunctionTypes": true,
        "strict": true
    },
    "exclude": [
        "node_modules",
        "**/node_modules/*"
    ]
}
```