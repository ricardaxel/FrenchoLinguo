# FrenchoLinguo on iPhone — technical plan (draft v0.1, 2026-10-29)

Target device: **iPhone**. Development machine: **Windows 10, no Mac**.
Template: the TurkoLinguo app (Expo SDK 57, React Native 0.86, TypeScript, Expo Router, ts-fsrs, expo-speech, AsyncStorage), which was Android-first.

Short version:

- **Develop in Expo Go on the iPhone** (free, no Apple account needed). **Start the skeleton on SDK 58, not 57**: Expo Go on the App Store has supported only SDK 58 since 22 October 2026 (§1.1).
- **Build in the cloud with EAS Build** (macOS machines run by Expo). You never need a Mac. The free plan covers 15 iOS builds a month.
- **Ship to the user's own iPhone through TestFlight.** This needs the Apple Developer Program ($99/yr). Without the program, Expo Go is the only way to run the app on an iPhone.
- **Keep Android as a secondary target.** Its APK workflow already works and costs nothing.

| Item | Cost | Needed for |
|---|---|---|
| Expo Go (App Store) | free | development through M4 |
| Expo account | free | Expo Go login, EAS |
| EAS Free plan | free: 15 iOS + 15 Android builds/month, low priority, 45 min timeout | dev / preview / production builds |
| EAS Starter (optional) | $19/month incl. $45 build credit (iOS ≈ $2 per medium build) | only if you need more builds or faster queues |
| Apple Developer Program | $99/year | any build of our own app on a real iPhone, TestFlight, App Store |
| Mac / Xcode | not needed | — |

---

## 1. Development on Windows with no Mac

### 1.1 Expo Go on iPhone: which SDK

- **The App Store version of Expo Go is now 58.0.5 (released 22 October 2026), and it supports SDK 58 only.** It needs iOS 16.4 or later.
- SDK 58 came out on 15 October 2026 with React Native 0.88. Expo warned that "the new Expo Go will support only SDK 58". So **the template's SDK 57 does not open in the current iOS Expo Go.**
- History: in September 2026 the App Store Expo Go moved to SDK 57. Before that it had sat on SDK 54 for months while new builds waited for Apple's approval.
- TurkoLinguo's README says the *Play Store* Expo Go was on SDK 57. Expo said the SDK 58 Play Store build would follow "within a few weeks". Check the Play Store before deciding how to run Android (§4.3).

**Decision for M2:** copy the template, then upgrade it to SDK 58 before writing anything new:

```bash
npx expo install expo@^58.0.0
npx expo install --fix
npx expo-doctor
```

SDK 58 changes that matter here:
- React Native 0.86 → 0.88.
- The Strict TypeScript API is now the default, so expect some typecheck fixes.
- `npx expo prebuild` now generates a `SceneDelegate` for the iOS 27 UIScene life cycle. This is handled for us because we don't commit `ios/`.
- Update `react-native-safe-area-context` with `npx expo install`.

Expect this cycle to repeat. Each time Expo ships a new Expo Go, it drops the previous SDK. If you ever lag behind, a development build (§1.4) keeps working on the old SDK.

**Login on both ends.** Since the SDK 57 Expo Go for iOS, running a project in Expo Go needs the same Expo account signed in on the computer and in the app. Development builds don't need this.

```bash
npx expo login
```

On the phone, open Expo Go → Home → avatar (top right) → sign in.

### 1.2 Connecting the iPhone to the Windows dev server

**LAN (default, fastest):**

1. Connect the PC and the iPhone to the same Wi-Fi. Guest networks and networks with "client isolation" block this; use the tunnel on those.
2. In `app/`, run `npm start`. Run it from **Windows**, not WSL2: WSL2 reports a virtual IP that the phone can't reach. The `npm run content` script can keep calling `wsl python3`.
3. On the first run, Windows Defender Firewall asks about Node.js. Allow it on **Private** networks. The Wi-Fi network profile must also be set to *Private* (Settings → Network & Internet → Wi-Fi → network → Private).
4. On the iPhone, scan the QR code with the **Camera** app. It opens Expo Go. Allow Expo Go's **Local Network** permission when asked; you can change it later in Settings → Privacy & Security → Local Network.
5. If the QR code shows the wrong IP (for example from a Hyper-V or VirtualBox adapter), set the IP by hand:
   ```powershell
   $env:REACT_NATIVE_PACKAGER_HOSTNAME = "192.168.1.23"; npm start
   ```

**Tunnel (when LAN fails):**

```bash
npx expo start --tunnel
```

This installs `@expo/ngrok` on first use and routes traffic through the internet. It is slower on reload but works across networks, including phone hotspot ↔ PC.

### 1.3 What Expo Go can't do

Expo Go is a prebuilt app that ships with a fixed set of native modules. It can't:

