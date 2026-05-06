```sh
npx create-expo-app@latest BusinessCard --template 
```
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