# CLAUDE.md — bouj-site

## Overview

Single-page frontend for the bouj moon calendar. Fetches lunar date from `bouj-backend` via the browser's Geolocation API and renders it in natural language.

The backend API is at `https://bouj-backend-production.up.railway.app`.

---

## What it does

1. On load, asks the browser for the user's lat/lon
2. Calls `GET /moon?lat=...&lon=...` on the backend
3. Renders the lunar date — either a normal reading or a gate day (two readings)
4. Language and theme preferences are saved in `localStorage`

---

## Key display logic

### Normal day
```
fourth moon day twenty-nine        (English)
四月廿九                            (Chinese)
```

### Gate day
```
sixth moon day thirty / seventh moon day one        (English)
六月三十 / 七月初一                                  (Chinese)
gate day · new moon at 12:11pm
交朔 · 12:11pm
```

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

Both are fixed to the top-right corner, stacked vertically:
- **Theme toggle**: light ↔ dark, saved to `localStorage('theme')`
- **Language toggle**: 中文 ↔ eng, saved to `localStorage('lang')`

Toggling language re-renders immediately from the cached API response (`lastData`) — no second fetch needed.

---

## What NOT to do

- Do not hardcode a location — always use the browser Geolocation API
- Do not fetch again on language toggle — re-render from `lastData`
- CORS is handled by the backend (`allow_origins=["*"]`) — no proxy needed
