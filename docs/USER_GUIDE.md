# Batroun Race — User Guide

A step-by-step guide for everyone who interacts with the registration system: runners, staff, admins, and race-day hostesses.

---

## The three URLs

| Page | URL | Who uses it |
|---|---|---|
| **Public registration** | https://register.batrounrace.com/ | Runners — anyone signing up |
| **Admin dashboard** | https://register.batrounrace.com/admin.html | Admins + staff — must sign in with Google (use Chrome) |
| **Race-day scanner** | https://register.batrounrace.com/scan.html | Hostesses on race day — same Google sign-in |

> ⚠ **Admin & scanner pages only work in Chrome.** Microsoft Edge silently blocks Firebase sign-in due to its tracking-prevention feature. The public registration page works in any browser.

---

## 1. For a runner — registering

1. Open the registration link on a phone or computer.
2. Fill out the form:
   - Pick a race (10K, 5K, Marathon, etc.)
   - Enter your date of birth — the **age category** dropdown fills automatically based on your DOB; you cannot pick the wrong one
   - Fill in name, phone, email, T-shirt size, etc.
   - Tick the two consent boxes
3. Click **Register**.
4. On the confirmation screen you'll see:
   - A unique **reference code** like `BTN-2026-AB3K` — write it down or screenshot it
   - **How to pay via Whish Money** — three steps:
     1. Open Whish, send the listed amount to *Batroun Race*
     2. Paste your reference code into the Whish **note / comment field**
     3. Save your reference (screenshot or write it down)

> 💡 **The reference is mandatory.** If you don't include it in your Whish note, payment cannot be matched to your registration.

5. Once an admin verifies your payment, you'll receive a **WhatsApp message** with:
   - Your bib number
   - A link to your **race ticket** (with a QR code)
   - Show this ticket at the pickup desk on race day

### One phone number = one registration
You cannot register twice with the same phone number. If you try, you'll get a clear "already registered" message.

---

## 2. For staff (Mary, Raed84, Casper, anyone added) — daily work

You can sign in to the admin dashboard but you see **only two tabs**:
- **Registrations** — every signup, with filters and search
- **Confirmed** — runners whose payment has been approved

### Confirming a Whish payment
1. Sign in to https://register.batrounrace.com/admin.html (Chrome only)
2. **Registrations** tab → find the runner whose Whish payment you've matched (search by reference code, name, or phone)
3. Click **Confirm pay**
4. Enter their **bib number**
5. Tick "Open WhatsApp message after confirming" (default on)
6. Click **Mark as paid**
7. A new tab opens with a pre-filled WhatsApp message — click **Send** in WhatsApp Web

The runner now appears in the **Confirmed** tab and receives their ticket.

### Deleting a spam or accidental registration
1. Click **Delete** on the row
2. A warning dialog asks you to confirm
3. After OK, a yellow toast appears at the bottom: *"… deleted — moved to Backup — 10s"* with an **Undo** button
4. Click Undo within 10 seconds if you changed your mind — otherwise it moves to the admin's Backup tab

> Deleted registrations are NOT permanently removed — admins can restore them from the Backup tab.

### Undoing a wrongly-confirmed payment
On the **Confirmed** tab, every row has a **↩ Revert** button. Clicking it clears the bib, payment date, and check-in, and moves the runner back to **Registrations** as *pending*. Use this when you confirmed the wrong person or matched the wrong Whish transfer.

### Re-sending a WhatsApp ticket
On any confirmed row, click the **WA** button to reopen the same pre-filled WhatsApp message. Useful if the runner says they didn't receive it.

---

## 3. For admins (raed, nizar, and anyone with full access) — full powers

You see all tabs in the admin nav:

| Tab | What it does |
|---|---|
| **Registrations** | All signups (pending + paid + free), with sortable columns and filters |
| **Confirmed** | Just the paid/free ones — used on race day |
| **Backup** | Soft-deleted registrations. Restore individually or with **Restore all**. Permanently delete to free up the phone number for re-registration. |
| **Dashboard** | Customize which filters/columns appear in the table (per-browser preference), and which fields appear on the hostess scanner card (saved to Firestore for all hostesses) |
| **Categories** | Add/edit/delete race categories. Each can have an optional **sub-question dropdown** (e.g. age brackets for 10K and 5K). Use **Seed default categories** to start from the Race.docx list. |
| **Form fields** | Add custom optional fields to the public registration form (T-shirt color, team name, etc.) |
| **Theme & copy** | Upload logo, set background image, change heading/subtitle/top banner text, customize logo size and the text shown next to it. All changes propagate to the public page on next reload. |
| **Competition** | Event name, year, registration opens/closes dates, race day |
| **Settings** | **Add or remove user access** — see below |

### Customizing the registration form
- **Categories tab** → Add a race. If you want an age-bracket dropdown like the 10K/5K examples, expand "Optional extra dropdown" and either click one of the preset buttons or paste your own options.
- **Form fields tab** → Add fields like *T-shirt color* (text/number/checkbox/dropdown). They appear automatically on the public form below the standard fields.

