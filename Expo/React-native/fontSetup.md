1. insert fonts into folder `src/assets/fonts`
2. insert fonts into `android/app/src/main/assets/fonts`
3. create file in main folder `react-native-config.js`
```js
module.exports = {
    project: {
        ios: {},
        android: {},
        assets: ["./src/assets/fonts"],
    }
}
```
4. install package `react-native-asset`
```sh
npm install react-native-asset
```

5. `constant/Fontfamily` example
```sh
export default {
    regular: "Poppins-Regular",
    medium: "Poppins-Medium",
    semiBold: "Poppins-SemiBold",
    bold: "Poppins-Bold",
}
```
6. Link the assets
```bash
npx react-native-asset
```
```sh
```