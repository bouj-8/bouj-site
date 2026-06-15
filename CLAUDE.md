# CLAUDE.md — bouj-site

## Overview

Two-page frontend for the bouj moon calendar. The public page (`index.html`) shows the lunar date. A password-gated inner page (`inner.html`) is a traditional blog.

Deployed on GitHub Pages at `https://bouj-8.github.io/bouj-site/`. Every push to `main` triggers an automatic redeploy (legacy build type, no Actions workflow needed).

The backend API is at `https://bouj-backend-production.up.railway.app`.

---

## What it does

### `index.html` (public)
1. On load, asks the browser for the user's lat/lon
2. Calls `GET /moon?lat=...&lon=...` on the backend
3. Renders the lunar date — either a normal reading or a gate day (two readings)
4. Language and theme preferences are saved in `localStorage`

### `inner.html` (password-gated blog)
1. On load, reads `bouj_token` from `localStorage` and calls `GET /me` to verify it
2. If the token is missing or expired, redirects to `index.html`
3. If valid, reveals the blog and shows the logged-in username
4. Has a logout button that clears `bouj_token` and `bouj_username` from `localStorage`

---

## Key display logic

### Normal day
```
fourth moon day twenty-nine        (English)
四月廿九                            (Chinese)
```

### Gate day
```
sixth moon day thirty · seventh moon day one        (English)
六月三十 · 七月初一                                  (Chinese)
gate at 12:11pm
交朔 · 12:11pm
```
Both readings render at equal weight (no muting). The divider `·` is muted.

### Leap month
```
leap seventh moon day eleven       (English)
闰七月十一                          (Chinese)
```

### Chinese day notation
- Days 1–10: 初一 … 初十
- Days 11–20: 十一 … 二十
- Days 21–29: 廿一 … 廿九
- Day 30: 三十
- Leap prefix: 闰
- Month 1: 正月 (not 一月)

---

## Toggles

Both are fixed to the top-right corner of `index.html`, stacked vertically:
- **Theme toggle**: light ↔ dark, saved to `localStorage('theme')`
- **Language toggle**: 中文 ↔ eng, saved to `localStorage('lang')`

Toggling language re-renders immediately from the cached API response (`lastData`) — no second fetch needed.

---

## Password / auth flow

The login prompt is a terminal-style bar at the bottom of the screen. It is hidden until triggered.

**Triggers:**
- Desktop: `Shift+/` (the `?` key)
- Mobile: long-press (600ms) on the moon date element

**Flow:**
1. User types password and hits Enter
2. Frontend posts `{ password }` to `POST /auth` on the backend
3. On success: stores `bouj_token` and `bouj_username` in `localStorage`, redirects to `inner.html`
4. On failure: prompt shakes and clears, user can try again
5. Escape closes the prompt without submitting

**Tokens expire after 24 hours.** On expiry, `inner.html` bounces the user back to `index.html`.

**localStorage keys:**
- `bouj_token` — JWT from the backend
- `bouj_username` — display name of the logged-in user
- `theme` — `'light'` or `'dark'`
- `lang` — `'zh'` or `'en'`

**iOS keyboard note:** the prompt uses `window.visualViewport` to lift itself above the software keyboard when it appears.

---

## Planned features

- **Gate day display toggle** (post-site, post-user-storage): let users choose between showing both readings or just one (opening / closing / auto based on time of day). Requires user preference storage to be in place first.

---

## What NOT to do

- Do not hardcode a location — always use the browser Geolocation API
- Do not fetch again on language toggle — re-render from `lastData`
- CORS is handled by the backend (`allow_origins=["*"]`) — no proxy needed
- Do not validate the JWT client-side — always call `GET /me` to verify; the backend checks the signature and expiry
