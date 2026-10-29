# FrenchoLinguo on the web — technical plan (draft v0.1, 2026-10-29)

Goal: a free way to use FrenchoLinguo on the iPhone as an icon on the Home Screen. It needs no Apple Developer Program ($99/yr) and no PC running a dev server. The web version is built from the **same Expo app**; there is no second codebase.

Related: [IOS.md](IOS.md) covers the native iPhone route (Expo Go, EAS, TestFlight). The two routes don't exclude each other. The web version can be the main way to use the app until paying for the native route is worth it.

Short version:

- Export the Expo Router app as a static website (`npx expo export -p web`).
- Host it free on **GitHub Pages**, deployed by GitHub Actions on every push to `main`.
- On the iPhone, open the site in Safari, then Share → **Add to Home Screen**. With a web app manifest it opens full screen, without the Safari UI.
- Progress stays on the phone in `localStorage` (AsyncStorage's web backend).

| Item | Cost |
|---|---|
| GitHub Pages hosting | free for a **public** repo. A private repo needs a paid GitHub plan; alternatives: Cloudflare Pages or Netlify (free), or EAS Hosting (free preview URLs; custom domains only on paid plans) |
| Custom domain (optional) | ~€10/yr |
| Apple account | not needed |

---

## 1. What the template already has

The TurkoLinguo app is already web-capable:
- `react-native-web` and `react-dom` are installed;
- the `npm run web` script exists;
- `app.json` has `web.favicon`;
- the README documents `npm run web` → http://localhost:8081.

So the web version is mainly about **export, hosting, a manifest, and fixing the few places where the web behaves differently** (§4).

## 2. Build and host

### 2.1 Export

In `app.json`:

```json
{
  "expo": {
    "web": {
      "output": "static",
      "favicon": "./assets/favicon.png"
    },
    "experiments": {
      "typedRoutes": true,
      "baseUrl": "/FrenchoLinguo"
    }
  }
}
```

- **`output: "static"`** writes one HTML file per route into `dist/`, which suits a static host. Dynamic routes (`session/[lesson]/[session]`, `grammar/[id]`, …) need `generateStaticParams` so that each lesson gets a page.
  - Simpler fallback: `"output": "single"`, a one-page app with only `index.html`. GitHub Pages has no SPA rewrites, so copy `index.html` to `404.html` so that deep links and reloads work.
  - **Recommended for M2: `single`** (less work). Move to `static` later if it's needed.
- **`baseUrl`** is only needed on GitHub Pages (`https://<user>.github.io/FrenchoLinguo/`). Leave it out for a custom domain or Cloudflare/Netlify.
  - Setting it also moves the **local dev** server under that path.
  - The manifest and icon links must include the base path.

Build locally:

```bash
npx expo export -p web
npx serve dist
```

`npx serve dist` checks the build in a browser on the PC. Files in `app/public/` are copied into `dist/` as they are.

### 2.2 Deploy with GitHub Actions: `.github/workflows/deploy-web.yml`

```yaml
# Exports the web app and publishes it on GitHub Pages.
name: Deploy web

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

defaults:
  run:
    working-directory: app

jobs:
  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          cache-dependency-path: app/package-lock.json

      - run: npm ci

      - name: Test
        run: |
          npm run typecheck
          npm run lint
          npm test -- --ci

      - run: npx expo export -p web

      - name: Single-page fallback for deep links
        run: cp dist/index.html dist/404.html && touch dist/.nojekyll

      - uses: actions/upload-pages-artifact@v3
        with:
          path: app/dist

      - id: deployment
        uses: actions/deploy-pages@v4
```

To set it up once, open GitHub → repo Settings → Pages → Source: **GitHub Actions**.

`.nojekyll` matters: Expo's output has folders that start with `_`, and Jekyll would drop them.

**Alternatives:**
- **EAS Hosting:** `npx expo export -p web`, then `eas deploy`. It needs an Expo account (also used for EAS Build) and gives a preview URL; production deploys need `eas deploy --prod`. Free tier limits aren't on the docs page; check expo.dev/pricing.
- **Cloudflare Pages / Netlify:** connect the repo, with build command `cd app && npm ci && npx expo export -p web` and output `app/dist`. Both have proper SPA rewrites, so no `404.html` trick is needed.

## 3. Home Screen app (PWA)

### 3.1 Manifest and iOS meta tags

Put `app/public/manifest.json` in place. With `baseUrl`, the paths include `/FrenchoLinguo/`:

```json
{
  "name": "FrenchoLinguo",
  "short_name": "Frencho",
  "start_url": "/FrenchoLinguo/",
  "scope": "/FrenchoLinguo/",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#FFFFFF",
  "theme_color": "#FFFFFF",
  "icons": [
    { "src": "/FrenchoLinguo/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/FrenchoLinguo/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

Use the real `colors.bg` value for the two colours.

Link it from the HTML.
- With `output: "single"`, run `npx expo customize public/index.html` and edit `<head>`.
- With `static`, edit `src/app/+html.tsx`.

```html
<link rel="manifest" href="/FrenchoLinguo/manifest.json" />
<link rel="apple-touch-icon" href="/FrenchoLinguo/apple-touch-icon.png" />  <!-- 180×180, no transparency -->
<meta name="apple-mobile-web-app-title" content="Frencho" />
<meta name="apple-mobile-web-app-status-bar-style" content="default" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
```

`viewport-fit=cover` is needed for the safe-area insets (§4.4).

The icons come from the same 1024×1024 master as the native icon ([IOS.md](IOS.md) §5.6).

### 3.2 Installing on the iPhone

1. Open the site in **Safari**.
2. Tap Share → **Add to Home Screen**. The name comes from `apple-mobile-web-app-title` or `short_name`. Recent iOS versions show an "Open as Web App" toggle; keep it on.
3. Open it from the icon. It runs full screen with its own storage (§4.1).

Put these steps on a small "Install on your iPhone" page or in the README. They can't be done from inside the app: iOS doesn't offer an install prompt.

### 3.3 Offline (later, not M2)

Without a service worker, the app needs the network at every launch. To work offline, add a **Workbox** service worker that caches the export:

```bash
npx expo export -p web
npx workbox-cli generateSW workbox-config.js
```

Register `sw.js` from the HTML `<head>`.

**Caution:** Expo's own docs warn that service workers "are known to cause unexpected behavior on web". A stale cache can keep users on an old version. Ship it only with an update strategy: a "New version, reload" banner, or skip-waiting on a new deploy.

**Recommended:** leave it out until M6. The app is small and is used online most of the time anyway.

## 4. What differs on the web (and what to do)

### 4.1 Progress storage

- AsyncStorage's web backend is **`localStorage`**, per origin. The ProgressContext code doesn't change.
- **Safari's 7-day rule:** a site used *in Safari* loses all script-written storage (localStorage, IndexedDB, service worker data) after 7 days of Safari use without the user visiting that site. Someone on a break would lose their review schedule.
- **Home Screen web apps are exempt.** Apps added to the Home Screen "are not part of Safari" and have their own counter of days of use, which use resets. WebKit says unexpected deletion there would be a bug. So **ask learners to install it to the Home Screen** rather than using it in a Safari tab.
- A Home Screen web app has **separate storage** from the same site in a Safari tab. Progress made in a tab doesn't appear in the installed app. Say so on the install page.
- Call `navigator.storage.persist()` once at start-up; it does nothing outside the web, so guard it with `Platform.OS === 'web'`. WebKit grants it by heuristics, and being a Home Screen web app counts in its favour.
- **Export / import progress** (JSON download and file upload) moves from "later" to **M4**. On the web it is the only backup: iCloud backup doesn't cover web app data the way it covers native app data.
- Data is lost if the user removes the Home Screen icon or clears website data.

### 4.2 Speech (`expo-speech` → Web Speech API)

- On the web, `expo-speech` calls the browser's `speechSynthesis`. iOS Safari uses the system voices, so `fr-FR` is there.
- Whether the downloaded **Enhanced/Premium** voices are offered to Safari has to be tested.
- `getAvailableVoicesAsync()` can return an empty list at first, because browsers load voices asynchronously. Retry or listen for `voiceschanged` before picking the voice, using the same `fr-FR` / Enhanced selection as in [IOS.md](IOS.md) §5.1.
- **User gesture:** iOS Safari ignores `speak()` that isn't started by a tap. That's fine for the play buttons. Any **auto-play** when a card appears (listen exercises) must wait for a first tap in the session, so show a "Tap to listen" button.
- **Silent switch:** reports conflict on whether Safari speech plays with the ring/silent switch on. `expo-audio`'s `setAudioModeAsync` has no effect on the web. Test on the phone; if it's muted, show the same "turn off silent mode" hint as native.
- iOS Safari ducks other audio during speech and for about a second after it. That's harmless.
- The player's own utterance queue (pause/resume) works on the web unchanged.
- Recorded audio (M6, Lingua Libre / Tatoeba) plays through `expo-audio`'s web player. Those files must be in `public/` or on a CDN, and count towards the first download unless they are loaded lazily.

### 4.3 Typing

- React Native Web maps `autoCorrect={false}`, `autoCapitalize="none"` and `spellCheck={false}` to the HTML `autocorrect`, `autocapitalize` and `spellcheck` attributes. Safari honours them. Also set `autoComplete="off"`.
- The **in-app accent bar** works the same. It matters more on the web, because a learner using a Safari tab may not have the French keyboard.
- `InputAccessoryView` is iOS-native only, so on the web keep the inline two-row bar.
- When the accent bar inserts a character and gives focus back to the input, Safari may close and reopen the keyboard. Test this. If the keyboard flickers, use `onPressIn` with `preventDefault` so the button doesn't take focus.
- `KeyboardAvoidingView` doesn't do anything useful on the web. Safari scrolls the focused input into view itself. Check that the accent bar stays visible above the keyboard in standalone mode.

### 4.4 Layout, safe areas, navigation

- In standalone mode the page goes under the status bar and the home indicator. That needs `viewport-fit=cover` (§3.1) plus padding from the safe-area insets.
- SDK 58 notes that Expo Router no longer provides safe-area insets on the web through `react-native-safe-area-context`. Use CSS `env(safe-area-inset-top)` and `env(safe-area-inset-bottom)`, for example in a small web-only wrapper style. Check this on the phone.
- A standalone web app has **no browser back button**. Every screen needs its own back/close control. Stack headers show a back button on the web; full-screen sessions already need a close button (IOS.md §5.4).
- Lock to portrait through the manifest. iOS may ignore it, so make sure landscape doesn't break the layout.
- Desktop browsers: centre the app in a phone-width column (max width ~480 px) so it doesn't stretch. It's cheap and makes testing on the PC nicer.

### 4.5 No haptics

- iOS Safari doesn't support the Vibration API.
- `expo-haptics` doesn't work on the web, so guard its calls with `Platform.OS !== 'web'`. Correct/wrong feedback must stay clear without haptics; colour and sound are already there.

### 4.6 Updates

- Without a service worker, a new deploy is picked up the next time the app is opened from scratch.
- Show the build version (commit SHA or tag) on the home screen's Developer/About section, so you can tell which version the phone is running.

## 5. Changes to the app (on top of IOS.md §6)

| File | Change |
|---|---|
| `app.json` | `web.output: "single"` (M2), `experiments.baseUrl: "/FrenchoLinguo"` if GitHub Pages |
| `public/manifest.json`, `public/icon-192.png`, `public/icon-512.png`, `public/apple-touch-icon.png` | new |
| `public/index.html` (via `npx expo customize public/index.html`) | manifest link, Apple meta tags, `viewport-fit=cover` |
| `src/app/_layout.tsx` | web only: `navigator.storage.persist()`, a phone-width column on desktop, safe-area padding |
| `src/ui/player.ts` | wait for voices on the web, and require a first tap before auto-play |
| `src/ui/controls.tsx` | accent bar keeps focus in the input (no keyboard flicker); `autoComplete="off"` |
| `src/session/*` | haptics guarded by `Platform.OS !== 'web'` |
| `src/progress/` | export/import progress (JSON), M4 |
| `.github/workflows/deploy-web.yml` | new (§2.2) |
| `README.md` | "Use it on your iPhone without the App Store": the site URL, Add to Home Screen, the separate-storage note |

The Jest tests and `ci.yml` stay the same. The web export runs in `deploy-web.yml`, so a web-only build break shows up there.

## 6. Milestones

**M2 — Skeleton:**

1. **M2.7** `npm run web` works with the empty course. Then `npx expo export -p web` with `output: "single"`, and `npx serve dist` on the PC.
2. **M2.8** Add `deploy-web.yml` and turn on GitHub Pages. This decides public vs private repo, or Cloudflare/Netlify instead.

**M4 — Vertical slice:**

3. **M4.x** Manifest, icons and meta tags. Add to the Home Screen on the iPhone and play the full lesson there. Test speech (voice, silent switch, tap-to-listen), the accent bar with the keyboard, safe areas, and back/close navigation.
4. **M4.x** Progress export/import, plus `navigator.storage.persist()`.

**M6 — Polish:**

5. **M6.x** Workbox offline cache with an update banner, recorded audio loaded lazily, and a custom domain (optional).

When the web version's limits start to hurt, switch to the native route in [IOS.md](IOS.md): $99/yr, TestFlight. The likely reasons are voice quality, the silent switch, offline use, or wanting haptics.

## 7. Open decisions

1. **Hosting:** GitHub Pages (needs a public repo, or a paid GitHub plan), Cloudflare Pages / Netlify (private repo fine, real SPA rewrites), or EAS Hosting.
2. **`single` vs `static` output:** `single` for M2 (recommended); `static` later if per-page HTML is useful.
3. **Custom domain:** whether to have one, and its name. With one, `baseUrl` isn't needed.
4. **Web as the main iPhone route until when:** for example until A1 is complete, then decide on the $99 native app.
5. **Offline support:** M6 (recommended) or earlier.

---

## Sources (checked 2026-10-29)

- Expo, Publish websites (`web.output` single/static, `expo export -p web`, GitHub Pages with `baseUrl` and `--nojekyll`, EAS Hosting): https://docs.expo.dev/guides/publishing-websites/
- Expo, Progressive web apps (manifest, `+html.tsx` / `public/index.html`, Workbox, the service-worker caution): https://docs.expo.dev/guides/progressive-web-apps/
- EAS Hosting (`eas deploy`, custom domains on paid plans): https://docs.expo.dev/eas/hosting/introduction/
- Expo SDK 58 changelog (no web safe-area insets through `react-native-safe-area-context`; use CSS `env()`): https://expo.dev/changelog/sdk-58
- WebKit, Full Third-Party Cookie Blocking and More (7-day cap on script-writable storage; Home Screen web apps have their own counter): https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/
- WebKit, Updates to Storage Policy (quotas, eviction, `StorageManager.persist()` heuristics incl. Home Screen web apps): https://webkit.org/blog/14403/updates-to-storage-policy/
- expo-speech docs (iOS silent mode, voices): https://docs.expo.dev/versions/latest/sdk/speech/
- Apple Developer Forums, Text-to-speech not working on iOS Safari (silent switch report): https://developer.apple.com/forums/thread/49875
- Apple Developer Forums, AVSpeechSynthesizer ignores mute switch on iOS 13 (the conflicting report): https://developer.apple.com/forums/thread/120270
- WebKit Bugzilla 278598 (iOS Safari `speechSynthesis` ducks other audio): https://bugs.webkit.org/show_bug.cgi?id=278598
