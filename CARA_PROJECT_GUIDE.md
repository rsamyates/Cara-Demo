# Cara — Project Guide

**This is the single source of truth for Cara's design, voice, and roadmap.**
Read this first in any new session before making changes. Update it whenever a decision is made, a feature ships, or something important is learned.

Last updated: June 20, 2026 (Session 6 — check-in logging reliability fix)

**⚠️ ACTIVE SITUATION:** Cara's backend (Sheet, Drive photos, Apps Script) is currently running on Sam's personal Google account as a workaround while ABLE's Google Workspace renewal is pending. See Section 6 for full detail and the restoration checklist. Don't be surprised if URLs/IDs referenced in older parts of this document point to the original ABLE Workspace setup — Section 6 is the current source of truth on this.

---

## 0. Starting a New Session — Read This First

**Paste this prompt at the start of every new Cara session:**

> "We're continuing work on Cara, a residential care accountability app for ABLE (A Broader Living Experience) in San Rafael, CA. I'm uploading the current project guide, index.html, and Code.gs — please read them before we start, especially Section 0 and the active backend migration flagged at the top of the guide. Live app: https://nimble-daifuku-2c6de8.netlify.app/ (temporary — see Section 8 for full deployment status). After July 1, 2026 use: https://monumental-mooncake-072ec3.netlify.app/"

**Then upload:** `CARA_PROJECT_GUIDE.md` + `index.html` + `Code.gs` (if backend work is planned).

**Why upload files?** The GitHub repo is private — Claude cannot fetch files from it directly. Always upload the latest versions from GitHub at the start of each session.

**Before ending any session:** ask Claude to update this guide. Download it and commit to GitHub.

**Critical context for Claude in every session:**
- This is a relational, values-driven care environment. Language should be warm, not clinical or transactional.
- ABLE's mission: "Together, we create a culture of care and belonging."
- Cara's philosophy: "The highest quality of care comes from a quality of presence."
- Staff are overnight DSPs working alone — Cara should feel like a supportive teammate, not a compliance tool.
- When suggesting UI text, character messages, or report language — always run it through the question: does this honor the dignity of both the resident and the caregiver?
- Privacy and HIPAA matter throughout. Resident identifiers are initials only. Photos never touch staff phones.
- Code decisions should favor simplicity and maintainability over cleverness — Sam is building this without a technical co-founder.
- **Backend is temporarily on Sam's personal Google account** (Sheet, Drive photos, Apps Script) due to an ABLE Google Workspace renewal gap. See Section 6 for the full restoration checklist — bring this up if Sam mentions Workspace being resolved, since several config values need updating back.

---

## 1. What Cara Is

Cara is a care accountability companion for overnight direct support staff at ABLE, a 9-resident ICF-DD/H in San Rafael, CA. It exists to make consistent, high-quality overnight care *likely, not just possible* — through structured check-ins, resident-specific clinical guidance, and a warm, encouraging tone.

**Founding philosophy:** "The highest quality of care comes from a quality of presence."

ABLE was founded by parents who wanted more than quality care for bodies — they wanted a warm, caring, welcoming home. Mission: **"Together, we create a culture of care and belonging."**

**Primary users:** Direct Support Professionals (DSPs). Overnight (NOC) is the primary launch use case.

**Administrators:**
- Sam Yates (Director/QIDP) — sam@broaderliving.org / 25deanza@att.net
- Valeriy (Leriy) Shichkov, RN — valeriy.shichkov74@gmail.com
- Alice (House Manager) — wave63alice@gmail.com

Cara is developed by Sam Yates personally (IP owned by LLC) with ABLE as founding pilot partner.

---

## 2. Design System

### 2.1 Colors

| Token | Hex | Usage |
|---|---|---|
| Primary blue | `#4A6CF7` | Headers, primary buttons, progress fill |
| Deep blue | `#1E3A8A` | Welcome screen, report header |
| Background | `#E8EDF8` | App background |
| Gold/accent | `#FFD166` | Points, streaks, "cara." period, philosophy quote |
| Teal (Remi) | `#2A9D8F` | Nurse Remi name |
| Will accent | `#E8A838` | Big Will name |
| Success green | `#10B981` | Yes answers, done states |
| Warning amber | `#F59E0B` | Due-soon status |
| Danger red | `#EF4444` | Overdue status, concern flags |

**Font:** Nunito (400/600/700/800), Google Fonts.

### 2.2 Character Portraits

SVG illustrations. **Source of truth lives in `index.html` as named constants. Always copy directly from the file — never recreate from description or retype by hand, even for mockups/previews.**

**Standard portraits** (`viewBox="0 0 300 360"`), used in dashboard, check-in steps, standard celebration, rushed alert, round-end:

- **`BEA_SVG` — Nana Bea.** Silver hair in a bun with a terracotta scarf and bow detail; circular wire "granny" glasses (slightly high on the nose); forehead/crow's-feet/smile wrinkles; warm peachy-orange blush; light blue scrubs with V-neck collar detail. Color: `#4A6CF7`.
- **`REMI_SVG` — Nurse Remi.** Dark hair in a low bun with a single side braid (teal hair-tie at the tip); visible ear; teal crew-neck scrubs; stethoscope draped on her left side; white badge with red cross pinned to her chest; eyebrows in warm brown `#4A2F1E` (lightened from near-black for visibility against dark hair — see Decisions Log). Color: `#2A9D8F`.
- **`WILL_SVG` — Big Will.** Gold/amber headband with a subtle stripe detail across the forehead; short tight fade with lighter temple shading; goatee (chin only); gold/amber shirt; broad-shoulder silhouette. Color: `#E8A838`.