| Limitation | Effect on FrenchoLinguo |
|---|---|
| Run native modules it doesn't ship, or **config plugins** | `plugins/withReleaseSigning.js` is ignored, which is fine (it's Android-only). Any future native module (Piper TTS, a custom audio engine, widgets) needs a dev build. |
| Apply `app.json` iOS settings: bundle id, icon, splash, display name, `infoPlist`, privacy manifest | None of these can be checked in Expo Go. |
| Run an SDK other than the store's current one | Forces upgrades on Expo's schedule (§1.1). |
| Store data under our own app | AsyncStorage data is kept inside Expo Go's container. Progress does **not** carry over to the real app. |
| Match real app performance and start-up | Expo Go runs in dev mode with a JS bundle from the PC. |
| Test notifications, background audio, App Store-only behaviour | Not needed before M6. |

What **does** work in Expo Go, and covers everything up to M4:
- all of the template's JS (Expo Router, ts-fsrs, AsyncStorage)
- `expo-speech`, `expo-audio`, `expo-haptics`
- `react-native-safe-area-context` and `react-native-svg`

### 1.4 When to switch to a development build

Switch when the first of these happens:
- the app needs a native module Expo Go doesn't have;
- you want to check the real icon, splash, name and privacy manifest;
- you need to stay on an SDK that Expo Go has dropped.

M6 (TestFlight) needs one in any case.

A development build is "your own Expo Go": the app with `expo-dev-client` added. It loads JS from `npx expo start`, so hot reload still works. To set one up:

1. Install the dev client:
   ```bash
   npx expo install expo-dev-client
   ```
   After this, `npx expo start` targets the dev build. Use `npx expo start --go` to keep opening the app in Expo Go.
2. Register the iPhone and make the build:
   ```bash
   npm install -g eas-cli
   eas login
   eas device:create
   eas build -p ios --profile development
   ```
   `eas device:create` gives you a link or QR code. Open it on the iPhone and install the profile it offers; this registers the phone's UDID.
3. Install the build from the QR code or link that EAS prints.
4. On the iPhone, turn on **Settings → Privacy & Security → Developer Mode** and restart. This is required for development and ad-hoc builds, not for TestFlight.
5. Run `npx expo start`.

This needs the paid Apple Developer Program. A free Apple account can only sign apps with Xcode on a Mac (7-day profiles, 3 devices), so a Windows-only setup can't use that route.

Without a Mac there is no iOS Simulator. EAS can build simulator `.app` files, but they only run on macOS, so skip `"simulator": true` profiles.

---

## 2. Builds (EAS Build, cloud macOS)

### 2.1 One-time setup

From `app/`:

```bash
npm install -g eas-cli
eas login
eas init
eas build:configure
```

`eas init` creates the EAS project and writes `extra.eas.projectId` to `app.json`. `eas build:configure` writes `eas.json`; replace its contents with the file below.

### 2.2 `eas.json`

```json
{
  "cli": {
    "version": ">= 19.1.0",
    "appVersionSource": "remote"
  },
  "build": {
    "base": {
      "node": "22.x"
    },
    "development": {
      "extends": "base",
      "developmentClient": true,
      "distribution": "internal",
      "channel": "development",
      "android": { "buildType": "apk" }
    },
    "preview": {
      "extends": "base",
      "distribution": "internal",
      "channel": "preview",
      "android": { "buildType": "apk" }
    },
    "production": {
      "extends": "base",
      "distribution": "store",
      "channel": "production",
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {
      "ios": {
        "ascAppId": "<App Store Connect app id, filled after the app record exists>"
      }
    }
  }
}
```

How each profile builds and installs:

| Profile | iOS signing | How it gets on the phone | Use |
|---|---|---|---|
| `development` | ad hoc, registered UDIDs only | QR/link from EAS; needs Developer Mode | daily dev once Expo Go isn't enough |
| `preview` | ad hoc (`distribution: internal`) | QR/link; needs Developer Mode | a release-like build (no dev menu) for your own phone or a few testers' phones without TestFlight |
| `production` | App Store distribution | TestFlight, then the App Store | real releases |

Notes:
- **`appVersionSource: "remote"` + `autoIncrement`**: EAS stores the iOS `buildNumber` (and the Android `versionCode` for EAS Android builds) and bumps it on every production build. App Store Connect rejects a repeated build number, so this avoids that error. The user-facing `version` (e.g. `0.2.0`) still comes from `app.json`.
- **`channel`** only matters if you add EAS Update (over-the-air JS updates) later. It's harmless now.
- **Adding a new device to `development` or `preview`** needs `eas device:create` and then a rebuild, or `eas build:resign`. In CI, add `--refresh-ad-hoc-provisioning-profile`.
- **Ad hoc limits:** 100 iPhones per year per app, and removed devices still count. A newly registered device can take 24–72 hours to work on a fresh membership.
- **The `android` keys** let EAS build APKs too, which is optional (§4.3). The current Android release path is still the GitHub Actions workflow.

### 2.3 Apple Developer Program: what works with and without it

