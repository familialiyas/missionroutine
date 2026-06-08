# Roukido — PWA

Gamified daily routine tracker for kids — set up your own player profiles and
go. Shared family-tablet app — pure HTML + vanilla JS, `localStorage`-first,
with optional Supabase cloud sync (see "Cloud sync setup" below).

## Folder structure

```
Routine Gamification_v2/
├── index.html        ← full app (single file)
├── manifest.json     ← PWA manifest
├── sw.js             ← service worker (cache-first + font stale-while-revalidate)
├── icons/
│   ├── icon-192.png  ← home-screen icon (192 × 192)
│   └── icon-512.png  ← splash / store icon (512 × 512)
└── README.md
```

---

## Test locally

You **must** serve the files over HTTP (not `file://`) for the service worker and manifest to work.

### Option A — npx serve (zero install)
```bash
npx serve .
```
Then open `http://localhost:3000` in Chrome or Edge.

### Option B — Python
```bash
python -m http.server 8080
```
Then open `http://localhost:8080`.

### Verify PWA is working
1. Open DevTools → **Application** tab
2. Check **Manifest** — should show name, icons, colours
3. Check **Service Workers** — status should be *activated and running*
4. Go offline (Network tab → throttle to Offline) and reload — app should still load

---

## Deploy to Netlify (drag and drop)

1. Go to [netlify.com](https://netlify.com) and log in (or create a free account)
2. From the dashboard click **"Add new site" → "Deploy manually"**
3. Drag the entire `Routine Gamification_v2` folder into the drop zone
4. Netlify gives you a URL like `https://random-name.netlify.app`
5. Open that URL on your phone — the install prompt will appear automatically

> The app works on any static host (GitHub Pages, Vercel, Cloudflare Pages, etc.)

---

## Install on Android (Chrome)

1. Open the deployed URL in **Chrome**
2. Tap the **⋮** menu → **"Add to Home screen"**
3. Confirm — the Roukido icon appears on your home screen
4. Tap it to launch in standalone (full-screen) mode

> Chrome also shows an automatic install banner at the bottom of the screen on supported devices.

---

## Install on iPhone / iPad (Safari)

1. Open the deployed URL in **Safari** (must be Safari — Chrome on iOS cannot install PWAs)
2. Tap the **Share** button (box with arrow pointing up)
3. Scroll down and tap **"Add to Home Screen"**
4. Tap **"Add"** — the icon appears on your home screen

> On iOS the app opens in standalone mode without the Safari address bar.

---

## Updating the cache

When you make changes to `index.html`, bump the cache version in `sw.js`:

```js
const CACHE_NAME = 'roukido-v2'; // increment this
```

This tells the browser to discard the old cache and install the new one on next visit.

## Cloud sync setup (Supabase + Google sign-in)

The app can back up & sync `players`, `sessions` and the parent PIN to a free
[Supabase](https://supabase.com) project, gated behind a single Google sign-in
for the parent. Kids still just tap their profile card — only the parent signs
in once per device. The app keeps working offline (it always reads/writes
`localStorage` first) and quietly syncs to the cloud in the background.

### 1. Create the Supabase project
1. Go to [supabase.com](https://supabase.com) → **New project** (free tier is fine).
2. Once it's created, open **Project Settings → API** and copy:
   - **Project URL** (looks like `https://xxxxxxxx.supabase.co`)
   - **anon public** key (a long JWT string)

### 2. Create the data table
Open **SQL Editor** in Supabase and run:

```sql
create table public.mission_routine_data (
  user_id    uuid primary key references auth.users(id) on delete cascade,
  players    jsonb not null default '[]',
  sessions   jsonb not null default '[]',
  pin        text,
  updated_at timestamptz not null default now()
);

alter table public.mission_routine_data enable row level security;

create policy "Users manage their own data"
  on public.mission_routine_data
  for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);
```

This stores one row per signed-in parent account, and Row Level Security
ensures nobody can read or write anyone else's row.

### 3. Turn on Google sign-in
1. In Supabase: **Authentication → Providers → Google** → toggle it on.
2. You'll need a Google OAuth Client ID/Secret — create one at
   [Google Cloud Console → Credentials](https://console.cloud.google.com/apis/credentials):
   - Application type: **Web application**
   - Authorized redirect URI: paste the **Callback URL** Supabase shows you
     on that same Google provider page (looks like
     `https://xxxxxxxx.supabase.co/auth/v1/callback`)
3. Paste the resulting **Client ID** and **Client Secret** back into the
   Supabase Google provider page and save.
4. In **Authentication → URL Configuration**, add the URL(s) you'll open the
   app from (e.g. your Vercel/Netlify URL and `http://localhost:3000`) to
   **Redirect URLs** — otherwise Google will refuse to send you back to the app.

### 4. Connect the app
Open `index.html` and find this near the top of the `<script>` block:

```js
const SUPABASE_URL      = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

Replace both placeholders with the values you copied in step 1, then redeploy
(and bump `CACHE_NAME` in `sw.js` as above). On next load, a **Sign in with
Google** screen appears before the profile picker — sign in once on each
device and everything syncs from there.

> Until you fill in real values, the app silently skips cloud sync and behaves
> exactly like the local-only version — nothing breaks for anyone who hasn't
> set this up.
