# RecruitLens — Changelog
TeamExyKings

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
