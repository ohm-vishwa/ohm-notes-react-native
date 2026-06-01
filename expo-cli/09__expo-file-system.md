```jsx
	import * as FS from 'expo-file-system';

    const saveFile = async (uri: string) => {
        //save file
        const filename = uri.split('/').pop() || 'photo.jpg'
        const source = new FS.File(uri)
        const destination = new FS.File(FS.Paths.document, filename)

        await source.copy(destination)
    }
```

```jsx
    const loadFile = async () => {
        if (!FS.Paths.document) {
            return;
        }
        const res = await FS.Paths.document.list()    
    }
```

```jsx
	const { name } = useLocalSearchParams<{ name: string }>()
    const fullUri = (FS.Paths.document.uri || '') + (name || '')
    
    const onDelete = async () => {
        const targetFile = new FS.File(fullUri)
        await targetFile.delete()
        router.back()
    }
```