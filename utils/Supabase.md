# Supabase + React Native — Complete Reference

A comprehensive reference for using Supabase in React Native / Expo apps.  
Covers client setup, auth (including secure session storage), database queries, realtime, storage, edge functions, and common patterns.

> **Install:**
> 
> ```bash
> npx expo install @supabase/supabase-js react-native-url-polyfill
> npx expo install @react-native-async-storage/async-storage
> # For Google/Apple native auth (optional):
> npx expo install expo-web-browser expo-auth-session
> ```

---

## Table of Contents

1. [Overview](#overview)
2. [Client Setup](#client-setup)
3. [Environment Variables](#environment-variables)
4. [Authentication — Email/Password](#authentication--emailpassword)
5. [Authentication — OAuth (Google, Apple, etc.)](#authentication--oauth-google-apple-etc)
6. [Authentication — Magic Link & OTP](#authentication--magic-link--otp)
7. [Session Management](#session-management)
8. [Auth State Listener](#auth-state-listener)
9. [Database — Select Queries](#database--select-queries)
10. [Database — Insert, Update, Delete](#database--insert-update-delete)
11. [Database — Filters Reference](#database--filters-reference)
12. [Database — Joins (Embedded Resources)](#database--joins-embedded-resources)
13. [RPC — Calling Postgres Functions](#rpc--calling-postgres-functions)
14. [Row Level Security (RLS)](#row-level-security-rls)
15. [Realtime Subscriptions](#realtime-subscriptions)
16. [Storage — File Upload/Download](#storage--file-uploaddownload)
17. [Edge Functions](#edge-functions)
18. [React Query Integration](#react-query-integration)
19. [Common Patterns](#common-patterns)
20. [Troubleshooting](#troubleshooting)

---

## Overview

<details> <summary>What Supabase provides for React Native apps</summary>

Supabase is a backend-as-a-service built on PostgreSQL. The `@supabase/supabase-js` client talks directly to Supabase's APIs over HTTPS/WebSocket — no custom backend required for most apps.

|Feature|What it replaces|
|---|---|
|Postgres Database|Your own database server|
|Auth|Firebase Auth / custom JWT server|
|Realtime|Socket.io / custom WebSocket server|
|Storage|S3 / Cloudinary|
|Edge Functions|Serverless functions (Lambda-style)|
|Row Level Security|API-layer authorization logic|

```
┌──────────────┐   HTTPS / WSS    ┌─────────────────────┐
│ React Native │ ───────────────► │  Supabase            │
│     App      │ ◄─────────────── │  (Postgres + Auth +   │
└──────────────┘                  │  Realtime + Storage)  │
                                   └─────────────────────┘
```

> ✅ Unlike MongoDB, it's safe and intended for the Supabase client to talk **directly** from your mobile app to the Supabase API — security is enforced via **Row Level Security (RLS)** policies on the Postgres side, not by hiding credentials.

</details>

---

## Client Setup

<details> <summary>Create the Supabase client with AsyncStorage session persistence</summary>

```ts
// lib/supabase.ts
import 'react-native-url-polyfill/auto';
import { createClient } from '@supabase/supabase-js';
import AsyncStorage from '@react-native-async-storage/async-storage';

const supabaseUrl = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseAnonKey, {
  auth: {
    storage: AsyncStorage,
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: false, // must be false in React Native
  },
});
```

> ⚠️ `react-native-url-polyfill/auto` must be imported at the very top of your entry file — Supabase's client relies on `URL` which isn't fully implemented in Hermes by default.

</details> <details> <summary>Using MMKV instead of AsyncStorage (faster, recommended)</summary>

```ts
// lib/mmkvStorage.ts
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV({ id: 'supabase-auth' });

export const mmkvStorageAdapter = {
  getItem: (key: string) => {
    const value = storage.getString(key);
    return value ?? null;
  },
  setItem: (key: string, value: string) => {
    storage.set(key, value);
  },
  removeItem: (key: string) => {
    storage.delete(key);
  },
};
```

```ts
// lib/supabase.ts
import 'react-native-url-polyfill/auto';
import { createClient } from '@supabase/supabase-js';
import { mmkvStorageAdapter } from './mmkvStorage';

export const supabase = createClient(
  process.env.EXPO_PUBLIC_SUPABASE_URL!,
  process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!,
  {
    auth: {
      storage: mmkvStorageAdapter,
      autoRefreshToken: true,
      persistSession: true,
      detectSessionInUrl: false,
    },
  }
);
```

</details> <details> <summary>Auto-refresh session on app foreground</summary>

```tsx
// App.tsx
import { AppState } from 'react-native';
import { useEffect } from 'react';
import { supabase } from './lib/supabase';

export default function App() {
  useEffect(() => {
    const subscription = AppState.addEventListener('change', (state) => {
      if (state === 'active') {
        supabase.auth.startAutoRefresh();
      } else {
        supabase.auth.stopAutoRefresh();
      }
    });
    return () => subscription.remove();
  }, []);

  return <MainApp />;
}
```

</details>

---

## Environment Variables

<details> <summary>Where to find your keys and how to store them</summary>

```bash
# .env (commit-safe — anon key is meant to be public, secured by RLS)
EXPO_PUBLIC_SUPABASE_URL=https://xxxxxxxxxxxx.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Find these in: **Supabase Dashboard → Project Settings → API**

|Key|Safe in mobile app?|Notes|
|---|---|---|
|`anon` / `publishable` key|✅ Yes|Designed to be public — protected entirely by RLS policies|
|`service_role` key|❌ **NEVER**|Bypasses RLS completely — server-only (Edge Functions, backend)|

> ⚠️ The anon key being "public" is by design, but it ONLY stays safe if you have proper RLS policies enabled on every table. Without RLS, anyone with the anon key can read/write your entire database.

</details>

---

## Authentication — Email/Password

<details> <summary><code>supabase.auth.signUp(credentials)</code></summary>

Registers a new user with email and password. Sends a confirmation email by default (configurable in dashboard).

```ts
async function signUp(email: string, password: string) {
  const { data, error } = await supabase.auth.signUp({
    email,
    password,
    options: {
      data: {
        full_name: 'Jane Doe', // stored in user_metadata
      },
    },
  });

  if (error) throw error;
  return data; // { user, session }
}
```

</details> <details> <summary><code>supabase.auth.signInWithPassword(credentials)</code></summary>

```ts
async function signIn(email: string, password: string) {
  const { data, error } = await supabase.auth.signInWithPassword({
    email,
    password,
  });

  if (error) throw error;
  return data; // { user, session }
}
```

</details> <details> <summary><code>supabase.auth.signOut()</code></summary>

```ts
async function signOut() {
  const { error } = await supabase.auth.signOut();
  if (error) throw error;
}
```

</details> <details> <summary><code>supabase.auth.resetPasswordForEmail(email)</code></summary>

```ts
async function requestPasswordReset(email: string) {
  const { error } = await supabase.auth.resetPasswordForEmail(email, {
    redirectTo: 'myapp://reset-password', // deep link back into your app
  });
  if (error) throw error;
}

// After user follows the link and you have a session:
async function updatePassword(newPassword: string) {
  const { error } = await supabase.auth.updateUser({ password: newPassword });
  if (error) throw error;
}
```

</details> <details> <summary><code>supabase.auth.updateUser(attributes)</code></summary>

```ts
async function updateProfile(updates: { email?: string; data?: object }) {
  const { data, error } = await supabase.auth.updateUser(updates);
  if (error) throw error;
  return data.user;
}

// Update metadata
await updateProfile({ data: { full_name: 'Jane Smith', avatar_url: url } });
```

</details>

---

## Authentication — OAuth (Google, Apple, etc.)

<details> <summary>OAuth flow with <code>expo-auth-session</code></summary>

OAuth in React Native requires opening a browser session and handling the redirect back into the app via a deep link.

```bash
npx expo install expo-web-browser expo-auth-session
```

```ts
// hooks/useGoogleAuth.ts
import * as WebBrowser from 'expo-web-browser';
import * as Linking from 'expo-linking';
import { supabase } from '../lib/supabase';

WebBrowser.maybeCompleteAuthSession();

const redirectTo = Linking.createURL('/auth-callback');

export async function signInWithGoogle() {
  const { data, error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo,
      skipBrowserRedirect: true, // we open the browser manually below
    },
  });

  if (error) throw error;

  const result = await WebBrowser.openAuthSessionAsync(data.url, redirectTo);

  if (result.type === 'success') {
    const { url } = result;
    await createSessionFromUrl(url);
  }
}

async function createSessionFromUrl(url: string) {
  const { params, errorCode } = Linking.parse(url) as any;
  if (errorCode) throw new Error(errorCode);

  const { access_token, refresh_token } = params;
  if (!access_token) return;

  const { data, error } = await supabase.auth.setSession({
    access_token,
    refresh_token,
  });
  if (error) throw error;
  return data.session;
}
```

```json
// app.json — required scheme for the deep link redirect
{
  "expo": {
    "scheme": "myapp"
  }
}
```

</details> <details> <summary>Native Apple Sign In (iOS) — using ID token directly</summary>

```bash
npx expo install expo-apple-authentication
```

```ts
import * as AppleAuthentication from 'expo-apple-authentication';
import { supabase } from '../lib/supabase';

async function signInWithApple() {
  const credential = await AppleAuthentication.signInAsync({
    requestedScopes: [
      AppleAuthentication.AppleAuthenticationScope.FULL_NAME,
      AppleAuthentication.AppleAuthenticationScope.EMAIL,
    ],
  });

  if (credential.identityToken) {
    const { data, error } = await supabase.auth.signInWithIdToken({
      provider: 'apple',
      token: credential.identityToken,
    });
    if (error) throw error;
    return data.session;
  }
}
```

</details> <details> <summary>Native Google Sign In — using ID token directly</summary>

```bash
npm install @react-native-google-signin/google-signin
```

```ts
import { GoogleSignin } from '@react-native-google-signin/google-signin';
import { supabase } from '../lib/supabase';

GoogleSignin.configure({
  webClientId: 'YOUR_WEB_CLIENT_ID.apps.googleusercontent.com',
});

async function signInWithGoogleNative() {
  await GoogleSignin.hasPlayServices();
  const userInfo = await GoogleSignin.signIn();
  const idToken = userInfo.data?.idToken;

  if (idToken) {
    const { data, error } = await supabase.auth.signInWithIdToken({
      provider: 'google',
      token: idToken,
    });
    if (error) throw error;
    return data.session;
  }
}
```

> ✅ Native ID-token sign-in (Apple/Google) is preferred over the browser OAuth flow — it's faster, feels native, and avoids the WebBrowser redirect dance.

</details>

---

## Authentication — Magic Link & OTP

<details> <summary><code>signInWithOtp</code> — email magic link</summary>

```ts
async function sendMagicLink(email: string) {
  const { error } = await supabase.auth.signInWithOtp({
    email,
    options: {
      emailRedirectTo: 'myapp://auth-callback',
    },
  });
  if (error) throw error;
}
```

</details> <details> <summary><code>signInWithOtp</code> — phone SMS OTP</summary>

```ts
// Step 1 — send the code
async function sendSmsOtp(phone: string) {
  const { error } = await supabase.auth.signInWithOtp({ phone });
  if (error) throw error;
}

// Step 2 — verify the code the user received
async function verifySmsOtp(phone: string, token: string) {
  const { data, error } = await supabase.auth.verifyOtp({
    phone,
    token,
    type: 'sms',
  });
  if (error) throw error;
  return data.session;
}
```

</details>

---

## Session Management

<details> <summary><code>supabase.auth.getSession()</code> — current session</summary>

```ts
const { data: { session }, error } = await supabase.auth.getSession();

if (session) {
  console.log('User ID:', session.user.id);
  console.log('Access token:', session.access_token);
  console.log('Expires at:', session.expires_at);
}
```

</details> <details> <summary><code>supabase.auth.getUser()</code> — current user, re-validated</summary>

Unlike `getSession()` (which reads the local stored session), `getUser()` re-validates the token against the Supabase server — slower but guarantees the session hasn't been revoked.

```ts
const { data: { user }, error } = await supabase.auth.getUser();
```

</details> <details> <summary><code>supabase.auth.refreshSession()</code></summary>

```ts
const { data, error } = await supabase.auth.refreshSession();
```

> 💡 Usually unnecessary to call manually — `autoRefreshToken: true` in the client config handles this automatically.

</details>

---

## Auth State Listener

<details> <summary><code>supabase.auth.onAuthStateChange(callback)</code> — global auth context</summary>

The standard pattern: subscribe once at the app root and drive your navigation/UI state from it.

```tsx
// context/AuthContext.tsx
import { createContext, useContext, useEffect, useState, ReactNode } from 'react';
import { Session, User } from '@supabase/supabase-js';
import { supabase } from '../lib/supabase';

interface AuthContextType {
  session: Session | null;
  user: User | null;
  loading: boolean;
}

const AuthContext = createContext<AuthContextType>({ session: null, user: null, loading: true });

export function AuthProvider({ children }: { children: ReactNode }) {
  const [session, setSession] = useState<Session | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session);
      setLoading(false);
    });

    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (_event, session) => {
        setSession(session);
        setLoading(false);
      }
    );

    return () => subscription.unsubscribe();
  }, []);

  return (
    <AuthContext.Provider value={{ session, user: session?.user ?? null, loading }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => useContext(AuthContext);
```

**Auth event types:**

|Event|Fires when|
|---|---|
|`SIGNED_IN`|User successfully signs in|
|`SIGNED_OUT`|User signs out|
|`TOKEN_REFRESHED`|Access token auto-refreshed|
|`USER_UPDATED`|`updateUser` succeeds|
|`PASSWORD_RECOVERY`|User opens a password reset link|
|`INITIAL_SESSION`|Initial session check on client load|

</details> <details> <summary>Expo Router auth guard using the session</summary>

```tsx
// app/_layout.tsx
import { useEffect } from 'react';
import { Slot, useRouter, useSegments } from 'expo-router';
import { AuthProvider, useAuth } from '../context/AuthContext';

function AuthGuard() {
  const { session, loading } = useAuth();
  const segments = useSegments();
  const router = useRouter();

  useEffect(() => {
    if (loading) return;
    const inAuthGroup = segments[0] === '(auth)';

    if (!session && !inAuthGroup) {
      router.replace('/login');
    } else if (session && inAuthGroup) {
      router.replace('/home');
    }
  }, [session, loading, segments]);

  return null;
}

export default function RootLayout() {
  return (
    <AuthProvider>
      <AuthGuard />
      <Slot />
    </AuthProvider>
  );
}
```

</details>

---

## Database — Select Queries

<details> <summary>Basic select</summary>

```ts
// Select all columns
const { data, error } = await supabase.from('posts').select('*');

// Select specific columns
const { data } = await supabase.from('posts').select('id, title, created_at');

// Single row
const { data: post, error } = await supabase
  .from('posts')
  .select('*')
  .eq('id', postId)
  .single();        // throws if 0 or >1 rows match

// Maybe single — returns null instead of throwing if not found
const { data: maybePost } = await supabase
  .from('posts')
  .select('*')
  .eq('id', postId)
  .maybeSingle();
```

</details> <details> <summary>Ordering, limiting, pagination</summary>

```ts
const { data } = await supabase
  .from('posts')
  .select('*')
  .order('created_at', { ascending: false })
  .limit(20);

// Pagination with range
const PAGE_SIZE = 20;
const page = 2;
const { data, count } = await supabase
  .from('posts')
  .select('*', { count: 'exact' })
  .order('created_at', { ascending: false })
  .range(page * PAGE_SIZE, page * PAGE_SIZE + PAGE_SIZE - 1);
```

</details> <details> <summary>Count queries</summary>

```ts
// Get count without fetching rows
const { count, error } = await supabase
  .from('posts')
  .select('*', { count: 'exact', head: true })
  .eq('published', true);

console.log('Total published posts:', count);
```

</details>

---

## Database — Insert, Update, Delete

<details> <summary><code>insert</code> — create rows</summary>

```ts
// Single insert
const { data, error } = await supabase
  .from('posts')
  .insert({ title: 'New Post', body: 'Content here', user_id: userId })
  .select()
  .single();

// Bulk insert
const { data } = await supabase
  .from('tags')
  .insert([
    { name: 'react' },
    { name: 'expo' },
    { name: 'supabase' },
  ])
  .select();
```

</details> <details> <summary><code>update</code> — modify rows</summary>

```ts
const { data, error } = await supabase
  .from('posts')
  .update({ title: 'Updated Title', published: true })
  .eq('id', postId)
  .select()
  .single();
```

</details> <details> <summary><code>upsert</code> — insert or update on conflict</summary>

```ts
const { data, error } = await supabase
  .from('user_settings')
  .upsert(
    { user_id: userId, theme: 'dark' },
    { onConflict: 'user_id' }
  )
  .select()
  .single();
```

</details> <details> <summary><code>delete</code> — remove rows</summary>

```ts
const { error } = await supabase
  .from('posts')
  .delete()
  .eq('id', postId);

// Delete with a returning select
const { data: deleted } = await supabase
  .from('posts')
  .delete()
  .eq('id', postId)
  .select()
  .single();
```

</details>

---

## Database — Filters Reference

<details> <summary>All available filter operators</summary>

```ts
const query = supabase.from('posts').select('*');

// Equality
query.eq('status', 'published');
query.neq('status', 'draft');

// Comparison
query.gt('views', 100);
query.gte('views', 100);
query.lt('views', 100);
query.lte('views', 100);

// Pattern matching
query.like('title', '%react%');       // case-sensitive
query.ilike('title', '%react%');      // case-insensitive

// Null checks
query.is('deleted_at', null);
query.not('deleted_at', 'is', null);

// Set membership
query.in('category', ['tech', 'design']);

// Array column contains
query.contains('tags', ['react-native']);
query.containedBy('tags', ['react-native', 'expo', 'mobile']);

// Range overlap (for date/numeric ranges)
query.overlaps('date_range', '[2024-01-01,2024-06-01]');

// Full-text search
query.textSearch('content', 'react native', { type: 'websearch' });

// Combine filters (AND is default by chaining)
const { data } = await supabase
  .from('posts')
  .select('*')
  .eq('published', true)
  .gte('created_at', '2024-01-01')
  .order('created_at', { ascending: false });

// OR conditions
const { data: orResults } = await supabase
  .from('posts')
  .select('*')
  .or('status.eq.published,author_id.eq.' + userId);
```

</details>

---

## Database — Joins (Embedded Resources)

<details> <summary>Fetch related data via foreign key relationships</summary>

```ts
// posts table has a foreign key: author_id → users.id

// Fetch posts with their author embedded
const { data, error } = await supabase
  .from('posts')
  .select(`
    id,
    title,
    created_at,
    author:users (
      id,
      name,
      avatar_url
    )
  `);

// data = [{ id, title, created_at, author: { id, name, avatar_url } }, ...]
```

</details> <details> <summary>One-to-many relationships</summary>

```ts
// Fetch a post with all its comments
const { data, error } = await supabase
  .from('posts')
  .select(`
    id,
    title,
    comments (
      id,
      body,
      created_at,
      author:users ( name, avatar_url )
    )
  `)
  .eq('id', postId)
  .single();
```

</details> <details> <summary>Filtering on related tables</summary>

```ts
// Get posts that have at least one comment from a specific user
const { data } = await supabase
  .from('posts')
  .select(`
    id,
    title,
    comments!inner ( id, author_id )
  `)
  .eq('comments.author_id', userId);
```

</details>

---

## RPC — Calling Postgres Functions

<details> <summary><code>supabase.rpc(functionName, params)</code></summary>

Call a stored Postgres function (great for complex logic that should live in the database, transactional operations, or anything that needs to be atomic).

```sql
-- In Supabase SQL Editor — define the function
create or replace function increment_like_count(post_id uuid)
returns void as $$
begin
  update posts set like_count = like_count + 1 where id = post_id;
end;
$$ language plpgsql security definer;
```

```ts
const { data, error } = await supabase.rpc('increment_like_count', {
  post_id: postId,
});
```

</details> <details> <summary>RPC with return values</summary>

```sql
create or replace function get_trending_posts(limit_count int default 10)
returns table (id uuid, title text, score numeric) as $$
begin
  return query
    select p.id, p.title, (p.like_count * 1.0 / extract(epoch from now() - p.created_at)) as score
    from posts p
    order by score desc
    limit limit_count;
end;
$$ language plpgsql;
```

```ts
const { data: trending, error } = await supabase.rpc('get_trending_posts', {
  limit_count: 5,
});
```

</details>

---

## Row Level Security (RLS)

<details> <summary>Why RLS is mandatory</summary>

Since the anon key is bundled in your app, **RLS policies are the only thing protecting your data**. Without them, any user can read/write any row in any table.

```sql
-- 1. Enable RLS on the table (required first step — disabled by default allows nothing once RLS is on)
alter table posts enable row level security;

-- 2. Add policies for each operation
create policy "Users can view published posts"
  on posts for select
  using (published = true);

create policy "Users can view their own unpublished posts"
  on posts for select
  using (auth.uid() = author_id);

create policy "Users can insert their own posts"
  on posts for insert
  with check (auth.uid() = author_id);

create policy "Users can update their own posts"
  on posts for update
  using (auth.uid() = author_id);

create policy "Users can delete their own posts"
  on posts for delete
  using (auth.uid() = author_id);
```

</details> <details> <summary>Common RLS patterns</summary>

```sql
-- Public read, authenticated write
create policy "Anyone can view" on products for select using (true);
create policy "Authenticated users can insert" on products for insert
  with check (auth.role() = 'authenticated');

-- Owner-only access
create policy "Owner full access" on tasks for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);

-- Team-based access (via a join table)
create policy "Team members can view" on documents for select
  using (
    exists (
      select 1 from team_members
      where team_members.team_id = documents.team_id
      and team_members.user_id = auth.uid()
    )
  );

-- Admin-only via custom claim
create policy "Admins can delete anything" on posts for delete
  using ((auth.jwt() ->> 'role') = 'admin');
```

</details>

---

## Realtime Subscriptions

<details> <summary>Subscribe to table changes</summary>

```tsx
import { useEffect, useState } from 'react';
import { supabase } from '../lib/supabase';

function useLiveMessages(roomId: string) {
  const [messages, setMessages] = useState<any[]>([]);

  useEffect(() => {
    // Initial fetch
    supabase
      .from('messages')
      .select('*')
      .eq('room_id', roomId)
      .order('created_at')
      .then(({ data }) => setMessages(data ?? []));

    // Realtime subscription
    const channel = supabase
      .channel(`room:${roomId}`)
      .on(
        'postgres_changes',
        { event: 'INSERT', schema: 'public', table: 'messages', filter: `room_id=eq.${roomId}` },
        (payload) => {
          setMessages(prev => [...prev, payload.new]);
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [roomId]);

  return messages;
}
```

</details> <details> <summary>Listening to all event types</summary>

```ts
const channel = supabase
  .channel('posts-changes')
  .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'posts' }, (payload) => {
    console.log('New post:', payload.new);
  })
  .on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'posts' }, (payload) => {
    console.log('Updated post:', payload.new, 'was:', payload.old);
  })
  .on('postgres_changes', { event: 'DELETE', schema: 'public', table: 'posts' }, (payload) => {
    console.log('Deleted post:', payload.old);
  })
  .subscribe();

// Or listen to all events with '*'
const allChanges = supabase
  .channel('all-posts')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'posts' }, (payload) => {
    console.log(payload.eventType, payload.new, payload.old);
  })
  .subscribe();
```

</details> <details> <summary>Presence — who's online</summary>

```tsx
function useRoomPresence(roomId: string, userId: string) {
  const [onlineUsers, setOnlineUsers] = useState<string[]>([]);

  useEffect(() => {
    const channel = supabase.channel(`room:${roomId}`, {
      config: { presence: { key: userId } },
    });

    channel
      .on('presence', { event: 'sync' }, () => {
        const state = channel.presenceState();
        setOnlineUsers(Object.keys(state));
      })
      .subscribe(async (status) => {
        if (status === 'SUBSCRIBED') {
          await channel.track({ online_at: new Date().toISOString() });
        }
      });

    return () => { supabase.removeChannel(channel); };
  }, [roomId, userId]);

  return onlineUsers;
}
```

</details> <details> <summary>Broadcast — ephemeral messages (typing indicators, cursors)</summary>

```ts
const channel = supabase.channel(`room:${roomId}`);

// Send a broadcast event
channel.send({
  type: 'broadcast',
  event: 'typing',
  payload: { userId, isTyping: true },
});

// Listen for broadcast events
channel
  .on('broadcast', { event: 'typing' }, (payload) => {
    console.log(`${payload.payload.userId} is typing:`, payload.payload.isTyping);
  })
  .subscribe();
```

</details>

---

## Storage — File Upload/Download

<details> <summary>Upload a file</summary>

```ts
import * as FileSystem from 'expo-file-system';
import { decode } from 'base64-arraybuffer';

async function uploadAvatar(userId: string, localUri: string) {
  const base64 = await FileSystem.readAsStringAsync(localUri, {
    encoding: FileSystem.EncodingType.Base64,
  });

  const filePath = `${userId}/avatar.jpg`;

  const { data, error } = await supabase.storage
    .from('avatars')
    .upload(filePath, decode(base64), {
      contentType: 'image/jpeg',
      upsert: true,
    });

  if (error) throw error;
  return data.path;
}
```

</details> <details> <summary>Get a public URL</summary>

```ts
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl(`${userId}/avatar.jpg`);

console.log(data.publicUrl);
// https://xxxxx.supabase.co/storage/v1/object/public/avatars/userId/avatar.jpg
```

</details> <details> <summary>Signed URLs — for private buckets</summary>

```ts
// Generate a temporary signed URL (expires after the given seconds)
const { data, error } = await supabase.storage
  .from('private-documents')
  .createSignedUrl(`${userId}/contract.pdf`, 3600); // 1 hour

console.log(data?.signedUrl);
```

</details> <details> <summary>Download a file</summary>

```ts
const { data, error } = await supabase.storage
  .from('avatars')
  .download(`${userId}/avatar.jpg`);

if (data) {
  const reader = new FileReader();
  reader.onload = () => {
    // data:image/jpeg;base64,...
    const base64 = reader.result;
  };
  reader.readAsDataURL(data);
}
```

</details> <details> <summary>List and delete files</summary>

```ts
// List files in a folder
const { data: files } = await supabase.storage
  .from('avatars')
  .list(userId, { limit: 100, sortBy: { column: 'created_at', order: 'desc' } });

// Delete a file
await supabase.storage.from('avatars').remove([`${userId}/avatar.jpg`]);

// Delete multiple
await supabase.storage.from('avatars').remove([
  `${userId}/old1.jpg`,
  `${userId}/old2.jpg`,
]);
```

</details> <details> <summary>Storage RLS policies</summary>

```sql
-- Users can only upload to their own folder
create policy "Users can upload own avatar"
  on storage.objects for insert
  with check (
    bucket_id = 'avatars'
    and (storage.foldername(name))[1] = auth.uid()::text
  );

create policy "Avatars are publicly viewable"
  on storage.objects for select
  using (bucket_id = 'avatars');
```

</details>

---

## Edge Functions

<details> <summary>Calling an Edge Function from React Native</summary>

```ts
const { data, error } = await supabase.functions.invoke('send-welcome-email', {
  body: { userId: user.id, email: user.email },
});

if (error) throw error;
console.log(data);
```

</details> <details> <summary>Edge Function with auth context (server side, for reference)</summary>

```ts
// supabase/functions/send-welcome-email/index.ts (Deno runtime)
import { createClient } from 'jsr:@supabase/supabase-js@2';

Deno.serve(async (req) => {
  const { userId, email } = await req.json();

  const supabaseAdmin = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!  // safe here — server-side only
  );

  await sendEmail(email, 'Welcome!');

  return new Response(JSON.stringify({ success: true }), {
    headers: { 'Content-Type': 'application/json' },
  });
});
```

</details>

---

## React Query Integration

<details> <summary>Wrapping Supabase queries with React Query</summary>

```ts
// hooks/usePosts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { supabase } from '../lib/supabase';

export function usePosts() {
  return useQuery({
    queryKey: ['posts'],
    queryFn: async () => {
      const { data, error } = await supabase
        .from('posts')
        .select('*, author:users(name, avatar_url)')
        .eq('published', true)
        .order('created_at', { ascending: false });
      if (error) throw error;
      return data;
    },
  });
}

export function useCreatePost() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: async (post: { title: string; body: string; author_id: string }) => {
      const { data, error } = await supabase.from('posts').insert(post).select().single();
      if (error) throw error;
      return data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
}

export function useDeletePost() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: async (id: string) => {
      const { error } = await supabase.from('posts').delete().eq('id', id);
      if (error) throw error;
    },
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['posts'] }),
  });
}
```

</details>

---

## Common Patterns

<details> <summary>Full auth flow — sign up, sign in, sign out</summary>

```tsx
// hooks/useAuthActions.ts
import { supabase } from '../lib/supabase';
import { useState } from 'react';

export function useAuthActions() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const signUp = async (email: string, password: string) => {
    setLoading(true);
    setError(null);
    const { error } = await supabase.auth.signUp({ email, password });
    setLoading(false);
    if (error) { setError(error.message); throw error; }
  };

  const signIn = async (email: string, password: string) => {
    setLoading(true);
    setError(null);
    const { error } = await supabase.auth.signInWithPassword({ email, password });
    setLoading(false);
    if (error) { setError(error.message); throw error; }
  };

  const signOut = async () => {
    await supabase.auth.signOut();
  };

  return { signUp, signIn, signOut, loading, error };
}
```

</details> <details> <summary>Profile table synced with auth via trigger</summary>

```sql
-- Automatically create a profile row when a user signs up
create table profiles (
  id uuid primary key references auth.users on delete cascade,
  full_name text,
  avatar_url text,
  updated_at timestamp default now()
);

create or replace function handle_new_user()
returns trigger as $$
begin
  insert into public.profiles (id, full_name)
  values (new.id, new.raw_user_meta_data->>'full_name');
  return new;
end;
$$ language plpgsql security definer;

create trigger on_auth_user_created
  after insert on auth.users
  for each row execute procedure handle_new_user();
```

```ts
// React Native — fetch the synced profile
const { data: profile } = await supabase
  .from('profiles')
  .select('*')
  .eq('id', session.user.id)
  .single();
```

</details> <details> <summary>Infinite scroll feed</summary>

```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
import { supabase } from '../lib/supabase';

const PAGE_SIZE = 20;

function useFeed() {
  return useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: async ({ pageParam = 0 }) => {
      const { data, error } = await supabase
        .from('posts')
        .select('*')
        .order('created_at', { ascending: false })
        .range(pageParam, pageParam + PAGE_SIZE - 1);
      if (error) throw error;
      return data;
    },
    getNextPageParam: (lastPage, allPages) =>
      lastPage.length === PAGE_SIZE ? allPages.length * PAGE_SIZE : undefined,
    initialPageParam: 0,
  });
}
```

</details> <details> <summary>Image upload from camera/picker to Storage + DB record</summary>

```ts
import * as ImagePicker from 'expo-image-picker';
import * as FileSystem from 'expo-file-system';
import { decode } from 'base64-arraybuffer';
import { supabase } from '../lib/supabase';

async function pickAndUploadPostImage(postId: string) {
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    quality: 0.8,
  });
  if (result.canceled) return;

  const asset = result.assets[0];
  const base64 = await FileSystem.readAsStringAsync(asset.uri, {
    encoding: FileSystem.EncodingType.Base64,
  });

  const filePath = `posts/${postId}/${Date.now()}.jpg`;

  const { error: uploadError } = await supabase.storage
    .from('post-images')
    .upload(filePath, decode(base64), { contentType: 'image/jpeg' });
  if (uploadError) throw uploadError;

  const { data: { publicUrl } } = supabase.storage
    .from('post-images')
    .getPublicUrl(filePath);

  await supabase.from('posts').update({ image_url: publicUrl }).eq('id', postId);

  return publicUrl;
}
```

</details> <details> <summary>Realtime chat screen</summary>

```tsx
function ChatScreen({ roomId }: { roomId: string }) {
  const { session } = useAuth();
  const [messages, setMessages] = useState<Message[]>([]);
  const [text, setText] = useState('');

  useEffect(() => {
    supabase
      .from('messages')
      .select('*')
      .eq('room_id', roomId)
      .order('created_at')
      .then(({ data }) => setMessages(data ?? []));

    const channel = supabase
      .channel(`room:${roomId}`)
      .on('postgres_changes',
        { event: 'INSERT', schema: 'public', table: 'messages', filter: `room_id=eq.${roomId}` },
        (payload) => setMessages(prev => [...prev, payload.new as Message])
      )
      .subscribe();

    return () => { supabase.removeChannel(channel); };
  }, [roomId]);

  const sendMessage = async () => {
    if (!text.trim()) return;
    await supabase.from('messages').insert({
      room_id: roomId,
      user_id: session!.user.id,
      body: text,
    });
    setText('');
  };

  return (
    <View style={{ flex: 1 }}>
      <FlatList
        data={messages}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => <MessageBubble message={item} />}
      />
      <View style={{ flexDirection: 'row' }}>
        <TextInput value={text} onChangeText={setText} placeholder="Message..." />
        <Button title="Send" onPress={sendMessage} />
      </View>
    </View>
  );
}
```

</details>

---

## Troubleshooting

<details> <summary>"URL.protocol is not implemented" or similar polyfill errors</summary>

- Ensure `import 'react-native-url-polyfill/auto';` is the **first** import in your entry file (`index.js` / `App.tsx`)
- Restart Metro with `npx expo start --clear` after adding the polyfill

</details> <details> <summary>Session not persisting between app restarts</summary>

- Confirm `persistSession: true` and a valid `storage` adapter are set in `createClient` options
- `detectSessionInUrl` must be `false` in React Native (it's a web-only feature and can throw)

</details> <details> <summary>RLS policy blocking expected access</summary>

- Use the Supabase SQL Editor's "Run as" feature to test policies as a specific user
- Temporarily run `select auth.uid();` in a query while authenticated to confirm the JWT is being sent
- Check that you enabled RLS (`alter table x enable row level security`) AND added at least one policy — RLS enabled with zero policies blocks everything

</details> <details> <summary>Realtime not receiving events</summary>

- Confirm Realtime is enabled for the table: **Dashboard → Database → Replication** → toggle the table on
- Check your RLS policies — Realtime respects RLS, so a user without SELECT access won't receive change events for rows they can't read

</details>

---

## Quick-Reference Cheatsheet

|API|Use case|
|---|---|
|`createClient(url, key, { auth: {...} })`|Initialize client with persisted session|
|`auth.signUp / signInWithPassword`|Email/password auth|
|`auth.signInWithOAuth`|Google/GitHub/etc via browser|
|`auth.signInWithIdToken`|Native Apple/Google sign-in|
|`auth.signInWithOtp`|Magic link / SMS OTP|
|`auth.onAuthStateChange`|Global session listener|
|`auth.getSession() / getUser()`|Read current session/user|
|`.from(table).select()`|Query rows|
|`.eq() .gt() .like() .in()`|Filter operators|
|`.select('*, related(*)')`|Embedded joins|
|`.insert() .update() .upsert() .delete()`|Mutations|
|`.rpc(fn, params)`|Call Postgres function|
|`alter table x enable row level security`|**Mandatory** for any public table|
|`.channel().on('postgres_changes', ...)`|Realtime table subscription|
|`.channel().on('broadcast', ...)`|Ephemeral pub/sub messages|
|`.channel({ config: { presence } })`|Online presence tracking|
|`storage.from(bucket).upload()`|File upload|
|`storage.from(bucket).getPublicUrl()`|Public file URL|
|`storage.from(bucket).createSignedUrl()`|Temporary private URL|
|`functions.invoke(name, { body })`|Call an Edge Function|

---

_Reference based on `@supabase/supabase-js` v2. Always check the [official docs](https://supabase.com/docs/guides/getting-started/tutorials/with-expo-react-native) for the latest updates._