| | Without ($0) | With ($99/yr) |
|---|---|---|
| Expo Go on the iPhone | ✅ | ✅ |
| Web build in Safari (`npm run web`) | ✅ | ✅ |
| EAS iOS build of our own app on a device (dev / preview / production) | ❌ (EAS needs paid-account signing credentials) | ✅ |
| TestFlight (internal: up to 100 testers, no review) | ❌ | ✅ |
| App Store | ❌ | ✅ |
| `eas go` (your own Expo Go on TestFlight for an older SDK) | ❌ | ✅ |

Enrol as an **Individual** at developer.apple.com/programs; it needs an Apple account with 2FA. The seller name on the App Store is then your legal name. An Organization account shows a company name but needs a D-U-N-S number.

**When to pay:** not before M4. Expo Go covers M2–M4. Pay when you want the app installed as itself (M5/M6) or need a native module.

### 2.4 Credentials handled by EAS

The first time you run `eas build -p ios` interactively, EAS asks for your Apple ID. On Windows this works in PowerShell, including the 2FA code. EAS then creates and stores:

- the **Distribution Certificate** (one per team, valid 1 year; EAS renews it);
- **Provisioning profiles**: ad hoc ones for development and preview (they include the registered UDIDs), and an App Store one for production;
- the **App ID** (bundle identifier) in the Apple Developer portal;
- an **App Store Connect API key** for EAS Submit. Create it once with `eas credentials` → iOS → App Store Connect API Key. It lets CI submit without an Apple ID or 2FA.

To inspect or reset them, run `eas credentials`.

iOS credentials are less fragile than the Android keystore. If they're lost or revoked, EAS can regenerate them, and users don't have to reinstall. App updates are tied to the bundle id and the team, not to one certificate.

If you ever want EAS to build Android too, upload the existing keystore under `eas credentials` → Android → Keystore, and **keep the backup**. The rules in TurkoLinguo's `docs/RELEASING.md` still apply.

Commit nothing secret. `.gitignore` from the template already covers `*.p8`, `*.p12`, `*.mobileprovision`, `*.jks` and `/ios`. If you ever use local credentials, also add `credentials.json`.

---

## 3. Distribution

### 3.1 Which channel

| Channel | Who | Limits | Fit |
|---|---|---|---|
| **TestFlight, internal testing** | up to 100 members of your App Store Connect team | no Apple review; each build expires after 90 days | **Recommended for the user's own iPhone (M6).** Installs like a normal app; no UDID or Developer Mode. |
| TestFlight, external testing | up to 10,000 people by email or public link | the first build of a version goes through a short beta review | for French-learner beta testers later |
| Ad hoc (`preview` profile) | registered devices only | 100 devices/year; Developer Mode; rebuild for each new device | quick release-like builds without App Store Connect; dev builds |
| App Store | everyone | full App Review | when the course is ready (after M5, or M7) |

TestFlight and the App Store both need an **app record** in App Store Connect. Create it once, at App Store Connect → Apps → + → New App, with the bundle id from §5.6 and a SKU such as `frencholinguo`. Put its numeric Apple ID into `submit.production.ios.ascAppId`.

### 3.2 EAS Submit

Build and send to TestFlight in one step:

```bash
eas build -p ios --profile production --auto-submit
```

Or submit the latest finished build later:

```bash
eas submit -p ios --latest
```

The build shows up in TestFlight about 5–15 minutes after the upload, once Apple has processed it. Releasing it to the App Store is a manual step in App Store Connect: add it to a version, fill in the metadata, then submit for review.

**Apple's upload rule:** since 28 April 2026, every upload, TestFlight included, must be built with Xcode 26+ against the iOS 26 SDK. EAS's default image for SDK 58 meets this. Don't pin an old `image` in `eas.json`. A similar rule for the iOS 27 SDK is reported for April 2027 (not confirmed on Apple's site).

### 3.3 App Store review points for a language-learning app

1. **Privacy manifest (`PrivacyInfo.xcprivacy`).** Expo generates it from `expo.ios.privacyManifests` and merges in the manifests that Expo and React Native packages ship. If Apple's post-upload email (ITMS-91053) lists a missing "required reason" API, add it to `app.json`. A typical entry if UserDefaults is flagged:
   ```json
   "ios": {
     "privacyManifests": {
       "NSPrivacyAccessedAPITypes": [
         { "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategoryUserDefaults", "NSPrivacyAccessedAPITypeReasons": ["CA92.1"] }
       ]
     }
   }
   ```
   Check after the first TestFlight upload. The email usually arrives within minutes.
