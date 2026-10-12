# Meal Rotation

A small PWA for planning meal combos, a weekly rotation, and the shopping list that goes with it.

## Deploying

1. Push all the files in this folder (`index.html`, `sync.js`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`) to a GitHub repo, or drag the folder onto https://app.netlify.com/drop.
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

## Linking to Well Tended

Checking off an item on the Batch cook tab still works exactly as before, and now also opens Well Tended at `?log=<meal name>` so you can log "Make <meal>" there (or cancel if it's the same batch re-listed). Unchecking does nothing extra.

The address lives in the `WELL_TENDED_URL` constant next to `toggleBatchCooked` in `index.html`; set it to `''` to turn the link off. Which Well Tended category receives the item is chosen in Well Tended's own Settings.

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

## Cloud sync

Optional, and separate from the Google Drive backup above. Tap the ⋯ menu (top right) → "Sign in with Google" under "Sync across devices" to keep your meals, weekly plan, and shopping list synced automatically between your phone and computer through a shared Supabase backend (the same one used by Well Tended and the journal app — one sign-in covers all three, since they share this origin).

- Everything still lives in `localStorage` first and the app works offline exactly as before; sync just pushes changes up and pulls changes made elsewhere in the background.
- The first time a device signs in and finds a different cloud copy already there, you'll be asked whether to keep this device's data or the cloud's — the one you don't pick is saved once on this device as a backup.
- Signing out affects all three apps, since they share one sign-in.
- The existing Google Drive backup above still works independently and is worth keeping as a safety net.

## Adjusting or disabling the backup reminder

If it's been too long since your last Google Drive backup (or you've never backed up), the app shows a banner right at the top of the screen as soon as it opens, plus a matching "stale" indicator on the backup pill in the header menu. Tapping "Back up now" on the banner backs up immediately; the `×` dismisses it until the next time you open the app.

Both are controlled by two constants near the top of the `<script>` block in `index.html`, right above the Google Drive backup code:

```js
const BACKUP_REMINDER_DAYS = 60;
const BACKUP_REMINDER_ALERT_ENABLED = true;
```

- `BACKUP_REMINDER_DAYS` — how many days may pass since the last backup before the reminder kicks in. Change `60` to whatever number you'd prefer.
- `BACKUP_REMINDER_ALERT_ENABLED` — set to `false` to turn off the top-of-screen banner entirely. (The header menu's pill still quietly turns amber past `BACKUP_REMINDER_DAYS`, but nothing pops up on open.)

Redeploy (push the updated `index.html`) after changing either value.
