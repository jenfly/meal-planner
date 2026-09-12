# Meal Rotation

A small PWA for planning meal combos, a weekly rotation, and the shopping list that goes with it.

## Deploying

1. Push all the files in this folder (`index.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`) to a GitHub repo, or drag the folder onto https://app.netlify.com/drop.
2. If using GitHub, enable GitHub Pages in the repo's Settings (source: root of `main`).
3. Visit the resulting URL on your phone, then:
   - iPhone: Share button → "Add to Home Screen"
   - Android: Chrome menu → "Install app"

HTTPS (which both GitHub Pages and Netlify provide automatically) is required for install and offline caching to work.

## Running locally

Opening `index.html` directly (`file://`) won't work — the service worker needs a real HTTP origin. Serve the folder instead:

```
cd meal-rotation-pwa
python3 -m http.server 8000
```

(or `npx serve` if you have Node). Then visit `http://localhost:8000`. `localhost` counts as a secure origin, so no HTTPS is needed for local testing.

To test Google Drive backup locally, add `http://localhost:8000` as an extra Authorized JavaScript origin on the same OAuth client (see below) — production and local can share one client ID.

## Setting up Google Drive backup

The "Back up your meals" feature (in the header menu, top right) saves just your meal list — not your weekly plan — to a file in your Google Drive. It needs a one-time setup before it will work:

1. Go to [Google Cloud Console](https://console.cloud.google.com).
2. Create a project (or use an existing one).
3. Enable the **Google Drive API** for that project (APIs & Services → Library → search "Google Drive API" → Enable).
4. Go to APIs & Services → Credentials → **Create Credentials → OAuth client ID**.
   - Application type: **Web application**
   - Under **Authorized JavaScript origins**, add the exact URL your app is deployed at (e.g. `https://yourusername.github.io`) — no trailing slash, no path.
5. Copy the generated **Client ID** (looks like `123456789-abc123.apps.googleusercontent.com`).
6. Open `index.html` in this folder, find this line near the top of the `<script>` block:
   ```js
   const GOOGLE_CLIENT_ID = 'YOUR_CLIENT_ID.apps.googleusercontent.com';
   ```
   and replace it with your own client ID.
7. Redeploy (push the updated `index.html`).

Notes:
- The app requests the `drive.file` scope, which only ever grants access to the one backup file it creates — never your whole Drive.
- If you skip this setup, tapping "Back up now" will just tell you it isn't configured yet, rather than failing silently.
- The backup menu nudges you if it's been 30+ days since your last backup, or if you've never backed up.