2. **App Privacy "nutrition label"** in App Store Connect: with no analytics, ads or accounts, answer **"Data Not Collected"**. Progress stays on the device. If analytics or crash reporting are added later, the answers change.
3. **Privacy policy URL**: every app needs one, even one that collects nothing. A short page saying that no data leaves the device is enough; a GitHub Pages page works.
4. **Export compliance**: the app uses no custom encryption. Set `"ios": { "config": { "usesNonExemptEncryption": false } }` so that each build doesn't stop at the encryption question in TestFlight.
5. **Account deletion (Guideline 5.1.1(v))**: not relevant now, since there are no accounts. If accounts or sync are ever added, the app must let users **delete the account from inside the app**, not just deactivate it. Sign in with Apple is required only if other third-party social logins are offered.
6. **Age rating**: Apple's new questionnaire (13+/16+/18+ tiers; answers required since 31 January 2026) must be filled in. Expected result: **4+**, because the app has no user-generated content, chat, web view or ads.
   - Talk tasks happen off-app (partner cards). If in-app contact with native speakers or open chat is ever added, the rating and the moderation rules (Guideline 1.2) change.
   - Check the sentence content (Tatoeba) for mature themes before release.
7. **Minimum functionality (4.2) / spam (4.3)**: a full course passes. Don't submit the M4 one-lesson slice to the App Store; use TestFlight for that. TurkoLinguo and FrenchoLinguo share code under one developer account, which is fine as long as each is a distinct, complete app.
8. **Third-party content licences**: list CC BY / CC BY-SA attributions (Tatoeba, Lingua Libre, Lexique) on an in-app "Credits" screen. Reviewers and the licences both expect this.
9. **iPad**: with `supportsTablet: true`, App Review also tests on iPad and asks for iPad screenshots. Set it to `false` unless the layout is made iPad-ready (open decision).
10. **Screenshots**: iPhone screenshots at the sizes App Store Connect currently asks for. Take them from a TestFlight build on the iPhone.

---

## 4. CI

### 4.1 What to keep

- `.github/workflows/ci.yml` (typecheck, lint, Jest on `ubuntu-latest`): **copy as is**. It is platform-independent.
- `.github/workflows/release-apk.yml`: **keep for Android** (§4.3). Rename `turkolinguo` → `frencholinguo`.
  - Re-check `withReleaseSigning.js` after the SDK 58 upgrade. It throws if the Gradle template changed, so a failure is loud, not silent.

### 4.2 New workflow: `.github/workflows/release-ios.yml`

Approach: GitHub Actions only **starts** the EAS build. It finishes in seconds and uses no macOS runner minutes. EAS builds on macOS and auto-submits to TestFlight.

```yaml
# Builds the iOS app on EAS and submits it to TestFlight.
#
# Publishing a release with a tag like v0.2.0 sets the app version to 0.2.0;
# EAS sets the build number (appVersionSource: remote). Running the workflow by
# hand builds the chosen profile from the current branch.
name: Release iOS

on:
  release:
    types: [published]
  workflow_dispatch:
    inputs:
      profile:
        description: EAS build profile
        type: choice
        options: [production, preview]
        default: preview

concurrency:
  group: release-ios-${{ github.ref }}
  cancel-in-progress: false

defaults:
  run:
    working-directory: app

jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          cache-dependency-path: app/package-lock.json

      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - run: npm ci

      - name: Test
        run: |
          npm run typecheck
          npm run lint
          npm test -- --ci

      - name: Set version from release tag
        if: github.event_name == 'release'
        env:
          TAG: ${{ github.event.release.tag_name }}
        run: |
          node -e '
            const fs = require("fs");
            const m = /^v?(\d+)\.(\d+)\.(\d+)$/.exec(process.env.TAG);
            if (!m) { console.error(`Tag "${process.env.TAG}" is not vX.Y.Z`); process.exit(1); }
            const app = JSON.parse(fs.readFileSync("app.json", "utf8"));
            app.expo.version = m.slice(1).map(Number).join(".");
            fs.writeFileSync("app.json", JSON.stringify(app, null, 2));
            console.log(`version ${app.expo.version}`);
          '

      - name: Start EAS build
        env:
          PROFILE: ${{ github.event_name == 'release' && 'production' || inputs.profile }}
        run: |
          if [ "$PROFILE" = production ]; then submit=--auto-submit; fi
          eas build --platform ios --profile "$PROFILE" --non-interactive --no-wait $submit
```

Notes:
- **Secrets**: `EXPO_TOKEN`, a personal access token from expo.dev → Account settings → Access tokens. The Apple side uses the App Store Connect API key stored in EAS (§2.4), so no Apple secrets are needed in GitHub.
- **First run**: do the first iOS build **interactively** on the PC (`eas build -p ios --profile production`) so that EAS can create the credentials. After that, `--non-interactive` works in CI.
- **Uncommitted version change**: the tag-to-version edit to `app.json` isn't committed, but EAS uploads the working directory as it is, so the build carries the tag's version. The same is true of the APK workflow today.
- **`--no-wait`**: the job goes green once the build is queued. Watch progress on expo.dev or by email. Remove the flag if a later step needs the result.
- **Preview builds in CI** reuse the existing ad-hoc profile. If you added a device since the last preview build, add `--refresh-ad-hoc-provisioning-profile`; it needs the ASC API key stored in EAS.
- **Alternative: EAS Workflows** (`.eas/workflows/*.yml`, run by Expo instead of GitHub). It does the same job; GitHub Actions is kept here so that CI and Android stay in one place.
- **Build budget**: one production build per release plus a few preview builds stays well inside the free 15 iOS builds a month.

