# RecruitLens — Changelog
TeamExyKings

---

## v1.80 (Play Store release build)

- **Play build switch.** `RL_PLAY_BUILD=1` (set automatically by the production profile and the local build script) hides every buying screen and button, and the three buying routes redirect to Settings. Plans and credits still show and are granted by the Super Admin. Normal test builds are unchanged.
- **Local .aab build** without EAS: `plugins/withUploadSigning.js` and `scripts/build-aab.mjs` (pre-flight checks, auto versionCode, SHA-256 of the result).
- Server messages no longer say "Upgrade" (4 functions: enqueue-analysis, analyse-resume, restructure-resume, export-candidates).
- Legal pages now at `teamexykings.in/recruitlens/`.
- Store graphics in `store-assets/` (placeholders). New test file `tests/playBuild.test.mjs`.

---

## v1.79 (Sign-in with Google, two-factor, account deletion, 16 resume templates, Play prep)

> **Run `19_account_security_hardening.sql` as soon as you can.** It closes holes
> that were already in the live database (first section below). It is a database
> change only, so it does not have to wait for the new app build.

### Security fixes (found while adding sign-in)
- **Any signed-in user could change their own role, plan, credits, domain and
  "blocked" flag**, because the protection trigger from migration 13 never actually
  blocked anyone. It is now a whitelist: a person can change only their name and
  phone. (Migration 19)
- **A new profile row could be created with any values** (for example super admin
  with 999,999 credits). The database now forces one of the two legitimate shapes:
  an Individual account, ready to use, or a company request that waits for the
  domain admin. (Migration 19)
- **Approval links could be forged by the person asking to join**, because they
  could read their own approval token. The token is now made on the server and the
  Approve / Reject links in the admin's email are signed. Old unsigned links stop
  working. New function `request-approval` sends the email.
- **Email function locked down.** `send-email` accepted calls from anyone holding
  the public key. Now only other server functions can send any type, and a
  signed-in user can send only an interview email to one of their own candidates.
- 15 Edge Functions now also refuse a half-signed-in session of a person who has
  two-factor on (the 14 listed in the Supabase README plus `admin-manage-user`).

### Sign in with Google
- **Continue with Google** on Login and Sign-up. The button appears only when that
  sign-in is switched on in Supabase, so nothing breaks while you set it up.
- Microsoft sign-in was built and then **left out on purpose**: setting it up needs
  an Azure account, and Google plus email already cover sign-in. If an organisation
  asks for it, it is one more entry in `SOCIAL_PROVIDERS` (`oauthService.js`) plus
  its setup in Supabase.
- A first-time person lands on **Complete Your Account**: name, optional phone,
  then Individual, or join a company with its domain code and a team. A company
  request still waits for the domain admin, like an email sign-up.
- The sign-in comes back through `recruitlens://auth-callback` using PKCE, so
  another app listening for the same link cannot take over a sign-in. It also copes
  with the phone closing the app while the browser is open.
- Someone who signed up with a password and later uses Google with the same
  verified email lands in the same account. Someone who only ever used Google
  can add a password in **Settings → Set a Password**.
- Step-by-step guide: `SOCIAL_SIGNIN_SETUP.md` (Supabase folder).
- **Forgot password** and **email confirmation** now work inside the app with a
  6-digit code (no web page, no link that has to open the app). The Reset Password
  and Confirm Signup email templates in Supabase must show `{{ .Token }}`.
- New code-entry control (six boxes, one real text field): pasting a whole code,
  Backspace and the number pad behave normally.

### Two-factor authentication (optional)
- **Settings → Account → Two-Factor Authentication.** Scan the QR code (or type
  the key) in an authenticator app, enter the first code, done. Turning it off asks
  for a current code.
- When it is on, Google sign-in and password reset ask for the code
  too.
- **The database enforces it**, not just the app: a session that has only passed
  the password sees no data. (Migration 20)
- **Lost phone:** a domain admin resets a member's two-factor from Settings →
  Domain Seats; the Super Admin can reset anyone from Admin → Users. Anyone who has
  nobody above them emails support; the one-line SQL is in the Supabase README.

### Delete account (Google Play requirement)
- **Settings → Account → Delete Account** (also on the waiting-for-approval and the
  Google "finish your account" screens). The person types DELETE and
  sends a request; they can cancel it from the same screen. A request only
  records the wish: the Super Admin checks it and runs
  `SELECT process_account_deletion('email');` within 30 days. (Migration 21)
