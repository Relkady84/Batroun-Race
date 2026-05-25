# Batroun Race Registration

Annual running competition registration system. Public registration page + admin dashboard, deployed on GitHub Pages with Firebase Firestore backend.

## Quick start

1. **Set up Firebase**: see `WORKFLOW.md` → Phase 0
2. **Configure**: replace `firebaseConfig` placeholder in `docs/index.html`
3. **Seed the database**: run `scripts/seed-competition.js` (after Phase 3)
4. **Deploy**: push to `main`, GitHub Pages serves `/docs`

## Stack

- Vanilla HTML/CSS/JS — no build step
- Firebase Firestore — database
- Firebase Auth (Microsoft provider) — admin only
- GitHub Pages — hosting
- Whish Money — payments (manual reconciliation)

## Docs

- `CLAUDE.md` — full project context (read this first)
- `WORKFLOW.md` — phase-by-phase build plan with ready-to-use prompts

## Status

| Phase | Status |
|-------|--------|
| 1. Public registration page | ✅ Done |
| 0. Firebase + GitHub setup | ⏳ Pending |
| 2. Security rules + capacity gate | ⏳ Pending |
| 3. Seed competition data | ⏳ Pending |
| 4. Admin dashboard | ⏳ Pending |
| 5. WhatsApp confirmation flow | ⏳ Pending |
| 6. Testing + polish | ⏳ Pending |

## Contact

Lycée Montaigne École, Beit Chabab, Lebanon.