`docs/RELEASING.md` gets a new "iOS (TestFlight)" section:
- the one-time setup: Apple program, app record, `eas credentials`, `EXPO_TOKEN`;
- cutting a release, which stays the same: `gh release create v0.2.0 --generate-notes` now triggers both workflows;
- that each TestFlight build expires after 90 days.

### 4.3 Keep Android as a secondary target? Yes

**Reasons:**
- It costs nothing extra. The APK workflow and signing are already written and tested in TurkoLinguo.
- Expo Go on Android is a second test device.
- The two platforms share all the code.

**The cost:** a second platform to check before each release.

**Rule:** iOS is the release target. Android must pass CI and get a quick smoke test.

If the Play Store Expo Go is still on SDK 57 when the skeleton moves to 58, test Android through an EAS `development` APK instead. Android dev builds need no paid account and no device registration.

---

## 5. iOS-specific app concerns

### 5.1 `expo-speech` French voices

- **Language.** Use `language: 'fr-FR'` in `src/ui/player.ts`, replacing `'tr-TR'`. Every iPhone ships with a compact `fr-FR` voice (Thomas or Amélie, depending on the iOS version). It is understandable but robotic.
- **Enhanced and Premium voices** are free downloads: Settings → Accessibility → Spoken Content → Voices → French → France → pick a voice marked *Enhanced* or *Premium*. Mention this in the app (onboarding or the Unit 0 sound lesson), the same way TurkoLinguo's README tells Android users to install a TTS voice.
- **Pick the best voice at runtime**, never by a hardcoded id:
  ```ts
  const voices = await Speech.getAvailableVoicesAsync();
  const fr = voices.filter((v) => v.language === 'fr-FR');
  const best = fr.find((v) => v.quality === Speech.VoiceQuality.Enhanced) ?? fr[0];
  Speech.speak(text, { language: 'fr-FR', voice: best?.identifier, rate });
  ```
  Avoid `fr-CA` voices: the target variety is France. Cache the choice in memory.
- **Silent switch.** The docs say *"On iOS physical devices, `expo-speech` won't produce sound if the device is in silent mode."* A learner who has the ring/silent switch on would hear nothing, and wouldn't know why.
  - Fix: add `expo-audio` and set the app's audio session at start-up in `_layout.tsx`:
    ```ts
    import { setAudioModeAsync } from 'expo-audio';
    setAudioModeAsync({ playsInSilentMode: true, interruptionMode: 'duckOthers' }).catch(() => {});
    ```
  - `expo-speech` uses the app's audio session by default (`useApplicationAudioSession`), so the speech should then play in silent mode.
  - **Check this on the device in M2.** It is the documented intent of both APIs, but I haven't seen it confirmed together.
  - If it still doesn't play, show a one-time hint ("Turn off silent mode to hear French"). Recorded audio played through `expo-audio` (M6) follows the same setting.
- **Speech rate.** iOS (AVSpeechSynthesizer) and Android scale speech rate differently, so `NORMAL_RATE = 0.9` and `SLOW_RATE = 0.5` need retuning by ear on the iPhone. 0.5 is likely too slow for normal speech. Consider per-platform constants.
- **Pausing.** The player's own utterance queue (a workaround for Android having no `pause()`) works on iOS too. Keep it.

### 5.2 French accents: iOS keyboard vs the in-app accent bar

**What iOS offers:**
- Long-press on the English keyboard: hold **e** → é è ê ë ē…, **a** → à â æ…, **c** → ç, **o** → ô œ…, **u** → ù û ü…, **i** → î ï….
- Users can also add the French keyboard (Settings → General → Keyboard → Keyboards → Add New Keyboard → French (France)). It uses AZERTY and the same long-press menus.

**Why keep the accent bar from `CURRICULUM.md` §5 anyway:**
- Long-press is slow and hard to find for beginners.
- It hides which accent is right among 5–7 options, and the curriculum *teaches* that choice.
- The bar works the same on Android.

**Make it iOS-friendly:**
- 13 keys (é è ê à â ç î ï ô û ù ë œ) don't fit in one row on a 375–440 pt screen at Apple's 44 pt minimum touch size. Use **two rows**: frequent keys (é è à ç ê ô) on top and the rest below. Alternatively, show a priority row with a "more" key.
- Optionally attach the bar to the keyboard with React Native's iOS-only `InputAccessoryView`, so it sits right above the keyboard and moves with it. Keep the current inline bar on Android.
- No capital accented letters are needed, because answer checking ignores case.

**Smart punctuation:** iOS turns `'` into `’` while you type. The planned normalisation of ' and ’ handles this; add a unit test for it.

