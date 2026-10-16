# RecruitLens
**Scan Your Resumes** · an Android app by TeamExyKings

RecruitLens reads a resume, scores it with an AI model, and keeps every candidate
in one place for an individual recruiter or a whole hiring team. It also has a
Resume Builder that turns a person's own resume into one of 16 templates (PDF or
Word).

| | |
|---|---|
| Platform | Android only (package `com.teamexykings.recruitlens`) |
| Internal drop | **v1.80** (shown as "Test build 1.80" on development and preview builds) |
| Store version | `1.0.0` (`app.json`), not yet on Google Play |
| Stack | Expo SDK 57 · React Native 0.86 · React 19 · Expo Router · Zustand |
| Backend | Supabase (Auth, Postgres with row-level security, Edge Functions) |
| Services | OpenRouter (AI models) · Razorpay (payments) · Brevo (email) |

---

## What it does

**Everyone**
* Scan a PDF or Word resume with the AI model of your choice: a score, a
  recommendation, matched and missing skills. The file itself is never stored;
  the text is read on the server and the file is discarded.
* Candidate tracker with an 11-step status pipeline, notes, a timeline, rescan
  with duplicate detection and version history, side-by-side compare, Excel export
  (paid plans), analytics.
* **Resume Builder**: restructure your own resume into one of **16 templates**
  (ATS-friendly, modern, two-column, bold), see a sample picture first, save as an
  A4 PDF or a Word file.
* Plans and credits, transaction history, notifications, Pitch Black / Black &
  Grey / White / System themes with 12 accent colours.
* Sign in with email and password, or with **Google**. Optional
  **two-factor authentication** (authenticator app) from Settings.
* **Delete Account** from Settings (Google Play requirement).

**Organisations (company accounts)**
* Join with a domain code; the Domain Admin approves by email link.
* Teams with tech stacks, team leads, member invites (temporary password), seat
  allocation, credit pool, one-time Domain Licence, Domain Log (who scanned what).
* Roles: `member`, `team_lead`, `spoc` (Domain Admin, with an optional Sub Admin),
  `superadmin`.

**Super Admin**
* Admin panel: models, users, skills review, domain on/off, Financial tab
  (revenue against AI cost), account-deletion processing (SQL).

---

## Project layout

```
src/
  app/            Screens (Expo Router, file-based)
    (auth)/       Login, sign-up, complete profile, waiting for approval, two-factor check
    (main)/       Tabs (Upload, Tracker, Analytics, Resume, Settings) and every detail screen
    delete-account.tsx, auth-callback.tsx
  components/     Shared UI (ResumeTemplatePicker, CodeInput, SocialSignIn, ModelDropdowns ...)
  constants/      Plans, models, themes, app info (names, links)
  hooks/          Theme, auth providers, access watcher
  services/       Supabase client and every call to the backend
  store/          Zustand stores (auth, candidates, app, theme)
  utils/          Pure logic, tested without a phone (resumeTemplates, resumeDocx,
                  accountDeletion, financeFormat, mfaQr, oauthUrl ...)
assets/
  images/         Icons and splash
  resume-previews/  The 16 sample pictures shown in the template picker
scripts/          make-resume-previews.mjs (regenerates those pictures)
tests/            node:test suites (no phone needed)
docs/             PLAY_STORE_NOTES.md, sample-resume.pdf
*.html            Pages served by GitHub Pages: index, privacy-policy,
                  terms-of-service, account-deletion
app.json          App settings (name, package, permissions, plugins)
app.config.js     Build-time tweaks driven by the EAS profile
eas.json          Build and submit profiles
```

The database and the Edge Functions live in the separate **Supabase** folder
(SQL migrations 01 to 21, 22 functions, set-up guides).

---

## Run it

You need Node 22, npm, an Android phone or emulator, and a **development build**
of the app (Razorpay and the sign-in return link are native, so plain Expo Go is
not enough).

```bash
npm install
npx expo start --dev-client
```