- What is deleted and what stays is listed on the screen, and differs for an
  Individual (everything they uploaded goes), an organisation member (the
  organisation keeps its records, the name becomes "Deleted user", unused credits go
  back to the pool) and a Domain Admin (must hand the role over first).
- New public page `account-deletion.html`, which Google requires, with the same
  steps and an email route.

### Resume Builder: 16 templates
- **Template picker** with a dropdown, a sample picture of the chosen template and a
  full-screen view. Sixteen templates in four groups: ATS-friendly (Classic,
  Minimal, Compact, Academic, Elegant, ATS Plain), Modern (Professional, Tech,
  Timeline, Fresher), Columns (Modern, Sidebar Light, Balanced) and Bold
  (Executive, Bold Header, Creative).
- **PDF is A4** now (it was US Letter), with the same top and bottom margin on every
  page, and headings never stranded alone at the bottom of a page.
- **Word (.docx) follows the chosen template**: its font, colours, heading style,
  section order, and the coloured band on the Executive template.
- Heading letter-spacing is capped in every template, because wide spacing made PDF
  readers (and applicant tracking systems) read SUMMARY as "S U M M A RY". A test
  keeps it that way.
- Fresher and Academic put Education before Experience; blank, "null" and "N/A"
  values from the AI are left out instead of printed.

### Financial tab (Super Admin)
- Organisations with no payments in the chosen period collapse to one quiet line;
  "No organisations yet" when there are none.
- Exchange-rate line reads in plain words (live / last known), shows the update time
  in your time zone, and has a **Refresh** button. "OpenRouter — Live Utilisation"
  says it is all-time.
- Model names and providers on separate lines; thousands in Indian grouping
  (1,23,456); icons instead of emoji on Money in / Money out.
- Fixed: the "No recharges / nothing here" notes drew in black on the dark theme
  (a missing style); every colour on the tab now follows the theme.

### Google Play preparation
- Privacy Policy and Terms rewritten for this version (sign-in methods, two-factor,
  Resume Builder, deletion, third parties); new `account-deletion.html` and
  `index.html` for GitHub Pages.
- `app.json`: blocks the advertising-ID permission as well as camera, microphone and
  storage; `app.config.js` drops the developer-only "draw over other apps"
  permission from preview and production builds; `eas.json` tags development and
  preview builds "Test build 1.79" and leaves production clean.
- `docs/PLAY_STORE_NOTES.md`: payments decision, Data safety answers, listing text,
  reviewer account, closed-test steps, first-build checks. `docs/sample-resume.pdf`
  for reviewers.
- Real `README.md` (it was still the Expo template).

### Smaller app, fewer permissions
- Removed libraries the app no longer uses (notifications, camera / image picker,
  biometrics, device info, two chart libraries, a UI kit, a spreadsheet library and
  more). The Android build now asks for only Internet and Vibrate: no camera,
  microphone, storage or advertising-ID permission.
- `app.json` cleaned up: Android only, black splash and background, backup off, the
  Expo splash-screen plugin instead of the old `splash` block, no iOS or web
  sections.
- Needs a new build to take effect.

### What to deploy
- **Supabase:** run `19`, `20`, `21` in that order; deploy `approve-user`,
  `send-email`, `request-approval`, `admin-manage-user` and the 14 guarded functions
  (list in the Supabase README); set email templates; delete any old Database
  Webhook that emails approval links.
- **App:** a **new native build** is needed only for the `app.json` changes: the
  permission clean-up, the black splash screen and adaptive-icon background, the System
  theme following the phone, and the smaller app. Everything else here (Google
  sign-in, two-factor, Delete Account, the templates, Settings, the Financial
  tab) is JavaScript plus Supabase, so it runs in your current development build through
  Metro. The web-browser module and the `recruitlens://` link have been in the project
  since 1.60, so a development build made since then already has them. If a screen shows
  a red "native module not found" error, that part needs the new build.
- **GitHub Pages:** `index.html`, `privacy-policy.html`, `terms-of-service.html`,
  `account-deletion.html`.

---

## v1.78 (Settings, About screen, pitch-black polish)

