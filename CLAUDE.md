# Batroun Race — Registration System

Project context for Claude Code. This file is read on every session. Keep it short and current.

## What this is

A registration system for the **Batroun Race** annual running competition in Lebanon. Public registration page + admin dashboard. Built to be reusable each year — never hardcode "2026", always use the `competitionId` (e.g. `batroun-race-2026`).

## Tech stack (locked in — don't change without asking)

- **Frontend**: vanilla HTML + CSS + JavaScript (no framework, no build step)
- **Database**: Firebase Firestore
- **Auth (admin only)**: Microsoft Azure / Office 365 via Firebase Auth, restricted to school domain
- **Hosting**: GitHub Pages (deploy from `main` branch, `/public` folder)
- **Payment**: Whish Money — static QR + manual admin reconciliation (no API integration yet)

Public registration requires **no auth** — riders shouldn't need accounts.

## File structure

```
batroun-race/
├── CLAUDE.md              (this file)
├── WORKFLOW.md            (phase-by-phase build plan)
├── README.md              (setup instructions for humans)
├── firestore.rules        (Firestore security rules)
├── firebase.json          (Firebase deploy config)
├── .firebaserc            (Firebase project link)
└── public/
    ├── index.html         (public registration page — DONE)
    ├── admin.html         (admin dashboard — TODO)
    └── assets/
        └── whish-qr.png   (school's Whish QR code image)
```

## Firestore data model

```
competitions/{competitionId}                    // e.g. "batroun-race-2026"
  ├─ name, year, registrationOpens, registrationCloses, raceDay
  ├─ waves: [{ id, label, endsAt }]
  └─ whish: { number, qrImageUrl }

competitions/{competitionId}/categories/{categoryId}
  ├─ name, distanceKm, timed, ageLimit
  ├─ prices: { wave1, wave2, wave3 }
  ├─ capacity, registeredCount
  └─ includes: [string]

competitions/{competitionId}/registrations/{regId}
  ├─ firstName, lastName, email, phone, dob, gender, nationality, city
  ├─ categoryId, categoryName, tshirtSize, club, medicalNotes
  ├─ emergencyContactName, emergencyContactPhone
  ├─ consents: { waiver, dataUsage, newsletter }
  ├─ waveAtRegistration, priceUSD
  ├─ whishReference          // "BTN-2026-A4K9" — unique, user-visible
  ├─ paymentStatus           // "pending" | "paid" | "refunded" | "free"
  ├─ paymentMethod           // "whish" | "free"
  ├─ bibNumber               // assigned by admin on payment confirmation
  ├─ registeredAt, paidAt
  └─ ageOnRaceDay            // denormalized for filtering
```

## Critical conventions

### Currency
USD throughout. The Race.docx prices are in USD. Display as `$X USD` for clarity.

### Phone numbers
Lebanese format: `+961XXXXXXXX` or local `XXXXXXXX`. Store as user-entered, normalize on display.

### WhatsApp links
Format: `https://wa.me/9613XXXXXXX?text=<urlencoded>`. The number `96171503555` is Café Molière's pattern — Batroun Race will have its own.

### Whish reference codes
- Format: `BTN-2026-XXXX` (4 chars, no confusing 0/O/1/I/L)
- Generated client-side at submit time
- Stored on registration doc
- Shown to user on confirmation — they include it in Whish payment memo
- Admin uses it to match incoming Whish payments

### Capacity gates (Phase 3 — not yet built)
**Must use Firestore transactions.** Two people submitting at the same time for the last spot is a real race condition. Use `runTransaction` to atomically read `registeredCount`, check against `capacity`, write the registration, increment count. Reject if full.

### Race day date
Currently hardcoded as `2026-11-22` in `index.html` (`RACE_DAY` constant). Move this to the `competitions/{id}` document when admin dashboard is built so it's editable without redeploying.

## Gotchas learned the hard way

- **Never edit JS via Python scripts that use string manipulation** — splits regex literals on `/`, doubles `const`, breaks template literals. If automating JS edits, run `node --check file.js` after every change.
- **Avoid template literals in any JS string that gets generated via Python f-strings or regex replacement.**
- **Firebase Auth `signInWithRedirect` is preferred over `signInWithPopup`** on mobile (popups get blocked). Café Molière learned this.
- **Disable App Check** during development; it blocks Firestore writes from localhost in confusing ways.
- **GitHub Pages cache is aggressive.** Hard reload (`Cmd+Shift+R`) after pushing or you'll think your fix didn't work.

## Race categories (from Race.docx — authoritative)

| ID            | Name                     | Timed | Age limit | Wave 1 | Wave 2 | Wave 3 |
|---------------|--------------------------|-------|-----------|--------|--------|--------|
| marathon      | Marathon 42.195 KM       | Yes   | 18+       | $40    | $50    | $65    |
| halfMarathon  | Half Marathon 21.1 KM    | Yes   | 17+       | $30    | $40    | $50    |
| 10k           | 10 KM Race               | Yes   | 12+       | $25    | $30    | $35    |
| 5k            | 5 KM Race                | No    | none      | $22    | $22    | $22    |
| para          | Para Athlete Race        | Yes   | 15+       | Free   | Free   | Free   |

### Waves
- Wave 1: now → **1 Sept 2026** (Early bird)
- Wave 2: 2 Sept → **10 Nov 2026** (Standard)
- Wave 3: 11 Nov → **20 Nov 2026** (Late)

Registration closes 20 Nov 2026 for everyone. Race day assumed 22 Nov 2026 (confirm with organizers).

### Includes (shown on all categories)
Race Registration · BIB & Tag · Finisher T-shirt · Finisher Medal · Goody Bag · Finish Time Certificate · LAF Fees

## How to test locally

```bash
# Quick static server (no Firebase needed for layout testing)
cd public && python3 -m http.server 8000

# For full Firestore testing:
firebase emulators:start --only firestore,auth
# then point firebaseConfig at the emulator
```

## How to deploy

GitHub Pages serves from `/public` on `main`. Just push.

```bash
git add . && git commit -m "..." && git push
```

GitHub Pages updates within 1–2 minutes. Hard reload to bypass cache.

## Status — what's done

- [x] Phase 1: Public registration page (`public/index.html`)
  - Wave detection, live price display, age validation, Whish reference generation, Firestore write, confirmation panel

## Status — what's next

See `WORKFLOW.md`.
