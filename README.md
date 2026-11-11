# Lock In: build and install on your iPhone

You need a **Mac with Xcode** to produce an iOS app. There is no way around this: Apple only lets Xcode sign iOS builds. If you don't have a Mac, see "No Mac?" at the bottom.

## 0. Try it first (no build needed)

Open `www/index.html` in any browser. Everything runs locally and saves to localStorage.

## 1. One-time setup on the Mac

1. Install Xcode from the App Store, open it once, accept the license.
2. Install Node.js (nodejs.org, LTS).
3. Make a free Apple ID developer account: in Xcode, Settings → Accounts → add your Apple ID. A paid Developer Program membership ($99/yr) is NOT required to run on your own phone.

## 2. Create the iOS project

Unzip this folder, open Terminal in it, then:

```bash
npm install
npx cap add ios
npx cap sync ios
npx cap open ios
```

Xcode opens the project (`ios/App/App.xcworkspace`).

## 3. Sign it

In Xcode: click **App** (top of the left sidebar) → **Signing & Capabilities** → check "Automatically manage signing" → pick your Apple ID under **Team**. If the bundle identifier is taken, change `com.malakai.lockin` to something unique, and keep `capacitor.config.json` in sync.

## 4a. Run on your iPhone (easiest)

1. Plug in the iPhone, tap Trust. On the phone: Settings → Privacy & Security → **Developer Mode** → on (it will restart).
2. In Xcode, pick your iPhone in the device dropdown at the top, press **Run** (▶).
3. First launch: Settings → General → VPN & Device Management → trust your Apple ID.

Free-account builds expire after **7 days**. Rerun from Xcode to renew. A paid account lasts a year and allows TestFlight.

## 4b. Export an actual .ipa

1. Device dropdown → **Any iOS Device (arm64)**.
2. Menu: **Product → Archive**.
3. In the Organizer window: **Distribute App → Debug Testing** (free account) or **Ad Hoc / App Store Connect** (paid) → follow prompts → **Export**.
4. You get `App.ipa`. Install it with Apple Configurator, or sideload with AltStore / Sideloadly (these re-sign with your Apple ID, so the 7-day limit still applies on a free account).

## 5. Updating the app after changes

Edit `www/index.html`, then `npx cap sync ios` and press Run again.

## Add the real app icon (optional)

Put a 1024×1024 PNG in `ios/App/App/Assets.xcassets/AppIcon.appiconset/` (or drag it onto AppIcon in Xcode).

## Calendar sync: how to get your links

- **Canvas:** Calendar → "Calendar Feed" (right sidebar) → copy the URL. Paste in Settings.
- **Google Calendar:** calendar.google.com → Settings → your calendar → Integrate calendar → **Secret address in iCal format**. Paste in Settings.
- Treat both links like passwords. Never post them or paste them into public tools.

Limits to know about:
- In a normal browser, these fetches are often blocked by CORS. Inside the iPhone app, `CapacitorHttp` (enabled in `capacitor.config.json`) makes native requests, so they work. Fallback: download the `.ics` and use "Import .ics".
- Recurring Google events (weekly classes, shifts) only appear as their first occurrence. The feed stores a repeat rule, which this app doesn't expand yet.
- Sync is manual (tap Sync). Background sync needs native code.

## No Mac?

- **Install as a home-screen app instead:** host the `www` folder on any HTTPS static host (GitHub Pages, Netlify, Cloudflare Pages), open it in Safari on the iPhone → Share → **Add to Home Screen**. Looks and runs like an app, no IPA, no 7-day expiry.
- **Cloud Mac:** rent a Mac in the cloud (MacinCloud etc.) for an hour and follow the steps above, or use a CI service with macOS runners.

## Known design trade-offs

- Quests marked "AUTO" are verified by the app (focus timer, completed tasks). The rest are honor-system. You can't cheat anyone but yourself, but note the leaderboard will still reward it.
- The leaderboard is simulated: rivals grow deterministically every day, so it behaves the same offline. Real multiplayer needs a backend.
- Strict focus mode treats "screen locked" the same as "left the app", which is why it's off by default.

## No Mac: build the IPA in the cloud, sideload from Windows or Mac

1. Create a GitHub repo and push this folder to it (the `.github/workflows/build-ipa.yml` file is included).
2. Repo → **Actions** → "Build unsigned IPA" → **Run workflow**. Takes about 5-10 minutes.
3. Download the `LockIn-ipa` artifact and unzip it to get `LockIn.ipa`.
4. Install **Sideloadly** (sideloadly.io) or AltStore, plug in the iPhone, drag in the IPA, enter your Apple ID. It signs the app and installs it.
5. On the phone: Settings → Privacy & Security → Developer Mode → on, then trust your Apple ID under VPN & Device Management.

Free Apple ID = the app expires every 7 days; re-run Sideloadly (or let AltStore refresh it) to renew.
I couldn't run this workflow in my sandbox, so if the build step fails, read the error in the Actions log. The usual fix is a scheme or workspace name mismatch.