**Celebration poses** (rare, high-praise moments only — see Section 4.2). Each has its own `viewBox`:

- **`BEA_CELEBRATE_SVG`** (`viewBox="0 0 320 380"`) — hand over heart, eyes closed in soft curved lines, single glistening tear, deep warm smile, flushed cheeks. Two straight even-width arms cross up to meet at the hand below her neck.
- **`REMI_CELEBRATE_SVG`** (`viewBox="0 0 320 400"`) — both arms raised at ~55°, simple circle hands (no fingers), wide excited eyes with raised eyebrows, small open "O" mouth cheering. Braid emerges in one continuous line from the temple.
- **`WILL_CELEBRATE_SVG`** (`viewBox="0 0 320 380"`) — single thumbs-up fist (oval + curved thumb) positioned in front of the chest, colored to match his face skin tone `#8B5E3C`. One eye winks (asymmetric eyebrows), bigger open smile with teeth.

**Usage rule:** celebration poses are reserved for rare milestones (Full Skin Care streak celebration is the only current trigger). The everyday celebration screen (`showStandardCelebration`) intentionally keeps the standard portraits — using the rare poses there would dilute their impact.

### 2.3 Resident Status Emoji (Dashboard)

| Status | Emoji | Threshold |
|---|---|---|
| Ready for first check-in | 🌙 | Before 7pm shift start or no check-in yet today |
| Recently checked | 😴 | >40 min remaining |
| Approaching due | 😕 | 21–40 min remaining |
| Due now | 😟 | 0–20 min remaining |
| Overdue | 😰 | Past due |
| Done this round | ✓ | Check-in submitted |

**7pm clock reset:** After 7pm, if last check-in was before 6:30pm, resident shows "Ready for first check-in" (🌙, soft blue card). After first check-in, 120-min clock starts.

### 2.4 Login Screen