- **Settings reorganised** into clear groups, top to bottom: profile card,
  Credits (paid plans), Account, Plan & Billing, Activity & Data, Appearance,
  Domain Management (admins only), Help & Support, About, Logout. Every group is
  one rounded card with a small heading; the last row of a group has no divider.
- Profile card shows the plan badge and, where it applies, a role badge
  (Super Admin / Sub Admin / Domain Admin / Team Lead).
- **Two-Factor Authentication** row added to Account as a placeholder; it is
  switched on in the next update.
- **About screen** (Settings → About): logo, version, test-build label, builder
  credit, website, Privacy Policy and Terms of Service.
- **Pitch Black is the default** everywhere: screens, tab bar, navigation
  background, status bar and the pre-load screen are true #000000, so there is
  no grey flash on start-up.
- **Contrast pass.** Gold, green, amber and red text use readable versions on
  every theme, and the text on every coloured button (including each plan
  button on Upgrade Plan) is picked automatically as black or white.
- Small layout fixes: Invite Member first/last name fields sit side by side,
  spacing between the notes and cards on Domain Admins, rounded logo on the
  start and login screens.
- Native-only (needs the next EAS build): the System theme following the phone,
  and the black splash / adaptive-icon background in app.json.

---

## v1.77 (Phase 8 — Themes & Colours)

