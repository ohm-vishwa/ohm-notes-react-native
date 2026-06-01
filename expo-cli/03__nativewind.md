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