Three-line tagline: "Your care companion / for every shift. / Built for the team at ABLE."
Philosophy quote in gold (#FFD166): "The highest quality of care comes from a quality of presence."

### 2.5 Language Support

Language selector on gate screen: English | Español | Kreyòl (Haitian Creole).
All UI strings live in the `STRINGS` object. `t('key')` helper used throughout. `getBaseline()` returns translated baseline questions at render time.

**Translation status:** Structural translations complete. Character message pools (Nana Bea, Nurse Remi, Big Will) are English-only pending native speaker review from ABLE staff. Kreyòl uses idiomatic phrasing ("Tcheke" for check-in). Spanish reviewed for warmth.

---

## 3. Character Voices

### 3.1 Nana Bea — Heart of the Team
Title: Compassionate Care Keeper 💛. Color: #4A6CF7.
Philosophy: Kristin Neff self-compassion. Founding story of ABLE woven in. Mission statement "Together, we create a culture of care and belonging" appears in her messages. Three lanes: compassion for residents, self-compassion for staff, compassion for teammates.

### 3.2 Nurse Remi — Clinical Guide
Title: Skin Care Champion 🩺. Color: #2A9D8F.
Philosophy: Presence IS good clinical practice. Skin care = quality of life. Moisture + pressure = skin breakdown. Also provides gentle "slow down" coaching for rushed check-ins — affirm the momentum first, then redirect.

### 3.3 Big Will — Strength Coach
Title: Care Warrior 💪. Color: #E8A838.
Philosophy: Physical self-care IS care for residents. Sees and honors the overnight shift explicitly.

### 3.4 Message Pools
Between 10pm–6am (`isNocHours()`), overnight-specific pools used. Global `pick(arr)` selects randomly. ~61 total messages. See `index.html` for full pool contents.

---

## 4. Current App State

Single-file `index.html`, vanilla JS, no build step.

### 4.1 Screens
1. Gate (site code) — includes language selector (English / Español / Kreyòl)
2. Login (staff chips + PIN)
3. Privacy acknowledgment — required before dashboard, every login
4. Dashboard — resident cards sorted by urgency, points/streak, character tabs
5. Check-in — Assess → Care → Position (if applicable) → Photo → Comment
6. Celebration — confetti, points, streak, character affirmation (standard portraits only)
7. Full Skin Care streak celebration overlay (see 4.2a — night sky + fireworks + celebration poses)
8. Rushed check-in alert overlay (Nurse Remi)
9. Round end

### 4.2 Check-in Flow

**Baseline assessment (all residents):**
- Breathing normally? (Yes / Concern + chips)
- Any signs of distress? (Yes / Concern + chips)
- Resident/bed dry? (Dry / Needs change) — only if `inco: true`

**Photos:**
- Required: one photo (blocks submit until captured)
- Optional: up to 9 additional photos (Photo 2–10), each a real clickable Drive link in the Sheet
- All photos via Apps Script to Drive — never on staff phone

**Optional comment:** "Staff Observation" field at bottom. Free text, optional, sent to Sheet and surfaced in report OBSERVATIONS section.

**Rushed check-in detection:** Under 2.5 min (or 5 min if change needed) triggers Nurse Remi popup before celebration. Also flagged in daily report PACE NOTE section.

**Submit button:** locked until all baseline questions + care tasks + position (if applicable) + required photo complete.

### 4.2a Full Skin Care Streak Celebration (visual detail)

Triggers at 3+ consecutive Full Skin Care check-ins within a single shift session (`showSkinStreakCelebration()` in `index.html`).

**Backdrop:** `.overlay.night-sky` — radial navy gradient (`#0B1F4D` → `#061230` → `#02060F`), replacing the standard dark overlay. ~40 twinkling stars (`.fw-star`, random position, opacity pulse via `fwTwinkle` keyframe).

**Cast build-up:**
| Streak count | Characters shown | Title badges |
|---|---|---|
| 3 | Big Will only | Care Warrior 💪 |
| 4 | + Nurse Remi | + Skin Care Champion 🩺 |
| 5+ | + Nana Bea | + Compassionate Care Keeper 💛 |

Each card uses the character's **celebration pose** (not standard portrait) at 96px width, on a warm champagne-gold background (`#FFF4DC`, border `#F2DDA8`). Cards show only the title badge (16px, bold) and a randomly-picked message — **character names are intentionally omitted** so the badge reads as recognition of the staff member, not introduction of the character.

Title text: "[N] Full Skin Care<br>checks in a row!" (line break keeps "checks in a row!" together). Subtitle: "[Resident] just received your full attention."

**Fireworks (`spawnFireworks()`):** two layers —
- Background layer (`#fireworks-container`, z-index 99, behind cards): 13 bursts of 26 particles each, sequenced top-to-bottom across the full screen height (y: 8%–92%) so the show isn't confined to the top.
- Foreground layer (`#fireworks-container-front`, z-index 101, **above the overlay and cards**): 5 lighter bursts of 14 particles each, smaller radius, slight opacity cap, so sparks visibly cross over the card content without obscuring text for long.

Both layers clear (`innerHTML = ''`) when "Keep going" is tapped, before routing to `showStandardCelebration()`.

### 4.3 Staff & Security
- Site code: `25deanza`
- Staff roster (in `STAFF` array):
  - Joi G. — 0188 *(overnight)*
  - Monyette J. — 9271 *(overnight)*
  - Alice W. — 0179 *(House Manager)*
  - Wilkens M. — 0259 *(pre-overnight)*
  - Lory M. — 0207 *(pre-overnight)*
  - Donly H. — 0275 *(pre-overnight)*
  - Jessalyn A. — 0270 *(pre-overnight)*
  - Other — 8087 *(catch-all)*

### 4.4 Privacy Screen
Shown after PIN entry, before dashboard, every login. Warm but clear confidentiality commitment. "I agree to keep this information private" button required to proceed. Translates with language selector.

### 4.5 State Management
`localStorage` for points/streak (per-device). In-memory `session` object for current shift. Cross-shift timing via `doGet` endpoint (see Section 6).

---

## 5. Resident Configuration

| Initials | Care Level | Repositioning | Special Notes |
|---|---|---|---|
| YC | Basic + Targeted | No | Toes, under breasts, heels/ankles; Nystatin; diaper tab check |
| TH | Basic Check-In | No | Kitchen safety / pica — Nurse Remi rotating reminder |
| MK | Full Skin Care | Yes | Mittens, heel protectors, pad placement |
| CJ | Full Skin Care | Yes | Wedge pillow, facial rash |
| AM | Basic Check-In | No | Lotion/scalp — daytime only (`nocTasks` + `nocRemi` in app) |
| SR | Basic + Targeted | No | Under breasts |
| BH | Full Skin Care | Yes | Bunions, poor circulation, eye history |
| KG | Full Skin Care | Yes | NEVER extend knees; armpit cushions |

---

## 6. Data Flow & Backend

**⚠️ TEMPORARY BACKEND MIGRATION (as of June 19, 2026) — READ THIS FIRST**

ABLE's Google Workspace trial ended and renewal was not approved, taking the original "Cara Check-In Log" Sheet, "Cara/Resident Photos/" Drive folder, and "Cara Check-In Backend" Apps Script offline. **As a workaround, Cara is currently running on Sam's personal Google account** — a new Sheet, new Drive folder, and new Apps Script project (newly deployed, not the original).

**This is intentionally temporary.** The goal is to restore ABLE's Google Workspace and migrate back as soon as it's approved/resolved. **Do not treat the personal-account setup as permanent** — when Workspace is restored:
1. Recreate (or reactivate) the Sheet, Drive folder, and Apps Script under the ABLE Workspace account
2. Update `PHOTO_FOLDER_ID` in Code.gs to the restored Drive folder
3. Redeploy as Web App, get the new `/exec` URL
4. Update `LOG_ENDPOINT` in `index.html` to the restored URL
5. Re-run `createDailyTrigger()` on the restored script
6. Confirm the report once again sends from `sam@broaderliving.org` (not personal Gmail)
7. Update this section and the Decisions Log to mark the migration back as complete

**Current (temporary) configuration:**
- Sheet: new personal Google Sheet (not the original "Cara Check-In Log")
- Photo storage: new personal Drive folder — `PHOTO_FOLDER_ID` updated in Code.gs accordingly
- Apps Script: new project bound to the personal Sheet, deployed fresh (this is Deployment Version 2+ on this new project — unrelated to version history on the original script)
- `LOG_ENDPOINT` in `index.html`: updated to the new deployment's `/exec` URL — **current value:** `https://script.google.com/macros/s/AKfycbzi1naQxPFaptdjgQx5RjHivP5695FxxyNnfI9_Jbd4PuNKPxoR0NihAmBAUN25aDIT2Q/exec` (recorded here so this value survives even if `index.html` is ever lost or rolled back; always treat the live `index.html` as authoritative if the two ever disagree)
- **Daily report now sends from Sam's personal Gmail address, not `sam@broaderliving.org`** — Leriy and Alice were given a heads-up so this doesn't look unfamiliar
- `REPORT_RECIPIENTS` array unchanged (still Sam, Leriy, Alice — those are just email addresses, unaffected by which account hosts the script)
- Deployment access setting: "Execute as: Me" / "Who has access: Anyone" — this is correct and intentional (see rationale below), not a security gap, since the real access control is the app's own site-code + PIN login layer

**Apps Script:** "Cara Check-In Backend" — bound to the (currently personal) Sheet.

**Functions:**
- `doGet(e)` — returns last check-in timestamp per resident (24-hr lookback) as JSON/JSONP. Called by app on login for cross-shift timing.
- `doPost(e)` — receives check-in JSON, writes to resident tab, saves photos to Drive
- `savePhoto()` — decodes base64, saves to `Cara/Resident Photos/` in Drive, returns URL
- `sendShiftReport()` — HTML email report, sent to 3 recipients
- `createDailyTrigger()` — run once to install 10am daily trigger (already installed)
- `fmt(date)` — time formatter helper

**Sheet schema (current — 15 columns per resident tab):**
`Timestamp | Staff | Care Level | Round | Breathing | Distress | Incontinence | Concern Details | Care Tasks Completed | Position | Staff Observation | Time since last check-in | Points Earned | Photo 1 | Photo 2 ... Photo 10`

**Notes on schema:** existing tabs were manually updated from old 12-col layout. The report reader is schema-aware (finds columns by header name). Column names that matter to the reader: "Staff Observation" (or fallback "Comment"), "Time since last check-in" (or fallback "Elapsed Since Prev (sec)"), "Photo 1" (or fallback "Photo Link").

**Cross-shift timing:** On login, app injects a `<script>` tag calling `LOG_ENDPOINT?callback=caraTimingCb_[timestamp]`. Apps Script `doGet` responds with JSONP containing last check-in time per resident. App updates `session.lastCheck[]` when response arrives. 8-second timeout; falls back to current time if Apps Script unreachable.

**Resolved (Session 6, June 20, 2026 — extended same day after live testing):** `doPost` previously used `mode: 'no-cors'`, which meant the app fired the request and immediately showed success regardless of whether the write actually reached the Sheet. This caused a real incident — see Session Log — where a staff member's MK check-in appeared done in the app but never landed in the Sheet, producing a false coverage-gap flag in the 6/20 morning report.

The first pass (confirmed write + 3 quick retries + dismissible banner) only solved *detection*, not *recovery* — a banner telling you data was lost isn't the same as not losing it. Sam flagged this directly after live-testing in airplane mode. The fix now has two layers:
1. **Background retry (in-memory only, full payload incl. photo):** keeps retrying every 45s for up to 20 minutes, plus immediately on the browser's `online` event. Self-heals a brief outage with no staff action. Never written to disk.
2. **Persistent queue (localStorage, text fields only — no photo):** survives the app closing or the phone restarting, and is flushed automatically on next login. Guarantees the clinical record (breathing/distress/incontinence/position/observations) isn't lost even if the in-memory retry never gets to run. The photo is deliberately excluded from this queue — Sam raised the concern that persisting resident photo data to phone storage would cut against the "photos never touch staff phones" rule and be a real exposure if the phone were compromised, and that's the right call. If the in-memory retry can't get the photo through before giving up, the banner says so explicitly, because the right fallback for a missing photo is a human re-checking the resident, not a software workaround.

`doPost` also now logs resident/staff/outcome on every call (it previously had zero logging), so a future incident can be diagnosed directly from the Executions panel. **Live-tested in airplane mode by Sam — confirmed the banner appears and dismisses correctly.**

**Round 2 finding:** the background retry (Layer 1) didn't actually fire on reconnect. Root cause: toggling airplane mode requires leaving the browser for the OS Settings app, which backgrounds the Cara tab — and mobile browsers (especially iOS Safari) throttle or fully pause JS timers in backgrounded tabs to save battery, so the 45s retry interval got no CPU time to run even after reconnecting. **Fix:** added a `visibilitychange` listener that triggers an immediate flush of both retry layers the moment the tab is foregrounded again (switching back from Settings, unlocking the phone), which doesn't depend on a backgrounded timer at all. Also fixed the `online` event handler, which was only flushing Layer 1 (in-memory) and not Layer 2 (persisted queue) — now flushes both.

**Worth knowing for context:** in a realistic overnight dead-zone (weak WiFi in a particular room), the Cara tab typically stays in the foreground the whole time — staff aren't leaving the app to toggle a setting, connectivity just silently drops and returns while the page stays open. In that case the original interval/online-event retry should work fine on its own. The airplane-mode test is actually a harder scenario than the common real-world case, because manually toggling it forces backgrounding — but it's a real scenario too (a staff member could background the app for other reasons mid-outage), so it's right that it surfaced this gap. A fully robust fix for arbitrary backgrounding would be a Service Worker with Background Sync — but that API isn't supported in iOS Safari at all (Apple hasn't implemented it), so it wouldn't fully solve this either if staff are on iPhones, and would be a much bigger lift for a partial win. The `visibilitychange` approach is simpler, has solid cross-platform support, and directly covers the failure mode that was actually observed.

**Not yet re-tested** — needs another deploy + airplane-mode round to confirm the visibility-triggered flush actually catches the reconnect.

---

## 7. Shift Report

**Schedule:** Daily at 10am via time-based Apps Script trigger.
**Recipients:** Sam, Leriy, Alice.
**Window:** Since last report (`lastReportTimestamp` in Script Properties).

**Sections:**
1. FLAGS AND CONCERNS — clinical concerns, prolonged wetness (wet after **130+ min** gap), coverage gaps (**2.5+ hr**), unchecked residents, missing photos
2. PACE NOTE — rushed check-ins (amber, coaching tone, not alarm)
3. OBSERVATIONS & NOTES — staff comments verbatim (blue, neutral)
4. RESIDENT SUMMARY — resident, care level, check-ins, positions, wet/changed, concerns, Photo 1 compliance
5. STAFF SUMMARY — check-ins, rounds, photo compliance, concerns, rushed count
6. RECOGNITION — named staff with 100% photo compliance + most check-ins
7. SHIFT OVERVIEW — totals, on-time residents (≤150 min gap, green), rushed count, observations count

**Thresholds:**
- Wetness flag: 130 min gap (10-min grace over the 120-min standard)
- Coverage gap flag: 150 min / 2.5 hours
- On-time category: no gap > 150 min
- Rushed: under 2.5 min standard, under 5 min if change needed

---

## 8. Deployment

**Current live (temporary):** `https://nimble-daifuku-2c6de8.netlify.app/`
Created via netlify.com/drop June 16, 2026 when free tier credits exhausted.

**Original site:** `https://monumental-mooncake-072ec3.netlify.app/`
GitHub auto-deploy reconnected. Resumes July 1, 2026 when credits reset. After July 1: use original URL, retire temporary site, update Section 0 prompt.

**To deploy updates (until July 1):**
1. Apps Script: paste Code.gs → Save → Deploy → Manage deployments → New version → Deploy
2. Netlify: drag index.html to Deploys tab drop zone on nimble-daifuku project
3. GitHub: commit index.html + CARA_PROJECT_GUIDE.md

**GitHub:** `BroaderLiving/Cara-App` (private). Claude cannot read this directly — always upload files at session start.

---

## 9. Roadmap

### Phase 1 — Done ✅
- Complete app: gate, login, privacy screen, dashboard, check-in flow, celebration, round-end
- Design system: blue palette, character portraits, Nunito font
- 6-state resident status (Ready 🌙, 😴😕😟😰✓) with 7pm clock reset
- Two-layer security (site code + PINs)
- Privacy acknowledgment screen (every login, translates with language)
- Language support: English / Español / Kreyòl across all screens
- Resident-specific care protocols for all 8 residents
- Three characters (~61 messages, time-aware overnight vs. daytime)
- AM daytime-only lotion/scalp tasks (`nocTasks` + `nocRemi`)
- Photo → Drive pipeline (10 photo slots per check-in, Photo 1–10 in Sheet)
- Optional "Staff Observation" comment on every check-in
- Rushed check-in alert (Nurse Remi popup + PACE NOTE in report)
- Full Skin Care streak celebration (per-shift: Will@3, +Remi@4, +Bea@5+)
- Cross-shift timing via Apps Script `doGet` + JSONP on login
- Pre-overnight staff (Wilkens, Lory, Donly, Jessalyn) in roster
- Automated 10am shift report with all sections
- Report: 130-min wetness threshold, 2.5-hr coverage gap, on-time category, PACE NOTE, OBSERVATIONS
- Character visual upgrade: detailed standard portraits (granny glasses, braid, stethoscope, badge, headband stripe, goatee) + dedicated celebration poses for all three characters
- Full Skin Care streak celebration: night-sky backdrop, two-layer fireworks (behind + in front of cards), champagne-gold cards, badge-only display (names removed)
- Confirmed check-in logging: `doPost` write confirmed via real response (no longer `no-cors`), with retry + dashboard banner on persistent failure

### Phase 2 — Next (PWA upgrade)
- Push notifications when approaching/overdue (screen off)
- Cross-shift streak persistence (currently per-shift)
- Vibration alerts (possible without full PWA)
- Periodic character pop-ups (~every 20 min)
- Dynamic resident config from Sheet
- Daytime shift support (AM lotion/scalp interval TBD)
- Native speaker review of ES/HT character messages
- Admin UI for resident/staff config
- Badges + Team Wall
- `qBasic` field cleanup in BASELINE
- Confirm `ABLE_App_IP_Ownership_Statement.docx` actually covers Claude-session work product — `Cara_IP_and_Asset_Record.docx` (Section 7) flags that the ownership statement predates several cataloged assets (character SVGs, copy, technical implementation) and recommends explicitly confirming coverage rather than assuming it

### Phase 3 — Scale
- Two independent pilot facilities
- Cara for Families (home care / IHSS)
- See `Cara_Business_Plan.docx`

---

## 10. Decisions Log

| Decision | Choice | Rationale |
|---|---|---|
| App name | Cara | Means "dear" in Italian/Irish |
| Character names | Nana Bea, Nurse Remi, Big Will | Each embodies a distinct support lane |
| Resident identifiers | Initials only | HIPAA mitigation |
| Report schedule | 10am daily | NOC ends ~7am; 10am gives morning team time to settle |
| Incontinence wording | "Resident/bed dry?" universally | Simpler, more human |
| Wetness flag threshold | 130 min (10-min grace) | Report flags urgent, not routine incontinence care |
| Coverage gap threshold | 2.5 hr (150 min) | Tightened from 3hr for earlier intervention |
| 7pm clock reset | Reset if last check-in before 6:30pm | Pre-overnight staff honored; overnight staff start clean |
| Privacy screen | Every login, before dashboard | HIPAA + relational culture; warm but clear |
| Language support | EN / ES / HT (Haitian Creole) | ABLE staff demographics; "Tcheke" for check-in in Kreyòl |
| Photo columns | Photo 1–10 individual columns | Clickable links, not comma-separated; Points Earned before photos |
| Sheet column headers | "Staff Observation", "Time since last check-in" | Sam's rename; Code.gs reader handles both old and new names |
| Rushed check-in threshold | 2.5 min standard, 5 min if change needed | Nurse Remi popup affirms momentum, then redirects |
| Skin streak build-up | Will@3, +Remi@4, +Bea@5+ | Escalating recognition mirrors depth of commitment |
| Nana Bea streak title | Compassionate Care Keeper | Parallels Will (Warrior) and Remi (Champion) |
| Cross-shift timing | doGet JSONP on login | No CORS issues; graceful fallback; pre-overnight clock inherited |
| Netlify deployment | Manual drag-and-drop | Credits exhausted; GitHub auto-deploy resumes July 1 |
| Photo storage | Drive via Apps Script | HIPAA; never on staff phone |
| Push notifications | PWA (Phase 2) | Requires Service Worker + manifest.json |
| Character visual detail | Full redesign per reference doc (granny glasses, braid+badge, headband stripe, etc.) | Replaces simpler original portraits; verified element-by-element match against source |
| Celebration poses | Dedicated SVGs, reserved for rare milestones only | Standard celebration screen keeps standard portraits intentionally — scarcity preserves impact |
| Remi eyebrow color | `#4A2F1E` (lightened from `#2A1008`), stroke-width 3.5 | Original near-black eyebrows blended into near-black hair at small display sizes |
| Streak celebration backdrop | Night sky gradient replaces cream card | More celebratory; cards become champagne-gold floating on navy |
| Streak celebration character names | Removed, badge-only | Badge reads as recognition of the staff member, not character introduction |
| Streak celebration card background | Champagne gold `#FFF4DC` | Festive, doesn't clash with Will/Remi/Bea accent colors |
| Fireworks layering | Two containers (z-index 99 behind, 101 in front) | Lets sparks visibly cross the cards without permanently obscuring text |
| Character portrait size in streak cards | 96px (50% increase from 64px) | Art is the visual centerpiece of the celebration |
| Title badge size in streak cards | 16px (settled after testing 12px → 18px → 16px) | Stands out without overwhelming the card |
| Backend host (temporary) | Sam's personal Google account | ABLE's Google Workspace trial ended, renewal not yet approved; **intended to be reverted to ABLE Workspace once restored — see Section 6 migration-back checklist** |
| Apps Script "Who has access" | Anyone | Required for the app's JSONP/fetch calls to reach the script from a phone browser with no Google sign-in; real access control is the app's own site-code + PIN layer, not this setting |
| Check-in logging confirmation | Switched from `no-cors` fire-and-forget to awaited fetch + JSON response check + retry (3 attempts, backoff) + two-layer recovery (in-memory background retry with photo, persistent text-only queue without photo) + dashboard banner | `no-cors` made backend write failures invisible to both app and staff — root cause of a false MK coverage-gap flag in the 6/20 report. First pass (banner only) solved detection, not recovery — Sam caught this in live testing and asked for an actual fix. Photo deliberately excluded from persistent storage to avoid putting resident photo data on phone disk; missing photo falls back to a human re-check, not a software retry. |
| IP ownership documentation | Two-document approach: existing `ABLE_App_IP_Ownership_Statement.docx` + new `Cara_IP_and_Asset_Record.docx` cataloging session-specific assets | Ownership statement predates most current assets; asset record fills the gap without replacing or re-litigating the original statement. **Open item:** confirm with Sam/legal that the original statement's terms actually extend to cover the cataloged assets — not yet verified. |

---

## 11. Session Log

### June 20, 2026 (Session 6 — check-in logging reliability fix)

**Trigger:** Sam reviewed the 6/20 morning report and flagged the MK coverage-gap alert (2:53 AM–5:27 AM) as inaccurate, since Joi G. reported the app showed MK as covered. Sam supplied the raw MK sheet log to verify.

**Investigation:** Walked the actual MK sheet rows against the report logic in `Code.gs`. The timestamps confirm the report's math is correct — there genuinely is no MK row between 2:53:03 AM and 5:27:48 AM (a 154m45s / 155-min gap), so the Sheet itself does not show coverage. The real bug is upstream: `submitCheckin()` in `index.html` posted to `doPost` with `mode: 'no-cors'` and never read the response — the app marks the resident "done" in local session state and shows the celebration screen unconditionally, regardless of whether the write to the Sheet succeeded. Combined with `no-cors` (which prevents the app from ever seeing a server error or failed write), this means a staff member can complete a check-in, see success, and have nothing actually saved — exactly consistent with what Joi G. described, and exactly the "Known limitation" already flagged (but not yet fixed) in Section 6.

**Root cause confirmed, not just theorized:** this explains both halves of the discrepancy — why the report is right (nothing in the Sheet) and why Joi G. is also right that the app told her MK was covered (the app's own state is optimistic/local and never re-verifies against the backend).

**Fix shipped:**
- ✅ `submitCheckin()` now calls a new `logCheckin()` function instead of a bare `no-cors` fetch
- ✅ `logCheckin()` removes `no-cors`, awaits the real response, and parses the JSON `status` field from `doPost`
- ✅ On failure (network error, non-ok status, or bad response), retries up to 2 more times with backoff (3s, 6s)
- ✅ If still failing after 3 total attempts, records a lightweight failure record (resident, staff, timestamp — no photo data) in `localStorage['cara_sync_failures']`
- ✅ New dismissible banner (`#syncBanner`) on the dashboard surfaces any unsynced check-ins in real time, telling staff to flag it to the House Manager and check the Sheet
- ✅ Celebration screen still shows immediately in all cases — the fix adds a safety net without slowing down the check-in flow at 3am

**Forensic follow-up — Apps Script Executions review:** Sam pulled the Executions panel for the Apps Script project covering the gap window. Mapping each `doPost` execution's end time (start + duration, since `new Date()` is called near the end of the function, after photo save) against the report's chronological check-in list: 8 of the 10 `doPost` calls inside the 2:53–5:27 AM window line up cleanly with other residents' logged check-ins (SR, YC, BH, CJ, KG, AM). Two calls — 3:48:47 AM and 5:23:08 AM — don't match any entry in the rushed-checkin excerpt and can't be confidently attributed to a resident after the fact.

**Important nuance:** Apps Script's "Completed" status only means the function returned without an *uncaught* exception — `doPost`'s own `catch` block swallows errors and returns a JSON error response instead of re-throwing, so a caught failure (e.g., a bad photo upload) still shows as "Completed" in the Executions list. Combined with the fact that `Code.gs` had **zero logging** in `doPost` (no `Logger.log` calls at all, success or failure), there was no way to retroactively determine which resident those two ambiguous executions were for, or whether they errored internally. So the executions log neither confirms nor rules out a caught, silent MK failure in that window — it just narrows it to two candidate timestamps.

**Fix shipped (Code.gs):**
- ✅ Added `Logger.log('doPost START resident=... staff=... round=...')` at the top of `doPost`, before any work happens
- ✅ Added `Logger.log('doPost OK resident=...')` on successful write
- ✅ Added `Logger.log('doPost ERROR resident=... staff=... error=...')` in the catch block, using the parsed payload captured before the try body (so it's available even if the failure happens after parsing)
- This doesn't change behavior — it makes every future execution forensically identifiable (resident, staff, success/failure) directly from the Executions panel, closing the exact gap that made this incident unresolvable after the fact

**Live testing (same day, after deploy correction):**
- First deploy attempt put the new `index.html` on the wrong Netlify site — corrected.
- Confirmed working: successful check-in writes to the Sheet normally; no false-positive banner on success.
- Airplane-mode test #1 (before the deploy correction): banner did not appear — expected, since the old `index.html` was still live on the actual site at that point, not a code bug.
- Airplane-mode test #2 (after deploy correction): banner appeared correctly and dismissed correctly with the ✕.
- Sam then pushed back on scope: a dismissible banner tells you data was lost, it doesn't recover it. That's a fair read of the original fix — it solved detection, not recovery. Built the two-layer recovery system described above in response (background retry + persistent text-only queue).
- **Round 2 test:** Sam retested airplane mode → banner appeared correctly again → turned airplane mode back off → check-in never sent → signed out → dashboard confirmed the check-in genuinely never made it through. Root cause: backgrounding the tab (required to reach OS Settings to toggle airplane mode) throttles/pauses the in-page retry timer. Fixed with a `visibilitychange` listener — see Section 6 for detail. Also fixed `online` event to flush both recovery layers, not just one.

**Still pending:**
- **Redeploy `index.html`** with the `visibilitychange` fix — not yet re-tested
- **Suggested retest:** airplane mode → check-in → banner appears → turn airplane mode off *but stay in the OS Settings app* for ~10s before switching back to the Cara tab → switching back to the tab should immediately trigger the flush and clear the banner within a couple seconds, without waiting for the 45s interval
- Migrate back to ABLE Google Workspace once restored (see Section 6 checklist) — **still a workaround as of this session, waiting on Workspace approval, not yet resolved**
- Native speaker review of ES/HT translations
- AM lotion/scalp daytime interval TBD
- All Phase 2 items (PWA, push notifications, cross-shift streak persistence, IP ownership statement confirmation, etc.)

### June 19, 2026 (Session 5 — backend migration to personal account)
**Trigger:** ABLE's Google Workspace trial ended; renewal not yet approved. This took the original Sheet, Drive photo folder, and Apps Script offline mid-operation, breaking check-in logging and the daily report.

**This is a temporary workaround, not a permanent architecture change.** Full restoration checklist is in Section 6 — to be run as soon as ABLE Workspace access is resolved.

**Migrated:**
- ✅ New personal Google Sheet created, receiving check-in data correctly (verified via live test)
- ✅ New personal Drive folder created for photos; `PHOTO_FOLDER_ID` updated in Code.gs; verified photo links open correctly
- ✅ New Apps Script project (bound to the new Sheet) — pasted current Code.gs, deployed as Web App ("Execute as: Me", "Who has access: Anyone")
- ✅ `LOG_ENDPOINT` in `index.html` updated to the new deployment URL (went through two URLs during setup — first deployment attempt vs. the actual "New deployment" version; second one was correct and is now live)
- ✅ `createDailyTrigger()` re-run on new script — 10am `sendShiftReport` trigger confirmed installed
- ✅ Cross-shift timing (`doGet`) verified two ways: direct browser test of the bare URL (returned correct JSON), then with `?callback=` param (returned correctly JSONP-wrapped). App-level test (log in as a second staff member shortly after a check-in) confirmed real elapsed time displays correctly, not a 120-min reset.

**Troubleshooting note for future reference:** at one point cross-shift timing appeared broken in the live app even though direct backend tests succeeded. Root cause was not a bug — the test check-ins had been made before 6:30pm, and the 7pm clock-reset logic (`isReadyForFirstCheckin()`) correctly classified them as "before tonight's shift," showing "Ready for first check-in" instead of elapsed time. This is correct behavior, not a defect. Worth remembering before assuming cross-shift timing is broken — check the time of day and the 6:30pm/7pm boundary first.

**Still pending:**
- Migrate back to ABLE Google Workspace once restored (see Section 6 checklist)
- Heads-up sent to Leriy and Alice about the report temporarily arriving from Sam's personal Gmail
- Native speaker review of ES/HT translations
- AM lotion/scalp daytime interval TBD
- All Phase 2 items (PWA, push notifications, cross-shift streak persistence, etc.)

### June 18, 2026 (Session 4 — character visual upgrade)
**Source:** A character reference doc (visual details only — names, philosophy, sample messages from that doc were NOT incorporated, only the SVG visuals) was pasted into chat. All three characters' standard portraits and celebration poses were rebuilt from this source.

**Built:**
- ✅ Replaced `BEA_SVG`, `REMI_SVG`, `WILL_SVG` with fully detailed versions (granny glasses, scarf bow, age lines on Bea; side braid, stethoscope, cross badge, visible ear on Remi; headband stripe, goatee, fade shading on Will)
- ✅ Added three new celebration-pose constants: `BEA_CELEBRATE_SVG`, `REMI_CELEBRATE_SVG`, `WILL_CELEBRATE_SVG` — wired into the Full Skin Care streak celebration only (not the everyday celebration screen, by design)
- ✅ Night-sky backdrop for streak celebration overlay (`.overlay.night-sky`), replacing the original cream/gold card background entirely
- ✅ Two-layer fireworks system — background layer behind cards (13 bursts, full screen height), foreground layer above cards (5 lighter bursts) so sparks cross visibly over the recognition cards
- ✅ Streak cards: champagne-gold background (`#FFF4DC`), character names removed (badge-only), portraits enlarged 50% (64px → 96px), title badges resized 12px → 18px → settled at 16px
- ✅ Title line break fix: "checks in a row!" stays together on its own line
- ✅ Remi's eyebrows recolored lighter (`#4A2F1E`) and thickened (3 → 3.5) for visibility against her dark hair — the one intentional deviation from the literal reference source

**Verification method:** After an earlier review (mid-session) found mockup inconsistencies, did a rigorous element-by-element set-diff between the actual deployed `index.html` SVG constants and the original pasted source for all 6 SVGs. Result: 4 of 6 are byte-perfect matches (Bea standard + celebrate, Will standard + celebrate); 2 of 6 (Remi standard + celebrate) differ only in the intentional eyebrow color/weight fix — no elements missing or dropped. **Lesson for future sessions: when showing mockups/previews of in-app visuals, pull the literal SVG content from the file rather than hand-retyping — hand-transcription is where drift was introduced in mid-session mockups (though never in the actual shipped file).**

**Still pending:**
- Native speaker review of ES/HT translations
- AM lotion/scalp daytime interval TBD
- All Phase 2 items (PWA, push notifications, cross-shift streak persistence, etc.)

### June 18, 2026 (Session 3)
**Report issues diagnosed and fixed:**
- Photos were not showing in report because old sheet tabs had a misaligned column schema (old "Responsive" column + missing Comment/Elapsed columns). Resolved by manually restructuring all 8 resident tabs. Code.gs reader now finds columns by header name with fallbacks for both old and new naming.
- Wetness flag threshold raised from 120 → 130 min (grace period)
- Coverage gap threshold lowered from 3hr → 2.5hr in report

**Built:**
- ✅ Privacy acknowledgment screen — warm but clear confidentiality commitment, every login, translates with language selector
- ✅ 7pm clock reset — residents show "Ready for first check-in" (🌙) until first check-in logged after 6:30pm; honors recent pre-shift check-ins
- ✅ Full language coverage — all app screens now respond to EN/ES/HT selector: dashboard, check-in flow, steps, baseline questions, photo zone, celebration, round-end, rushed alert, streak overlay. Character message pools remain English pending native speaker review.
- ✅ Photo columns 1–10 — each extra photo gets its own clickable Drive link column in the Sheet. Points Earned precedes photo columns. doPost now captures and logs all extra photo links.
- ✅ Staff Observation (comment) column + Time since last check-in column properly aligned in sheet

**Sheet current schema (15 columns):**
`Timestamp | Staff | Care Level | Round | Breathing | Distress | Incontinence | Concern Details | Care Tasks Completed | Position | Staff Observation | Time since last check-in | Points Earned | Photo 1 ... Photo 10`

**Still pending:**
- Native speaker review of ES/HT translations (especially privacy screen and care prompts)
- AM lotion/scalp daytime interval TBD
- All Phase 2 items

### June 17, 2026 (Session 2)
Built: pre-overnight staff, bigger photo button, optional comment field, Nurse Remi rushed-check-in popup, Full Skin Care streak celebration (per-shift), cross-shift timing via doGet + JSONP, report PACE NOTE + OBSERVATIONS sections.

### June 15–16, 2026 (Session 1)
Built complete v1 from scratch. First real overnight shift June 15-16. Netlify credits exhausted; temporary site created.

---

## 12. How To Use This Document

**Starting a session?** Use Section 0 prompt. Upload guide + index.html + Code.gs.

**Made a decision?** Add to Section 10.

**Shipped a feature?** Add to Section 9 Phase 1 ✅ and Session Log.

**Ending a session?** Ask Claude to update this guide → download → commit to GitHub.

**Language and tone reminders for Claude:**
- Cara's voice is warm, relational, and clinically grounded — never bureaucratic
- When writing care prompts: plain language, human dignity, specific to context
- When writing character messages: voice must feel distinctly like that character
- When writing report language: clinical precision, actionable framing, not alarmist
- When suggesting changes: consider impact on staff experience at 3am, alone, working hard
