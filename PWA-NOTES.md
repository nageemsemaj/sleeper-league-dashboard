# The Ledger — PWA + sharing deploy notes

## Files to commit (all at repo root, same folder as index.html)
- `index.html` (updated)
- `manifest.json`
- `sw.js`
- `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon.png`

Paths are relative (`./`), so this works on GitHub Pages under a subpath
(`nageemsemaj.github.io/sleeper-dashboard/`) without changes.

**HTTPS is required** for service workers. GitHub Pages is HTTPS, so you're fine.

## What's new

### 1. Installable app (PWA)
- **Android/Chrome:** browser shows an "Install app" prompt automatically.
- **iPhone:** Safari only. Share button -> "Add to Home Screen".
  iOS does NOT auto-prompt, which is why the app shows a one-time hint on the
  entry screen telling iPhone users exactly what to tap. Dismissible; the
  dismissal is remembered.
- Opens fullscreen (no browser chrome), dark theme, own icon.

### 2. Deep links (the big one for adoption)
- `?league=<LEAGUE_ID>` — opens straight into that league, no typing.
- `?user=<USERNAME>` — opens that user's league picker.

Example to paste in your league chat:
```
https://nageemsemaj.github.io/sleeper-dashboard/?league=1326671212276883456
```
Friends tap it and immediately see their own league. No instructions needed.

### 3. Share button
In the header once a league is loaded. Uses the native share sheet on mobile
(so it goes straight to iMessage / group chat), falls back to copying the link
on desktop with a "Link copied" toast.

### 4. Saved leagues
Every successfully loaded league is remembered (id, name, type) and listed on
the entry screen with its type badge. Tap to reload instantly. Individually
removable, plus "Clear all". Capped at 8, most recent first.
This is what makes the installed app feel like an app: open it, tap your league.

## Service worker caching strategy (important)
- Caches the app SHELL only: html, manifest, icons, Chart.js.
- **Never caches Sleeper or FantasyCalc API responses** — league data stays live.
  Standings, rosters, and Chopped eliminations are never served stale.
- Navigations are network-first, so a new deploy is picked up immediately;
  the cached shell is only used when offline.

### Deploying an update
Bump `CACHE_VERSION` in `sw.js` (e.g. `ledger-shell-v1` -> `v2`) whenever you
change the shell. Old caches are deleted on activate.

## Verify after deploy
1. Load over HTTPS, open DevTools -> Application -> Service Workers (should be active).
2. Application -> Manifest (icons + name resolve, no errors).
3. Try `?league=<id>` in a fresh tab.
4. Load a league, hit Share, confirm the copied link works.
5. Reload the entry screen: the league should appear under "Saved".
6. On an iPhone, confirm the Add to Home Screen hint appears and the installed
   icon opens fullscreen.