- **Theme system.** Settings → Appearance → Theme & Colours. Themes: Pitch Black
  (new default, true #000000), Black & Grey (the previous look), White, and System
  (follows the phone). The choice is saved on the phone and applied before the
  first screen draws, so there is no colour flash.
- **12 accent colours.** White (becomes Black on the White theme so it never
  vanishes), Red, Blue, Green, Orange, Purple, Pink, Teal, Yellow, Cyan, Indigo,
  Lime. The accent colours primary buttons, active chips, selected tabs and the
  tab-bar highlight; text on top of it switches between black and white
  automatically so it stays readable.
- **Live preview** on the Theme & Colours screen.
- All 35 screens now read colours from the theme instead of a fixed list; warning,
  success and error cards have light-theme versions.
- Replaced the old non-functional Dark Mode switch.
- Note: "System" needs `userInterfaceStyle: "automatic"` (set in app.json from
  v1.78) and takes full effect with the next native build.

---

## v1.16 – v1.76 (summary)

Domain Log and per-user User Log, notifications with mark-read, resume
rescan with duplicate detection and version History, Resume Builder
(PDF/Word, monthly free limit), domain admins and sub admins (set, remove,
swap), seat allocation and permanent member removal, domain-wide
disable/enable, server-side blocked-account checks on all sensitive
functions, database-level blocked-account rule, automatic sign-out when
access is withdrawn, per-model financial view, free-model fallback queue.

The Domain Licence (introduced as an annual licence in v1.11, below) became a
one-time ₹25,000 purchase: 15,000 credits go into the organisation's pool, there
is no renewal and no expiry date, and the pool carries forward. The database
still calls it `annual_license` internally.

---

## v1.15

### Interview Email — 3 Fixes
- Added editable **Interview Location** field on Candidate Detail —
  shown in the email right after the company name, and again as its
  own line
- Interview emails now use the domain's official **Contact Email**
  (new field, set once by the Primary Admin under Domain Admins →
  e.g. hr@yourcompany.com) instead of whichever admin happened to
  click Send
- Removed the personal email from every email template's footer —
  just "RecruitLens by TeamExyKings" now

---

## v1.14 (Phase 1 — Bug Fixes)

- **Team reassignment / live-refresh bug** — Upload, Tracker, and
  Team Management now refetch on every screen focus
  (`useFocusEffect`), not just once on mount. New teams and
  reassignments show up immediately, no logout/login needed.
- **Delete Candidate** — added to both Tracker (trash icon per
  card) and Candidate Detail (header icon). Permanently removes
  the DB row + child records; the Domain Log entry stays by design.
- **Export Excel crash fixed** — `expo-file-system`'s
  `writeAsStringAsync` was deprecated in the installed version;
  pinned to `expo-file-system/legacy`.
- **Domain-Wide Audit Log (new)** — Settings → Domain Log. Every
  resume analysis across the domain, survives candidate deletion
  since `usage_log` has no foreign key to `candidates`.

---

## v1.13

- **Donate button fixed** — was a raw UPI intent with a placeholder
  VPA. Now a proper modal (₹100/500/1000 presets + custom amount)
  routed through the same Razorpay flow as every other purchase.
  Donations are recorded as their own transaction type, no credits
  granted.
- **Project Tracker** — new 3-sheet Excel (Roadmap, Completed
  Features, Queued Features) to track build progress going forward.

---

## v1.12

- Rename + Delete team (Manage Teams screen)
- Transaction History screen
- Edit Profile screen
- Current Plan row now opens Upgrade Plan screen
- 2FA / Notifications rows show an honest "Coming Soon" toast
  instead of doing nothing
- Privacy Policy / Terms of Service — full legal content written
  (kept separately, see repo-assets — not bundled into this zip)
- Fixed `Invite Members` silently swallowing its real error message

---

## v1.11

### ⚠️ 2FA + Google/Outlook OAuth — Deferred
Dropped from this version — plain email + password remains the
only login method. Revisit in a later version if needed.

### 1. Domain Email Restriction
`domains.email_domain` (e.g. `sterlingsoftware.co.in`) — set by
the Primary/Sub Admin. Enforced server-side in `send-invite`: any
invited email that doesn't end with `@<email_domain>` is rejected
before an account is even created.

### 2. Corporate Invites — Temp Password Model (replaces invite links)
`send-invite` now creates the auth user AND the approved
`user_master` row immediately — no accept-link, no waiting. The
Welcome email carries the temp password. New Change Password
screen (Settings) lets anyone set their own password afterward.
The old `(auth)/invite/[token].tsx` accept-link screen is removed.

### 3. Annual Domain License — ₹25,000/year, 15 Seats
New Domain License screen (Settings → Domain Management):
- SPOC/Primary Admin purchases via Razorpay (price/credits looked
  up server-side)
- On success: all existing approved domain users (up to 15) get
  1,000 credits + `plan='pro'`; remainder sits in
  `domains.domain_pool_balance`
- **Known gap:** manually reallocating a freed seat's unused
  credits back to a new seat isn't built yet
- Seat cap enforced in `send-invite` (blocks the 16th invite)

**Grace period:** 5 days past `subscription_end_date` before seats
drop to Free tier. Enforced lazily per-user in `analyse-resume`,
plus a domain-wide sweep + 3-day renewal reminder via
`check-license-expiry` — needs external daily scheduling.

### 4. Primary Admin + Sub Admin
New Domain Admins screen. Primary Admin can assign a Sub Admin and
swap the two roles — useful for handover without needing Super
Admin intervention.

---

## v1.10

### 1. Free Tier: 5 Scans/Month (was 10, cumulative-forever)
### 2. No More File Storage — Text Extraction Instead
**Major architectural change.** PDFs/Word docs are text-extracted
server-side (`unpdf` for PDFs, `JSZip` + XML stripping for
`.docx`), then sent as plain text to the AI — works with any free
model now, not just vision-capable ones. **No file is ever saved
to Storage**. Upload only accepts PDF and modern Word (.docx).
### 3. Interview Email with Optional Teams Link
### 4. Editable Fields Expanded (Tech Stack, Role, Source)
### 5. FAQ Screen
### 6. Export All Data (Pro) — needs `npx expo install expo-sharing`

---

## v1.5

- Models read live from the `model_config` database table instead
  of a hardcoded file — fixing a broken model is now "edit a table
  row," no app update needed. New "Sync Free Models" button pulls
  OpenRouter's live catalog.
- Session bootstrap fix — cold start never saved the logged-in
  user's profile into the global store, causing "invalid uuid:
  undefined" errors throughout the app.
- Real error messages from Edge Functions — Supabase's client
  library hides the actual error behind a generic wrapper;
  `analysisService.js` now unwraps it.
- Brevo swap (was Resend), `team_invite` template merged in.

---

## v1.4

### Razorpay Payments
`upgrade-plan.tsx`, `topup.tsx`, `paymentService.js`,
`create-razorpay-order` / `verify-razorpay-payment`. Prices/credits
always looked up server-side, never trusted from the app.

**⚠️ Requires a development build — does not work in plain Expo
Go.** `eas build --profile development --platform android`

---

## Earlier / Foundational (pre-v1.4, not individually versioned)
- Core auth: signup, login, OTP, domain-code join flow, pending
  approval
- Teams: creation, tech stack config, member invites
- Core screens: Upload, Tracker, Candidate Detail, Analytics,
  Settings
- Candidate reassignment, notes, activity timeline
