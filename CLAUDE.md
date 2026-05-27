# Batroun Race — Registration System

Project context for Claude Code. This file is auto-read on every session. **Keep it accurate.**

## What this is

A complete registration system for the **Batroun Race** annual running event in Lebanon (Lycée Montaigne, Beit Chabab). Public registration page + admin dashboard + race-day QR scanner. Built reusable each year — never hardcode "2026" anywhere; always use `competitionId` (e.g. `batroun-race-2026`). One deliberate exception: the Whish reference code format (see Critical conventions).

## Tech stack (locked — don't change without asking)

- **Frontend**: vanilla HTML + CSS + JavaScript. **No build step, no framework.** Inter font from Google Fonts at runtime.
- **Database**: Firebase Firestore (free Spark plan)
- **Auth**: Google sign-in via Firebase Auth, restricted to a dynamic allowlist (super admins hardcoded, additional admins/staff stored in Firestore `/config/access`)
- **Hosting**: GitHub Pages serves `/docs` from `main` branch (Pages only supports `/` or `/docs`; that's why the folder is named that)
- **Payment**: Whish Money, manual reconciliation. Runner pastes their reference code into the Whish note; admin matches it and clicks Confirm.
- **WhatsApp**: pre-composed `wa.me` links open in WhatsApp Web after payment confirmation — admin clicks Send. Semi-automatic, no API costs.
- **QR**: generated client-side via `qrcodejs` (cdnjs CDN). No external QR API at runtime.
- **No Firebase Storage** — Storage requires Blaze (paid); we use base64 in Firestore for the logo + background image instead.

## File structure

```
batroun-race/
├── CLAUDE.md              (this file)
├── README.md              (human-facing — points at user guide)
├── WORKFLOW.md            (historical build plan — kept for reference)
├── firestore.rules        (Firestore security rules — re-publish manually in Firebase console after edits)
├── storage.rules          (Firebase Storage rules — deny-by-default; we don't use Storage, see Tech stack)
├── firebase.json          (Firebase CLI config)
├── .firebaserc            (Firebase project link → "batroun-race")
├── .gitignore             (ignores build artifacts and local zips)
├── .devcontainer/         (GitHub Codespaces config — auto-installs Claude Code + Firebase CLI)
└── docs/                  (served by GitHub Pages)
    ├── index.html         (public registration page)
    ├── admin.html         (admin dashboard — Chrome only)
    ├── scan.html          (race-day QR scanner)
    ├── ticket.html        (public ticket page with QR — linked from WhatsApp)
    └── USER_GUIDE.md      (comprehensive non-technical manual for all roles)
```

## Live URLs

- Public registration: https://relkady84.github.io/Batroun-Race/
- Admin: https://relkady84.github.io/Batroun-Race/admin.html
- Race-day scanner: https://relkady84.github.io/Batroun-Race/scan.html
- Ticket page (per runner): https://relkady84.github.io/Batroun-Race/ticket.html?ref=...&name=...&bib=...

## Roles & access

| Role | Where defined | Sees |
|---|---|---|
| **Super admin** | Hardcoded in code + rules — `raedelkady@gmail.com`, `nizarelkady@gmail.com` — cannot be removed | Everything |
| **Admin** | `/config/access.admins` array in Firestore (managed via admin Settings tab) | Everything including Settings |
| **Staff / viewer** | `/config/access.viewers` array | Registrations · Confirmed · scanner only |
| **Public** | No auth | Just the registration page |

`firestore.rules` `isAdmin()` checks super-admin list OR `/config/access.admins`. `isViewer()` checks `/config/access.viewers`. Rules read the access doc via `get()`.

## Firestore data model

```
competitions/{competitionId}                    // "batroun-race-2026"
  ├─ name, year, registrationOpens, registrationCloses, raceDay, raceStartTime, raceLocation

competitions/{competitionId}/categories/{categoryId}
  ├─ name, distanceKm, timed
  ├─ ageLimit (min), ageLimitMax (optional)
  ├─ price (single value — no waves)
  ├─ capacity (nullable), registeredCount
  ├─ bibRangeStart, bibRangeEnd                 // when set → confirm-payment auto-assigns the next unused bib in [start, end], FIFO
  ├─ isRelay: bool                              // when true → public form collects a teammate
  └─ subQuestion: { label, options[] } | null   // e.g. age-bracket dropdown

competitions/{competitionId}/config/{configId}   // public read except /access
  ├─ theme:      { accent, accent2, ink, bg1, bg2, bgImage, logoImage, logoText, logoHeight, whishLogo, heading, subtitle, bannerText }
  ├─ form:       { fields: [{ key, label, type, required, options }] }   // extra public-form fields
  ├─ publicForm: { builtIn: { <key>: { visible, required, label, deleted } }, includes: [string] }   // toggles + label overrides for built-in fields (email, gender, nationality, country, city, tshirt, blood, club, note, emergency) and the "Registration includes" list shown on the price card
  ├─ scanner:    { fields: [...] }    // which fields the hostess scanner shows
  └─ access:     { admins: [email], viewers: [email] }   // authenticated read only

competitions/{competitionId}/registrations/{regId}    // doc ID = normalizePhone(phone)
  ├─ firstName, lastName, email, phone, dob, gender, nationality, country, city
  ├─ categoryId, categoryName, subQuestionAnswer
  ├─ tshirtSize, bloodType, club, medicalNotes
  ├─ emergencyContactName, emergencyContactPhone
  ├─ consents: { waiver, dataUsage, newsletter }
  ├─ priceUSD, whishReference   // "BTN-2026-A4K9"
  ├─ paymentStatus              // "pending" | "paid" | "refunded" | "free"
  ├─ paymentMethod              // "manual" | "free"
  ├─ bibNumber, paidAt
  ├─ registeredAt, ageOnRaceDay
  ├─ custom: { ...customFieldValues }
  ├─ deletedAt, deletedBy       // soft delete; admin Backup tab restores
  └─ checkedInAt, checkedInBy   // set by hostess scanner
```

## Critical conventions

### Registration doc ID is the Whish reference
Each new registration's Firestore document ID **is** its `whishReference` (e.g. `BTN-2026-A4K9`). This unlocks two things:
- A single mobile can register **multiple participants** — each gets its own doc (the old phone-as-ID model is gone).
- The public **sign-in flow** (`index.html` → "Already registered? Sign in") looks up a registration via `getDoc(refs/<ref>)`. Firestore rules allow public `get` on `/registrations/{regId}` for this exact pattern; `list` is still locked to admin/viewer so we don't leak the whole table. The client verifies that the stored `phone` (normalised) matches what the user typed before exposing the doc's contents.

### Whish reference codes
- Format: `BTN-2026-XXXX` (4 chars, no confusing 0/O/1/I/L)
- **Deliberate exception to the "no hardcoded year" rule** — keeping the year in the ref code lets admins instantly tell a current-year payment apart from a stale one from a prior edition. When rolling over to the next race, bump the literal in `docs/index.html` (search `BTN-2026-`) alongside the `competitionId`.
- Generated client-side at submit time
- Shown prominently on the confirmation panel
- Runner must include in Whish note/comment — admin matches manually

### Sub-question (age brackets per category)
Each category can have a `subQuestion: { label, options }` field. On the public form, when the runner picks the category AND enters their DOB, the matching bracket is auto-filled as a **readonly** field (parser handles `(YYYY-YYYY)`, `X-Y years`, `X+`, etc.). The runner cannot override — enforces correct bracket-to-DOB match.

### Capacity gates
Categories may have a `capacity` and a `registeredCount`. **Currently not enforced.** If/when we need it: use `runTransaction` to atomically read count, check capacity, write registration, increment count.

### Theme/logo upload
Admin uploads images via the Theme tab. Images are compressed client-side (canvas → JPEG ≤ 800 KB for backgrounds, ≤ 250 KB for logos) and stored as `data:image/jpeg;base64,...` strings on `/config/theme`. **No Firebase Storage** (Blaze required).

### Soft delete + 10s undo
When viewer/admin clicks Delete on a registration:
1. Warning dialog
2. `updateDoc` sets `deletedAt + deletedBy` (NOT `deleteDoc`)
3. Yellow toast at bottom with **Undo** button, 10s countdown
4. Clicking Undo within 10s clears `deletedAt`/`deletedBy`
5. After 10s, registration stays soft-deleted; only visible in admin **Backup** tab

Admin can **Restore all** / **Permanent delete all** from Backup. Permanent delete frees the phone-based doc ID for re-registration.

### Confirmed → Revert
Confirmed tab has a **↩ Revert** button per row that clears `paymentStatus`/`bibNumber`/`paidAt`/`checkedInAt`/`checkedInBy`, moving the runner back to Registrations as pending.

## Gotchas learned the hard way

- **NEVER edit JS via Python scripts that use string manipulation** — splits regex literals, doubles `const`, breaks template literals. Use the Edit tool.
- **Always `node --check` the inline admin.html module after edits** — a duplicate `const` once silently broke the entire admin (sign-in button looked dead because the whole script failed to parse). Extract with `awk '/<script type="module">/,/<\/script>/' docs/admin.html | sed 's/<script[^>]*>//;s/<\/script>//' > /tmp/x.mjs && node --check /tmp/x.mjs`.
- **Admin must be opened in Chrome.** Edge's tracking prevention silently blocks Firebase Auth's cross-origin token handoff between `relkady84.github.io` and `batroun-race.firebaseapp.com` — `getRedirectResult` resolves to null. Public page is unaffected.
- **Firebase Auth `signInWithPopup` first, then `signInWithRedirect` fallback** on blocked-popup error codes. The admin and scanner pages do this.
- **Firestore `onSnapshot` fires twice on initial load** (cache then server). Any editor that populates inputs from a listener needs a "dirty" flag, or user-typed values get wiped between keystrokes and Save. See `competitionDirty` in admin.html.
- **GitHub Pages cache is aggressive.** Hard reload (`Ctrl+Shift+R`) after pushing.
- **GitHub Pages folder must be `/docs`** — Pages only supports root or `/docs`, not `/public`. The folder is named `docs/` for this reason.
- **`firestore.rules` changes require manual re-publish** in Firebase Console → Firestore → Rules tab. Pushing to GitHub doesn't deploy rules.
- **Don't reintroduce `api.qrserver.com`** — was the only external dependency at runtime, swapped for client-side `qrcodejs` so 1000+ registrations don't hit any rate limit.
- **WhatsApp messages: no emojis** — some Android devices render unknown emojis as "?" in WhatsApp. Use plain text labels. (Rule is WhatsApp-specific; admin UI and docs can use emojis freely.)

## How to test locally

```bash
cd docs && python3 -m http.server 8000
# then open http://localhost:8000
```

For Firebase emulator testing:
```bash
firebase emulators:start --only firestore,auth
# then point firebaseConfig at the emulator
```

## How to deploy

GitHub Pages serves `/docs` on `main`. Just push.

```bash
git add . && git commit -m "..." && git push
```

Updates within 1–2 minutes. Hard-reload to bypass cache.

If `firestore.rules` changed, also re-publish manually in Firebase Console.

## Branch conventions

- `main` — production. GitHub Pages deploys from `/docs` on this branch, so every push is live within 1–2 minutes.
- `claude/...` — feature branches used by Claude Code on the web. Develop and push here; merge to `main` via PR (or fast-forward locally) only when you're ready to ship to production.
- Never push directly to `main` from a Claude Code session unless explicitly asked — the deploy is immediate and irreversible from the user's perspective.

## Status — what's built

- ✅ Public registration page with custom form fields, Whish payment instructions, phone-uniqueness, sub-question (age bracket) auto-fill from DOB, registration-open window enforcement
- ✅ Admin dashboard:
  - Registrations table with 17 columns, 16 filters, all sortable, per-browser customization
  - Confirmed tab for paid/free only (also visible to staff)
  - Backup tab (admin only) with Restore / Permanent delete / Restore all / Permanent delete all
  - Categories CRUD with sub-question editor + preset 10K/5K brackets
  - Form fields CRUD (text/number/textarea/checkbox/dropdown)
  - Theme tab: logo upload, background upload, colors, copy, banner, logo text, logo size
  - Competition tab: name/year/dates
  - Dashboard customization tab: filter/column visibility + scanner card fields
  - Settings tab: add/remove additional admins + staff
- ✅ Soft delete + 10s undo toast (stacked, multiple simultaneous undos work)
- ✅ Payment approval flow → auto-opens pre-composed WhatsApp message with ticket link
- ✅ Public ticket page (ticket.html) with client-side QR
- ✅ Race-day scanner (scan.html) with camera + manual fallback + admin-customizable result card fields + check-in tracking
- ✅ Two-tier access: super admins hardcoded, additional admins + staff dynamic via Firestore
- ✅ Codespaces devcontainer for cloud development

## Open ideas (not yet built)

- Capacity gate via transaction (data model is ready; UI not wired)
- Cloud Function or Twilio integration for fully-automatic WhatsApp send (currently semi-manual)
- Admin-side "Check in manually" button on a registration (currently only scanner does it)
- Sticky Actions column on the registrations table (sometimes scrolls out of view with many columns enabled)
- Re-enable a "default background photo" preset that admin can apply with one click

## Who to ask

- **Project owner**: Raed Elkady (`raedelkady@gmail.com`)
- **Co-admin**: Nizar Elkady (`nizarelkady@gmail.com`)
- For the day-of: staff are added/removed via the admin **Settings** tab.
