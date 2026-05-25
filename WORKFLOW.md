# Workflow — Batroun Race Build Plan

Sequential build phases. Each phase has a goal, acceptance criteria, and a **ready-to-paste prompt** for Claude Code. Work one phase at a time, commit at the end of each phase.

---

## Phase 1 — Public registration page ✅ DONE

Shipped in `public/index.html`. Wave-aware pricing, age validation, Whish reference, Firestore write, confirmation panel.

**Still TODO in this phase:**
- Replace `firebaseConfig` placeholder with real values (see Phase 0 setup below)
- Replace QR placeholder `<div>` with `<img src="assets/whish-qr.png">`

---

## Phase 0 — Setup (do this first, before any new code)

### Goal
Get Firebase project + GitHub repo wired up.

### Steps
1. Create Firebase project at https://console.firebase.google.com — name it `batroun-race`
2. Enable **Firestore** (start in production mode, rules come in Phase 2)
3. Enable **Authentication** → add Microsoft provider (you'll need Azure tenant ID — same as Café Molière setup probably)
4. From Project Settings → General → "Your apps" → add a Web app → copy the `firebaseConfig` object
5. Paste into `public/index.html` (replace the `REPLACE_ME` block)
6. Create GitHub repo `batroun-race` (public)
7. Enable GitHub Pages: Settings → Pages → Deploy from `main` branch, `/public` folder
8. Drop the school's Whish QR code as `public/assets/whish-qr.png`
9. Commit and push everything

### Acceptance
- Page loads at `https://[your-username].github.io/batroun-race/`
- Filling the form successfully writes a document to Firestore (check Firebase console)
- Whish reference appears on the confirmation panel

---

## Phase 2 — Firestore security rules + capacity gate

### Goal
Lock down the database so nobody can tamper with registrations or bypass capacity.

### Prompt for Claude Code

```
Read CLAUDE.md for context.

Create `firestore.rules` with the following requirements:

1. Public can CREATE registrations in /competitions/{competitionId}/registrations/{regId}
   - Validate required fields exist and are correct types
   - Force paymentStatus to "pending" or "free" on create (never "paid")
   - Force bibNumber to null on create
   - Force registeredAt to server timestamp
   - Block writes if competition's registrationCloses is past

2. Public can READ competitions/{competitionId} and its categories subcollection
   (so the page can show wave dates, capacity remaining, etc.)

3. Public CANNOT read /registrations/* — privacy. Only admins.

4. Admins (authenticated users with email ending in the school domain) can:
   - Read/write everything under competitions/
   - Update paymentStatus, bibNumber, paidAt on registrations
   - Update capacity and registeredCount on categories

5. Nobody can delete registrations (audit trail).

Then update `public/index.html` to use a Firestore TRANSACTION instead of plain addDoc:
- Read /categories/{categoryId} doc
- Check registeredCount < capacity (if capacity is set)
- Create the registration doc
- Increment registeredCount atomically
- If full, throw and show "Sorry, this category is full" to user

Test with the Firebase emulator before deploying rules.
```

### Acceptance
- Rules deployed: `firebase deploy --only firestore:rules`
- Attempting to write a registration with `paymentStatus: "paid"` from client fails
- Two browser tabs hitting submit at the same time when only 1 spot is left — exactly one succeeds, the other gets "category full"
- Reading the registrations collection from an unauthenticated browser fails

---

## Phase 3 — Seed the competition + categories

### Goal
Put the actual competition and category data into Firestore so the app reads from it instead of hardcoding.

### Prompt for Claude Code

```
Read CLAUDE.md.

Create a one-shot Node.js seed script `scripts/seed-competition.js` that uses 
firebase-admin SDK to write:

- competitions/batroun-race-2026 with:
  - name, year: 2026
  - registrationOpens: 2026-02-15
  - registrationCloses: 2026-11-20T23:59:59
  - raceDay: 2026-11-22 (CONFIRM with user before running)
  - waves: array of 3 from CLAUDE.md
  - whish: { number: "ASK USER", qrImageUrl: "assets/whish-qr.png" }

- competitions/batroun-race-2026/categories/{id} for all 5 categories
  from the table in CLAUDE.md, with:
  - capacity: null for now (admin will set per category)
  - registeredCount: 0

Document in the script how to run it:
  npm install firebase-admin
  Provide service account key path
  node scripts/seed-competition.js

Then refactor `public/index.html` to fetch competition + categories from 
Firestore on page load instead of using the hardcoded CATEGORIES/WAVES 
constants. Keep the constants as a fallback if Firestore is unreachable.
```

### Acceptance
- Firestore console shows the competition doc and 5 category docs
- `index.html` reads them on load — change a price in Firestore, refresh page, new price shows
- Hardcoded constants serve as offline fallback

---

## Phase 4 — Admin dashboard

### Goal
Private dashboard for school staff to manage registrations, confirm payments, assign bibs, export data.

### Prompt for Claude Code

```
Read CLAUDE.md.

Build `public/admin.html` — admin dashboard for Batroun Race registrations.

Auth:
- Microsoft sign-in via Firebase Auth (signInWithRedirect)
- Reject anyone whose email doesn't end with the school's domain (ask user 
  what the domain is — likely @lyceemontaigne.edu.lb or similar)
- Sign-out button visible

Layout (single page, same dark blue/teal aesthetic as index.html, reuse 
the CSS variables):

1. Top stats cards: total registrations, pending payment, paid, free, 
   total $ collected, breakdown by category

2. Filters bar: category dropdown, payment status dropdown, wave 
   dropdown, search by name/email/reference

3. Registrations table:
   - Columns: ref code, name, category, wave, price, status, bib, 
     registered date, actions
   - Sortable columns
   - Pagination (50 per page)

4. Row actions:
   - "Confirm payment" → opens modal, asks for bib number, marks as paid, 
     sets paidAt timestamp
   - "View details" → modal showing all fields + emergency contact + medical notes
   - "Send WhatsApp" → generates wa.me link with reference + bib info, 
     opens in new tab

5. Capacity panel (sidebar or top section):
   - For each category, show registered/capacity with a progress bar
   - Inline edit of capacity (admin-only, writes back to Firestore)

6. Export button: dump current filtered view to CSV (XLSX optional later)

Real-time updates: use Firestore onSnapshot for the registrations list so 
multiple admin tabs stay in sync (Café Molière pattern).

NO deletion functionality. Audit trail matters.
```

### Acceptance
- Admin can sign in with Microsoft, non-school accounts are rejected
- Table shows all registrations with correct data
- Confirming a payment updates Firestore: paymentStatus → "paid", bibNumber set, paidAt timestamp
- Capacity editing works and shows immediately on public page
- CSV export downloads correctly

---

## Phase 5 — WhatsApp confirmation flow

### Goal
When admin confirms a payment, generate a pre-filled WhatsApp message to the registrant.

### Prompt for Claude Code

```
Read CLAUDE.md.

In admin.html, when "Confirm payment" succeeds, automatically open a new 
tab with a pre-filled WhatsApp message to the registrant's phone:

Template:
  Hi {firstName}, your Batroun Race 2026 registration is confirmed! 🏃
  
  Reference: {whishReference}
  Bib number: {bibNumber}
  Category: {categoryName}
  Race day: {raceDay} — see you at the start line!
  
  Pickup details and race pack info will follow closer to the event.

Phone normalization: strip spaces/dashes, ensure it starts with country 
code (961 for Lebanon).

URL: https://wa.me/{normalizedPhone}?text={urlencoded}

Add a setting in admin to toggle "auto-open WhatsApp on payment confirm" 
on/off (stored in localStorage).
```

### Acceptance
- Clicking "Confirm payment" opens WhatsApp Web with the right message
- Phone numbers in various formats all get normalized correctly

---

## Phase 6 — Testing + polish

### Goal
End-to-end tests, edge cases, deployment hardening.

### Checklist
- [ ] Submit a Marathon registration → confirm pending → admin sees it
- [ ] Submit a Para Athlete registration → confirms as free immediately
- [ ] Try age 17 for Marathon → rejected with clear error
- [ ] Try to set capacity = 1, submit two registrations simultaneously → one succeeds
- [ ] Try to access /admin.html without signing in → redirected to sign-in
- [ ] Try to sign in with non-school email → rejected with message
- [ ] Test on iPhone Safari (mobile first audience for parents)
- [ ] Test on Android Chrome
- [ ] Hard-reload after deploy actually shows new version
- [ ] Whish QR loads correctly on confirmation page
- [ ] CSV export works in Excel (UTF-8 BOM, French characters render)

---

## Open questions (decide before running phases that need them)

- [ ] **Race day date** — confirmed 22 Nov 2026? Or different?
- [ ] **School Whish number** — what's the actual Whish business number to display?
- [ ] **Admin email domain** — what's the domain to allow? (`@lyceemontaigne.edu.lb`? Other?)
- [ ] **Per-category capacity numbers** — Marathon: ?, Half: ?, 10K: ?, 5K: ?, Para: ?
- [ ] **Race day timing details** — start time per category, will go on public page
