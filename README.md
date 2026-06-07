# Mission Routine — PWA

Gamified daily routine tracker for Uzaiyr (age 10) and Imtiyaz (age 7).  
Shared family-tablet app — no backend, no login, pure HTML + vanilla JS, localStorage only.

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
3. Confirm — the Mission Routine icon appears on your home screen
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
const CACHE_NAME = 'mission-routine-v2'; // increment this
```

This tells the browser to discard the old cache and install the new one on next visit.