### Customizing the look
- **Theme & copy tab**:
  - Upload a **logo** image (auto-compressed, displays top-left on both public and admin pages)
  - Add **text next to the logo** (e.g. "Batroun Race 2026") — gets the same white→teal gradient as the heading
  - Adjust the **logo height** (40–200 px)
  - Upload a **background image** for the public page
  - Change the **page heading**, **subtitle**, and **top banner text**
  - Pick **accent colors** and **gradient backgrounds**
- After making changes, click **Save theme & copy** at the bottom

### Managing event dates
- **Competition tab** → set Registration opens / Registration closes
- Before "opens", the public page shows *"Registration opens on [date]"* instead of the form
- After "closes", it shows *"Registration is now closed"*
- Both are also enforced by Firestore rules — no one can sneak a registration in outside the window

### Backup tab actions
- **View** — see the full registration
- **Restore** — move back to Registrations as if not deleted
- **Permanent delete** — actually wipes from Firestore (frees the phone number)
- **Restore all** / **Permanently delete all** at the top

---

## 4. For admins — adding/removing user access (Settings tab)

### Two-tier access
- **Super admins** (locked in code, never removable): `raedelkady@gmail.com`, `nizarelkady@gmail.com`
- **Additional admins**: full access including Settings — add via Settings tab
- **Staff / viewers**: limited access (Registrations · Confirmed · scanner) — add via Settings tab

### Adding a user
1. **Settings** tab in admin
2. Type their Gmail in **Add admin** or **Add staff** input
3. Click the button — they're saved immediately to Firestore
4. They can sign in to admin (and scanner) on their next visit

### Removing a user
Click the red **Remove** button next to their email. Takes effect immediately — they'll be signed out on next page load.

---

## 5. For race-day hostesses — using the scanner

The day of the race, open https://register.batrounrace.com/scan.html on a phone (Chrome or Safari).

1. **Sign in with Google** — use your approved staff email (admins set this up in Settings before the event)
2. **Allow camera access** when prompted
3. Point the camera at the runner's QR code (from their WhatsApp ticket message)
4. The result card appears immediately:

| Card color | What it means | What to do |
|---|---|---|
| 🟢 **Green — PAID** | Registration is valid and not yet checked in | Click **"Mark as checked in → hand out bib"** → hand out the bib + race pack |
| 🟡 **Yellow — ALREADY CHECKED IN** | This runner already collected their bib (shows when + who scanned them) | Don't issue a second bib. Refer to admin if there's a dispute. |
| 🔴 **Red — NOT PAID** | Payment hasn't been confirmed | Send them to the admin / payment desk |
| 🔴 **Red — Not found** | The QR doesn't match any registration | Refer to admin |
| 🔴 **Red — Deleted** | This registration was deleted | Refer to admin |

### Backup if the QR won't scan
Under the camera, there's a text input. Type the reference code manually (e.g. `BTN-2026-AB3K`) and click **Look up reference**. Same result card.

### What fields show on the result card
Admins control this in **Dashboard tab → Race-day scanner card** (16 possible fields: name, bib, category, sub-category, reference, payment status, age, t-shirt, etc.). Defaults are: Name, Bib, Category, Sub-category, Reference, Payment.

### After the event
Admin can use the **Dashboard tab → "Check-in status" filter** (after enabling it) to see who didn't arrive. The **Confirmed tab** shows a green check on every checked-in row.

---

## Quick reference — what to do when…

| Situation | Where to go |
|---|---|
| Runner says they paid but I can't find their reference | **Registrations tab** → search by name or phone |
| Runner says they never got the WhatsApp ticket | **Confirmed tab** → find them → click **WA** to resend |
| Confirmed someone by mistake | **Confirmed tab** → click **↩ Revert** on that row |
| Spam / accidental registration | **Registrations tab** → **Delete** → 10s undo if you change your mind |
| Spam survived past 10s | **Backup tab** → **Permanent delete** |
| Adding new staff for the event | **Settings tab** → **Add staff** |
| Want a different logo/color/banner | **Theme & copy tab** → upload/edit → **Save theme & copy** |
| Want to know who hasn't arrived on race day | Enable "Check-in status" filter (Dashboard tab) → set to "Not checked in" |
| Forgot to mark someone as checked in at the scanner | Open the registration in admin → currently no UI for this (let admin know if you need it) |

---

## Stack & limits (for the curious)

- **Frontend**: vanilla HTML / CSS / JavaScript on GitHub Pages — no build step, no framework
- **Backend**: Firebase Firestore (free Spark plan) for storage + Firebase Auth (Google) for staff sign-in
- **QR codes**: generated client-side in the browser, no external API
- **WhatsApp**: pre-composed `wa.me` links open in WhatsApp Web — admin clicks Send manually
- **Free tier supports 1000+ registrations comfortably**

If something looks broken, hard-reload with **Ctrl+Shift+R** first — GitHub Pages caches aggressively.