### Environment variables
The app reads two public values (the Supabase address and its public "anon" key;
Row-Level Security is what protects the data, not secrecy of this key):

```
EXPO_PUBLIC_SUPABASE_URL=https://<project-ref>.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=<anon key>
```

* **On your computer**: put them in a `.env.local` file in the project root.
* **On EAS (cloud builds)**: files that Git ignores are not uploaded, so the two
  values must exist as EAS environment variables for every environment you build
  with. Check, and set what is missing:

```bash
eas env:list --environment development
eas env:list --environment preview
eas env:list --environment production
eas env:set --name EXPO_PUBLIC_SUPABASE_URL --value "https://<project-ref>.supabase.co" --environment production --visibility plaintext
eas env:set --name EXPO_PUBLIC_SUPABASE_ANON_KEY --value "<anon key>" --environment production --visibility plaintext
```

A build without them starts and then fails at once, so check **before** you
spend a build. (Set them for all three environments and it does not matter which
one a profile uses.)

---

## Tests

```bash
npm test          # node --test tests/*.test.mjs
```

77 checks cover the template engine, the Word export, finance formatting,
account-deletion wording and rules, sign-in link parsing, two-factor helpers and
error messages. The Word-export tests need `jszip` (it is a dependency, so
`npm install` is enough; without it those 8 are skipped). The database tests
(138 checks) and the Edge Function tests (83 checks) are in the Supabase folder
(`tests/`).

Regenerate the template pictures after changing a template:

```bash
node scripts/make-resume-previews.mjs     # see the header of the script for what it needs
```

---

## Builds and releases

| Profile | Result | Use |
|---|---|---|
| `development` | Debug app that loads JavaScript from your computer | Daily testing |
| `preview` | Installable APK, JavaScript inside | Final check on a phone |
| `production` | App bundle (.aab), `versionCode` raised by EAS | Google Play |

```bash
eas build --platform android --profile development
eas build --platform android --profile preview
eas build --platform android --profile production
eas submit --platform android --profile production
```

* `app.json` is the one place for settings. `app.config.js` only adds the
  "Test build" label (development and preview) and drops the developer-only "draw
  over other apps" permission (preview and production).
* Dangerous permissions (camera, microphone, storage, advertising ID) are blocked
  in `app.json`. A Play build asks the phone for no permission at all.
* There is **no over-the-air update**. A change reaches phones only through a new
  build.
* Native changes (a new native library, `app.json`) need a new build. JavaScript
  changes do not need one while testing on the development build.
* Everything about Google Play is in [`docs/PLAY_STORE_NOTES.md`](docs/PLAY_STORE_NOTES.md).

---

## Documentation

| File | What it is |
|---|---|
| [`CHANGELOG.md`](CHANGELOG.md) | What changed in every version |
| [`docs/PLAY_STORE_NOTES.md`](docs/PLAY_STORE_NOTES.md) | Google Play checklist, listing text, Data safety answers, payments decision |
| `Supabase/README.md` | Database order, Edge Functions, secrets, schedules, runbooks |
| `Supabase/SOCIAL_SIGNIN_SETUP.md` | Google sign-in, step by step |
| `index.html`, `privacy-policy.html`, `terms-of-service.html`, `account-deletion.html` | Public pages on our own domain: `https://teamexykings.in/recruitlens/` (the `recruitlens` folder of the TeamExyKings website repo) |

---

## Security notes

* Never commit: the Supabase **service-role** key, the Razorpay secret, the
  OpenRouter and Brevo keys, an Android keystore, or the Google Play service
  account file. They live in Supabase secrets and in EAS. `.gitignore` already
  excludes keystores and `*service-account*.json`.
* The public repository contains only the app. The anon key is public by design.
* All data rules are enforced in the database (row-level security plus
  restrictive rules for blocked accounts and for two-factor), not only in the app.

---

© TeamExyKings · see [LICENSE](LICENSE)