### 5.3 Autocorrect and autocapitalize off for answer inputs

The template's `TurkishInput` already sets `autoCapitalize="none"`, `autoCorrect={false}` and `spellCheck={false}`. Keep these on the new `FrenchInput`. Also add:

```tsx
autoComplete="off"
textContentType="none"   // no AutoFill suggestions on iOS
keyboardType="default"
```

On the device, check that the QuickType suggestion bar doesn't offer the right answer. With the French keyboard enabled, iOS predicts accented words.

For English-answer inputs (FR → EN), keep the same settings.

### 5.4 Safe areas, gestures, haptics

- **Safe areas.** The template already uses `react-native-safe-area-context` with `SafeAreaView edges={['top','bottom']}` on full-screen session screens. Stack headers handle the top inset on the other screens.
  - Check the Dynamic Island and the home indicator on the user's iPhone.
  - SDK 58 notes that some new devices may need `SafeAreaView` inside scroll views.
- **Gestures.** Sessions set `gestureEnabled: false`, which disables iOS swipe-back so a session can't be lost by accident. So every full-screen screen needs a visible close button.
- **Keyboard.** `KeyboardAvoidingView` with `behavior="padding"` on iOS is already in `SessionRunner`. Re-test it with the taller two-row accent bar.
- **Haptics.** Add `expo-haptics` (it works in Expo Go): `Haptics.notificationAsync(Success)` for a correct answer and `(Error)` for a wrong one, plus `selectionAsync()` on chip taps.
  - It follows the user's system haptics setting.
  - It does nothing on web; guard with `Platform.OS !== 'web'`.

### 5.5 AsyncStorage persistence and iCloud backup

- On iOS, AsyncStorage writes to files in the app's *Application Support* directory. That data:
  - survives app updates;
  - is **included in iCloud and computer backups**, so it is restored when the user moves to a new iPhone from a backup;
  - is **deleted when the app is uninstalled**;
  - **does not sync** live between devices.
- Expo Go, dev builds (if they use a different bundle id), TestFlight and the App Store build keep **separate** data. That's expected; say so in the README.
- The iCloud backup size is negligible: FSRS state is a few hundred KB at most.
- Later (not M2): an "Export / import progress" JSON through the share sheet, as a safety net for uninstalls. Live iCloud sync would need CloudKit or iCloud key-value storage, which means native code and a dev build; leave it out.

### 5.6 Bundle identifier, name, icons, splash

- **Bundle identifier**: `com.ricardaxel.frencholinguo`, matching TurkoLinguo's Android package naming. Use the same value for `android.package`. It is **permanent** once the app is in App Store Connect.
- **Display name**: "FrenchoLinguo" is 13 characters, and the iPhone home screen cuts names at about 11–12 characters (it may show "FrenchoLing…"). Choose a short `CFBundleDisplayName` such as "Frencho" (open decision). The App Store name can stay "FrenchoLinguo".
- **App icon**: 1024×1024 PNG, **no transparency** (App Store Connect rejects an alpha channel), square corners (iOS rounds them).
  - Optional: `ios.icon` can take `{ "light", "dark", "tinted" }` variants for iOS 18+ dark and tinted icons.
  - iOS 26 "Liquid Glass" layered icons are made in Apple's Icon Composer, which needs a Mac. A plain PNG is fine.
