# MongoDB + React Native — Complete Reference

A comprehensive reference for using MongoDB with React Native apps.  
Covers why a direct driver isn't used, the three real architectures (Atlas Data API, Realm/Device Sync, backend API), full code for each, schema design, and common patterns.

---

## Table of Contents

1. [Overview — Why React Native Can't Use the MongoDB Driver Directly](#overview--why-react-native-cant-use-the-mongodb-driver-directly)
2. [Choosing Your Architecture](#choosing-your-architecture)
3. [Architecture A — Atlas Data API (REST)](#architecture-a--atlas-data-api-rest)
4. [Architecture B — Atlas Device SDK (Realm) with Sync](#architecture-b--atlas-device-sdk-realm-with-sync)
5. [Architecture C — Custom Backend API](#architecture-c--custom-backend-api)
6. [Atlas Data API — Full Reference](#atlas-data-api--full-reference)
7. [Realm React Native SDK — Full Reference](#realm-react-native-sdk--full-reference)
8. [Realm Schema Definition](#realm-schema-definition)
9. [Realm Queries](#realm-queries)
10. [Realm Sync Configuration](#realm-sync-configuration)
11. [Backend API — Mongoose Schema Reference](#backend-api--mongoose-schema-reference)
12. [Backend API — Express Routes Reference](#backend-api--express-routes-reference)
13. [Authentication Patterns](#authentication-patterns)
14. [Offline-First Patterns](#offline-first-patterns)
15. [Common Patterns](#common-patterns)
16. [Security Checklist](#security-checklist)

---

## Overview — Why React Native Can't Use the MongoDB Driver Directly

<details> <summary>The core problem — no TCP sockets, no secrets on device</summary>

The official `mongodb` npm driver requires:

- Raw TCP socket connections (not available in React Native's JS runtime)
- Direct database credentials embedded in the client (a major security risk — anyone could extract your connection string from the app bundle and access your entire database)

```
❌ This does NOT work in React Native:
import { MongoClient } from 'mongodb';
const client = new MongoClient('mongodb+srv://user:pass@cluster...');
// Fails: no TCP socket support in RN's JS engine (Hermes/JSC)
// Also: NEVER put DB credentials in a mobile app bundle
```

**You have three legitimate options instead:**

|Option|What it is|Best for|
|---|---|---|
|**A. Atlas Data API**|MongoDB's official HTTPS REST API for Atlas|Simple apps, prototypes, serverless|
|**B. Atlas Device SDK (Realm)**|Embedded local database + automatic cloud sync|Offline-first apps, real-time sync|
|**C. Custom Backend API**|Your own Node/Express (or any) server using the real MongoDB driver|Production apps, full control, complex business logic|

</details>

---

## Choosing Your Architecture

<details> <summary>Decision guide</summary>

```
Need offline support / real-time multi-device sync?
├── Yes → Architecture B (Realm + Device Sync)
└── No
    │
    Need custom business logic, complex auth, third-party integrations?
    ├── Yes → Architecture C (Custom Backend API)
    └── No
        │
        Quick prototype or simple CRUD app?
        └── Yes → Architecture A (Atlas Data API)
```

|Factor|Data API|Realm Sync|Custom Backend|
|---|---|---|---|
|Setup complexity|Low|Medium|Medium-High|
|Offline support|❌|✅ Built-in|⚠️ Manual (MMKV/SQLite cache)|
|Real-time sync|❌|✅ Built-in|⚠️ Manual (WebSockets)|
|Custom business logic|⚠️ Limited (Atlas Functions)|⚠️ Limited|✅ Full control|
|Cost at scale|Pay per request|Pay per sync|Your infra cost|
|Production readiness|Prototypes / simple apps|✅ Production-ready|✅ Production-ready|
|Maintenance|None|Low|You maintain the server|

</details>

---

## Architecture A — Atlas Data API (REST)

<details> <summary>How it works</summary>

MongoDB Atlas exposes your cluster through an auto-generated HTTPS REST API. Your React Native app calls this API directly with `fetch`, authenticated via API keys or Atlas App Services auth.

```
┌──────────────┐      HTTPS       ┌──────────────────┐      ┌─────────────┐
│ React Native │ ───────────────► │  Atlas Data API   │ ───► │ MongoDB     │
│     App      │ ◄─────────────── │  (auto-generated)  │ ◄─── │ Atlas       │
└──────────────┘      JSON        └──────────────────┘      └─────────────┘
```

**Setup steps:**

1. In Atlas UI: **App Services** → **Data API** → Enable
2. Create an API Key (or use App Services Authentication for per-user auth)
3. Copy your Data API base URL and App ID

</details>

---

## Architecture B — Atlas Device SDK (Realm) with Sync

<details> <summary>How it works</summary>

Realm is an embedded object database that lives **on the device**. Atlas Device Sync automatically and bidirectionally syncs data between the local Realm database and your MongoDB Atlas cluster — including offline support.

```
┌──────────────────────────┐                    ┌─────────────┐
│      React Native App     │                    │   MongoDB   │
│  ┌────────────────────┐  │   Atlas Device Sync │   Atlas     │
│  │  Local Realm DB     │◄─┼───────────────────►│   Cluster   │
│  │  (works offline)    │  │   (auto, bg sync)   │             │
│  └────────────────────┘  │                    └─────────────┘
└──────────────────────────┘
```

**Setup steps:**

1. In Atlas UI: **App Services** → Create an App
2. Enable **Device Sync**, choose a cluster, define data access rules
3. Install `realm` SDK in your React Native app
4. Define your schema in JS, matching your sync rules

</details>

---

## Architecture C — Custom Backend API

<details> <summary>How it works</summary>

You run your own server (Node.js + Express most commonly) that uses the real `mongoose`/`mongodb` driver. Your React Native app talks to **your API**, never to MongoDB directly.

```
┌──────────────┐    HTTPS/REST     ┌──────────────────┐   mongodb://    ┌─────────────┐
│ React Native │ ───────────────►  │  Your Backend     │ ──────────────► │ MongoDB     │
│     App      │ ◄───────────────  │  (Node/Express)   │ ◄────────────── │ Atlas       │
└──────────────┘    JSON            └──────────────────┘                 └─────────────┘
```

> ✅ This is the **recommended approach for most production apps** — it gives you full control over validation, auth, rate limiting, and business logic, and keeps DB credentials entirely server-side.

</details>

---

## Atlas Data API — Full Reference

<details> <summary>Setup — enable and get credentials</summary>

```
1. Atlas Dashboard → App Services → Create a New App (or use existing)
2. Left sidebar → Data API → Enable the Data API
3. Note your:
   - Data API Base URL: https://data.mongodb-api.com/app/<app-id>/endpoint/data/v1
   - App ID
4. Authentication → Create an API Key (simplest) or set up Email/Password auth
```

</details> <details> <summary>Client setup — fetch wrapper</summary>

```ts
// api/mongoClient.ts
const DATA_API_URL = process.env.EXPO_PUBLIC_MONGODB_DATA_API_URL!;
const DATA_API_KEY = process.env.EXPO_PUBLIC_MONGODB_DATA_API_KEY!;

const DATA_SOURCE = 'mongodb-atlas';
const DATABASE = 'myAppDb';

async function dataApiRequest(action: string, body: object) {
  const response = await fetch(`${DATA_API_URL}/action/${action}`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'api-key': DATA_API_KEY,
    },
    body: JSON.stringify({
      dataSource: DATA_SOURCE,
      database: DATABASE,
      ...body,
    }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error || 'Data API request failed');
  }

  return response.json();
}

export default dataApiRequest;
```

> ⚠️ Storing a Data API key in `EXPO_PUBLIC_*` env vars means it's bundled in your app and extractable. Only use API-key auth for prototypes. For production, use App Services user authentication (JWT) instead — see below.

</details> <details> <summary><code>findOne</code> — fetch a single document</summary>

```ts
async function getUser(userId: string) {
  const result = await dataApiRequest('findOne', {
    collection: 'users',
    filter: { _id: { $oid: userId } },
  });
  return result.document; // null if not found
}
```

</details> <details> <summary><code>find</code> — fetch multiple documents</summary>

```ts
async function getPosts(limit = 20, skip = 0) {
  const result = await dataApiRequest('find', {
    collection: 'posts',
    filter: { published: true },
    sort: { createdAt: -1 },
    limit,
    skip,
  });
  return result.documents;
}
```

</details> <details> <summary><code>insertOne</code> / <code>insertMany</code> — create documents</summary>

```ts
async function createPost(post: { title: string; body: string; authorId: string }) {
  const result = await dataApiRequest('insertOne', {
    collection: 'posts',
    document: {
      ...post,
      createdAt: { $date: new Date().toISOString() },
      published: false,
    },
  });
  return result.insertedId;
}

async function createManyTags(tags: string[]) {
  const result = await dataApiRequest('insertMany', {
    collection: 'tags',
    documents: tags.map(name => ({ name, createdAt: { $date: new Date().toISOString() } })),
  });
  return result.insertedIds;
}
```

</details> <details> <summary><code>updateOne</code> / <code>updateMany</code> — modify documents</summary>

```ts
async function publishPost(postId: string) {
  const result = await dataApiRequest('updateOne', {
    collection: 'posts',
    filter: { _id: { $oid: postId } },
    update: { $set: { published: true, publishedAt: { $date: new Date().toISOString() } } },
  });
  return result.modifiedCount;
}

async function incrementLikeCount(postId: string) {
  await dataApiRequest('updateOne', {
    collection: 'posts',
    filter: { _id: { $oid: postId } },
    update: { $inc: { likeCount: 1 } },
  });
}
```

</details> <details> <summary><code>deleteOne</code> / <code>deleteMany</code> — remove documents</summary>

```ts
async function deletePost(postId: string) {
  const result = await dataApiRequest('deleteOne', {
    collection: 'posts',
    filter: { _id: { $oid: postId } },
  });
  return result.deletedCount;
}
```

</details> <details> <summary><code>aggregate</code> — run an aggregation pipeline</summary>

```ts
async function getTopAuthors() {
  const result = await dataApiRequest('aggregate', {
    collection: 'posts',
    pipeline: [
      { $match: { published: true } },
      { $group: { _id: '$authorId', postCount: { $sum: 1 } } },
      { $sort: { postCount: -1 } },
      { $limit: 10 },
    ],
  });
  return result.documents;
}
```

</details> <details> <summary>BSON type wrappers in JSON</summary>

Atlas Data API uses MongoDB Extended JSON. Wrap special BSON types when sending requests.

```ts
// ObjectId
filter: { _id: { $oid: '64f1a2b3c4d5e6f7a8b9c0d1' } }

// Date
update: { $set: { updatedAt: { $date: new Date().toISOString() } } }

// Numbers (when precision matters)
filter: { price: { $numberDouble: '19.99' } }
filter: { count: { $numberInt: '42' } }
filter: { bigValue: { $numberLong: '9223372036854775807' } }
```

</details> <details> <summary>React Query hook wrapper</summary>

```ts
// hooks/usePosts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import dataApiRequest from '../api/mongoClient';

export function usePosts() {
  return useQuery({
    queryKey: ['posts'],
    queryFn: async () => {
      const result = await dataApiRequest('find', {
        collection: 'posts',
        filter: { published: true },
        sort: { createdAt: -1 },
      });
      return result.documents;
    },
  });
}

export function useCreatePost() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (post: { title: string; body: string }) =>
      dataApiRequest('insertOne', { collection: 'posts', document: post }),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['posts'] }),
  });
}
```

</details>

---

## Realm React Native SDK — Full Reference

<details> <summary>Install</summary>

```bash
npx expo install realm @realm/react

# Add to babel.config.js plugins (if not using @realm/react's bare setup)
```

```js
// babel.config.js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: ['@realm/babel-plugin'],
  };
};
```

</details> <details> <summary>Connect to your App Services app</summary>

```tsx
// realm/app.ts
import Realm from 'realm';

export const app = new Realm.App({ id: 'your-atlas-app-id' });
```

</details> <details> <summary>Authenticate a user</summary>

```ts
import { app } from './app';
import Realm from 'realm';

// Email/Password
async function loginWithEmail(email: string, password: string) {
  const credentials = Realm.Credentials.emailPassword(email, password);
  const user = await app.logIn(credentials);
  return user;
}

// Anonymous (no login UI needed)
async function loginAnonymously() {
  const credentials = Realm.Credentials.anonymous();
  const user = await app.logIn(credentials);
  return user;
}

// Custom JWT (bring your own auth provider)
async function loginWithJWT(token: string) {
  const credentials = Realm.Credentials.jwt(token);
  const user = await app.logIn(credentials);
  return user;
}

// Logout
async function logout() {
  await app.currentUser?.logOut();
}
```

</details>

---

## Realm Schema Definition

<details> <summary>Defining object schemas (classes)</summary>

```ts
// realm/schemas.ts
import Realm, { BSON } from 'realm';

export class Task extends Realm.Object<Task> {
  _id!: BSON.ObjectId;
  title!: string;
  isComplete!: boolean;
  priority!: number;
  createdAt!: Date;
  ownerId!: string;

  static schema: Realm.ObjectSchema = {
    name: 'Task',
    primaryKey: '_id',
    properties: {
      _id: { type: 'objectId', default: () => new BSON.ObjectId() },
      title: 'string',
      isComplete: { type: 'bool', default: false },
      priority: { type: 'int', default: 0 },
      createdAt: { type: 'date', default: () => new Date() },
      ownerId: 'string',
    },
  };
}

export class Project extends Realm.Object<Project> {
  _id!: BSON.ObjectId;
  name!: string;
  tasks!: Realm.List<Task>;

  static schema: Realm.ObjectSchema = {
    name: 'Project',
    primaryKey: '_id',
    properties: {
      _id: { type: 'objectId', default: () => new BSON.ObjectId() },
      name: 'string',
      tasks: 'Task[]',   // embedded list relationship
    },
  };
}
```

</details> <details> <summary>Property type reference</summary>

|Type|Description|
|---|---|
|`'string'`|Text|
|`'int'`|32-bit integer|
|`'double'`|Floating point|
|`'bool'`|Boolean|
|`'date'`|Date/timestamp|
|`'objectId'`|MongoDB ObjectId|
|`'uuid'`|UUID|
|`'data'`|Binary data|
|`'decimal128'`|High-precision decimal|
|`'mixed'`|Any type|
|`'Task[]'`|List of `Task` objects (relationship)|
|`'Task?'`|Optional/nullable property|
|`{ type: 'list', objectType: 'string' }`|List of primitive strings|

```ts
properties: {
  _id: 'objectId',
  title: 'string',
  description: 'string?',           // optional string
  tags: 'string[]',                 // list of strings
  dueDate: 'date?',                 // optional date
  assignee: 'User?',                // optional relationship
  comments: 'Comment[]',            // list of related objects
  metadata: 'mixed',                // any type
}
```

</details>

---

## Realm Queries

<details> <summary>Open a realm and access objects</summary>

```tsx
import Realm from 'realm';
import { Task } from './schemas';

const realm = await Realm.open({
  schema: [Task],
});

// Get all objects of a type
const allTasks = realm.objects<Task>('Task');
```

</details> <details> <summary>Filtering with RQL (Realm Query Language)</summary>

```ts
// Equality
const incomplete = realm.objects<Task>('Task').filtered('isComplete == false');

// Comparison operators
const highPriority = realm.objects<Task>('Task').filtered('priority >= 3');

// String contains / starts with
const search = realm.objects<Task>('Task').filtered('title CONTAINS[c] $0', 'meeting');
//                                                              ^ case-insensitive

// Multiple conditions
const myUrgentTasks = realm.objects<Task>('Task')
  .filtered('ownerId == $0 AND priority >= 4 AND isComplete == false', userId);

// Sorting
const sorted = realm.objects<Task>('Task').sorted('createdAt', true); // true = descending

// Sort by multiple fields
const multiSort = realm.objects<Task>('Task').sorted([
  ['priority', true],
  ['createdAt', false],
]);
```

</details> <details> <summary>Writing data — create, update, delete</summary>

```ts
// Create
realm.write(() => {
  realm.create<Task>('Task', {
    title: 'Buy groceries',
    priority: 2,
    ownerId: currentUser.id,
  });
});

// Update
realm.write(() => {
  const task = realm.objectForPrimaryKey<Task>('Task', taskId);
  if (task) {
    task.isComplete = true;
  }
});

// Delete
realm.write(() => {
  const task = realm.objectForPrimaryKey<Task>('Task', taskId);
  if (task) {
    realm.delete(task);
  }
});

// Delete all of a type
realm.write(() => {
  const allTasks = realm.objects<Task>('Task');
  realm.delete(allTasks);
});
```

</details> <details> <summary>Listening for changes</summary>

```ts
const tasks = realm.objects<Task>('Task');

const listener = (collection: Realm.Collection<Task>, changes: Realm.CollectionChangeSet) => {
  console.log('Inserted:', changes.insertions);
  console.log('Modified:', changes.newModifications);
  console.log('Deleted:', changes.deletions);
};

tasks.addListener(listener);

// Cleanup
tasks.removeListener(listener);
```

</details> <details> <summary><code>@realm/react</code> hooks — React-friendly API</summary>

```tsx
// realm/RealmProvider.tsx
import { createRealmContext } from '@realm/react';
import { Task, Project } from './schemas';

export const { RealmProvider, useRealm, useQuery, useObject } = createRealmContext({
  schema: [Task, Project],
});
```

```tsx
// App.tsx
import { RealmProvider } from './realm/RealmProvider';

export default function App() {
  return (
    <RealmProvider>
      <TaskListScreen />
    </RealmProvider>
  );
}
```

```tsx
// screens/TaskListScreen.tsx
import { useRealm, useQuery } from '../realm/RealmProvider';
import { Task } from '../realm/schemas';
import { FlatList, Text, Button, View } from 'react-native';

function TaskListScreen() {
  const realm = useRealm();
  const tasks = useQuery(Task).filtered('isComplete == false').sorted('createdAt', true);

  const addTask = (title: string) => {
    realm.write(() => {
      realm.create(Task, { title, ownerId: 'user123' });
    });
  };

  const toggleTask = (task: Task) => {
    realm.write(() => {
      task.isComplete = !task.isComplete;
    });
  };

  return (
    <FlatList
      data={tasks}
      keyExtractor={(item) => item._id.toHexString()}
      renderItem={({ item }) => (
        <View>
          <Text>{item.title}</Text>
          <Button title="Toggle" onPress={() => toggleTask(item)} />
        </View>
      )}
    />
  );
}
```

</details>

---

## Realm Sync Configuration

<details> <summary>Flexible Sync — subscribe to specific data subsets</summary>

```ts
import Realm from 'realm';
import { Task } from './schemas';

const app = new Realm.App({ id: 'your-app-id' });
const user = await app.logIn(Realm.Credentials.anonymous());

const realm = await Realm.open({
  schema: [Task],
  sync: {
    user,
    flexible: true,
  },
});

// Subscribe to only the current user's tasks
await realm.subscriptions.update((mutableSubs) => {
  mutableSubs.add(
    realm.objects<Task>('Task').filtered('ownerId == $0', user.id)
  );
});

await realm.subscriptions.waitForSynchronization();
```

</details> <details> <summary><code>@realm/react</code> with sync provider</summary>

```tsx
// realm/RealmProvider.tsx
import { createRealmContext } from '@realm/react';
import Realm from 'realm';
import { Task } from './schemas';

const app = new Realm.App({ id: 'your-app-id' });

export const { RealmProvider, useRealm, useQuery, useUser, useApp } = createRealmContext({
  schema: [Task],
});
```

```tsx
// App.tsx
import { AppProvider, UserProvider } from '@realm/react';
import { RealmProvider } from './realm/RealmProvider';

const APP_ID = 'your-app-id';

export default function App() {
  return (
    <AppProvider id={APP_ID}>
      <UserProvider fallback={<LoginScreen />}>
        <RealmProvider
          sync={{
            flexible: true,
            initialSubscriptions: {
              update: (mutableSubs, realm) => {
                mutableSubs.add(realm.objects('Task'));
              },
            },
          }}
        >
          <MainApp />
        </RealmProvider>
      </UserProvider>
    </AppProvider>
  );
}
```

</details> <details> <summary>Connection state monitoring</summary>

```ts
realm.syncSession?.addProgressNotification(
  Realm.ProgressDirection.Download,
  Realm.ProgressMode.ReportIndefinitely,
  (transferred, transferable) => {
    console.log(`Synced ${transferred} of ${transferable}`);
  }
);

// Check connection state
const state = realm.syncSession?.connectionState;
// Realm.ConnectionState.Connected | Connecting | Disconnected
```

</details>

---

## Backend API — Mongoose Schema Reference

<details> <summary>Connection setup (Node.js server, NOT React Native)</summary>

```ts
// server/db.ts
import mongoose from 'mongoose';

export async function connectDB() {
  await mongoose.connect(process.env.MONGODB_URI!, {
    dbName: 'myAppDb',
  });
  console.log('MongoDB connected');
}
```

> ⚠️ This code runs on your **server**, never inside the React Native app.

</details> <details> <summary>Schema with validation and relationships</summary>

```ts
// server/models/Post.ts
import mongoose, { Schema, Document } from 'mongoose';

interface IPost extends Document {
  title: string;
  body: string;
  author: mongoose.Types.ObjectId;
  tags: string[];
  likeCount: number;
  published: boolean;
  createdAt: Date;
}

const postSchema = new Schema<IPost>({
  title: { type: String, required: true, maxlength: 200 },
  body: { type: String, required: true },
  author: { type: Schema.Types.ObjectId, ref: 'User', required: true, index: true },
  tags: [{ type: String, lowercase: true }],
  likeCount: { type: Number, default: 0, min: 0 },
  published: { type: Boolean, default: false, index: true },
}, {
  timestamps: true, // adds createdAt, updatedAt automatically
});

postSchema.index({ title: 'text', body: 'text' }); // full-text search

export const Post = mongoose.model<IPost>('Post', postSchema);
```

</details> <details> <summary>User schema with password hashing</summary>

```ts
// server/models/User.ts
import mongoose, { Schema, Document } from 'mongoose';
import bcrypt from 'bcrypt';

interface IUser extends Document {
  email: string;
  passwordHash: string;
  name: string;
  comparePassword(candidate: string): Promise<boolean>;
}

const userSchema = new Schema<IUser>({
  email: { type: String, required: true, unique: true, lowercase: true, trim: true },
  passwordHash: { type: String, required: true },
  name: { type: String, required: true },
}, { timestamps: true });

userSchema.methods.comparePassword = function (candidate: string) {
  return bcrypt.compare(candidate, this.passwordHash);
};

userSchema.statics.hashPassword = (password: string) => bcrypt.hash(password, 10);

export const User = mongoose.model<IUser>('User', userSchema);
```

</details>

---

## Backend API — Express Routes Reference

<details> <summary>Basic CRUD routes</summary>

```ts
// server/routes/posts.ts
import { Router } from 'express';
import { Post } from '../models/Post';
import { requireAuth } from '../middleware/auth';

const router = Router();

// GET /api/posts — list published posts
router.get('/', async (req, res) => {
  const { limit = 20, skip = 0 } = req.query;
  const posts = await Post.find({ published: true })
    .sort({ createdAt: -1 })
    .limit(Number(limit))
    .skip(Number(skip))
    .populate('author', 'name email');
  res.json(posts);
});

// GET /api/posts/:id
router.get('/:id', async (req, res) => {
  const post = await Post.findById(req.params.id).populate('author', 'name');
  if (!post) return res.status(404).json({ error: 'Not found' });
  res.json(post);
});

// POST /api/posts — create (requires auth)
router.post('/', requireAuth, async (req, res) => {
  const post = await Post.create({
    ...req.body,
    author: req.user._id, // from auth middleware
  });
  res.status(201).json(post);
});

// PATCH /api/posts/:id — update (requires auth + ownership check)
router.patch('/:id', requireAuth, async (req, res) => {
  const post = await Post.findById(req.params.id);
  if (!post) return res.status(404).json({ error: 'Not found' });
  if (post.author.toString() !== req.user._id.toString()) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  Object.assign(post, req.body);
  await post.save();
  res.json(post);
});

// DELETE /api/posts/:id
router.delete('/:id', requireAuth, async (req, res) => {
  const post = await Post.findById(req.params.id);
  if (!post) return res.status(404).json({ error: 'Not found' });
  if (post.author.toString() !== req.user._id.toString()) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  await post.deleteOne();
  res.status(204).send();
});

export default router;
```

</details> <details> <summary>Pagination, search, and aggregation routes</summary>

```ts
// GET /api/posts/search?q=keyword
router.get('/search', async (req, res) => {
  const { q } = req.query;
  const posts = await Post.find(
    { $text: { $search: q as string }, published: true },
    { score: { $meta: 'textScore' } }
  ).sort({ score: { $meta: 'textScore' } }).limit(20);
  res.json(posts);
});

// GET /api/posts/stats — aggregation example
router.get('/stats', async (req, res) => {
  const stats = await Post.aggregate([
    { $match: { published: true } },
    {
      $group: {
        _id: '$author',
        postCount: { $sum: 1 },
        totalLikes: { $sum: '$likeCount' },
      },
    },
    { $sort: { totalLikes: -1 } },
    { $limit: 10 },
    {
      $lookup: {
        from: 'users',
        localField: '_id',
        foreignField: '_id',
        as: 'authorInfo',
      },
    },
  ]);
  res.json(stats);
});
```

</details> <details> <summary>React Native client calling your backend</summary>

```ts
// api/client.ts — React Native side
const API_URL = process.env.EXPO_PUBLIC_API_URL!;

class ApiClient {
  private token: string | null = null;

  setToken(token: string) {
    this.token = token;
  }

  private async request(path: string, options: RequestInit = {}) {
    const response = await fetch(`${API_URL}${path}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...(this.token ? { Authorization: `Bearer ${this.token}` } : {}),
        ...options.headers,
      },
    });
    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new Error(error.error || `Request failed: ${response.status}`);
    }
    return response.status === 204 ? null : response.json();
  }

  getPosts(limit = 20, skip = 0) {
    return this.request(`/api/posts?limit=${limit}&skip=${skip}`);
  }
  getPost(id: string) {
    return this.request(`/api/posts/${id}`);
  }
  createPost(data: { title: string; body: string }) {
    return this.request('/api/posts', { method: 'POST', body: JSON.stringify(data) });
  }
  updatePost(id: string, data: object) {
    return this.request(`/api/posts/${id}`, { method: 'PATCH', body: JSON.stringify(data) });
  }
  deletePost(id: string) {
    return this.request(`/api/posts/${id}`, { method: 'DELETE' });
  }
}

export const apiClient = new ApiClient();
```

```tsx
// React Query hooks wrapping the client
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { apiClient } from '../api/client';

export function usePosts() {
  return useQuery({ queryKey: ['posts'], queryFn: () => apiClient.getPosts() });
}

export function useCreatePost() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: apiClient.createPost.bind(apiClient),
    onSuccess: () => qc.invalidateQueries({ queryKey: ['posts'] }),
  });
}
```

</details>

---

## Authentication Patterns

<details> <summary>JWT auth with backend API</summary>

```ts
// server/middleware/auth.ts
import jwt from 'jsonwebtoken';
import { User } from '../models/User';

export async function requireAuth(req, res, next) {
  const header = req.headers.authorization;
  if (!header?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }
  try {
    const token = header.split(' ')[1];
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as { userId: string };
    const user = await User.findById(payload.userId);
    if (!user) return res.status(401).json({ error: 'Invalid token' });
    req.user = user;
    next();
  } catch {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
}
```

```ts
// server/routes/auth.ts
router.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (!user || !(await user.comparePassword(password))) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET!, { expiresIn: '7d' });
  res.json({ token, user: { id: user._id, email: user.email, name: user.name } });
});
```

```ts
// React Native — store token securely with MMKV (encrypted)
import { MMKV } from 'react-native-mmkv';
import { apiClient } from './client';

const storage = new MMKV({ id: 'auth', encryptionKey: ENCRYPTION_KEY });

async function login(email: string, password: string) {
  const { token, user } = await apiClient.login(email, password);
  storage.set('auth.token', token);
  apiClient.setToken(token);
  return user;
}
```

</details> <details> <summary>Atlas App Services authentication (for Data API / Realm)</summary>

```ts
// Email/Password registration via Atlas App Services
async function registerUser(app: Realm.App, email: string, password: string) {
  await app.emailPasswordAuth.registerUser({ email, password });
  const credentials = Realm.Credentials.emailPassword(email, password);
  return app.logIn(credentials);
}

// Get a user access token for Data API calls
async function getDataApiToken(user: Realm.User) {
  return user.accessToken; // pass as Authorization: Bearer <token>
}
```

</details>

---

## Offline-First Patterns

<details> <summary>Realm — automatic, built-in</summary>

```tsx
// With Realm + Flexible Sync, offline support is automatic.
// Writes made while offline are queued and synced when connectivity returns.

realm.write(() => {
  realm.create(Task, { title: 'Works offline automatically' });
});
// No special offline handling needed — Realm manages the local store
// and syncs in the background whenever connectivity is available.
```

</details> <details> <summary>Backend API — manual offline queue with MMKV</summary>

```ts
// utils/offlineQueue.ts
import { MMKV } from 'react-native-mmkv';
import NetInfo from '@react-native-community/netinfo';
import { apiClient } from '../api/client';

const queue = new MMKV({ id: 'offline-queue' });

interface QueuedAction {
  id: string;
  type: 'createPost' | 'updatePost' | 'deletePost';
  payload: any;
  timestamp: number;
}

export function enqueueAction(action: Omit<QueuedAction, 'id' | 'timestamp'>) {
  const id = `${Date.now()}-${Math.random()}`;
  const item: QueuedAction = { ...action, id, timestamp: Date.now() };
  const existing = JSON.parse(queue.getString('pending') ?? '[]');
  queue.set('pending', JSON.stringify([...existing, item]));
}

export async function processQueue() {
  const { isConnected } = await NetInfo.fetch();
  if (!isConnected) return;

  const pending: QueuedAction[] = JSON.parse(queue.getString('pending') ?? '[]');
  const remaining: QueuedAction[] = [];

  for (const action of pending) {
    try {
      if (action.type === 'createPost') await apiClient.createPost(action.payload);
      if (action.type === 'updatePost') await apiClient.updatePost(action.payload.id, action.payload.data);
      if (action.type === 'deletePost') await apiClient.deletePost(action.payload.id);
    } catch {
      remaining.push(action); // retry later
    }
  }

  queue.set('pending', JSON.stringify(remaining));
}

// Listen for connectivity changes and flush
NetInfo.addEventListener(state => {
  if (state.isConnected) processQueue();
});
```

</details>

---

## Common Patterns

<details> <summary>Realm: tasks app with sync</summary>

```tsx
import { useRealm, useQuery, useUser } from './realm/RealmProvider';
import { Task } from './realm/schemas';

function TasksScreen() {
  const realm = useRealm();
  const user = useUser();
  const tasks = useQuery(Task)
    .filtered('ownerId == $0', user.id)
    .sorted('createdAt', true);

  const addTask = (title: string) => {
    realm.write(() => {
      realm.create(Task, { title, ownerId: user.id });
    });
  };

  return (
    <FlatList
      data={tasks}
      keyExtractor={t => t._id.toHexString()}
      renderItem={({ item }) => <TaskRow task={item} />}
    />
  );
}
```

</details> <details> <summary>Data API: infinite scroll feed</summary>

```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
import dataApiRequest from '../api/mongoClient';

function useFeed() {
  return useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: async ({ pageParam = 0 }) => {
      const result = await dataApiRequest('find', {
        collection: 'posts',
        filter: { published: true },
        sort: { createdAt: -1 },
        limit: 20,
        skip: pageParam,
      });
      return result.documents;
    },
    getNextPageParam: (lastPage, allPages) =>
      lastPage.length === 20 ? allPages.length * 20 : undefined,
    initialPageParam: 0,
  });
}
```

</details> <details> <summary>Backend API: image upload + MongoDB reference</summary>

```ts
// server/routes/upload.ts — store image in S3/Cloudinary, save URL in MongoDB
router.post('/posts/:id/image', requireAuth, upload.single('image'), async (req, res) => {
  const imageUrl = await uploadToS3(req.file);
  const post = await Post.findByIdAndUpdate(
    req.params.id,
    { $set: { imageUrl } },
    { new: true }
  );
  res.json(post);
});
```

```tsx
// React Native — upload via FormData
async function uploadPostImage(postId: string, imageUri: string) {
  const formData = new FormData();
  formData.append('image', {
    uri: imageUri,
    name: 'photo.jpg',
    type: 'image/jpeg',
  } as any);

  const response = await fetch(`${API_URL}/api/posts/${postId}/image`, {
    method: 'POST',
    headers: { Authorization: `Bearer ${token}` },
    body: formData,
  });
  return response.json();
}
```

</details> <details> <summary>Real-time updates without Realm — polling fallback</summary>

```tsx
// Simple polling for near-real-time updates when not using Realm Sync
function useLivePostCount(postId: string) {
  return useQuery({
    queryKey: ['post-likes', postId],
    queryFn: () => apiClient.getPost(postId),
    refetchInterval: 5000, // poll every 5s
  });
}

// Better: WebSocket-based real-time updates
import { io } from 'socket.io-client';

const socket = io(API_URL);

useEffect(() => {
  socket.on(`post:${postId}:updated`, (data) => {
    queryClient.setQueryData(['post', postId], data);
  });
  return () => socket.off(`post:${postId}:updated`);
}, [postId]);
```

</details>

---

## Security Checklist

<details> <summary>Critical security rules</summary>

- ❌ **Never** embed a `mongodb://` connection string in your React Native app
- ❌ **Never** use a Data API key with full read/write permissions bundled in `EXPO_PUBLIC_*` env vars for production apps
- ✅ Use **Atlas App Services rules** to restrict Data API / Realm access per-user (e.g. users can only read/write their own documents)
- ✅ For custom backends, validate and sanitize all input server-side — never trust client data
- ✅ Use parameterized queries (Mongoose/driver do this by default) — never string-concatenate into queries
- ✅ Hash passwords with bcrypt/argon2 — never store plaintext
- ✅ Use HTTPS only for all API communication
- ✅ Rate-limit your backend API endpoints
- ✅ Store JWTs/tokens in encrypted MMKV or `expo-secure-store`, not plain AsyncStorage
- ✅ Set short JWT expiry + refresh token rotation for sensitive apps
- ✅ Enable MongoDB Atlas network access rules (IP allowlist) for your backend server only

```ts
// Atlas App Services rule example — restrict to owner only
{
  "roles": [
    {
      "name": "owner-only",
      "read": { "ownerId": "%%user.id" },
      "write": { "ownerId": "%%user.id" }
    }
  ]
}
```

</details>

---

## Quick-Reference Cheatsheet

|Approach|Key APIs|
|---|---|
|**Atlas Data API**|`findOne`, `find`, `insertOne`, `updateOne`, `deleteOne`, `aggregate` via `fetch`|
|**Realm SDK**|`Realm.open()`, `realm.write()`, `realm.objects()`, `.filtered()`, `.sorted()`|
|**@realm/react**|`useRealm()`, `useQuery()`, `useObject()`, `RealmProvider`|
|**Realm Sync**|`Realm.Credentials`, `app.logIn()`, `realm.subscriptions.update()`|
|**Backend (Mongoose)**|`Schema`, `model()`, `find()`, `findById()`, `create()`, `aggregate()`|
|**Backend (Express)**|Standard REST routes — your app calls these via `fetch`|
|**Auth**|JWT (custom backend) or `Realm.Credentials` (App Services)|
|**Offline**|Realm Sync (automatic) or MMKV-based manual queue (custom backend)|

|Concept|Direct Driver|Data API|Realm|Backend API|
|---|---|---|---|---|
|Works in React Native|❌|✅|✅|✅|
|Offline support|—|❌|✅|⚠️ Manual|
|Real-time sync|—|❌|✅|⚠️ Manual (WebSocket)|
|Custom logic|—|⚠️ Limited|⚠️ Limited|✅|
|Production-grade|—|⚠️ Simple apps|✅|✅|

---

_Reference covers MongoDB Atlas Data API, Atlas Device SDK (Realm) v12+, and custom Node/Express + Mongoose backends. Always check the [Atlas Data API docs](https://www.mongodb.com/docs/atlas/app-services/data-api/) and [Realm React Native docs](https://www.mongodb.com/docs/realm/sdk/react-native/) for the latest updates._