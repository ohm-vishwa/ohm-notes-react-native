# Modern `expo-file-system` Notes (SDK 54+) — New API

Expo SDK 54 introduced the new object-based FileSystem API using:

```js
import {
  File,
  Directory,
  Paths,
} from 'expo-file-system';
```

The new API is:

- Object-oriented
    
- Faster
    
- Cleaner
    
- Blob compatible
    
- Better for React Native New Architecture
    

Expo moved the old API to:

```js
import * as FileSystem from 'expo-file-system/legacy';
```

([docs.expo.dev](https://docs.expo.dev/versions/v54.0.0/sdk/filesystem)) ([Expo Documentation](https://docs.expo.dev/versions/v54.0.0/sdk/filesystem?utm_source=chatgpt.com "FileSystem - Expo Documentation"))

---

# Installation

```bash
npx expo install expo-file-system
```

---

# Import

```js
import {
  File,
  Directory,
  Paths,
} from 'expo-file-system';
```

---

# Main Concepts

|Class|Purpose|
|---|---|
|`File`|Represents a file|
|`Directory`|Represents a folder|
|`Paths`|Built-in system paths|

---

# Paths

```js
Paths.cache
```

Temporary storage.

---

```js
Paths.document
```

Permanent storage.

---

```js
Paths.bundle
```

Read-only bundled app assets.

Expo says `Paths.cache` and `Paths.document` replace the old `cacheDirectory` and `documentDirectory`. ([Expo Documentation](https://docs.expo.dev/versions/v54.0.0/sdk/filesystem?utm_source=chatgpt.com "FileSystem - Expo Documentation"))

---

# Create File

```js
import {
  File,
  Paths,
} from 'expo-file-system';

const file = new File(
  Paths.cache,
  'hello.txt'
);

file.create();

file.write('Hello World');
```

---

# Read File

## Async

```js
const text = await file.text();

console.log(text);
```

---

## Sync

```js
const text = file.textSync();

console.log(text);
```

---

# Complete `File` Notes

```jsx
import {
  File,
  Paths,
} from 'expo-file-system';

// =====================================================
// CREATE FILE
// =====================================================

const file = new File(
  Paths.cache,
  'demo.txt'
);

file.create();

// =====================================================
// WRITE TEXT
// =====================================================

file.write('Hello World');

// =====================================================
// READ TEXT
// =====================================================

await file.text();

file.textSync();

// =====================================================
// READ BYTES
// =====================================================

await file.bytes();

file.bytesSync();

// =====================================================
// ARRAY BUFFER
// =====================================================

await file.arrayBuffer();

// =====================================================
// BASE64
// =====================================================

await file.base64();

file.base64Sync();

// =====================================================
// FILE INFO
// =====================================================

file.exists;

file.size;

file.name;

file.type;

file.uri;

file.creationTime;

file.modificationTime;

file.md5;

// =====================================================
// DELETE FILE
// =====================================================

file.delete();

// =====================================================
// COPY FILE
// =====================================================

const copiedFile = new File(
  Paths.document,
  'copy.txt'
);

file.copy(copiedFile);

// =====================================================
// MOVE FILE
// =====================================================

file.move(Paths.document);

// OR

file.move(
  new Directory(
    Paths.document,
    'downloads'
  )
);

// =====================================================
// RENAME FILE
// =====================================================

file.rename('new-name.txt');

// =====================================================
// FILE INFO OBJECT
// =====================================================

await file.info();

// =====================================================
// OPEN FILE HANDLE
// =====================================================

const handle = await file.openHandleAsync();

// =====================================================
// STREAMS
// =====================================================

const stream = file.readableStream();

const writable = file.writableStream();
```

---

# Create Directory

```js
import {
  Directory,
  Paths,
} from 'expo-file-system';

const dir = new Directory(
  Paths.document,
  'downloads'
);

dir.create();
```

---

# Complete `Directory` Notes

```jsx
import {
  Directory,
  File,
  Paths,
} from 'expo-file-system';

// =====================================================
// CREATE DIRECTORY
// =====================================================

const dir = new Directory(
  Paths.document,
  'images'
);

dir.create();

// =====================================================
// CREATE NESTED DIRECTORY
// =====================================================

const nestedDir = new Directory(
  Paths.document,
  'media',
  'videos'
);

nestedDir.create();

// =====================================================
// CHECK EXISTS
// =====================================================

dir.exists;

// =====================================================
// DIRECTORY INFO
// =====================================================

dir.name;

dir.uri;

dir.size;

// =====================================================
// GET INFO
// =====================================================

await dir.info();

// =====================================================
// LIST FILES
// =====================================================

const items = dir.list();

// =====================================================
// CREATE FILE INSIDE DIRECTORY
// =====================================================

const imageFile =
  dir.createFile(
    'photo.jpg',
    'image/jpeg'
  );

// =====================================================
// CREATE SUB DIRECTORY
// =====================================================

const subDir =
  dir.createDirectory('backup');

// =====================================================
// COPY DIRECTORY
// =====================================================

dir.copy(
  new Directory(
    Paths.cache,
    'copied-folder'
  )
);

// =====================================================
// MOVE DIRECTORY
// =====================================================

dir.move(Paths.cache);

// =====================================================
// RENAME DIRECTORY
// =====================================================

dir.rename('new-folder');

// =====================================================
// DELETE DIRECTORY
// =====================================================

dir.delete();
```

---

# Download File

```js
import {
  Directory,
  File,
  Paths,
} from 'expo-file-system';

const url =
  'https://example.com/file.pdf';

const destination =
  new Directory(
    Paths.cache,
    'pdfs'
  );

destination.create();

const downloadedFile =
  await File.downloadFileAsync(
    url,
    destination
  );

console.log(downloadedFile.uri);
```

Expo recommends `File.downloadFileAsync()` in the new API. ([Expo Documentation](https://docs.expo.dev/versions/v54.0.0/sdk/filesystem?utm_source=chatgpt.com "FileSystem - Expo Documentation"))

---

# Upload File

Use `expo/fetch`.

```js
import { fetch } from 'expo/fetch';

const response = await fetch(
  'https://example.com/upload',
  {
    method: 'POST',
    body: file,
  }
);
```

The new `File` class implements `Blob`, so it works directly with `fetch`. ([Expo Documentation](https://docs.expo.dev/versions/v54.0.0/sdk/filesystem?utm_source=chatgpt.com "FileSystem - Expo Documentation"))

---

# Upload Using FormData

```js
import { fetch } from 'expo/fetch';

const formData = new FormData();

formData.append(
  'image',
  file
);

await fetch(
  'https://example.com/upload',
  {
    method: 'POST',
    body: formData,
  }
);
```

---

# Pick File (Android)

```js
const pickedFile =
  await File.pickFileAsync();

console.log(pickedFile.name);
```

---

# Pick Directory

```js
const dir =
  await Directory.pickDirectoryAsync();
```

---

# Save Image Permanently

```js
const savedFile = new File(
  Paths.document,
  'profile.jpg'
);

tempFile.copy(savedFile);
```

---

# Read Folder Recursively

```js
function printDir(
  directory,
  indent = 0
) {
  console.log(
    ' '.repeat(indent) +
      directory.name
  );

  const items = directory.list();

  for (const item of items) {
    if (item instanceof Directory) {
      printDir(item, indent + 2);
    } else {
      console.log(
        ' '.repeat(indent + 2) +
          item.name
      );
    }
  }
}
```

Expo documentation shows recursive directory listing using `Directory.list()`. ([Expo Documentation](https://docs.expo.dev/versions/v54.0.0/sdk/filesystem?utm_source=chatgpt.com "FileSystem - Expo Documentation"))

---

# File Properties

|Property|Description|
|---|---|
|`exists`|File exists or not|
|`size`|File size|
|`name`|File name|
|`uri`|File path|
|`md5`|MD5 hash|
|`type`|MIME type|
|`creationTime`|Created date|
|`modificationTime`|Last modified|

---

# Directory Properties

|Property|Description|
|---|---|
|`exists`|Directory exists|
|`size`|Total size|
|`name`|Folder name|
|`uri`|Directory path|

---

# Real-World Examples

## Cache Image

```js
const image = await File.downloadFileAsync(
  imageUrl,
  new Directory(
    Paths.cache,
    'images'
  )
);
```

---

## Offline PDF Reader

```js
const pdfDir = new Directory(
  Paths.document,
  'pdfs'
);

pdfDir.create();
```

---

## Audio Recorder Save

```js
const audioFile = new File(
  Paths.document,
  'recording.m4a'
);
```

---

# Old vs New API

|Old API|New API|
|---|---|
|`readAsStringAsync()`|`file.text()`|
|`writeAsStringAsync()`|`file.write()`|
|`makeDirectoryAsync()`|`directory.create()`|
|`moveAsync()`|`file.move()`|
|Function-based|Object-based|

Expo officially states the new API is now stable in SDK 54 and the old API moved to `/legacy`. ([Expo](https://expo.dev/changelog/sdk-54?utm_source=chatgpt.com "Expo SDK 54 - Expo Changelog"))

---

# Common Errors

## File Already Exists

```js
file.create();
```

Can throw error if file exists.

Check first:

```js
if (!file.exists) {
  file.create();
}
```

---

## Wrong Class

This causes error:

```js
new File(existingDirectoryPath)
```

Use correct type:

```js
new Directory(path)
```

Expo docs mention constructors throw if wrong class type is used for an existing path. ([Expo Documentation](https://docs.expo.dev/versions/v54.0.0/sdk/filesystem?utm_source=chatgpt.com "FileSystem - Expo Documentation"))

---

# Android SAF Notes

Android uses Storage Access Framework.

Picked directories may return:

```txt
content://
```

URIs instead of:

```txt
file://
```

The new API supports SAF better than legacy API. ([Expo](https://expo.dev/blog/expo-file-system?utm_source=chatgpt.com "Expo File System gets a major upgrade in SDK 54"))

---

# Best Practices

- Use `Paths.cache` for temporary files
    
- Use `Paths.document` for permanent storage
    
- Delete unused cache files
    
- Avoid huge Base64 conversions
    
- Use `arrayBuffer()` instead of fetch(fileUri)
    
- Prefer object API over legacy API
    
- Store large media in cache
    

Community reports show `File.arrayBuffer()` is safer than `fetch(fileUri)` on newer Android versions. ([Reddit](https://www.reddit.com/r/expo/comments/1sfshtx/sdk_55_fetchfileuri_failing_on_android_with/?utm_source=chatgpt.com "[SDK 55] fetch(fileUri) failing on Android with \"Network request failed\" after upgrading to Expo 55 / RN 0.83"))

---

# Recommended Expo Libraries

- [Expo Image Picker](https://docs.expo.dev/versions/latest/sdk/imagepicker/?utm_source=chatgpt.com)
    
- [Expo Document Picker](https://docs.expo.dev/versions/latest/sdk/document-picker/?utm_source=chatgpt.com)
    
- [Expo Sharing](https://docs.expo.dev/versions/latest/sdk/sharing/?utm_source=chatgpt.com)
    
- [Expo Media Library](https://docs.expo.dev/versions/latest/sdk/media-library/?utm_source=chatgpt.com)
    

---

# Full Minimal Example

```jsx
import React from 'react';
import {
  View,
  Button,
} from 'react-native';

import {
  File,
  Directory,
  Paths,
} from 'expo-file-system';

export default function App() {
  const handleFile = async () => {
    // Create directory
    const dir = new Directory(
      Paths.document,
      'notes'
    );

    if (!dir.exists) {
      dir.create();
    }

    // Create file
    const file = new File(
      dir,
      'demo.txt'
    );

    if (!file.exists) {
      file.create();
    }

    // Write
    file.write(
      'Hello Expo FileSystem'
    );

    // Read
    const content =
      await file.text();

    console.log(content);
  };

  return (
    <View>
      <Button
        title="Create File"
        onPress={handleFile}
      />
    </View>
  );
}
```