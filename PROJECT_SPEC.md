# Journal App — Project Spec

A personal mood journaling PWA built as a single HTML file, hosted on GitHub Pages, installable on iPhone via Safari. No backend, no accounts, no cost.

---

## Goals & Objectives

The core philosophy of this project is **privacy-first, zero-dependency personal tooling**. Everything runs in the browser. Nothing touches a server. The app should feel as native as possible on iPhone while remaining trivially simple to host and maintain.

- **Full offline capability** — the app must work without any network connection after first load
- **True end-to-end encryption** — entries are encrypted with AES-GCM in the browser before being written to IndexedDB; the key never leaves memory
- **No accounts, no subscriptions** — free forever, hosted on GitHub Pages or Netlify
- **Installable as a PWA** — Add to Home Screen on iOS Safari, full-screen, no browser chrome
- **Long-term personal archive** — the data model and export format should be stable enough to trust for years of daily entries

---

## What's Built So Far

- Password lock screen with PBKDF2 key derivation (310,000 iterations, SHA-256)
- AES-GCM 256-bit encryption per entry, unique IV per write
- IndexedDB storage (no 5MB cap, handles photos)
- Monthly calendar view with mood color dots
- Per-day entry: mood (6-step green→red scale), weather (multi-select chips), notes, single photo
- Photo auto-resize to 1200px before storage
- Export to JSON (decrypted, portable)
- Import from JSON (re-encrypted on ingest)
- Change password with full re-encryption of all entries
- Toast notifications

---

## Roadmap

### Features

**Year heatmap view**
A full 365-day grid colored by mood — like GitHub's contribution graph but for your emotional state. Probably the highest visual payoff of anything on this list. Build as a second tab or swipe view alongside the monthly calendar.

**Mood trend chart**
A simple line or bar chart showing average mood per week/month over time. Pair with weather to start surfacing correlations ("you tend to log lower moods on rainy days"). Use the Canvas API or a lightweight lib like Chart.js — no need for anything heavy.

**Tags & activities**
A set of tappable chips per entry (Exercise, Social, Work, Travel, Creative, Rest, etc.) that let you annotate what kind of day it was — separate from mood. Over time this becomes the raw material for pattern analysis: "your mood is highest on days you logged Exercise."

**Streak tracker**
Consecutive days logged, personal best streak, days logged this month. Small but motivating. Show in the header or a dedicated stats card.

**"One word for today" field**
A single short text input at the top of each entry — forces a moment of reflection, and looks great rendered in the year heatmap as a tooltip or overlay.

**PWA Service Worker**
Register a Service Worker to cache the app shell so it loads instantly and works fully offline even on first open after install. Required for a proper PWA experience. The current version requires a network hit to load fonts on first launch.

**Siri Shortcut**
A Shortcuts automation that opens the journal URL directly to today's entry. Can be triggered by voice, time of day, or location. No code needed — pure iOS Shortcuts config, but worth documenting here as a setup step.

---

### Security & Encryption

**Argon2id instead of PBKDF2** *(stretch goal)*
Argon2id is the modern recommendation for password hashing — it's memory-hard, making GPU-based brute-forcing much more expensive. PBKDF2 with 310k iterations is reasonable, but if you want to go deeper this is the upgrade. Requires a WASM build (e.g. `argon2-browser`).

**Lock on background**
Detect when the app moves to the background (Page Visibility API) and re-show the lock screen, clearing the in-memory key. Currently the key persists for the lifetime of the browser session.

**Encrypted export option**
Offer a second export format that keeps entries encrypted with the current key — useful for encrypted cloud backups where you don't want plaintext JSON sitting in iCloud Drive.

---

### Sync

**iCloud sync via CloudKit JS**
Apple exposes a public CloudKit JS API that lets you read/write to a private iCloud container from a web app — no Apple Developer account required for private use. This would let entries sync between your iPhone and Mac transparently. Genuinely hard to implement, but the most elegant possible sync story for an Apple-only personal app.

**Manual iCloud Drive sync** *(simpler alternative)*
Export the JSON backup directly to iCloud Drive, and add an "Open latest backup" button that reads from a fixed iCloud Drive path. Not automatic, but requires zero infrastructure.

---

## Maintenance & Tech Debt

- **Split JS into separate files** — the current single-file approach is fine for now but will get unwieldy. Split into at least: `crypto.js`, `db.js`, `calendar.js`, `entry.js`, `app.js`. Use ES modules (`type="module"`) and a build step (Vite is ideal — zero config, produces a single optimised bundle for deployment)
- **Move to a proper build pipeline** — Vite + vanilla JS or Vite + Svelte would let you use real imports, hot reload during dev, and minified output for production. The single HTML file can stay as the deployment artefact
- **Font loading** — self-host DM Sans and DM Serif Display rather than loading from Google Fonts. Removes the network dependency for first load and is required for true offline support
- **Keyboard navigation** — the app is entirely tap-driven right now. Add keyboard support for the calendar (arrow keys to move between days, Enter to select) for when you use it on a Mac via browser
- **Error boundaries** — if IndexedDB fails (storage quota, private browsing mode, corruption) the app currently fails silently. Add graceful error states with user-facing messages
- **Entry validation** — currently you can save a completely empty entry. Decide whether that's intentional (some days you just want to mark as visited) or add a soft warning
- **Export filename collision** — if you export twice in one day you get the same filename. Append a timestamp to the filename
- **Lint + format** — add ESLint and Prettier with a pre-commit hook if you move to a proper repo structure. Keeps code consistent as the project grows
- **README.md** — write a setup guide for your future self: how to run locally, how to deploy to GitHub Pages, how to update the hosted version

---

## Project Principles

1. No data ever leaves the device without explicit user action
2. No external dependencies at runtime (fonts excluded for now)
3. The export format must be human-readable and self-describing
4. Forgetting your password means losing your data — document this prominently
5. The app should work on a 5-year-old iPhone with a slow connection