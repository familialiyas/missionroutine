# Roukido — CLAUDE.md

## What this project is

A gamified daily routine tracker built as a **single-file PWA**, originally
built for the developer's own kids (Uzaiyr, 10, blue `theme-0`; Imtiyaz, 7,
pink `theme-1`) and now generalized so any family can set up their own player
profiles from scratch — see "Onboarding" below. There is no hardcoded roster:
new accounts start with zero players and walk through a setup flow to create
their first one.

Designed to run on a shared family tablet. No build step, no bundler.

`localStorage` is always the source of truth the app reads/writes first — an
**optional** Supabase layer (parent-only Google sign-in + cloud backup/sync
across devices) sits on top of it without changing that. See "Cloud sync" below.

## Tech stack

| Layer | Choice |
|---|---|
| UI | Pure HTML + CSS + vanilla JS (no frameworks) |
| Fonts | Google Fonts CDN — Nunito + Space Mono |
| Data | `localStorage` (keys prefixed `mr_`), optionally mirrored to Supabase |
| Cloud sync (optional) | Supabase (Postgres + Auth) — Google OAuth, one row per parent account |
| PWA | `manifest.json` + `sw.js` (cache-first strategy) |
| Icons | `icons/icon-192.png`, `icons/icon-512.png` (System.Drawing generated) |

## File layout

```
Routine Gamification_v2/
├── index.html        ← entire app lives here (HTML + CSS + JS in one file)
├── manifest.json     ← PWA manifest
├── sw.js             ← service worker
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
├── README.md         ← local testing + deployment instructions
└── CLAUDE.md         ← this file
```

## Key architecture decisions

### Single-file design (do not split)
Everything intentionally lives in `index.html`. No bundler, no npm, no build step. This was a deliberate choice for simplicity and portability — the app can be opened as a file on any device. Do not refactor into separate JS/CSS files unless explicitly requested.

### Cloud sync (optional, Supabase) — `localStorage` is still the source of truth
Added so a parent can sign in with Google once per device and have
`players`/`sessions`/`pin` back up to the cloud and sync across devices,
**without** changing how kids use the app (they still just tap their profile
card — no login for them). The data model is **multi-tenant by design** —
every account's row is keyed by `user_id` and isolated via Row Level Security
— so this also works as the foundation for selling the app to other families,
each with their own completely separate Google account and data.

- **Gate**: a `view-signin` screen ("Sign in with Google") shown only when
  Supabase is configured AND no session exists. If `SUPABASE_URL`/
  `SUPABASE_ANON_KEY` are left as placeholders, `sb` is `null` and the whole
  layer no-ops — the app behaves exactly like the original local-only PWA.
- **Auth**: `sb.auth.signInWithOAuth({ provider:'google' })`; one Postgres row
  per signed-in account in `mission_routine_data` (`user_id`, `players`,
  `sessions`, `pin`, `updated_at`), protected by Row Level Security so each
  account can only ever read/write its own row.
- **Account switching on shared devices**: `signOutCloud()` triggers
  `resetLocalCloudState()`, which wipes `mr_players`/`mr_sessions`/`mr_pin`/
  `mr_synced_at` from localStorage and resets in-memory state back to
  `DEFAULT_PLAYERS`. This is deliberate — it stops one account's cached data
  from being pushed into a different account's cloud row when someone signs
  out and a different person signs in on the same tablet (important once
  multiple buyers/families could use the same physical device or browser).
- **Sync model**: last-write-wins on the whole blob, compared via
  `updated_at` timestamps (`mr_synced_at` in localStorage vs. the row's
  `updated_at`). `pullRemote()` runs once on sign-in; `queueCloudSync()` →
  `pushRemote()` (debounced ~1.5s) fires from `save()` whenever
  `mr_players`/`mr_sessions` change, and from `saveParentPin()`. Push failures
  (offline) are swallowed and retried on the browser's `online` event — the
  app never blocks on the network.
- **Setup**: see README.md → "Cloud sync setup (Supabase + Google sign-in)"
  for the SQL schema + Google OAuth provider steps (external account setup,
  can't be scripted from here).

### localStorage key schema
```
mr_tasks        — JSON, full task list for both profiles (overrides DEFAULT_TASKS)
mr_completions  — JSON object, keys like "2026-5-11-p0-morning-m1" → "done"|"failed"|true
mr_streaks      — JSON array [streak_p0, streak_p1]
mr_avatars      — JSON array [emoji_p0, emoji_p1]
```
Completions use `todayKey()` → `"YYYY-M-D"` (no zero-padding — intentional, do not change).

### Auto-fail logic
Tasks auto-fail when `nowMins() > timeToMins(task.end)` and no completion exists. This runs on every `renderTasks()` call (every second via `setInterval`).

### Phase detection
`autoDetectPhase()` sets morning/evening based on whether current time is before or after noon. Users can manually override with the phase pills.

### Theme system
`body.theme-0` = blue (Uzaiyr), `body.theme-1` = pink (Imtiyaz). CSS custom properties cascade from these classes. Profile selection applies the theme via `applyTheme(pid)`.

### Service worker cache strategy
- **Static assets** (`/`, `/index.html`, `/manifest.json`, icons): cache-first
- **Google Fonts** (`fonts.googleapis.com`, `fonts.gstatic.com`): stale-while-revalidate
- Cache name: `roukido-v1` — bump to `v2` etc. when deploying updates

## How to test locally

```bash
npx serve .
# then open http://localhost:3000
```

Service workers require HTTPS or localhost — `file://` will not work.

## How to deploy

Drag the folder into Netlify drop zone at netlify.com → instant HTTPS URL → install prompt appears on mobile.

## What NOT to change without discussion

- The `todayKey()` date format (changing it invalidates all stored completions)
- The `compKey()` structure (same reason)
- `localStorage` key names (same reason)
- The single-file architecture
- Functionality, logic, styling, or behaviour (this CLAUDE.md was created during a PWA scaffolding pass — scope was scaffolding only)

## Adding new tasks

Tasks are defined in `DEFAULT_TASKS` inside `index.html` (around line 486). Parents can also edit task times/XP live via Parent Settings → saved to `mr_tasks` in localStorage. To add a new task type, add an entry to `CHEER_MAP` so the mascot has something to say about it.

## Icons

Current icons are placeholder PNGs (dark background + purple circle + white "M") generated via PowerShell `System.Drawing`. Replace with a proper icon by overwriting `icons/icon-192.png` and `icons/icon-512.png` — no other changes needed.
