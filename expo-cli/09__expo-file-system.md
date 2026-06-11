# Expo FileSystem — Complete Reference

A comprehensive reference for `expo-file-system`.  
Covers directories, file operations, downloads, uploads, Storage Access Framework (Android), and common patterns.

> **Install:**
> 
> ```bash
> npx expo install expo-file-system
> ```

---

## Table of Contents

1. [Directory Constants](#directory-constants)
2. [File Info](#file-info)
3. [Reading Files](#reading-files)
4. [Writing Files](#writing-files)
5. [Copying & Moving](#copying--moving)
6. [Deleting](#deleting)
7. [Directories](#directories)
8. [Downloading Files](#downloading-files)
9. [Resumable Downloads](#resumable-downloads)
10. [Uploading Files](#uploading-files)
11. [Encoding Types](#encoding-types)
12. [Storage Access Framework (Android)](#storage-access-framework-android)
13. [Background Sessions — iOS](#background-sessions--ios)
14. [Common Patterns](#common-patterns)

---

## Directory Constants

<details> <summary><code>FileSystem.documentDirectory</code></summary>

URI to the app's document directory. Files here **persist** across app restarts and OS-level cache clears. Back up to iCloud on iOS if the user enables it.

|||
|---|---|
|**Type**|`string \| null`|
|**Persists**|✅ Yes|
|**Backed up (iOS)**|✅ Yes|

```jsx
import * as FileSystem from 'expo-file-system';

const docDir = FileSystem.documentDirectory;
// iOS:     file:///var/mobile/.../Documents/
// Android: file:///data/user/0/com.app/files/

const filePath = `${FileSystem.documentDirectory}user-data.json`;
```

> ✅ Use for: user-created data, downloaded content the user should keep, app settings.

</details> <details> <summary><code>FileSystem.cacheDirectory</code></summary>

URI to the app's cache directory. Files here may be **deleted by the OS** when storage is low. Not backed up.

|||
|---|---|
|**Type**|`string \| null`|
|**Persists**|⚠️ May be cleared by OS|
|**Backed up (iOS)**|❌ No|

```jsx
const cacheDir = FileSystem.cacheDirectory;
// iOS:     file:///var/mobile/.../Caches/
// Android: file:///data/user/0/com.app/cache/

const cachedImage = `${FileSystem.cacheDirectory}thumbnail_${id}.jpg`;
```

> ✅ Use for: downloaded images, temp files, cached API responses.

</details> <details> <summary><code>FileSystem.bundleDirectory</code> · 🍎 iOS</summary>

URI to the app's bundle directory (read-only). Contains assets bundled at build time.

|||
|---|---|
|**Type**|`string \| null`|
|**Platform**|iOS|
|**Writable**|❌ Read-only|

```jsx
const bundleDir = FileSystem.bundleDirectory;
// file:///var/containers/Bundle/Application/.../
```

</details>

---

## File Info

<details> <summary><code>FileSystem.getInfoAsync(fileUri, options?)</code></summary>

Returns metadata about a file or directory — whether it exists, its size, modification time, and whether it is a directory.

|Option|Type|Default|Description|
|---|---|---|---|
|`md5`|`boolean`|`false`|Include MD5 hash of the file|
|`size`|`boolean`|`false`|Include file size in bytes|

**Returns:** `Promise<FileInfo>`

|FileInfo field|Type|Description|
|---|---|---|
|`exists`|`boolean`|Whether the file/directory exists|
|`uri`|`string`|URI of the item|
|`size`|`number`|File size in bytes (if `size: true` or file exists)|
|`isDirectory`|`boolean`|Whether it's a directory|
|`modificationTime`|`number`|Unix timestamp of last modification|
|`md5`|`string`|MD5 hash (if `md5: true`)|

```jsx
import * as FileSystem from 'expo-file-system';

const info = await FileSystem.getInfoAsync(`${FileSystem.documentDirectory}notes.txt`);

if (info.exists) {
  console.log('Size:', info.size);
  console.log('Modified:', new Date(info.modificationTime * 1000));
  console.log('Is directory:', info.isDirectory);
} else {
  console.log('File does not exist');
}

// With MD5 hash
const infoWithHash = await FileSystem.getInfoAsync(fileUri, { md5: true });
console.log('MD5:', infoWithHash.md5);
```

</details>

---

## Reading Files

<details> <summary><code>FileSystem.readAsStringAsync(fileUri, options?)</code></summary>

Reads the entire contents of a file as a string.

|Option|Type|Default|Description|
|---|---|---|---|
|`encoding`|`EncodingType`|`'utf8'`|`'utf8'` or `'base64'`|
|`position`|`number`|`0`|Byte offset to start reading from|
|`length`|`number`|whole file|Number of bytes to read|

**Returns:** `Promise<string>`

```jsx
// Read UTF-8 text file
const content = await FileSystem.readAsStringAsync(
  `${FileSystem.documentDirectory}notes.txt`
);
console.log(content);

// Read as base64 (for binary files, images)
const base64 = await FileSystem.readAsStringAsync(imageUri, {
  encoding: FileSystem.EncodingType.Base64,
});
const dataUri = `data:image/jpeg;base64,${base64}`;

// Read a slice (first 1024 bytes)
const chunk = await FileSystem.readAsStringAsync(fileUri, {
  encoding: FileSystem.EncodingType.UTF8,
  position: 0,
  length: 1024,
});
```

</details> <details> <summary><code>FileSystem.readDirectoryAsync(fileUri)</code></summary>

Returns an array of file and directory names inside the given directory URI. Returns names only (not full URIs).

**Returns:** `Promise<string[]>`

```jsx
const entries = await FileSystem.readDirectoryAsync(FileSystem.documentDirectory);
// ['notes.txt', 'images', 'settings.json']

// Build full URIs
const fullPaths = entries.map(name => `${FileSystem.documentDirectory}${name}`);
```

</details>

---

## Writing Files

<details> <summary><code>FileSystem.writeAsStringAsync(fileUri, contents, options?)</code></summary>

Writes a string to a file. Creates the file if it doesn't exist; overwrites it if it does.

|Option|Type|Default|Description|
|---|---|---|---|
|`encoding`|`EncodingType`|`'utf8'`|`'utf8'` or `'base64'`|

**Returns:** `Promise<void>`

```jsx
// Write a text file
await FileSystem.writeAsStringAsync(
  `${FileSystem.documentDirectory}notes.txt`,
  'Hello, World!'
);

// Write JSON
const data = { name: 'Alice', score: 42 };
await FileSystem.writeAsStringAsync(
  `${FileSystem.documentDirectory}data.json`,
  JSON.stringify(data)
);

// Write binary data from base64
await FileSystem.writeAsStringAsync(
  `${FileSystem.cacheDirectory}image.jpg`,
  base64ImageData,
  { encoding: FileSystem.EncodingType.Base64 }
);
```

</details>

---

## Copying & Moving

<details> <summary><code>FileSystem.copyAsync(options)</code></summary>

Copies a file or directory from one URI to another. The destination is overwritten if it already exists.

|Option|Type|Required|Description|
|---|---|---|---|
|`from`|`string`|✅|Source URI|
|`to`|`string`|✅|Destination URI|

**Returns:** `Promise<void>`

```jsx
// Copy a file
await FileSystem.copyAsync({
  from: `${FileSystem.cacheDirectory}downloaded.jpg`,
  to: `${FileSystem.documentDirectory}saved-photo.jpg`,
});

// Copy a directory
await FileSystem.copyAsync({
  from: `${FileSystem.documentDirectory}old-folder/`,
  to: `${FileSystem.documentDirectory}backup/`,
});

// Copy a bundled asset to document directory
await FileSystem.copyAsync({
  from: Asset.fromModule(require('./assets/template.json')).uri,
  to: `${FileSystem.documentDirectory}template.json`,
});
```

</details> <details> <summary><code>FileSystem.moveAsync(options)</code></summary>

Moves a file or directory. The original is deleted after a successful copy. Destination is overwritten if it exists.

|Option|Type|Required|Description|
|---|---|---|---|
|`from`|`string`|✅|Source URI|
|`to`|`string`|✅|Destination URI|

**Returns:** `Promise<void>`

```jsx
// Move a downloaded file to permanent storage
await FileSystem.moveAsync({
  from: `${FileSystem.cacheDirectory}temp_download.pdf`,
  to: `${FileSystem.documentDirectory}documents/report.pdf`,
});
```

</details>

---

## Deleting

<details> <summary><code>FileSystem.deleteAsync(fileUri, options?)</code></summary>

Deletes a file or directory. Deleting a directory also deletes all its contents recursively.

|Option|Type|Default|Description|
|---|---|---|---|
|`idempotent`|`boolean`|`false`|If `true`, does not throw if the file doesn't exist|

**Returns:** `Promise<void>`

```jsx
// Delete a file (throws if not found)
await FileSystem.deleteAsync(`${FileSystem.documentDirectory}old-notes.txt`);

// Safe delete — won't throw if already gone
await FileSystem.deleteAsync(
  `${FileSystem.cacheDirectory}temp/`,
  { idempotent: true }
);

// Delete a directory and all its contents
await FileSystem.deleteAsync(`${FileSystem.documentDirectory}exports/`);
```

</details>

---

## Directories

<details> <summary><code>FileSystem.makeDirectoryAsync(fileUri, options?)</code></summary>

Creates a directory at the given URI.

|Option|Type|Default|Description|
|---|---|---|---|
|`intermediates`|`boolean`|`false`|If `true`, creates all intermediate directories (like `mkdir -p`)|

**Returns:** `Promise<void>`

```jsx
// Create a single directory
await FileSystem.makeDirectoryAsync(
  `${FileSystem.documentDirectory}exports/`
);

// Create nested directories in one call
await FileSystem.makeDirectoryAsync(
  `${FileSystem.documentDirectory}projects/2024/january/`,
  { intermediates: true }
);
```

</details>

---

## Downloading Files

<details> <summary><code>FileSystem.downloadAsync(uri, fileUri, options?)</code></summary>

Downloads a file from a remote URL and saves it to a local URI. Simple one-shot download.

|Option|Type|Description|
|---|---|---|
|`headers`|`object`|HTTP request headers|
|`md5`|`boolean`|Verify MD5 after download|
|`sessionType`|`FileSystemSessionType`|iOS background session type|
|`cache`|`boolean`|Use cached response if available|

**Returns:** `Promise<FileSystemDownloadResult>`

|Result field|Type|Description|
|---|---|---|
|`uri`|`string`|Local URI of the saved file|
|`status`|`number`|HTTP status code|
|`headers`|`object`|Response headers|
|`md5`|`string`|MD5 hash (if requested)|

```jsx
import * as FileSystem from 'expo-file-system';

// Simple download
const result = await FileSystem.downloadAsync(
  'https://example.com/file.pdf',
  `${FileSystem.documentDirectory}report.pdf`
);
console.log('Saved to:', result.uri);
console.log('Status:', result.status);

// Download with auth headers
const result = await FileSystem.downloadAsync(
  'https://api.example.com/private/data.json',
  `${FileSystem.cacheDirectory}data.json`,
  { headers: { Authorization: `Bearer ${token}` } }
);

// Download with MD5 verification
const result = await FileSystem.downloadAsync(
  'https://example.com/archive.zip',
  `${FileSystem.cacheDirectory}archive.zip`,
  { md5: true }
);
console.log('MD5:', result.md5);
```

</details>

---

## Resumable Downloads

<details> <summary><code>FileSystem.createDownloadResumable(uri, fileUri, options?, callback?, resumeData?)</code></summary>

Creates a `DownloadResumable` object for downloads that can be paused, resumed, and tracked with progress. Use for large files.

|Parameter|Type|Description|
|---|---|---|
|`uri`|`string`|Remote URL to download|
|`fileUri`|`string`|Local path to save to|
|`options`|`object`|Same as `downloadAsync` options|
|`callback`|`function`|Progress callback `({ totalBytesWritten, totalBytesExpectedToWrite })`|
|`resumeData`|`string`|Serialized resume data from a previous session|

```jsx
const downloadResumable = FileSystem.createDownloadResumable(
  'https://example.com/large-video.mp4',
  `${FileSystem.cacheDirectory}video.mp4`,
  {},
  ({ totalBytesWritten, totalBytesExpectedToWrite }) => {
    const progress = totalBytesWritten / totalBytesExpectedToWrite;
    setDownloadProgress(progress);
  }
);
```

</details> <details> <summary><code>downloadResumable.downloadAsync()</code></summary>

Starts or resumes the download. Resolves when download completes.

**Returns:** `Promise<FileSystemDownloadResult | undefined>`

```jsx
try {
  const result = await downloadResumable.downloadAsync();
  if (result) {
    console.log('Download complete:', result.uri);
  }
} catch (error) {
  console.error('Download failed:', error);
}
```

</details> <details> <summary><code>downloadResumable.pauseAsync()</code></summary>

Pauses the download. Returns a result containing `resumeData` which can be saved to AsyncStorage for later resumption.

**Returns:** `Promise<FileSystemDownloadResult>`

```jsx
const pauseResult = await downloadResumable.pauseAsync();

// Save resume data to AsyncStorage for later
await AsyncStorage.setItem(
  'downloadResumeData',
  JSON.stringify(pauseResult)
);
```

</details> <details> <summary><code>downloadResumable.resumeAsync()</code></summary>

Resumes a paused download from where it left off.

**Returns:** `Promise<FileSystemDownloadResult | undefined>`

```jsx
await downloadResumable.resumeAsync();
```

</details> <details> <summary><code>downloadResumable.cancelAsync()</code></summary>

Cancels the download entirely.

**Returns:** `Promise<void>`

```jsx
await downloadResumable.cancelAsync();
```

</details> <details> <summary><code>downloadResumable.savable()</code></summary>

Returns a plain-object snapshot of the download that can be serialized to JSON and used to reconstruct the download later (e.g. after app restart).

**Returns:** `DownloadPauseState`

```jsx
const savableState = downloadResumable.savable();
await AsyncStorage.setItem('download', JSON.stringify(savableState));

// Later, to reconstruct:
const saved = JSON.parse(await AsyncStorage.getItem('download'));
const resumed = FileSystem.createDownloadResumable(
  saved.url,
  saved.fileUri,
  saved.options,
  progressCallback,
  saved.resumeData  // pass resume data here
);
await resumed.downloadAsync();
```

</details>

---

## Uploading Files

<details> <summary><code>FileSystem.uploadAsync(url, fileUri, options?)</code></summary>

Uploads a local file to a remote URL. Supports both binary and multipart form uploads.

|Option|Type|Default|Description|
|---|---|---|---|
|`httpMethod`|`string`|`'POST'`|HTTP method: `'POST'`, `'PUT'`, `'PATCH'`|
|`uploadType`|`FileSystemUploadType`|`BINARY_CONTENT`|`BINARY_CONTENT` or `MULTIPART`|
|`fieldName`|`string`|`'file'`|Field name (multipart only)|
|`mimeType`|`string`|auto-detected|MIME type of the file|
|`headers`|`object`|`{}`|Custom request headers|
|`parameters`|`object`|`{}`|Extra form fields (multipart only)|
|`sessionType`|`FileSystemSessionType`|—|iOS background session type|

**Returns:** `Promise<FileSystemUploadResult>`

|Result field|Type|Description|
|---|---|---|
|`status`|`number`|HTTP response status code|
|`headers`|`object`|Response headers|
|`body`|`string`|Response body|

```jsx
import * as FileSystem from 'expo-file-system';

// Binary upload (PUT)
const result = await FileSystem.uploadAsync(
  'https://api.example.com/upload',
  `${FileSystem.documentDirectory}photo.jpg`,
  {
    httpMethod: 'PUT',
    uploadType: FileSystem.FileSystemUploadType.BINARY_CONTENT,
    headers: {
      'Content-Type': 'image/jpeg',
      Authorization: `Bearer ${token}`,
    },
  }
);
console.log('Response:', result.status, result.body);

// Multipart form upload
const result = await FileSystem.uploadAsync(
  'https://api.example.com/photos',
  `${FileSystem.documentDirectory}photo.jpg`,
  {
    httpMethod: 'POST',
    uploadType: FileSystem.FileSystemUploadType.MULTIPART,
    fieldName: 'photo',
    mimeType: 'image/jpeg',
    parameters: {
      userId: '123',
      album: 'vacation',
    },
  }
);
```

</details> <details> <summary><code>FileSystem.createUploadTask(url, fileUri, options?, callback?)</code></summary>

Creates an `UploadTask` object — similar to `DownloadResumable` but for uploads. Provides progress tracking.

|Callback param|Type|Description|
|---|---|---|
|`totalBytesSent`|`number`|Bytes uploaded so far|
|`totalBytesExpectedToSend`|`number`|Total file size|

```jsx
const uploadTask = FileSystem.createUploadTask(
  'https://api.example.com/upload',
  fileUri,
  {
    httpMethod: 'POST',
    uploadType: FileSystem.FileSystemUploadType.MULTIPART,
    fieldName: 'file',
  },
  ({ totalBytesSent, totalBytesExpectedToSend }) => {
    setUploadProgress(totalBytesSent / totalBytesExpectedToSend);
  }
);

const result = await uploadTask.uploadAsync();
console.log('Upload status:', result.status);
```

</details>

---

## Encoding Types

<details> <summary><code>FileSystem.EncodingType</code></summary>

Enum used for `readAsStringAsync` and `writeAsStringAsync`.

|Value|String|Use case|
|---|---|---|
|`FileSystem.EncodingType.UTF8`|`'utf8'`|Plain text, JSON, XML, CSV|
|`FileSystem.EncodingType.Base64`|`'base64'`|Images, binary files, PDFs|

```jsx
// Write text
await FileSystem.writeAsStringAsync(path, 'Hello', {
  encoding: FileSystem.EncodingType.UTF8,
});

// Write binary as base64
await FileSystem.writeAsStringAsync(path, base64String, {
  encoding: FileSystem.EncodingType.Base64,
});

// Read as base64
const b64 = await FileSystem.readAsStringAsync(imagePath, {
  encoding: FileSystem.EncodingType.Base64,
});
```

</details> <details> <summary><code>FileSystem.FileSystemUploadType</code></summary>

Enum for upload type in `uploadAsync`.

|Value|Description|
|---|---|
|`FileSystem.FileSystemUploadType.BINARY_CONTENT`|Raw binary upload — Content-Type header determines format|
|`FileSystem.FileSystemUploadType.MULTIPART`|Multipart form-data — like an HTML form file upload|

```jsx
// Binary (for REST APIs expecting raw body)
uploadType: FileSystem.FileSystemUploadType.BINARY_CONTENT

// Multipart (for form-based file uploads)
uploadType: FileSystem.FileSystemUploadType.MULTIPART
```

</details> <details> <summary><code>FileSystem.FileSystemSessionType</code> · 🍎 iOS</summary>

Controls whether downloads/uploads continue in the background on iOS.

|Value|Description|
|---|---|
|`FileSystem.FileSystemSessionType.BACKGROUND`|Continues when app is backgrounded|
|`FileSystem.FileSystemSessionType.FOREGROUND`|Stops when app is backgrounded (default)|

```jsx
await FileSystem.downloadAsync(url, fileUri, {
  sessionType: FileSystem.FileSystemSessionType.BACKGROUND,
});
```

</details>

---

## Storage Access Framework (Android)

<details> <summary>Overview — what SAF is and when to use it</summary>

Android's **Storage Access Framework (SAF)** allows apps to access files in any location the user picks — including external storage, SD cards, and cloud providers — without requiring broad `READ_EXTERNAL_STORAGE` permissions.

Use SAF when you need to:

- Let users pick a directory on external storage
- Write to SD cards or USB drives
- Access files outside the app's sandbox

```jsx
import * as FileSystem from 'expo-file-system';
const SAF = FileSystem.StorageAccessFramework;
```

</details> <details> <summary><code>SAF.requestDirectoryPermissionsAsync(initialFileUri?)</code></summary>

Shows a system directory picker and requests read/write permission for the chosen directory. Returns the URI of the chosen directory.

**Returns:** `Promise<{ granted: boolean, directoryUri: string }>`

```jsx
const result = await FileSystem.StorageAccessFramework.requestDirectoryPermissionsAsync();

if (result.granted) {
  console.log('Access granted to:', result.directoryUri);
  // Store this URI in AsyncStorage for future use
  await AsyncStorage.setItem('externalDirUri', result.directoryUri);
} else {
  console.log('Permission denied');
}
```

</details> <details> <summary><code>SAF.readDirectoryAsync(dirUri)</code></summary>

Lists files and subdirectories in the given SAF directory. Returns an array of content URIs.

**Returns:** `Promise<string[]>`

```jsx
const dirUri = await AsyncStorage.getItem('externalDirUri');
const files = await FileSystem.StorageAccessFramework.readDirectoryAsync(dirUri);
// ['content://com.android.externalstorage.../file1.txt', ...]
```

</details> <details> <summary><code>SAF.createFileAsync(parentDirUri, fileName, mimeType)</code></summary>

Creates a new file inside a SAF directory. Returns the URI of the new file.

**Returns:** `Promise<string>`

```jsx
const fileUri = await FileSystem.StorageAccessFramework.createFileAsync(
  parentDirUri,
  'exported-data.csv',
  'text/csv'
);
console.log('Created:', fileUri);

// Then write to it
await FileSystem.writeAsStringAsync(fileUri, csvContent, {
  encoding: FileSystem.EncodingType.UTF8,
});
```

</details> <details> <summary><code>SAF.readAsStringAsync(uri, options?)</code></summary>

Reads a SAF file's contents as a string.

```jsx
const content = await FileSystem.StorageAccessFramework.readAsStringAsync(
  safFileUri,
  { encoding: FileSystem.EncodingType.UTF8 }
);
```

</details> <details> <summary><code>SAF.writeAsStringAsync(uri, contents, options?)</code></summary>

Writes string content to a SAF file.

```jsx
await FileSystem.StorageAccessFramework.writeAsStringAsync(
  safFileUri,
  JSON.stringify(data),
  { encoding: FileSystem.EncodingType.UTF8 }
);
```

</details> <details> <summary><code>SAF.copyAsync(options)</code></summary>

Copies a file to a SAF directory. `from` is a standard file URI; `to` is a SAF directory URI.

```jsx
await FileSystem.StorageAccessFramework.copyAsync({
  from: `${FileSystem.documentDirectory}notes.txt`,
  to: safDirUri,
});
```

</details> <details> <summary><code>SAF.moveAsync(options)</code></summary>

Moves a file to a SAF location, deleting the original.

```jsx
await FileSystem.StorageAccessFramework.moveAsync({
  from: `${FileSystem.cacheDirectory}export.json`,
  to: safDirUri,
});
```

</details> <details> <summary><code>SAF.deleteAsync(fileUri, options?)</code></summary>

Deletes a SAF file or directory.

```jsx
await FileSystem.StorageAccessFramework.deleteAsync(safFileUri);
// or idempotent:
await FileSystem.StorageAccessFramework.deleteAsync(safFileUri, { idempotent: true });
```

</details> <details> <summary><code>SAF.makeDirectoryAsync(parentUri, dirName)</code></summary>

Creates a subdirectory inside a SAF directory.

**Returns:** `Promise<string>` — URI of the new directory

```jsx
const newDirUri = await FileSystem.StorageAccessFramework.makeDirectoryAsync(
  parentSafUri,
  'Photos'
);
```

</details> <details> <summary><code>SAF.getContentUriAsync(fileUri)</code></summary>

Converts a standard `file://` URI to a `content://` URI for sharing via Android intents.

**Returns:** `Promise<string>`

```jsx
const contentUri = await FileSystem.StorageAccessFramework.getContentUriAsync(
  `${FileSystem.documentDirectory}report.pdf`
);
// content://com.expo.modules.filesystem.provider/...

// Use with Sharing
await Sharing.shareAsync(contentUri);
```

</details>

---

## Background Sessions — iOS

<details> <summary>Background downloads and uploads on iOS</summary>

On iOS, downloads/uploads can continue even when the app is backgrounded or terminated, using `FileSystemSessionType.BACKGROUND`.

```jsx
// In app.json / app.config.js — required for background tasks
{
  "expo": {
    "ios": {
      "infoPlist": {
        "UIBackgroundModes": ["fetch", "remote-notification"]
      }
    }
  }
}

// Background download
const result = await FileSystem.downloadAsync(
  'https://example.com/large-file.zip',
  `${FileSystem.documentDirectory}file.zip`,
  {
    sessionType: FileSystem.FileSystemSessionType.BACKGROUND,
    md5: true,
  }
);

// Background resumable download
const resumable = FileSystem.createDownloadResumable(
  url,
  fileUri,
  { sessionType: FileSystem.FileSystemSessionType.BACKGROUND },
  progressCallback
);
await resumable.downloadAsync();
```

</details>

---

## Common Patterns

<details> <summary>Check if a file exists before reading</summary>

```jsx
async function readFileIfExists(filename) {
  const path = `${FileSystem.documentDirectory}${filename}`;
  const info = await FileSystem.getInfoAsync(path);

  if (!info.exists) {
    console.log('File not found:', filename);
    return null;
  }

  return await FileSystem.readAsStringAsync(path);
}
```

</details> <details> <summary>Save and load JSON data</summary>

```jsx
const DATA_FILE = `${FileSystem.documentDirectory}app-data.json`;

// Save
async function saveData(data) {
  await FileSystem.writeAsStringAsync(DATA_FILE, JSON.stringify(data));
}

// Load
async function loadData(defaultValue = {}) {
  const info = await FileSystem.getInfoAsync(DATA_FILE);
  if (!info.exists) return defaultValue;

  const content = await FileSystem.readAsStringAsync(DATA_FILE);
  return JSON.parse(content);
}

// Usage
await saveData({ user: 'Alice', score: 42 });
const data = await loadData({ user: 'Guest', score: 0 });
```

</details> <details> <summary>Download and cache a remote image</summary>

```jsx
async function getCachedImage(url) {
  const filename = url.split('/').pop().split('?')[0]; // strip query params
  const cachedPath = `${FileSystem.cacheDirectory}${filename}`;

  const info = await FileSystem.getInfoAsync(cachedPath);
  if (info.exists) {
    return cachedPath; // return cached version
  }

  const result = await FileSystem.downloadAsync(url, cachedPath);
  if (result.status !== 200) {
    throw new Error(`Download failed: ${result.status}`);
  }
  return cachedPath;
}

// Usage
const localUri = await getCachedImage('https://example.com/hero.jpg');
<Image source={{ uri: localUri }} />
```

</details> <details> <summary>Resumable download with progress bar and persistence</summary>

```jsx
import AsyncStorage from '@react-native-async-storage/async-storage';

const RESUME_KEY = 'downloadResume';

function useResumableDownload(url, filename) {
  const [progress, setProgress] = useState(0);
  const [status, setStatus] = useState('idle'); // idle | downloading | paused | done
  const downloadRef = useRef(null);

  const fileUri = `${FileSystem.documentDirectory}${filename}`;

  const startOrResume = async () => {
    const saved = await AsyncStorage.getItem(RESUME_KEY);
    const resumeData = saved ? JSON.parse(saved).resumeData : undefined;

    downloadRef.current = FileSystem.createDownloadResumable(
      url, fileUri, {},
      ({ totalBytesWritten, totalBytesExpectedToWrite }) => {
        setProgress(totalBytesWritten / totalBytesExpectedToWrite);
      },
      resumeData
    );

    setStatus('downloading');
    try {
      const result = await downloadRef.current.downloadAsync();
      if (result) {
        setStatus('done');
        await AsyncStorage.removeItem(RESUME_KEY);
      }
    } catch (e) {
      setStatus('paused');
    }
  };

  const pause = async () => {
    if (!downloadRef.current) return;
    const pauseState = await downloadRef.current.pauseAsync();
    await AsyncStorage.setItem(RESUME_KEY, JSON.stringify(pauseState));
    setStatus('paused');
  };

  const cancel = async () => {
    if (!downloadRef.current) return;
    await downloadRef.current.cancelAsync();
    await AsyncStorage.removeItem(RESUME_KEY);
    setProgress(0);
    setStatus('idle');
  };

  return { progress, status, startOrResume, pause, cancel };
}
```

</details> <details> <summary>List all files in a directory with metadata</summary>

```jsx
async function listDirectory(dirUri = FileSystem.documentDirectory) {
  const info = await FileSystem.getInfoAsync(dirUri);
  if (!info.exists || !info.isDirectory) {
    throw new Error('Not a directory: ' + dirUri);
  }

  const names = await FileSystem.readDirectoryAsync(dirUri);

  const entries = await Promise.all(
    names.map(async (name) => {
      const uri = `${dirUri}${name}`;
      const meta = await FileSystem.getInfoAsync(uri, { size: true });
      return {
        name,
        uri,
        size: meta.size,
        isDirectory: meta.isDirectory,
        modified: meta.modificationTime
          ? new Date(meta.modificationTime * 1000).toLocaleDateString()
          : null,
      };
    })
  );

  return entries.sort((a, b) => a.name.localeCompare(b.name));
}
```

</details> <details> <summary>Copy a bundled asset to the document directory</summary>

```jsx
import { Asset } from 'expo-asset';
import * as FileSystem from 'expo-file-system';

async function copyAssetToDocument(module, filename) {
  const dest = `${FileSystem.documentDirectory}${filename}`;
  const info = await FileSystem.getInfoAsync(dest);
  if (info.exists) return dest; // already copied

  const asset = Asset.fromModule(module);
  await asset.downloadAsync();

  await FileSystem.copyAsync({ from: asset.localUri, to: dest });
  return dest;
}

// Usage
const path = await copyAssetToDocument(
  require('./assets/default-config.json'),
  'config.json'
);
```

</details> <details> <summary>Clean the cache directory</summary>

```jsx
async function cleanCache() {
  const info = await FileSystem.getInfoAsync(FileSystem.cacheDirectory);
  if (!info.exists) return;

  const files = await FileSystem.readDirectoryAsync(FileSystem.cacheDirectory);

  await Promise.all(
    files.map(file =>
      FileSystem.deleteAsync(`${FileSystem.cacheDirectory}${file}`, {
        idempotent: true,
      })
    )
  );

  console.log(`Deleted ${files.length} cached files`);
}
```

</details> <details> <summary>Upload a photo from image picker</summary>

```jsx
import * as ImagePicker from 'expo-image-picker';
import * as FileSystem from 'expo-file-system';

async function pickAndUpload() {
  const picked = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    quality: 0.8,
  });

  if (picked.canceled) return;

  const asset = picked.assets[0];
  setUploading(true);

  try {
    const result = await FileSystem.uploadAsync(
      'https://api.example.com/photos',
      asset.uri,
      {
        httpMethod: 'POST',
        uploadType: FileSystem.FileSystemUploadType.MULTIPART,
        fieldName: 'photo',
        mimeType: 'image/jpeg',
        headers: { Authorization: `Bearer ${token}` },
        parameters: { album: 'profile' },
      }
    );

    if (result.status === 200) {
      const { photoUrl } = JSON.parse(result.body);
      setAvatar(photoUrl);
    }
  } finally {
    setUploading(false);
  }
}
```

</details> <details> <summary>Export data to external storage (Android SAF)</summary>

```jsx
import * as FileSystem from 'expo-file-system';
const SAF = FileSystem.StorageAccessFramework;

async function exportToExternalStorage(data, filename) {
  // Request directory access (shows native folder picker)
  const { granted, directoryUri } = await SAF.requestDirectoryPermissionsAsync();
  if (!granted) return;

  // Create the file in the selected directory
  const fileUri = await SAF.createFileAsync(
    directoryUri,
    filename,
    'application/json'
  );

  // Write content to it
  await SAF.writeAsStringAsync(fileUri, JSON.stringify(data, null, 2), {
    encoding: FileSystem.EncodingType.UTF8,
  });

  Alert.alert('Exported', `Saved to ${filename}`);
}

// Usage
await exportToExternalStorage({ users: [...] }, 'backup.json');
```

</details> <details> <summary>Read a file in chunks (large files)</summary>

```jsx
async function readInChunks(fileUri, chunkSize = 1024 * 64) {
  const info = await FileSystem.getInfoAsync(fileUri, { size: true });
  const totalSize = info.size;
  const chunks = [];

  for (let position = 0; position < totalSize; position += chunkSize) {
    const length = Math.min(chunkSize, totalSize - position);
    const chunk = await FileSystem.readAsStringAsync(fileUri, {
      encoding: FileSystem.EncodingType.UTF8,
      position,
      length,
    });
    chunks.push(chunk);
  }

  return chunks.join('');
}
```

</details> <details> <summary>Ensure a directory exists before writing</summary>

```jsx
async function ensureDir(dirPath) {
  const info = await FileSystem.getInfoAsync(dirPath);
  if (!info.exists) {
    await FileSystem.makeDirectoryAsync(dirPath, { intermediates: true });
  }
}

async function writeToPath(relativePath, content) {
  const fullPath = `${FileSystem.documentDirectory}${relativePath}`;
  const dir = fullPath.substring(0, fullPath.lastIndexOf('/') + 1);
  await ensureDir(dir);
  await FileSystem.writeAsStringAsync(fullPath, content);
}

// Usage — creates nested dirs automatically
await writeToPath('projects/2024/notes.txt', 'My notes');
```

</details>

---

## Quick-Reference Cheatsheet

|API|Use case|
|---|---|
|`FileSystem.documentDirectory`|Persistent user data|
|`FileSystem.cacheDirectory`|Temporary / cached files|
|`getInfoAsync(uri)`|Check existence, size, type|
|`readAsStringAsync(uri)`|Read file contents|
|`writeAsStringAsync(uri, data)`|Write / overwrite a file|
|`readDirectoryAsync(uri)`|List directory contents|
|`makeDirectoryAsync(uri)`|Create directory|
|`copyAsync({ from, to })`|Copy file or directory|
|`moveAsync({ from, to })`|Move / rename file|
|`deleteAsync(uri)`|Delete file or directory|
|`downloadAsync(url, dest)`|One-shot download|
|`createDownloadResumable(...)`|Pausable download with progress|
|`uploadAsync(url, file)`|Upload file to server|
|`createUploadTask(...)`|Upload with progress tracking|
|`EncodingType.UTF8`|Text / JSON files|
|`EncodingType.Base64`|Binary / image files|
|`SAF.requestDirectoryPermissionsAsync()`|Android external storage access|
|`SAF.createFileAsync(dir, name, mime)`|Create file in SAF directory|
|`FileSystemSessionType.BACKGROUND`|iOS background transfers|

---

_Reference based on `expo-file-system` SDK 50+. Always check the [official docs](https://docs.expo.dev/versions/latest/sdk/filesystem/) for the latest updates._