- **Splash**: configure the `expo-splash-screen` plugin (the template has `splash-icon.png` but doesn't configure it). Use an image about 200 pt wide and `backgroundColor` matching `colors.bg`; add a `dark` variant only if dark mode is ever supported (`userInterfaceStyle` is `light`).
- Check all of these in the first `preview` or TestFlight build; Expo Go can't show them.

---

## 6. Changes to the TurkoLinguo template (read-only review of `TurkoLinguo/app`)

### `app.json`

| Key | Template | FrenchoLinguo |
|---|---|---|
| `name` / `slug` / `scheme` | TurkoLinguo / turkolinguo / turkolinguo | FrenchoLinguo / frencholinguo / frencholinguo |
| `version` | 0.1.0 | 0.1.0 |
| `ios` | only `supportsTablet: true` | add `bundleIdentifier: "com.ricardaxel.frencholinguo"`, `supportsTablet: false` (decision), `config.usesNonExemptEncryption: false`, `infoPlist.CFBundleDisplayName` (decision), `privacyManifests` if Apple asks (§3.3) |
| `android.package` | com.ricardaxel.turkolinguo | com.ricardaxel.frencholinguo; `versionCode: 1` stays (the APK workflow sets it from the tag) |
| `plugins` | expo-router, expo-status-bar, ./plugins/withReleaseSigning | add `expo-splash-screen` (with config) and `expo-audio`; keep `withReleaseSigning` for Android |
| `extra.eas.projectId` | — | written by `eas init` |
| icons / splash | Turkish assets | new French icon set (§5.6) |

About the `expo-audio` plugin: the app only plays audio and never records. If Apple's upload email asks for `NSMicrophoneUsageDescription`, use the plugin option that turns off the microphone permission (check the option name in the SDK 58 `expo-audio` docs).

### `package.json`

- `name` → `frencholinguo`.
- SDK 58 versions: run `npx expo install expo@^58.0.0 && npx expo install --fix`. Don't edit the versions by hand.
- Add, with `npx expo install`:
  - `expo-audio` (audio session now, recordings in M6)
  - `expo-haptics`
  - `expo-splash-screen`, if it isn't already a dependency
  - `expo-dev-client`, only when the first dev build is made (§1.4)
- Scripts:
  - Add `"ios": "expo start --ios"` only as an alias of `start`; it can't open a simulator on Windows. Better: add `"start:tunnel": "expo start --tunnel"`.
  - `content` stays (`wsl python3 -I ../scripts/build_content.py`).

### `plugins/withReleaseSigning.js`

Android-only; it only changes `build.gradle`. It has no effect on iOS prebuild or EAS iOS builds. Keep it for the APK workflow, and run `npx expo prebuild --platform android --no-install` once after the SDK 58 upgrade to catch template changes.

### `src/`

| File | Change |
|---|---|
| `ui/player.ts` | `'tr-TR'` → `'fr-FR'`; runtime voice choice (§5.1); retune `NORMAL_RATE` / `SLOW_RATE` on the iPhone |
| `app/_layout.tsx` | title → FrenchoLinguo; one-time `setAudioModeAsync({ playsInSilentMode: true, … })` |
| `ui/controls.tsx` | `TurkishInput` → `FrenchInput`; Turkish `LETTERS` → French accent bar in two rows (§5.2); add `autoComplete="off"` and `textContentType="none"`; optional iOS `InputAccessoryView`; placeholder "Type in French…" |
| `session/*` | add haptic feedback on grading (§5.4); check every full-screen session screen has a close button (swipe-back is off) |
| `engine/*` | rewrite (PLAN.md §3). It isn't iOS-specific, but make sure the French engine runs on Hermes: `String.prototype.normalize('NFC')` and `toLocaleLowerCase('fr')` both work on Hermes, so add a Jest test anyway |
| `progress/ProgressContext.tsx` | change the storage `KEY` to a FrenchoLinguo-specific name; otherwise unchanged |
| `content/content.json` | empty course for M2 |

### Repo files

- **`README.md`**: rewrite "Run it on your Android phone" as "Run it on your iPhone", using §1.1–1.2: Expo Go from the App Store, log in on both ends, LAN or tunnel, the Local Network permission, enhanced French voices, the silent switch. Keep a short Android section, and replace "SDK 57 matches the Play Store version" with the SDK line from §1.1.
- **`AGENTS.md`**: copy as is. Its "Expo has changed, read the versioned docs" rule now points to v58.
- **`.gitignore`**: already ignores `/ios`, `/android` and the credential file types. Add `credentials.json`.
- **`.github/workflows/`**: copy `ci.yml`; copy `release-apk.yml` with the renames; add `release-ios.yml` (§4.2).
- **`docs/RELEASING.md`**: add the iOS section (§4.2). Rename the keystore and alias to `frencholinguo`. **Create a new Android keystore**; don't reuse TurkoLinguo's.

---

## 7. Milestones (fit into PLAN.md M2, with the later items marked)

**M2 — Skeleton (no Apple payment needed):**

1. **M2.1** Copy `TurkoLinguo/app`, the pipeline and CI. Rename the app (§6). Strip the Turkish content and engine down to an empty course.
2. **M2.2** Upgrade to **SDK 58**:
   ```bash
   npx expo install expo@^58.0.0
   npx expo install --fix
   npx expo-doctor
   ```
   Fix the typecheck issues from the Strict TypeScript API. CI goes green.
3. **M2.3** Run it on the iPhone in Expo Go: LAN, with the tunnel as fallback, logged in on both ends. Write the README iPhone section.
4. **M2.4** iOS basics in JS:
   - `fr-FR` voice selection;
   - `expo-audio` silent-mode session, **checked with the switch on**;
   - rate retuning;
   - French accent bar (two rows);
   - input flags;
   - haptics;
   - safe areas and close buttons.
5. **M2.5** `eas init` + `eas.json` (§2.2) committed. `app.json` gets the bundle id, `usesNonExemptEncryption`, the splash plugin and `supportsTablet: false` (pending the decision). No build yet.
6. **M2.6** Android: rename the APK workflow, create a new keystore, and confirm the APK workflow still builds on SDK 58.

**M4 — Vertical slice:** playable in Expo Go on the iPhone, as PLAN.md already says.

**Before M6 — the first real iOS build ($99 starts here):**

7. **M5/M6.1** Enrol in the Apple Developer Program. Create the App Store Connect app record. Do the first interactive build:
   ```bash
   eas build -p ios --profile production
   ```
   This creates the credentials. Then create the ASC API key with `eas credentials`.
8. **M6.2** `eas submit -p ios --latest`, then install from TestFlight on the iPhone. Check the icon, name, splash, privacy-manifest email and silent mode.
9. **M6.3** Add `release-ios.yml` + `EXPO_TOKEN`. From then on, a GitHub release produces both a TestFlight build and an APK.
10. **Later** App Store submission:
    - privacy policy page;
    - "Data Not Collected" label;
    - age rating questionnaire (4+ expected);
    - in-app Credits screen;
    - screenshots.

Suggested edits to `PLAN.md` (not made here):
- M2: "SDK 58, iPhone via Expo Go".
- M6: "TestFlight build (iOS) + signed APK (Android)".
- §1, CI row: "+ EAS iOS workflow".
- §7: the open decisions below.

---

## 8. Open decisions

1. **SDK 58 for the skeleton** (recommended, because App Store Expo Go only runs 58) vs staying on 57 and using a dev build or `eas go`. Both of those need the $99 program now.
2. **When to buy the Apple Developer Program**: at M5/M6 (recommended) or earlier, if a dev build is wanted sooner.
3. **Home-screen name**: "FrenchoLinguo" (may be cut off on the home screen) vs a short display name such as "Frencho".
4. **iPad support**: `supportsTablet: false` (recommended: fewer screenshots, no iPad review) vs making layouts iPad-ready.
5. **Android**: keep as a secondary target with the existing APK workflow (recommended), or drop it.
6. **Accent bar layout**: two inline rows vs iOS `InputAccessoryView` above the keyboard.
7. **Distribution beyond the user's own phone**: TestFlight external beta for learners, or straight to the App Store after A1.
8. **Privacy policy hosting**: GitHub Pages, or another page.

---

## Sources (checked 2026-10-29)

- Expo Go on the App Store, version 58.0.5 (22 Oct 2026, SDK 58, iOS 16.4+): https://apps.apple.com/us/app/expo-go/id982107779
- Expo SDK 58 changelog (15 Oct 2026; RN 0.88; Expo Go only SDK 58; UIScene; safe-area notes): https://expo.dev/changelog/sdk-58
- How to upgrade to SDK 58 (21 Oct 2026; new Expo Go supports only SDK 58): https://expo.dev/blog/how-to-upgrade-to-sdk-58
- Expo Go and the App Store, May 2026 (SDK 54 lag, `eas go` via TestFlight): https://expo.dev/changelog/expo-go-and-app-store-may-2026
- Expo Go with SDK 57 needs login on both ends: https://dev.to/expo/running-an-expo-sdk-57-app-in-expo-go-you-now-need-to-be-logged-in-on-both-ends-32ef
- Expo Go setup (iOS, `eas go`, Apple Developer requirement): https://docs.expo.dev/get-started/expo-go/
- From Expo Go to a development build: https://docs.expo.dev/develop/development-builds/expo-go-to-dev-build/
- EAS pricing (Free: 15 Android + 15 iOS builds, low priority, 45-min timeout; Starter $19/mo with $45 credit): https://expo.dev/pricing
- EAS plans (free limits reset monthly): https://docs.expo.dev/billing/plans/
- EAS usage-based pricing (example iOS build ≈ $2 medium / $4 large): https://docs.expo.dev/billing/usage-based-pricing/
- Internal distribution (ad hoc, `eas device:create`, 100 devices/year, `--refresh-ad-hoc-provisioning-profile`): https://docs.expo.dev/build/internal-distribution/
- Building on CI (`EXPO_TOKEN`, `expo/expo-github-action@v8`, `--non-interactive --no-wait`): https://docs.expo.dev/build/building-on-ci/
- Automate submissions (`--auto-submit` → TestFlight): https://docs.expo.dev/build/automate-submissions/
- Apple privacy manifests in Expo: https://docs.expo.dev/guides/apple-privacy/
- expo-speech (silent mode on iOS, `getAvailableVoicesAsync`, `VoiceQuality`, `useApplicationAudioSession`): https://docs.expo.dev/versions/latest/sdk/speech/
- expo-audio (`setAudioModeAsync`, `playsInSilentMode`): https://docs.expo.dev/versions/latest/sdk/audio/
- Apple Developer Program ($99/year): https://developer.apple.com/programs/
- Apple membership comparison (free Apple account: Xcode-only, 7-day profiles, 3 devices): https://developer.apple.com/support/compare-memberships/
- Apple upcoming requirements (Xcode 26 / iOS 26 SDK for uploads since 28 Apr 2026): https://developer.apple.com/news/upcoming-requirements/
- Apple updated age ratings (13+/16+/18+, questionnaire deadline 31 Jan 2026): https://developer.apple.com/news/?id=ks775ehf
- App Store Review Guidelines (1.2, 4.2, 4.3, 5.1.1(v) account deletion): https://developer.apple.com/app-store/review/guidelines/
