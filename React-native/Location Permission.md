```jsx
const requestLocation = async () => {

        const askPermission = async () => {
            if (Platform.OS === "android") {
                const granted = await PermissionsAndroid.request(
                    PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION
                );
                return granted === PermissionsAndroid.RESULTS.GRANTED;
            } else {
                const auth = await Geolocation.requestAuthorization("whenInUse");
                return auth === "granted";
            }
        };

        let permissionGranted = false;

        while (!permissionGranted) {
            permissionGranted = await askPermission();

            if (!permissionGranted) {
                const userAction = await new Promise((resolve) => {
                    Alert.alert(
                        "Location Required",
                        "Please allow location access",
                        [
                            {
                                text: "Retry",
                                onPress: () => {
                                    resolve("retry")
                                },
                            },
                            {
                                text: "Open Settings",
                                onPress: () => resolve("settings"),
                            },
                        ]
                    );
                });

                if (userAction === "settings") {
                    Linking.openSettings();
                    return;
                }
            }
        }
    };
```