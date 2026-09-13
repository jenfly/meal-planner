# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A small offline-first PWA ("Meal Planner" / "Meal Rotation") for tracking saved meal combos, assigning them to days of a weekly rotation, and auto-generating a shopping list from that week's meals. No build step, no framework, no package manager — it's three static files plus two icons.

## Running locally

There is no build/lint/test tooling in this repo. To run the app:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000`. Opening `index.html` via `file://` does not work — the service worker requires a real HTTP origin (`localhost` counts as secure, so no HTTPS needed locally).

To test the Google Drive backup feature locally, add `http://localhost:8000` as an extra Authorized JavaScript origin on the OAuth client referenced in `index.html` (see "Google Drive backup" below).

## Architecture

Everything — markup, CSS, and JS — lives in `index.html` as a single file. There is no bundler, no modules, no npm dependencies.

- **State**: four in-memory globals (`meals`, `week`, `shopExtras`/`shopChecked`/`shopOrder`) persisted to `localStorage` under the `mealRotation.` key prefix (`STORAGE_PREFIX`). Every mutation calls a matching `save*()` function immediately after updating state, then a `render*()` function to redraw. There is no diffing — each `render*()` fully replaces its container's `innerHTML` from current state.
- **No framework**: DOM updates are hand-rolled string templates assigned to `innerHTML`. Event handlers are wired two ways — inline `onclick="..."` attributes calling functions attached to `window` (needed because they're referenced from generated HTML strings), and `addEventListener` for static elements queried by id at load time.
- **Three tabs, one page**: `week` (weekly plan), `shop` (shopping list), `meals` (saved meal library) — panels are toggled via `.active` class, not routing.
- **Data model**: a meal (`{id, name, parts, items, lastMade, createdAt}`) is the source of truth. `week` maps day name (`Mon`..`Sun`) to an array of meal ids. The shopping list in `renderShop()` is *derived*, not stored directly — it aggregates `items` from every meal referenced anywhere in `week`, grouped case-insensitively by item text, plus manually-added `shopExtras`. `shopOrder` persists manual drag-to-reorder state and is reconciled against derived rows on every render (stale ids dropped, new ids appended — manual items to the top, meal-derived items to the bottom).
- **XSS**: all user-provided strings are inserted via `escapeHtml()` before going into template strings. When adding new fields that render user input, escape them the same way.
- **Icons**: SVGs are inlined as JS string constants (`ICON_*`) rather than `<img>` tags or a sprite sheet.
- **Google Drive backup**: optional, off by default until `GOOGLE_CLIENT_ID` in `index.html` is set (see README "Setting up Google Drive backup"). Uses Google Identity Services (loaded dynamically from a CDN) and the `drive.file` OAuth scope, which only grants access to the single backup file this app creates — never the whole Drive. Backup only covers `meals`, not the weekly plan or shopping list state.
- **Offline support**: `sw.js` is a cache-first service worker caching exactly the app shell files listed in `FILES_TO_CACHE`. Bump `CACHE_NAME` (e.g. `meal-planner-v3` → `v4`) whenever cached file contents change, so the old cache is evicted on activate.

## Making changes

- Since there's no bundler, edits to `index.html` take effect on a hard refresh; the service worker may serve a stale cached copy otherwise — bump `CACHE_NAME` in `sw.js` when shipping changes to cached files, or hard-reload/unregister the SW while developing.
- Deployment is just pushing the static files (`index.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`) to GitHub Pages or Netlify Drop — no CI/build step.

## Show screenshots for UI changes

Whenever a change affects the app's visual appearance or layout (new components, styling tweaks, layout changes, new screens/tabs, icons, etc.), render and share a screenshot of the affected view(s) before considering the task done — don't just describe the change in text. Use realistic sample data so the change is easy to evaluate at a glance. If a change touches multiple views or states (e.g. open/closed, empty/populated), show each relevant state.
