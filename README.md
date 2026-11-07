# Giftify

A web app for building **personal, shareable gifts**. A sender fills in memories,
photos, wishes, music and a few details; Giftify seals it behind a secret pass
and hands back a private link. The recipient opens the link, enters the pass,
and unwraps the gift at their own pace — complete with a memory museum, a
chronological photo puzzle, an optional MCQ challenge and flip-card wishes.

> Originally built as "Nazrana" and "Petrichor"; the project now ships under the
> **Giftify** brand.

---

## Features

### Marketing site (`Code/files/Index/index_2.html`)
- Hero, "how it works" steps, feature grid, demo preview, testimonials and a
  final call-to-action.
- Responsive layout with the cream / coral / charcoal palette and
  Cormorant Garamond · Nunito Sans · Caveat typography.
- Onboarding popup slides driven by `index_2.js`.
- The account icon in the nav opens the sign-in page.

### Account access (`Code/files/Index/login.html`)
- Sign in with either an **email address or a phone number** plus a password,
  validated in a single field.
- **Sign up** mode with name, email/phone and password, toggled by the page
  links or `#signup` / `#signin` hashes.
- **Continue with Google** opens a consent modal that requests the user's
  **name** and **email address**, then creates an account on first use or signs
  into the existing one.
- **Forgot password?** opens a recovery modal with inline validation and a
  confirmation state.
- Show/hide password, loading buttons, toast feedback, focus-trapped dialogs
  and an `inert` page background while a modal is open.
- Demo front-end auth: accounts and the session are kept in
  `localStorage` / `sessionStorage`, and success redirects to the gift builder.
  Replace the demo hooks with the FastAPI auth endpoints and a Google OAuth
  client ID before going live.
- Lighthouse: **100 Accessibility / 100 Best Practices / 100 SEO**.

### Gift builder (`Code/files/form/form.html`)
Seven guided sections, tracked by a sticky progress rail:

1. **Basic details** — loved one's name, your name, secret pass, pass hint,
   custom link (auto-generated from the two names) and their age.
2. **Sweet greetings** — optional short message, long message and final note,
   with live character counters.
3. **Memory museum** — photo uploads with 5 MB limit and live previews:
   - *Polaroid*: up to 4 square (1:1) photos.
   - *Special moments*: up to 4 photos (4:3 or 3:4).
   - *Chronological game*: ordered photos the receiver gets as a jumbled puzzle,
     with up/down reordering.
4. **MCQ puzzle (optional)** — manually add questions and 2–6 options, mark the
   correct answer, add/remove with `+` buttons.
5. **Music** — paste a Spotify / YouTube / Apple Music / SoundCloud link.
6. **Wishes** — up to 5 wishes, each rendered as a hover/click flip card.
7. **Review details** — live summary of everything entered, with "Edit"
   shortcuts back to each section.

Other builder niceties:
- In-memory state (no page reloads while editing), live review updates.
- Photo validation (type + 5 MB), slot removal, reordering.
- Show/hide secret pass toggle.
- Success modal with a copyable shareable link.
- Accessible focus management (`inert` modal), keyboard-flippable wishes.
- Lighthouse: **100 Accessibility / 100 Best Practices / 100 SEO**.

### Backend (`Code/backend`)
- **FastAPI** + **SQLAlchemy** + **Neon PostgreSQL**.
- Tables: `gifts`, `gift_photos`, `gift_questions`, `gift_question_options`,
  `gift_wishes`.
- Secret passes are **bcrypt-hashed** (SHA-256 pre-hash, never stored in plain text).
- Database-level safeguards mirror the form: age 1–120, max 4 polaroids/moments,
  chronological positions `0..n-1`, photos ≤ 5 MB, max 5 wishes, one correct
  option per question (partial unique index), unique slug, cascade deletes.
- Endpoints: create gift, lock-screen summary, unlock with secret pass, grade
  MCQ answers server-side, health check.
- CORS, environment-driven config, `pool_pre_ping` for Neon's scale-to-zero.

---

## How it works

1. **Join** — the sender signs in with an email/phone and password, creates an
   account, or continues with Google. The demo stores the account in
   `localStorage` and sends them to the builder.
2. **Build** — the sender opens the gift builder, fills the seven sections and
   uploads photos (kept as local previews in the browser).
3. **Upload media** — photos are uploaded to object storage (e.g. AWS S3,
   already configured via the `AWS_*` keys); the resulting HTTPS URLs are sent
   to the backend as `storage_url`.
4. **Seal** — `POST /api/gifts` validates the payload, hashes the secret pass
   and stores the gift. The API returns a `share_url` such as
   `giftify.gift/ayesha-from-shahwaz`.
5. **Share** — the sender copies the link from the success modal and sends it
   to their loved one.
6. **Unlock** — the recipient opens the link, sees only the lock screen (name,
   hint, MCQ flag), and enters the secret pass. `POST /api/gifts/{slug}/unlock`
   verifies it (bcrypt) and returns the full gift.
7. **Experience** — the recipient reads the greetings, browses the memory
   museum, solves the chronological puzzle and MCQ, listens to the music and
   flips through the wishes.
8. **Grade** — MCQ answers are checked with `POST /api/gifts/{slug}/answers`,
   which returns the score without leaking correct options beforehand.

---

## Running locally

### Frontend
Serve `Code/files` over HTTP (the form uses relative links):

```powershell
cd D:\Nazrana\Code\files
python -m http.server 8123
```

Then open `http://localhost:8123/Index/index_2.html`,
`http://localhost:8123/Index/login.html` and
`http://localhost:8123/form/form.html`.

### Backend
See `Code/backend/README.md`. Short version:

```powershell
cd D:\Nazrana\Code\backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
# add DATABASE_URL to .env (Neon connection string)
python init_db.py
uvicorn app.main:app --reload --port 8000
```

---

## Tech stack

- **Frontend:** HTML5, CSS3, vanilla JavaScript (no build step).
- **Backend:** Python, FastAPI, SQLAlchemy 2.x, Pydantic v2, bcrypt.
- **Database:** PostgreSQL on Neon.
- **Media:** object storage (AWS S3 compatible) referenced by URL.

---

## Project layout

```
D:\Nazrana
├── Code
│   ├── files
│   │   ├── Index        marketing site
│   │   └── form         gift builder
│   ├── backend          FastAPI + SQLAlchemy API
│   └── dumps            archived copies
└── Media Files          onboarding popup assets
```

---

## Notes

- The builder is currently front-end only in preview; wiring it to the backend
  means uploading photos to storage and calling `POST /api/gifts`.
- Draft gifts (status `draft`) are never visible to receivers.
- Add rate limiting on `/unlock` and restrict `CORS_ORIGINS` before going live.
