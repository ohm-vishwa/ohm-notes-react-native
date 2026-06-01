https://docs.expo.dev/versions/latest/sdk/camera/

```sh
npx expo install expo-camera
```

`app.json`
```json
{
  "expo": {
    "plugins": [
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access your camera",
          "microphonePermission": "Allow $(PRODUCT_NAME) to access your microphone",
          "recordAudioAndroid": true,
          "barcodeScannerEnabled": true
        }
      ]
    ]
  }
}

```

`Usage`
```jsx
const CameraScreen = () => {
    const [permission, requestPermission] = useCameraPermissions()
    const [facing, setFacing] = useState<CameraType>('back')
    const [picture, setPicture] = useState<CameraCapturedPicture>()

    const camera = useRef<CameraView>(null)

    useEffect(() => {
        if (permission && !permission.granted && permission.canAskAgain) {
            requestPermission()
        }
    }, [permission])


    const toggleCameraFacing = () => {
        setFacing((current) => (current === 'back' ? 'front' : 'back'));
    }

    if (!permission?.granted) {
        return <ActivityIndicator size={30} />
    }

    const takePicture = async () => {
        if (!camera.current) return;

        try {
            const options = { quality: 1.0, skipProcessing: false };
            const res = await camera.current.takePictureAsync(options);

            if (res && res.uri) {
                console.log("Picture saved to cache:", res.uri);

                setTimeout(() => {
                    setPicture(res);
                }, 100);

            }
        } catch (error) {
            console.error("Capture error:", error);
        }
    };

    if (picture) {
        return (
            <View style={{ flex: 1 }}>
                <Image
                    key={picture.uri}
                    source={{ uri: picture.uri }}
                    style={StyleSheet.absoluteFill}
                />
                <AntDesign
                    style={styles.close}
                    name="close" size={24}
                    color="black"
                    onPress={() => {
                        setPicture(undefined)
                    }}
                />
            </View>
        )
    }

    return (
        <View>
            <CameraView ref={camera} style={styles.camera} facing={facing} mirror={true}>

                <View style={styles.footer}>
                    <View style={{ width: 25 }} />
                    <Pressable style={styles.recordButton} onPress={takePicture} />

                    <Ionicons
                        name="camera-reverse-sharp"
                        size={24}
                        color="white"
                        onPress={toggleCameraFacing}
                    />
                </View>
            </CameraView>
            <AntDesign
                style={styles.close}
                name="close" size={24}
                color="white"
                onPress={() => router.back()}
            />

            <Link href={'/'}>Home</Link>
        </View>
    )
}

export default CameraScreen
const styles = StyleSheet.create({
    camera: {
        width: '100%',
        height: "100%"
    },
    close: {
        position: 'absolute',
        top: 45,
        left: 30
    },
    footer: {
        marginTop: 'auto',
        padding: 20,
        paddingBottom: 50,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        backgroundColor: '#00000099'
    },
    recordButton: {
        width: 50,
        height: 50,
        backgroundColor: 'white',
        borderRadius: 50
    }
})
```