# Batroun Race Registration

Registration system for the annual **Batroun Race** in Lebanon. Public registration page, admin dashboard, race-day QR scanner — all running on a free-tier Firebase + GitHub Pages stack.

## Live URLs

| Page | URL | Audience |
|---|---|---|
| Registration | https://relkady84.github.io/Batroun-Race/ | Runners (any browser) |
| Admin | https://relkady84.github.io/Batroun-Race/admin.html | Staff / admins (**Chrome only**) |
| Race-day scanner | https://relkady84.github.io/Batroun-Race/scan.html | Hostesses (Chrome / Safari on phone) |

## Docs

- **[docs/USER_GUIDE.md](docs/USER_GUIDE.md)** — everything a runner, staff, admin, or hostess needs to know
- **CLAUDE.md** — project context for AI-assisted development
- **WORKFLOW.md** — historical build plan (kept for reference)

## Features

- Public registration with category + sub-category (age-bracket) auto-fill from DOB, age-range validation, phone-number uniqueness, customizable form fields, customizable theme/logo/banner.
- Admin dashboard with role-based access (super admin · admin · staff), registration management, payment confirmation with auto-WhatsApp ticket, soft-delete with 10s undo, Backup tab for restoration.
- Per-category sub-question dropdowns (e.g. age brackets for 10K and 5K) — auto-filled from runner's DOB.
- Race-day QR scanner — staff sign in, scan runner tickets, check them in.
- Settings tab to add/remove user access without touching code.

## Stack

- Vanilla HTML/CSS/JS — no build step, runs straight from `/docs` on GitHub Pages
- Firebase Firestore (free Spark plan) for data
- Firebase Auth (Google) for staff sign-in
- Client-side QR generation (no third-party API)
- Inter font · custom dark/teal theme

## Local dev

```bash
cd docs && python3 -m http.server 8000
# then open http://localhost:8000
```

## Deploy

Just push to `main`. GitHub Pages serves `/docs` automatically. After a push wait ~1 min, then hard-reload (`Ctrl+Shift+R`).

If `firestore.rules` was changed, also paste the new rules into Firebase Console → Firestore → Rules → Publish.

## Contact

Lycée Montaigne École, Beit Chabab, Lebanon.
