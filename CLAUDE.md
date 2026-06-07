# Mission Routine — CLAUDE.md

## What this project is

A gamified daily routine tracker built as a **single-file PWA** for two kids:
- **Uzaiyr** — age 10, blue theme (`theme-0`)
- **Imtiyaz** — age 7, pink/magenta theme (`theme-1`)

Designed to run on a shared family tablet. No backend, no login, no build step.

## Tech stack

| Layer | Choice |
|---|---|
| UI | Pure HTML + CSS + vanilla JS (no frameworks) |
| Fonts | Google Fonts CDN — Nunito + Space Mono |
| Data | `localStorage` only (keys prefixed `mr_`) |
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
- Cache name: `mission-routine-v1` — bump to `v2` etc. when deploying updates

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
