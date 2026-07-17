# Cara — Project Guide

**This is the single source of truth for Cara's design, voice, and roadmap.**
Read this first in any new session before making changes. Update it whenever a decision is made, a feature ships, or something important is learned.

Last updated: July 16, 2026 (Session 24 — CJ decision-tree text stripped from reports; Section 0 version rule corrected; end-of-session guide habit established)

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
- **Version bumping — read carefully, the rule is asymmetric.** `APP_VERSION` (index.html) and `CURRENT_VERSION` (Code.gs) exist so the backend can spot a phone running a stale cached copy of the app (see Section 11, Session 21).
  - **If `index.html` changes → bump BOTH constants to the same new value.** Miss this and the feature is pointless: nobody ever shows as stale after a real deploy.
  - **If ONLY `Code.gs` changes → bump NOTHING.** The client contract hasn't moved. Bumping `CURRENT_VERSION` alone would flag *every* phone in the building as `STALE_VERSION` and bury the real signal in noise.
  - Date-based values are simplest (`2026-07-15-a`, then `-b` for a second deploy the same day).
  - *(This rule was originally written in Session 21 as "any edit to either file must bump both," which is wrong — corrected Session 24 after a backend-only fix would have tripped it.)*
- **⚠️ ALWAYS start from the latest shipped file, never from an earlier upload in the conversation.** In Session 16 Claude re-read `index.html` from the files uploaded at the *start* of the conversation instead of the latest output, and silently reverted a fix that had shipped one session earlier (the CJ duplicate-card bug, Session 15) — nobody noticed until staff hit it in production days later. Before editing either file, **verify lineage**: check that the version constant and a known recent fix are actually present in the file you're about to edit. Cheap to check, expensive to miss.
- **⚠️ The guide on GitHub may be BEHIND the working copy.** Sam deploys manually, so a guide fetched from the repo can be older than what a session has already produced. Never overwrite newer session work with a fetched copy without checking the `Last updated` header first — same lineage trap as above, different file.
- **📌 END EVERY WORK SESSION by asking Sam to update the project guide before closing.** Established Session 24 at Sam's request. This is the discipline that keeps the guide current and makes every new session fast — a guide that's one session stale is how the Session 16 regression happened. Don't wait to be asked; raise it as the session winds down, and say specifically what needs logging.
- **Photo bytes must never be written to the phone's storage.** Long-standing, deliberate privacy rule — see the comment in `queuePendingCheckin`. Drive URLs are considered acceptable there (a link, not image data); raw base64 is not, ever. Read that comment before touching anything photo-related — Claude nearly violated this in Session 23 by proposing a localStorage photo backup, and caught it only by reading the existing code.

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
  - Belem E. — 0280 *(pre-overnight)*
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
- Photo storage: personal Drive folder — `PHOTO_FOLDER_ID` = `1w4tl67nnrVL0YoPIkgVUTElEpsbEQgK1` (owned by rsamyates@gmail.com, same account running Apps Script — **see Session 8 photo-loss incident below before ever touching this folder or its ID again**)
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

**Confirmed (same session):** Sam redeployed and retested per the suggested protocol above — reconnect now triggers an immediate resend on foregrounding the tab, banner clears correctly, no more reliance on the 45s interval alone. This closes out the MK coverage-gap investigation: detection, in-tab retry, and the backgrounded-tab gap are all confirmed working, not just theorized.

**⚠️ Incident (Session 8, June 30, 2026): Drive photo folder deleted, all photos failing silently for ~24 hours**

**What happened:** Sam was clearing up Drive storage (account was at 81%, dropped to 66% after cleanup) and, in the process, deleted the actual `PHOTO_FOLDER_ID` folder — not just photos inside it. Because `savePhoto()` runs *before* the Sheet row is written in `doPost`, every check-in with a photo failed completely at the photo-save step. **Despite this, check-in data still showed up in the Sheet — with blank photo columns — which made the symptom look like "photos aren't saving" rather than "every photo check-in is failing."** That happened because Cara's recovery system (Section 6) has a text-only persistent retry queue with no photo and no expiration, which kept retrying and eventually succeeding once the underlying request stopped erroring in a way the client could parse — landing real data with empty photo slots. This is a legitimate, separate side effect of the retry architecture, not a bug in the retry logic itself.

**Two real problems were tangled together and diagnosed in sequence:**
1. **A genuine concurrency bug** (now fixed): `flushPendingRetries()` and `flushPendingQueue()` in `index.html` fired every backlogged check-in *simultaneously* instead of one at a time. `doPost`'s `LockService.getScriptLock()` only allows one execution at a time; a burst of ~14 simultaneous requests caused most to time out waiting for the lock (10s timeout), throwing an *uncaught* exception that showed as "Failed" in Apps Script's Executions panel with zero diagnostic trail (Logger.log calls only run after the lock is acquired). This was a real bug, was fixed, and is documented below — but it was not the actual cause of the photo loss. It's the kind of bug that could resurface independently in the future and is worth keeping fixed.
2. **The actual root cause**: the photo folder itself no longer existed. `DriveApp.getFolderById(PHOTO_FOLDER_ID)` threw `"No item with the given ID could be found"` on every single photo save attempt. This was only made visible by adding a new Debug Log sheet tab (see below) — Apps Script's Executions panel was too cumbersome to navigate on mobile to get a clear answer quickly.

**Fixed:**
- `PHOTO_FOLDER_ID` updated to a live folder: `1w4tl67nnrVL0YoPIkgVUTElEpsbEQgK1` (confirmed owned by rsamyates@gmail.com, same account running the Apps Script project — ownership match is required since the script saves as "Execute as: Me")
- `flushPendingRetries()` and `flushPendingQueue()` in `index.html` converted from parallel (`forEach` + bare `fetch`) to sequential (`for...of` + `await`) — never more than one outstanding request to `LOG_ENDPOINT` at a time, eliminating the lock-pileup failure mode
- New **"Debug Log" sheet tab**, auto-created by `writeDebugLog()` in `Code.gs`, written on every `doPost` call: timestamp, resident, staff, whether photo data was present, whether it actually saved, outcome (`OK` / `PHOTO_SAVE_ERROR` / `LOCK_TIMEOUT` / `ERROR`), and detail text. Self-trims to the most recent 500 rows. This is the fastest way to diagnose a backend issue going forward — far easier than Apps Script's Executions panel, especially from a phone.
- Lock-wait timeout is now caught explicitly (previously uncaught) and logged to the Debug Log as `LOCK_TIMEOUT` before returning a clean `{status:'error'}` response, instead of silently producing an unparseable response that left the client's retry logic in the dark
- `lock.releaseLock()` now only called if the lock was actually acquired (`gotLock` flag), avoiding a secondary error if `waitLock()` itself had failed

**Operational lesson — read before any future Drive cleanup on the personal account:** the personal Google account hosting Cara's backend (Sam's, while ABLE Workspace renewal is pending) has a shared 15GB quota across Gmail/Drive/Photos, and periodic manual cleanup is expected while this temporary setup persists. **Before deleting anything in Drive on this account, confirm the Cara photo folder (current ID: `1w4tl67nnrVL0YoPIkgVUTElEpsbEQgK1`) is not the target — deleting individual old photos *inside* the folder is fine and was the original intent, but deleting the folder itself silently breaks all photo uploads with no immediate visible symptom** (check-in data continues to flow normally for hours, masking the failure). The new Debug Log tab will catch this immediately going forward if it ever happens again — check it first if photos stop appearing.

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
- Rushed check-in alert (Nurse Remi popup + PACE NOTE in report) — SR, TH, AM exempt; consecutive-pattern only (two rush-eligible check-ins in a row, any resident, shift-wide per staff member), not single-occurrence
- Round-end affirmations: two-pool system — `STRONG_END_MSGS` (incl. Will's "more work than most people do in a day") reserved for full rounds with zero rush triggers; `GENERAL_END_MSGS` for partial or rushed rounds
- Weekly staff performance report (static HTML, print CSS for PDF export): character cards with celebration/standard poses, personal data stats, constructive feedback section, Sam Memoji + staff avatar integration
- Belem E. added to staff roster (pin 0280)
- Full Skin Care streak celebration (per-shift: Will@3, +Remi@4, +Bea@5+)
- Cross-shift timing via Apps Script `doGet` + JSONP on login
- Pre-overnight staff (Wilkens, Lory, Donly, Jessalyn) in roster
- Automated 10am shift report with all sections
- Report: 130-min wetness threshold, 2.5-hr coverage gap, on-time category, PACE NOTE, OBSERVATIONS
- Character visual upgrade: detailed standard portraits (granny glasses, braid, stethoscope, badge, headband stripe, goatee) + dedicated celebration poses for all three characters
- Full Skin Care streak celebration: night-sky backdrop, two-layer fireworks (behind + in front of cards), champagne-gold cards, badge-only display (names removed)
- Confirmed check-in logging: `doPost` write confirmed via real response (no longer `no-cors`), with two-layer recovery (in-memory background retry with photo, persistent text-only queue without photo), `visibilitychange`-triggered resend, and dashboard banner — live-tested and confirmed working, including the backgrounded-tab case
- Photo folder recovery + sequential retry/queue flush (fixes a real photo-loss incident — see Section 6 and Session Log) + Debug Log sheet tab for fast diagnosis without Apps Script Executions panel
- **Translation completion (Session 9)** — full ES/HT coverage: chip labels, all 8 resident Remi tips/tasks/photo notes (canonical English for backend, translated display via `tr()`), position labels, care level labels, all character message pools (Bea/Remi/Will dashboard, opening, celebration, round-end), rushed alert messages, skin streak celebration tags and messages, remaining UI strings (login screen, dashboard progress, round labels, sync banner). Architecture: `tr()`, `trPick()`, `trTemplate()` resolvers with English fallback; canonical English always sent to backend.
- **Care timing logic (Session 9)** — three-tier task system: core tasks (every check-in), `firstCheckTasks` (first check of shift — shift-start skin inspection), `afterThreeTasks` (once after 3am — second skin pass). Implemented for YC, MK, CJ, SR, BH, KG. Remi tip upgrades per tier (`firstCheckRemi`, `afterThreeRemi`). Photo note upgrades on first check (`firstCheckPhotoNote`). AM handled by existing `nocTasks`/`nocRemi` override.
- **Remi post-change reminder (Session 9)** — `REMI_CEL_CHANGE` pool (4 messages, all translated) fires every time a check-in includes "Needs change" — Remi always gets the celebration slot in that case, not left to random pool selection. Messages focus on using the change as a skin inspection and photo opportunity.
- **Companion Moments (Session 10)** — full-screen character pop-up (`s-greeting` screen) that fires mid-shift at 5 moments per session: early greeting (check-ins 1–3, randomized) + milestones at check-ins 10, 22, 34, 40 (each ±3 offset). Character zooms in with spring easing, speech bubble fades up, "Let's go →" returns to dashboard. Random blink loop (2–6s) using skin-tone ellipse lids. Character rotates randomly, never repeating back-to-back. "Other" staff greets as "caregiver."
- **Check-in de-duplication + delayed-sync detection (Session 11)** — every check-in generates a `checkinId` (resident + timestamp + random string) and `capturedAt` (ISO timestamp of the actual tap) at the moment of capture, both riding through every retry path automatically. Backend (`Code.gs`) checks the last 30 rows for a matching `checkinId` before writing — a retry of an already-successful submission gets silently skipped instead of creating a duplicate row. Shift reports now include a "Connectivity Note" section (only rendered when something's flagged) listing any check-in where `Captured At` and the row's arrival time differ by more than 3 minutes — surfacing delayed-sync events without confusing them for late care. Live-tested and confirmed working via a standalone local test tool (`dedup-test.html`, not part of the deployed app) that posts a duplicate `checkinId` directly to the backend.
- **Full Skin Care two required photos (Session 11)** — CJ, MK, BH, KG now get two big, high-contrast required photo prompts instead of one: a red "PHOTO 1: SENSITIVE SKIN AREA" badge (subtext: "Tap to take required photo") and a blue "PHOTO 2: REPOSITIONING (New Position)" badge (subtext: the resident-specific photo note, e.g. "Bottom — shift end and each repositioning round."). Submit stays disabled until both are captured; optional extra photos unlock only after both required ones are in. All other care levels (YC, TH, AM, SR) keep the original single-required-photo flow, untouched. No `Code.gs` changes needed — the repositioning photo is sent as `extraPhotos[0]`, landing in the existing "Photo 2" spreadsheet column automatically (Photo 1 = skin check, Photo 2 = repositioning, Photo 3+ = optional). Fully translated EN/ES/HT.
  **Addendum (Session 12):** BH temporarily exempted from the skin-photo requirement — repositioning photo only, for now, per Sam. Controlled by a single helper, `skinPhotoRequired(r)` (`r.level === 'Full Skin Care' && r.id !== 'BH'`), used consistently across the render, `checkReady()`, `gotPhoto()`'s post-capture branch, and `addExtraPhoto()`'s slot numbering — reversible by editing one line. BH's single required photo now behaves like a normal single-photo resident under the hood (fills `ck.photo`/`ck.photoData`, lands in the sheet's Photo 1 column) but keeps the bold FSC-style badge, relabeled "PHOTO 1: REPOSITIONING (New Position)."
- **Repositioning "Last check-in" note (Session 12 design, Session 13 build + bugfix)** — a light gray chip above the position buttons on Step 3 showing the resident's last recorded position, the clock time, and a relative "how long ago." Backend (`doGet`) scans each resident's last 50 rows for the most recent non-empty Position value, preferring `capturedAt` over arrival time. **Bugfix:** the chip was only ever populated once, at login — repositioning a resident multiple times mid-shift never updated it, so it kept showing stale, login-time data (caught by Sam testing MK). Fixed by updating `session.lastPosition` locally, immediately, the moment any check-in with a position is submitted — no longer waiting on a server round-trip.
- **On-time streak feature — Tonight's Streak + Perfect Shift Streak (designed Session 12, built and completed Session 13)** — two separate, confirmed mechanics:
  - **Tonight's Streak** (session-scoped, drives celebration screens): per-resident, starts counting at the 2nd consecutive on-time check-in of the shift (≤120 min since the resident's last check), resets to 0 on any late one. Full-screen celebration fires every time the streak reaches 2+, featuring each companion's real celebration pose (`WILL_CELEBRATE_SVG`/`REMI_CELEBRATE_SVG`/`BEA_CELEBRATE_SVG` — arms raised, thumbs up, tears of joy respectively — not the calm everyday portrait), a northern-lights aurora backdrop, an ambient sparkle layer drifting behind everything for depth, and each companion's signature multiplied element (Bea = hearts, Will = balloons, Remi = bubbles) — one per point in the streak, scattered across the whole screen (not confined to a row), each popping in then continuing to float/wobble independently. Character motion uses a punchy combined bounce+sway+scale-pulse animation (`streakCelebrate` keyframe), distinctly more energetic than the calm Companion Moments zoom-in. Confetti was deliberately avoided — already the signature of the standard check-in completion celebration.
  - **Perfect Shift Streak** (persists indefinitely, tied to a specific caregiver + resident pair): a shift counts as "perfect" for that pair if every check-in that caregiver made for that resident during that shift was on-time, with a "gimme" on the first check-in of any new shift (so the inevitable gap between shifts never falsely breaks it). Scoped entirely to each caregiver's own shifts — a different caregiver checking the same resident on a different night never affects another caregiver's number, which is what makes it fair across a rotating team. Personal best is tracked permanently and never decreases, even after a reset. Backend: new `Perfect Shift Streaks` sheet tab (`Staff, Resident, Current Streak, Personal Best, Last Updated`), written via a `recordShiftEnd` POST action fired at every sign-out (`index.html`'s `logout()`/`recordShiftEnd()`), read back via `doGet`'s `?staff=` parameter at every login. **Displayed** as a small amber "Perfect Shift Streak: N — Personal best: M" chip on the check-in screen (deliberately not the dashboard, which already has an unrelated device-level "🔥" streak — a second, differently-scoped streak number in the same spot would've been confusing) — only shown when there's something to show. **New personal best** gets its own quiet, distinct moment: computed entirely client-side at logout (no server round-trip needed, since the data to compute it is already on hand), shown on a warm dusk-gradient screen, always Bea, briefly delaying sign-out by exactly one screen — never skipped, never rushed.
  - **Important history:** most of this was already substantially built in an earlier, uncaptured part of this session before the "let's pick up where we left off" continuation — Tonight's Streak was fully working; Perfect Shift Streak's frontend (tracking, sending, fetching) was fully built, but `Code.gs` had zero handling for any of it, meaning every sign-out had been silently sending data into a backend with no route for it. Confirmed via Sam pasting the actual live `Code.gs` mid-session. The missing backend (`handleRecordShiftEnd`, `getStreakSheet`, `findStreakRow`, `doGet`'s `perfectStreaks` section) was completed and shipped this session.
- **Report bugfix: `onTimeResidents` (Session 13)** — found during a routine confirmation pass that Session 12's `capturedAt`-preference fix (delayed-sync correction) had been applied to `coverageGaps` and `prolongedWetness` but missed a third, adjacent calculation — the Shift Overview's "Residents with consistent on-time check-ins" list was still using raw arrival `Timestamp` only. Fixed to match the other two.
- **Backend security hardening (Session 13)** — the two items flagged as recommended-but-not-built in Session 12, now shipped:
  - **Shared secret (`apiToken`)** — a matching constant in both `index.html` and `Code.gs`, checked first thing in `doPost` on every action (regular check-ins and `recordShiftEnd` alike) before any data is touched. A mismatched or missing token is rejected and logged to Debug Log as `UNAUTHORIZED`. Closes the exact hole the Session 11 de-dup test tool demonstrated — writing to the sheet with zero credentials. **Important:** both files must be updated together if this is ever rotated, or the live app starts getting rejected by its own backend.
  - **Resident-ID whitelist** — any check-in for a `resident` value not in the actual `RESIDENTS` list is rejected and logged as `UNKNOWN_RESIDENT`, closing the same kind of hole that let the de-dup test tool spawn a `ZZ_TEST_DEDUP` sheet tab on purpose.
  - **Scope note:** this only covers `doPost` (writing). `doGet` (reading cross-shift timing, positions, streak data) remains unauthenticated — a smaller, different category of exposure, deliberately left out of this pass since protecting it would mean putting a secret in a URL (weaker, shows up in logs) rather than a POST body. Flagged, not yet decided whether it's worth doing.

### Phase 2 — Next (PWA upgrade)
- Push notifications when approaching/overdue (screen off)
- Cross-shift streak persistence (currently per-shift)
- Vibration alerts (possible without full PWA)
- Periodic character pop-ups (~every 20 min)
- **In-app sync reset button** — clear stuck retry queue without navigating Safari settings. Agreed design: escalates from existing sync banner after ~10 min of persistent failure. Low priority — not yet built.
- **Dynamic resident config from Google Sheet** — defer until ABLE Workspace restored. `Cara_Resident_Setup_v2.xlsx` Night + Day tabs finalized. Wire up once Workspace is stable.
- **Daytime care** — `Cara_Resident_Setup_v2.xlsx` Day tab captures all daytime protocols for Phase 2 (AM lotion/scalp, SR/BH/KG shower-linked checks, YC toes/breasts daytime timing).
- **Native speaker review of ES/HT translations** — Spanish is a confident first pass. Haitian Creole needs Leriy or another fluent speaker to review clinical content. Corrections fed back into `RESIDENT_TEXT` and `STRINGS` in `index.html`.
- Daytime shift support (full Phase 2 feature)
- Admin UI for resident/staff config
- Badges + Team Wall
- `qBasic` field cleanup in BASELINE
- Confirm `ABLE_App_IP_Ownership_Statement.docx` covers Claude-session work product

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
| Rushed care flag — resident exemption | SR, TH, AM exempt from rushed alerts and report flags | Care level and nature of these residents (basic wellness, self-repositioning) makes fast check-ins appropriate; flagging them was generating noise |
| Rushed care flag — consecutive pattern | Alert and report flag only fire when two consecutive rush-eligible check-ins (any resident, shift-wide per staff member) are both under threshold | Initially built as per-resident consecutive tracking, then corrected after live testing — Sam tested five different residents back-to-back and nothing fired, which revealed the per-resident design didn't match the intent. Shift-wide tracking catches a pattern of rushing across the round, which is the actual clinical concern. First check-in of a session always has null elapsed time and cannot be the "first" of a pair — in practice, the alert fires on the second consecutive rushed eligible check-in after the session is underway. |
| Round-end affirmation pool split | `STRONG_END_MSGS` for full + rush-free rounds; `GENERAL_END_MSGS` for all others | Will's "more work than most people do in a day" line lost credibility when delivered after a short or rushed round — gating it to actually-earned moments restores Will's credibility |
| Weekly staff performance report | Static HTML with print CSS, character SVGs inline, staff/admin avatars base64-embedded | Same as NOC report format principle — no JavaScript rendering dependency, saveable as PDF via browser print; delivered by email or Netlify Drop link |
| Photo folder recovery | New `PHOTO_FOLDER_ID` (`1w4tl67nnrVL0YoPIkgVUTElEpsbEQgK1`) after the original was deleted during Drive storage cleanup | Original folder ID became invalid, causing every photo-attached check-in to fail at the photo-save step for ~24 hours; confirmed new folder is owned by the same account (rsamyates@gmail.com) running the Apps Script project |
| Retry/queue flush — sequential, not parallel | `flushPendingRetries()` and `flushPendingQueue()` in `index.html` now send one request at a time (`for...of` + `await`), never simultaneously | Parallel flushing of a backlog caused a burst of ~14 simultaneous `doPost` calls, overwhelming the backend's single script lock and causing most to fail with an uncaught, unlogged `LockTimeoutException` |
| Debug Log sheet tab | New auto-created "Debug Log" tab, written on every `doPost` call (resident, staff, photo presence/save status, outcome, detail) | Apps Script's Executions panel is hard to navigate on mobile and doesn't surface custom log lines without digging into each row; a Sheet tab is something Sam can scan directly |
| Translation architecture | `tr()` / `trPick()` / `trTemplate()` resolvers on `{en,es,ht}` objects; canonical English always sent to backend (careLevel, positions, chip values); display-only translation at render time | Backend stability requires stable English values in the Sheet regardless of UI language; translation is a display concern only |
| Care timing — three-tier tasks | Core tasks (every check-in), `firstCheckTasks` (first check of shift), `afterThreeTasks` (once after 3am per resident per session) | Avoids overwhelming staff with skin inspection tasks on every visit while ensuring clinical inspections happen at the right frequency; `isFirstCheck()` uses `session.lastCheck[id]` (set on submit); `isAfterThree()` uses `session.afterThreeShown[id]` (set on open) |
| Remi post-change reminder | `REMI_CEL_CHANGE` pool fires every time a check-in includes "Needs change" — Remi always gets the celebration slot | A change is the highest-risk moment for skin damage and the best natural opportunity for a close look and photo; leaving it to random 25% pool selection undersells the clinical importance |

---

## 11. Session Log

### July 16, 2026 (Session 24 — CJ decision-tree text stripped from reports at read time)

**Version: unchanged at `2026-07-15-b` — `Code.gs` only, no client change.** (This is the case that exposed the Session 21 version rule as wrong; see Section 0.)

**The problem:** Sam reported the CJ decision-tree summary was STILL appearing under
OBSERVATIONS & NOTES in the daily report — e.g. *"Position: Lying down (mittens on,
pillow under head, area checked for hazards). Location: Couch (safe distance from
edge)."* — despite Session 16 removing the client-side generation.

**Why removing it from the client wasn't enough:** the text is permanently baked into
the **Staff Observation column of rows already written to the Sheet** during the
Sessions 14–16 window. `sendShiftReport` reads that column. Stopping the client from
generating new instances did nothing about rows that already exist — and the report
re-reads history every night. Verified the current client is clean (no `dtreeSummary`,
no trace of the phrasing anywhere in `index.html`).

**Fix — strip at read time, not write time.** Added `stripDtreeSummary()` + the
`DTREE_SUMMARY_RE` pattern to `Code.gs`, applied where `allRows` is built, so every
downstream consumer (observations filter, rendering) sees the cleaned value from a
single point. This handles historical rows **and** anything still arriving from a
device on a stale cached build.

**Critical implementation detail — do not "simplify" this later.** The Session 14
generator did `dtreeSummary + ' ' + staffComment`, so a real staff observation can be
sitting *after* the auto-text in the same field. The strip removes **only the generated
prefix** and keeps the remainder. Blanket-deleting any comment starting with
"Position:" would destroy genuine clinical observations from that window. Tested
against all three exact phrasings, auto-text-plus-real-comment, real-comment-only, and
a comment that merely mentions repositioning (correctly untouched).

**What this data actually is** (per Sam, worth recording so nobody "restores" it): it's
purely a care-guidance decision tree — sitting up means fewer care tasks, lying on the
couch more, in bed more still. It tells staff what to do in the moment. It is not
report-worthy and never should have been in the observation stream.

**Live diagnostic worth following up:** one of the flagged rows was **Lory M. at 7/15
8:59 PM** — *after* the `2026-07-15-b` deploy (Sam's test at 5:18 PM proves `-b` was
live). A phone on `-b` cannot produce that text, so her device is almost certainly
**still on a stale cached build** — consistent with her recurring `UNAUTHORIZED`
history (Sessions 18–19). **Check the Debug Log around 8:59 PM for a `STALE_VERSION`
row with her name**; if present, the Session 21 mechanism earned its keep on day one
and points at exactly which device needs attention. (Julia B. 11:13 AM and Belem E.
12:53 PM predate the deploy and are expected.)

**Also this session:**
- **Corrected the Section 0 version-bump rule**, which was wrong as written (Session
  21). It said *any* edit to either file must bump both; in fact a backend-only change
  must bump **nothing**, or every phone falsely flags as `STALE_VERSION`. This fix was
  the case that surfaced it.
- **Added the end-of-session guide-update habit to Section 0**, at Sam's request — as a
  standing rule rather than something one session happens to remember.
- **Added a Section 0 warning that the GitHub guide may lag the working copy.** Sam
  deploys manually; a fetched guide can be older than what a session has produced.
  Same lineage trap as Session 16, different file. (Came up when Sam pointed at the
  repo URL this session and Claude declined to pull it.)
- **Repo URL confirmed unchanged:** `rsamyates/Cara-App` (private, personal account —
  the deliberate IP-separation decision). A `BroaderLiving/Cara-App` link in Sam's
  session-opener was a typo. Live app remains `https://rsamyates.github.io/Cara-App/`.

**Still pending:**
- Check Debug Log for `STALE_VERSION` / Lory M. around 7/15 8:59 PM, and get her device
  onto the current build
- Have Leriy review a real 2400px/0.92 photo of an actual area of concern — the one
  open question on compression
- Confirm Sam is OK with Drive URLs (not bytes) persisting in the localStorage queue,
  or revert per Session 23
- Watch whether the "app kicks me out" reports stop — real-world confirmation of the
  memory theory
- Watch the next several reports to confirm the Session 19 gap-calculation fix
- Continue watching MK's room signal as the extender position is tuned
- The gentle-lateness-encouragement decision (Session 14, awaiting Sam)
- Native speaker review of ES/HT translations
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- `doGet` auth layer decision

### July 15, 2026 (Session 22–23 — staff feedback round; photo-loss root cause found and fixed)

**Version deployed this session: `2026-07-15-b`** (both `APP_VERSION` in index.html and `CURRENT_VERSION` in Code.gs).

---

#### ⚠️ PROCESS FAILURE THAT MUST NOT REPEAT

Sam reported the CJ duplicate wedge-pillow/bedrails card was **back**, despite
being fixed in Session 15. Cause: in Session 16, Claude pulled a "fresh" copy
of `index.html` from the **original files uploaded at the very start of the
conversation** rather than from the latest shipped output. That original still
contained the pre-Session-15 CJ data, so Session 16's edit was built on top of
a stale base and silently reverted the Session 15 fix. Session 16's own fix
(removing the CJ dtree summary) survived, so nothing looked wrong at the time.

**Rule going forward: always start from the latest file in outputs, never from
an earlier upload in the conversation, and verify before editing** (check the
version constant / a known recent fix is present). Claude ran an explicit
file-lineage check mid-session at Sam's prompting and should do this
automatically at the start of any session that touches code.

Re-applied the Session 15 fix: `CJ.tasks` emptied, duplicate lines stripped
from `CJ.firstCheckTasks`. Confirmed live in testing — the submitted test row
shows `Care Tasks Completed: 0/0`, which is correct for CJ now that wedge
pillow and bed rails live only in the decision tree.

---

#### The photo-loss bug — root cause found

**Report:** during daytime care, staff changing CJ in the bathroom captured the
sensitive-skin photo, then there was a delay before repositioning. During that
delay the app reloaded on its own and the photo was lost. Staff did not
refresh or navigate away. Sam wondered whether his deleting/replacing
`index.html` on GitHub mid-shift could have caused it.

**Ruled out:** GitHub deploys cannot affect an already-open tab — there is no
mechanism that pushes updates to a live session. Only the *next* page load
sees a change.

**Actual cause — memory pressure.** Investigation found the app had **no
compression or downscaling anywhere**: `readAsDataURL` held the raw camera
file as base64. Phone photos are 2–5MB raw, ~1.35× as base64 → 3–7MB each.
A Full Skin Care check-in held two required photos plus extras — **10–20MB of
live JS strings sitting in the tab for the entire duration of the check-in.**
Mobile browsers evict tabs under memory pressure, and "idle tab holding tens
of MB" is precisely that profile. This explains **both** reported bugs with one
cause: the lost bathroom photo, and Joi's "app kicks me out and I start over"
reports (Session 18) — both happened during a *pause*, tab idle, photos in
memory.

**Design path (worth recording — the first proposal was wrong):**
Claude initially proposed splitting the check-in into two backend writes
(photo creates a partial row, submit completes it). On review this was
rejected for three concrete reasons:
1. **It collided with the de-dup guard.** Session 11's logic skips any write
   whose `checkinId` already exists — so the completion write would have been
   silently swallowed as a "duplicate" and the client told it succeeded. A
   data-loss bug introduced by the fix.
2. **Partial rows would suppress real coverage-gap flags.** `coverageGaps`,
   `residentSummary`, `onTimeResidents` and photo-compliance all treat "a row
   with a timestamp" as "resident was checked." An abandoned check-in would
   have counted as a completed one — making the report *less* truthful in the
   direction that matters for resident safety.
3. **It tripled lock contention** on a backend already producing real
   `LOCK_TIMEOUT` errors (July 13/14 Debug Log), and completion writes would
   have had to *scan* the sheet, holding the lock longer than an append.

**What was built instead — Drive-first upload.** Each photo now uploads to
Drive the instant it's captured; the client keeps only the short URL and frees
the bytes immediately (`ck.photoData = null`). Memory during a check-in drops
from 10–20MB to a few strings. No partial rows, no de-dup collision, no schema
change, and the row is still written exactly once at submit.

- New `savePhotoOnly` action in `Code.gs`, routed **before `waitLock`** — it
  only touches Drive, never the Sheet, so it deliberately adds **zero** lock
  contention. This was the point: add photo traffic without adding lock traffic.
- `photoSlots` payload replaces `photoBase64`/`extraPhotos`. Each entry is
  `{url}` (uploaded at capture — normal) or `{b64}` (upload failed — bytes ride
  along as fallback). Mixed is expected. Legacy base64 path retained for stale
  clients.
- Slot order is the contract: index 0 → Photo 1, 1 → Photo 2, etc. Assembled by
  **role, not capture time** — so an extra photo taken *before* the required
  ones still lands in Photo 3. Verified by test.
- Failed capture-time upload degrades to exactly the old behavior — never worse.

**Compression — decided WITH Sam, do not change unilaterally.** Sam raised a
well-founded clinical objection: seeing skin closely really matters. Subtle
erythema is a low-contrast gradient and is exactly what JPEG quantization
damages first; the camera file is *already* a JPEG, so re-encoding stacks
generational loss. Settled on **2400px long edge / quality 0.92** (~0.15mm per
pixel on a sacrum photo — far beyond what redness/rash assessment needs, 4–8×
smaller). `compressPhoto()` **skips re-encoding entirely if the image is
already ≤2400px**, avoiding needless generational loss. Constants
`PHOTO_MAX_EDGE` / `PHOTO_QUALITY` sit at the top of the photo pipeline in
`index.html` — raise them or disable compression if quality is ever questioned;
**Drive-first is what fixes the memory bug and works fine at full resolution.**
Open question left with Sam: have Leriy eyeball a real 2400/0.92 photo of an
actual area of concern to confirm clinical detail holds up.

**Other staff-feedback fixes this session:**
- **"Add another photo" now always available** — was gated behind required
  photos being captured first. Staff sometimes need to capture something
  incidental *before* the required shots. Button is visible from screen load.
- **Privacy-screen reload no longer dumps staff back to the access-code
  screen** (a real regression from Session 20's cache-bust reload). The staff
  name is stashed before reload; `resumeAfterFreshLoad()` restores them
  straight to the dashboard. Safe because it's the same login continuing — gate
  and PIN were already passed seconds earlier.

**Privacy decision — flagged to Sam, needs his ruling if he disagrees.** The
persistent (localStorage) queue now retains Drive **URLs** while still
stripping all photo **bytes**. Reasoning: the documented rule is that photo
bytes never touch phone storage; a URL is a link, not image data, and that
queue already persists clinical PHI (breathing, distress, incontinence,
observations) by design — so the line drawn was about image bytes specifically.
Upside: before this, a check-in recovering via the persistent queue always
landed photo-less and tripped a false "missing photo" flag even though the
photo existed. **To revert: drop the `.filter` in `queuePendingCheckin` and
strip `photoSlots` entirely.**

(Note: earlier in the session Claude nearly built a localStorage photo-backup
that would have *directly* violated the no-bytes rule, and caught it only by
reading the existing comment. The rule is load-bearing and easy to trip over —
read `queuePendingCheckin`'s comment before touching anything photo-related.)

**Live test — passed end to end.** Three photos uploaded at capture
(`PHOTO_UPLOADED` slots 1/2/3, all sharing `checkinId=CJ-1784160930344-4i8yax`)
**before the check-in was submitted** — i.e. already safe on Drive while the
check-in didn't yet exist. That is the exact scenario that lost the bathroom
photo. On submit: only **3** files in Drive (no re-upload → URLs were used),
row landed with Photo 1 = skin, Photo 2 = repositioning, Photo 3 = extra,
`Captured At` 3s before `Timestamp`.

**Check-in duration — no column needed after all.** Because `checkinId` is now
generated at *screen open* and `capturedAt` records *submit*, duration is
already derivable from two existing columns (the test check-in computes to
2m 34s). This supersedes the Session 21 plan to add a Duration column — there
is nothing to build in the app; it's report-side math only, if/when wanted.
**Caveat:** rows written before `2026-07-15-b` generated `checkinId` at submit,
so they compute as ~0s — any duration math must ignore anything before
July 15, 2026.
**Values caution carried forward:** log/derive it, but think hard before putting
it in the report. A stopwatch on caregivers is a very different artifact than
the existing gentle, pattern-based pace note. Duration also gets *noisier*
precisely in the delay scenarios that motivated this work (a 25-minute
"duration" for 6 minutes of care).

**Still pending:**
- Have Leriy review a real 2400px/0.92 photo of an actual area of concern —
  the one open question on compression
- Confirm Sam is OK with Drive URLs (not bytes) persisting in the localStorage
  queue, or revert per above
- Watch whether the "app kicks me out" reports stop — that's the real-world
  confirmation the memory theory was right
- Watch the next several reports to confirm the Session 19 gap-calculation fix
  eliminates large/nonsensical gap numbers
- Continue watching MK's room signal as the extender position is tuned
- The gentle-lateness-encouragement decision (three options presented Session
  14, awaiting Sam's decision)
- Native speaker review of ES/HT translations
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- `doGet` auth layer decision

### July 15, 2026 (Session 21 — version-lockout mechanism built)

**Built the full version visibility mechanism, log-but-accept, per Sam's decision**

- `index.html`: added `APP_VERSION = '2026-07-15-a'`, sent in every
  check-in payload and every `recordShiftEnd` payload.
- `Code.gs`: added matching `CURRENT_VERSION`. In `doPost`, right after the
  existing `apiToken` check, a mismatch between `data.appVersion` and
  `CURRENT_VERSION` gets logged to the Debug Log as `STALE_VERSION` — with
  the device's actual sent version and what was expected — but the
  check-in is **never rejected**. Explicit decision (Sam, this session):
  an old-version submission is still real care and must still count;
  this is purely a visibility signal, same spirit as the delayed-sync
  note, not a hard gate like `apiToken`.
- Missing `appVersion` entirely (undefined) — e.g. a client from before
  this feature shipped — is treated the same as a mismatch: logged,
  not blocked.

**What this gives you that Session 20's auto-refresh alone didn't:**
confirmation. The Session 20 fix reduces how often a stale cache persists,
but this is what lets you actually *see* it happening — if a staff
member's name keeps showing up with `STALE_VERSION` in the Debug Log,
that's a clear, specific signal their device needs a real refresh, as
opposed to a wifi/connectivity problem (which would show as no log entry
at all, or a different outcome).

**Critical ongoing requirement, now documented in Section 0:** any future
edit to `index.html` or `Code.gs` must bump BOTH `APP_VERSION` and
`CURRENT_VERSION` to the same new value, together, or the whole mechanism
reports everyone as "stale" forever after a real deploy. Sam asked whether
this happens automatically — it doesn't; it's something whoever edits the
code (Claude, in a session, or Sam directly) needs to remember each time.
Added to Section 0's critical-context list specifically so this isn't
just a code comment easy to miss.

**Still pending:**
- After the next deploy, do one deliberate test: bump one file's version
  but not the other, submit a test check-in, confirm `STALE_VERSION` shows
  up correctly in the Debug Log — verify the mechanism actually works
  before relying on it
- Watch the next several reports to confirm the Session 19 gap-calculation
  fix eliminates large/nonsensical gap numbers going forward
- Get Joi's answers on the mid-check-in app-restart bug (Session 18)
- Continue watching MK's room signal specifically as the extender
  position is tuned
- The gentle-lateness-encouragement decision (three options presented Session
  14, awaiting Sam's decision)
- Native speaker review of ES/HT translations
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- `doGet` auth layer decision

### July 15, 2026 (Session 20 — auto-refresh on privacy-screen agreement)

**Built a one-time-per-session cache-busting reload, triggered on privacy agreement**

Addresses the stale-cache risk surfaced repeatedly this week (Lory M.'s
recurring `apiToken` rejections, Sessions 17–19) with a lightweight,
immediately-shippable fix — separate from and smaller in scope than the
version-lockout mechanism still pending.

**How it works:** in `agreeToPrivacy()` (`index.html`), right after someone
taps "I agree" on the mandatory privacy screen, the app checks a flag in
`sessionStorage`. If not yet set, it reloads the page with a cache-busting
query parameter (a unique timestamp), forcing the browser to fetch the real
current `index.html` from GitHub Pages instead of serving a cached copy —
then sets the flag so it doesn't fire again for that browser tab session.
If the flag's already set, nothing happens and login proceeds straight to
the dashboard as normal.

**Deliberate placement, with a known tradeoff:** this fires after PIN entry
(not at the login screen), per Sam's request. If a reload actually happens,
the in-memory session is wiped, so staff need to re-enter their PIN once
after the page comes back. Accepted as a fair cost for guaranteeing
everyone's on current code before logging care for the night. Noted in-code
that moving this check to the top of `buildLogin()` instead would avoid
that extra PIN entry entirely (nothing typed yet at that point) if it ever
becomes annoying in practice.

**What this does and doesn't solve:**
- Fixes plain browser HTTP caching — the most common and most likely
  explanation for what happened to Lory M.
- Zero cost on logins where the cache was already fresh — only fires once
  per phone/browser session.
- Does NOT help if a bookmark/home-screen icon points at a dead URL
  entirely (already handled separately — Netlify sites deleted, Session on
  Netlify shutdown).
- Does NOT give visibility into who was on a stale version or confirm the
  fix is working — that visibility is still what the version-lockout
  mechanism (logging a version mismatch to the Debug Log) would add.

**Still pending:**
- Build the full version-lockout mechanism (index.html + Code.gs) for
  visibility/confirmation — this session's fix reduces the problem but
  doesn't let us verify it from the Debug Log
- Watch the next several reports to confirm the Session 19 gap-calculation
  fix eliminates large/nonsensical gap numbers going forward
- Get Joi's answers on the mid-check-in app-restart bug (Session 18)
- Continue watching MK's room signal specifically as the extender
  position is tuned
- The gentle-lateness-encouragement decision (three options presented Session
  14, awaiting Sam's decision)
- Native speaker review of ES/HT translations
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- `doGet` auth layer decision

### July 15, 2026 (Session 19 — MK 3911-minute gap: root cause found and fixed)

**Root cause confirmed: not wifi, not a care gap, not device caching — a report-logic sort-order bug**

Traced end-to-end using the Debug Log, Apps Script execution history/Cloud
logs, and the five actual report emails (July 11–15):

- Confirmed all daily `sendShiftReport` runs fired on schedule, `Completed`,
  with normal check-in counts (39–49) every day — ruling out a broken
  trigger or a window that failed to reset.
- Confirmed no manual (`Editor`-type) executions existed — ruling out a
  stray manual run with a stale `lastReportTimestamp`.
- The July 14 report's own stated window was a completely normal
  `7/13 10:49 AM to 7/14 10:49 AM`, yet it contained a flag reading "No
  check-in between 7/11 4:18 AM and 7/13 9:30 PM" for MK — both endpoints
  outside the report's own window. That's only possible if a single MK
  check-in was **captured** (per its `capturedAt` field) on 7/11 4:18 AM but
  didn't **arrive** at the server (its `Timestamp`) until sometime inside
  the 7/13–7/14 window — i.e., sat queued client-side for roughly two days
  before finally syncing, consistent with the Session 6 backgrounding/retry
  behavior, just as an extreme delay rather than an outright loss.

**The actual bug:** `prolongedWetness`, `coverageGaps`, and `onTimeResidents`
all sorted each resident's rows by raw `timestamp` (server arrival order)
but then measured gaps using capturedAt-preferred time. That mismatch is
invisible almost all the time, since arrival and capture are normally
seconds apart — but for this one extremely-delayed row, sorting by arrival
time placed it next to the wrong neighbor in sequence, and the
capturedAt-based gap math between mismatched neighbors produced a large,
meaningless number (3911 minutes) instead of a real one.

**Fix:** added an `effectiveTime()` helper (capturedAt-preferred, falling
back to timestamp) and switched all three loops to **sort** by it, not just
use it for the gap calculation after the fact. Every resident's rows are now
placed in true chronological order regardless of when they happened to
sync, so gaps are always measured between genuinely adjacent real-world
check-ins. Shipped in `Code.gs`.

**Investigation trail (for reference — this took several passes to isolate):**
1. Initial theory: wifi dead zone at the far end of the house (TH/MK/AM
   rooms) — partially right (real intermittent signal issues did explain a
   separate, smaller gap the same week) but not the cause of this specific
   flag.
2. Second theory: Lory M.'s recurring `UNAUTHORIZED` rejections (stale
   `apiToken`, pre-Session-13 cached app) — a real, separate, already-
   understood bug, but not this one either.
3. Third theory: the report trigger skipped a day or failed to reset its
   window — ruled out via Executions/Cloud log review (Session 18).
4. Fourth theory: a stray manual run with a stale timestamp — ruled out,
   no `Editor`-type executions exist.
5. Actual cause: found by comparing the flagged gap's dates against the
   report's own stated coverage window, which is what exposed the
   sort-vs-measure mismatch above.

**Practical implication:** this also means MK's original 65-hour "gap" was
never a real coverage lapse — DSPs were checking on him normally; the
report just badly mis-displayed one late-syncing row. Worth a brief
reassurance to staff if this was raised as a concern with anyone.

**Still pending:**
- Watch the next several reports to confirm the fix eliminates
  large/nonsensical gap numbers going forward
- Get Joi's answers on the mid-check-in app-restart bug (Session 18)
- Build the version-lockout mechanism (index.html + Code.gs) — separate
  issue, still relevant given Lory M.'s recurring stale-token rejections
- Continue watching MK's room signal specifically as the extender
  position is tuned
- The gentle-lateness-encouragement decision (three options presented Session
  14, awaiting Sam's decision)
- Native speaker review of ES/HT translations
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- `doGet` auth layer decision

### July 15, 2026 (Session 18 — coverage-gap investigation, continued)

**Debug Log review corrected the initial theory — this was a real solo-coverage picture, not a simple network story**

Sam pulled the actual Debug Log rows for the night of July 13–14 (the report
date was off by a day — the 21:xx/22:xx/00:xx entries below are all 7/13
into early 7/14). Key findings from reading the raw log against staffing:

- **TH and AM were NOT silently dropped both times.** During the 22:01–22:02
  mini-round (Lory M. and Wilkens M. still on-site helping) TH and AM both
  logged `OK`. During Joi's solo 22:57–22:59 round less than an hour later —
  same phone, same rooms — TH, MK, and AM are missing entirely from the
  Debug Log (no row at all, success or failure). Same location transmitting
  fine one round and failing the next points to a **marginal/intermittent
  signal**, not a hard dead zone — consistent with an extender sitting right
  at the edge of usable range.
- **MK failed both rounds** — the weakest of the three rooms signal-wise,
  worth watching specifically as the extender position is tuned.
- **Lory M. hit `UNAUTHORIZED` four times in a row** (21:45:56–21:46:05,
  SR/CJ×2/KG) — a completely separate, already-understood issue: her phone
  was almost certainly running a stale cached copy of `index.html` from
  before the Session 13 `apiToken` fix, so it never sent the token at all.
  This is a caching problem, not a network problem — flagged for a proper
  fix (see below).
- **Ruled out:** residents were NOT skipped by staff. Joi attempted full
  rounds both times; the gaps are the app failing to transmit, not gaps in
  care.

**New info from Sam, changing the picture further:**
- **Extender move (closer to the router, roughly halfway to the dead zone)
  seems to have helped** — no missed rounds observed the following night.
  Not confirmed yet as a fix, just an early positive signal. **Keep watching.**
- **MK's 65-hour gap (7/11 4:18 AM – 7/13 9:30 PM) is NOT a technical
  issue** — Sam confirmed MK never left the site during that window. This
  needs to be looked at as a genuine care-coverage question, separate from
  the network investigation, and isn't something Cara's code can explain or
  fix.
- **New bug reported, not yet investigated:** starting Monday night (the
  same night as the data above), Joi reported the app sometimes "kicks her
  out" mid-check-in and she has to start over. This is a different failure
  mode than a slow/lossy network — if it's an actual page reload mid-flow,
  a check-in in progress could be lost outright rather than queued for
  retry (the Session 6 retry/queue fixes wouldn't help with this at all).
  Sam will ask Joi follow-up questions (does it happen mid-photo-capture
  specifically or anywhere in the flow; does "starting over" mean back at
  login or back at the top of one resident's check-in; what phone/browser)
  — answers not yet available.
- **Idea raised, not yet built:** add a version number to `index.html`, sent
  with every check-in, checked server-side in `Code.gs`. Any mismatch would
  be rejected with a clear `STALE_VERSION` reason in the Debug Log (same
  pattern as the `apiToken` fix), which would definitively rule the
  stale-cache theory in or out — for Lory M.'s issue and potentially for
  Joi's "kicked out" reports too, if that turns out to be cache-related.
  Deliberately not bundled into this session — worth its own careful build
  and test next time, touching both `index.html` and `Code.gs`.

**Still pending:**
- Watch extender performance over the next several shifts before calling it
  fixed
- Watch MK's room specifically — weakest signal of the three even after the
  move
- Get Joi's answers on the mid-check-in restart bug
- Build the version-lockout mechanism (index.html + Code.gs) to rule out
  stale-cache issues for both Lory M. and Joi
- Have Lory M. force-refresh/reinstall the Cara icon before her next shift
  regardless, as a cheap interim mitigation
- Consider whether the app should surface `UNAUTHORIZED`/failed-submission
  errors to the DSP in the moment, rather than failing silently from their
  point of view
- Follow up on MK's 65-hour gap as a care-coverage question (not a Cara/tech
  item)
- The gentle-lateness-encouragement decision (three options presented Session
  14, awaiting Sam's decision)
- Native speaker review of ES/HT translations
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- `doGet` auth layer decision

### July 15, 2026 (Session 17 — removed wet-at-shift-start flag; coverage-gap/wifi investigation opened)

**Removed the "wet at start of shift" flag from the daily report**

Sam confirmed this flag wasn't useful — it fired on the first check-in of a
shift whenever incontinence was "Needs change," but duration was always
unknown at that point (could've been minutes), so it read as routine noise
rather than an actionable signal. `Code.gs`'s `prolongedWetness` array now
only ever contains true mid-shift entries — a real, measured gap between two
consecutive check-ins on the same resident. The report render was simplified
to match (dropped the branch for the now-impossible `gap: null` case).

**Coverage-gap investigation opened — not yet resolved**

Sam noticed the July 14 report showed coverage gaps for TH, MK, and AM
between ~10:01pm–12:58am, but Joi G. actually completed a full round of all 8
residents around 11pm — those three check-ins simply never reached the
backend. TH/MK/AM's rooms are at the far end of the house from the wifi
router; cell coverage there is also known to be weak. Sam has moved the wifi
extender closer to the router (previously positioned nearer the far rooms)
to test whether that changes anything.

Discussed likely causes with Claude:
- **Wifi dead zone at the far end of the house** — the leading theory, given
  the pattern lines up exactly with room location, not with round timing or
  staff identity.
- **Phone backgrounding pausing the retry queue** (see Session 6 fix,
  `visibilitychange` listener) — if a phone's screen locks between check-ins
  in a weak-signal area and never returns to the foreground before the next
  round, queued submissions may not get a chance to flush. Wifi weakness and
  this queue behavior could compound each other rather than being separate
  causes.
- **The Debug Log tab is the fastest way to tell these apart**: if a request
  reached the Apps Script backend at all (even to fail — LOCK_TIMEOUT, ERROR,
  UNAUTHORIZED, etc.), it'll have a row there. No row at all for that
  resident/time window means the request never left the phone — network-side,
  consistent with the wifi theory. A row that shows an error means it reached
  the server and something else went wrong.

**Still pending:**
- Check the Debug Log tab for the specific July 14 ~10–11pm window (TH, MK,
  AM) to confirm whether those requests ever reached the backend at all
- See if extender relocation changes the pattern on the next shift
- The gentle-lateness-encouragement decision (three options presented Session
  14, awaiting Sam's decision)
- Native speaker review of ES/HT translations
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- `doGet` auth layer decision

### July 15, 2026 (Session 16 — removed CJ dtree summary from daily report)

**Removed CJ's Positioning & Safety Check auto-summary from the comment field / daily report**

Sam flagged that the daily report's Observations section was showing lines like
"Position: Lying down (mittens on, pillow under head, area checked for
hazards). Location: Bed (wedge pillow in place, side rails up)." for every CJ
check-in — auto-generated noise, not useful information. Root cause: Session 14's
CJ decision tree had no dedicated sheet column for its lying/sitting and couch/bed
answers, so `submitCheckin()` built a `dtreeSummary` sentence from those answers
and prepended it to the comment field ahead of anything staff actually typed.

**Fix:** removed `dtreeSummary` generation entirely. `comment` now sends only
`ck.comment` — the staff member's own typed note (Sam's example: MK's "Safety
belt from shower chair created temporary pressure..." note) — exactly as before
the decision tree existed. The checklist itself (mittens, pillow, hazard check,
wedge pillow, bed rails) is unchanged and still fully required/tracked
client-side per check-in; only the auto-summary sentence that leaked into the
report is gone. Left a code comment noting that if these answers are ever needed
again, they should get their own payload field/sheet column rather than riding
along in `comment`.

**Still pending:**
- The gentle-lateness-encouragement decision (three options presented Session 14,
  awaiting Sam's decision)
- Native speaker review of ES/HT translations
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- `doGet` auth layer decision

### July 14, 2026 (Session 15 — CJ duplicate-card bug fix)

**Fixed the Session 14 known bug: third, unlabeled wedge-pillow/bedrails card on CJ's check-in**

Root cause was in the CJ data object itself, not the render logic. Session 14's
decision tree moved wedge pillow / bed rails into CJ's Positioning & Safety Check
(two legitimate render spots: the 8pm–5:30am assumed-lying-in-bed block, and the
interactive lying+bed branch — both already correctly ordered, bed rails then
wedge pillow). A code comment claimed CJ's regular task list had been emptied out
as part of that move, but `CJ.tasks` and `CJ.firstCheckTasks` still held the old
two items in the old order (wedge pillow first). The generic Step 2 task renderer
doesn't care which decision-tree branch is active — it renders whatever's in
`activeTasks` unconditionally — and CJ's Step 2 title is deliberately suppressed
(decision-tree content replaces it), so the leftover items appeared as a third,
unlabeled card in every case. Matched Sam's screenshots exactly (wrong order,
appears regardless of sitting/lying/couch/bed selection).

**Fix:** emptied `CJ.tasks`; removed the two duplicate lines from
`CJ.firstCheckTasks`, leaving only its skin-check and face-check items (which
aren't duplicated anywhere else). Left a code comment warning against
reintroducing wedge pillow / bed rails there.

**Still pending:**
- The gentle-lateness-encouragement decision (three options presented Session 14,
  awaiting Sam's decision)
- Everything carried forward from Session 13/14 (native speaker review of ES/HT,
  in-app sync queue reset button, dynamic resident config from Sheet, Workspace
  migration back to ABLE, `doGet` auth layer decision)

### July 14, 2026 (Session 14 — CJ decision tree, staff additions, celebration/points fixes)

**CJ Positioning & Safety Check — decision tree, IN PROGRESS, known bug remaining**
Built a two-question branching decision tree for CJ (lying/sitting, then couch/bed),
replacing the old static "Wedge pillow in place" / "Bed side rails up" tasks. Sitting
up correctly cascades to remove the couch/bed question, position picker, and Photo 2
(repositioning) — only Photo 1 (sensitive skin area) remains. Lying + bed shows
"Bed side rails up" then "Wedge pillow in place" (bedrails first, confirmed order).
Added an 8pm–5:30am assumed-lying-in-bed window (locked, no override) that skips
the Q1/Q2 buttons but still requires the full safety checklist.

**KNOWN BUG — FIXED in Session 15.** See the Session 15 log entry below for the
root cause and fix; kept this paragraph as-is for the historical record of how
the bug was first found.

**Also shipped this session:**
- 7 new staff members added (Julie E., Julia B., Dave K., Duke K., Miggs O.,
  Danny M., Monse H.) with PINs, checked against existing for collisions
- Celebration overload fix — when a streak celebration and a milestone (or first
  greeting) become eligible on the same check-in, the milestone/greeting is now
  permanently marked as fired without ever showing, instead of just being deferred
  to resurface later as a third screen
- +15 on-time bonus points — awarded silently, with a small green "On time! +15"
  badge shown only when earned; nothing shown or implied when not earned, per
  Sam's explicit request to avoid anything that could read as finger-pointing
- Gentle-encouragement-for-lateness question raised but NOT built — three options
  presented (silence/do nothing beyond the bonus, positive-only rotating message,
  forward-looking pre-check-in nudge), awaiting Sam's decision

**Still pending (as of this session — see Session 15 above for the bug fix):**
- ~~FIX: the duplicate wedge pillow/bedrails card described above~~ — fixed Session 15
- The gentle-lateness-encouragement decision (see above)
- Everything carried forward from Session 13 (apiToken/whitelist already shipped;
  streak emoji still unconfirmed working per Sam — "never worked for me")

### July 10, 2026 (Session 13 — Perfect Shift Streak backend, security fixes, repositioning note bug fix, streak celebration visuals)

**Context: this session picked up from the Session 12 pending list**, working through it roughly in order: repositioning note first, then a routine confirmation pass on Session 12's report fixes (which turned up an additional bug), then the on-time streak feature, then the `apiToken`/whitelist security fix. A significant part of the streak work — see below — turned out to already be substantially built from earlier in this same session, in a part not fully visible when the "let's pick up where we left off" continuation began; this entry captures the full, true state as confirmed by direct code inspection, not just what was newly written today.

**Repositioning note — confirmed working, then a real bug found and fixed**
Built the backend (`doGet` extended to scan each resident's last 50 rows for the most recent position, `capturedAt`-preferred) and frontend (chip display, `fmtClockTime`/`fmtRelativeAgo` helpers reusing existing translation keys) exactly per the Session 12 mockup. Confirmed working via the Network tab after initial confusion traced to Apps Script's first-call-after-redeploy slowness, not a real bug. **Then Sam caught a genuine bug during real use:** repositioned MK multiple times across 7 check-ins, and the note kept showing the same stale position from login. Root cause: `session.lastPosition` was only ever populated once, at login, via the background fetch — nothing updated it locally afterward. Fixed by writing to `session.lastPosition` immediately, client-side, the instant any check-in with a position is submitted, rather than relying solely on the once-per-login server fetch.

**Confirming Session 12's report fixes — found a third instance of the same bug**
Sam pasted the actual live `Code.gs` to confirm the `capturedAt`-preference fixes from Session 12 had deployed correctly. `coverageGaps` and `prolongedWetness` both confirmed correct. While reviewing, found `onTimeResidents` (the Shift Overview's "Residents with consistent on-time check-ins" list) — sitting right next to the other two, computing a conceptually identical gap — had been missed and was still using raw `Timestamp` only. Fixed to match.

**On-time streak feature — full status, both mechanics**
See the Phase 1 shipped-features entry above for the complete technical description. Summary of what happened this session specifically:
- Tonight's Streak: confirmed already fully built and working, no changes needed.
- Perfect Shift Streak: confirmed the frontend was already fully built (tracking, `recordShiftEnd` at logout, fetch at login) but `Code.gs` had zero handling for any of it — every sign-out had been silently sending data into a backend with no route, confirmed directly from Sam's pasted live file. Built and shipped the missing backend: `handleRecordShiftEnd`, `getStreakSheet`, `findStreakRow`, and `doGet`'s `perfectStreaks` section.
- Added the two pieces that were designed but never decided/built: where the streak displays (check-in screen, not the dashboard — avoids colliding with the existing unrelated device-level "🔥" streak already there) and whether a new personal best gets its own celebration (yes — a quiet, once-per-shift, Bea-only moment at sign-out, computed client-side so it never delays logout).

**Streak celebration visuals — two rounds of feedback, both addressed**
First pass (bigger emoji, scattered positions, punchier character sway) shipped, then Sam reported two problems: the character was still using the calm everyday portrait rather than a real celebration pose, and the emoji had vanished entirely. Investigation:
- The celebration pose issue was real and straightforward — `WILL_CELEBRATE_SVG`, `REMI_CELEBRATE_SVG`, `BEA_CELEBRATE_SVG` already existed in the codebase (used by the older Full Skin Care streak celebration, `showSkinStreakCelebration`) but `showStreakCelebration` had never been pointed at them, using the plain `COMPANION_CHARS` portraits instead. Fixed — now uses the correct celebratory assets with their own viewBoxes, with the blink overlay dropped since those assets don't have lid coordinates calibrated for it (the pose already carries the expression).
- The "no emoji" report couldn't be reproduced through code review — the generation logic was traced end-to-end in isolation (ran the exact string-building code standalone) and produces correct, valid output. Leading theory: with 7 check-ins in one test session, Sam likely also triggered a regular milestone Companion Moment along the way, which never had emoji or aurora by design (calmer, different purpose) — easy to conflate with the streak celebration before the pose fix made the two visually distinct. Added a second, genuinely new layer regardless — ambient sparkle drifting behind everything — partly per Sam's ask for more background dynamism, partly because if a real emoji bug does surface on the next test (celebratory pose showing, sparkle showing, but the main multiplied element still missing), that's a clean, isolated signal to dig into specifically.

**Security hardening — both recommended items from Session 12, now built**
`apiToken` shared secret (checked first in `doPost`, before any data is touched, on every action) and resident-ID whitelist, both described fully in the Phase 1 entry above. `doGet` remains intentionally out of scope for now.

**Still pending, carried forward:**
- Confirm on next real test: does the streak celebration correctly show emoji alongside the new celebratory pose? (Isolates whether Session 13's "no emoji" report was a real bug or a different-screen mix-up.)
- Native speaker (Leriy) review of Haitian Creole clinical content
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- Decide whether `doGet` deserves its own auth layer, given it's currently the one unprotected read path left

### July 6, 2026 (Session 12 — GitHub Pages migration, on-time streak design, repositioning note mockup)

**Report threshold fixes (`Code.gs`) — built, not yet confirmed deployed**
Two report flags corrected to a consistent 150-minute floor and both now prefer `capturedAt` over arrival `Timestamp` when computing gaps, same fix pattern as Session 11's delayed-sync work:
- **Coverage gaps** — already had a 150-min floor (`gap > 150`) but was computing purely off arrival time. Now uses `capturedAt` when available on both rows being compared, so a round that happened on time but synced late no longer reads as a false gap.
- **Prolonged wetness risk** — this one actually had a *lower* threshold than intended (130 min, not 150). Raised to match and given the same `capturedAt` correction.
- Untouched on purpose: the "resident already needed a change at the very first check-in of the shift" flag — no gap to measure there, it's flagging unknown prior-shift duration, not a connectivity artifact.
- **Action item:** confirm this has actually been deployed to the live Apps Script project — it was built and handed off this session but deployment wasn't explicitly confirmed.

**Hosting migration: Netlify → GitHub Pages, complete and live**
Both of Sam's Netlify accounts ran out of free-tier credits (300/month cap, hard stop, no overage purchase available on free plans). Rather than create a third free account or lean on Netlify's Agent Runner "400 extra credits" signup incentive — which draws from the same capped pool and doesn't remove the underlying risk of the whole site going dark mid-shift — the decision was made to move to GitHub Pages, which uses a flat-fee, non-metered model with generous soft limits instead of a shared credit pool.

*Personal IP framing, decided deliberately:* Cara is Sam's separate venture, with ABLE as founding pilot partner rather than owner. Given that framing, hosting costs and account ownership were kept cleanly personal rather than routed through ABLE — partly to avoid private-inurement optics for a 501(c)(3), partly to keep a future IP conversation simple. Concretely:
- GitHub account renamed from `BroaderLiving` (Sam's original personal account, just named after ABLE) to `rsamyates`
- Primary email changed to a personal Gmail address; ABLE email kept only as backup recovery, not primary
- GitHub Pro (~$4/month) paid on Sam's personal card, not ABLE's
- Domain naming: explicitly decided against `ablecara.app` or anything tied to ABLE/BroaderLiving branding, for the same separation-of-IP reasons — currently running on the default `rsamyates.github.io/Cara-App/` URL, no custom domain purchased yet

*Technical setup:* `Cara-App` repo confirmed private under the renamed personal account; GitHub Pages enabled (Settings → Pages → Deploy from branch → `main` / root); the current `index.html` (with all of Session 11 and this session's BH exemption) uploaded before Pages was switched on, so the first live version at the new URL was already fully caught up — one combined cutover rather than two separate events. HTTPS enforced automatically. No `Code.gs`/backend changes of any kind — hosting is purely a front-end concern, completely decoupled from where the data lives.

**Live URL: `https://rsamyates.github.io/Cara-App/`**

*Status:* staff have the new link; Sam has tested it himself; no full overnight shift has run on it yet by staff. Netlify accounts intentionally left dormant (not deleted) as a fallback until a real shift confirms everything holds up — they can't serve anyone right now anyway (out of credits), so there's no cost to waiting. **Once a clean shift is confirmed, both Netlify sites can be deleted** (Site settings → Danger zone → Delete site) — purely tidying up at that point, not a live decision anymore.

**Security review — discussed, not yet built**
Prompted by the hosting conversation, a review of the existing two-screen login (site-wide password + staff PIN) found both checks are entirely client-side, comparing against plaintext constants sitting in the page's own source (`SITE_CODE = '25deanza'` and the `STAFF` array's PINs). More importantly, `Code.gs`'s `doPost` validates nothing at all before writing to the sheet — confirmed directly, since the Session 11 de-dup test tool wrote to the sheet successfully without ever touching the app's login flow. The login screens are real and valuable for their actual purpose — preventing accidental encounters and establishing staff accountability — but they are not a security boundary against anyone who deliberately investigates, on any host.

**Recommended, not yet built:**
- A shared secret token (`apiToken`) sent with every check-in payload and checked first thing in `doPost`, rejecting anything that doesn't match. Closes the specific hole demonstrated in Session 11 — right now literally anyone on the internet who finds the Apps Script URL can write to the sheet with zero credentials.
- A resident-ID whitelist in `doPost`, rejecting any `resident` value not in the known `RESIDENTS` list — stops arbitrary/junk sheet tabs like the test tool's `ZZ_TEST_DEDUP` from being creatable by anyone, independent of the token fix.
- **Separately flagged, worth Sam checking directly:** the Google Drive sharing settings for the photo folder — the actual privacy of resident photos depends entirely on that folder's own access settings, not on anything in the app, and this is arguably a bigger exposure surface than the login screens.
- Out of scope as "simple": real hashed-password, server-side session authentication. Would require Cara to stop being a purely static site — a genuine architecture change, not something to chase opportunistically.

**On-time streak feature — fully designed this session, not yet built**
Two distinct, separately-tracked mechanics, both keyed per caregiver per resident:

**1. Tonight's streak (session-scoped, drives the celebration screens)**
- Per resident. Starts counting at a resident's 2nd on-time check-in of the shift (i.e. "in a row" language only appears once it's genuinely 2+), climbs by 1 with every consecutive on-time check-in for that resident, resets to 0 the moment any check-in for that resident that shift exceeds 120 minutes since the previous one.
- "On-time" = elapsed time since `session.lastCheck[r.id]` (the existing per-resident last-check tracking) is ≤ 120 minutes.
- Fires the companion celebration screen every time the streak is ≥ 2.

**2. Perfect Shift Streak (persists indefinitely across shifts, tied to a specific caregiver + resident pair)**
- A much bigger, slower number — the "50, 75, 200" scale Sam wants staff to feel motivated to protect.
- A shift counts as "perfect" for a given caregiver-resident pair if every check-in that caregiver made for that resident during that shift was on-time.
- **The "gimme":** the first check-in of any new shift for a caregiver-resident pair automatically counts as on-time, regardless of how many hours passed since the caregiver's last shift ended — without this, every shift would start with an automatic "late" purely from the gap between shifts, making a perfect shift nearly unachievable.
- If the caregiver's most recent shift with that resident was perfect and this one is too, the streak increments by 1. Any late check-in during their own shift resets it to 0.
- **Critically, this is scoped to each caregiver's own shifts only** — this is what solves the team-fairness problem raised this session. A different caregiver checking on the same resident during their own separate shift has zero effect on another caregiver's Perfect Shift Streak. Joi's number only ever looks at Joi's own shifts with that resident; Monyette checking MK on a night Joi isn't working doesn't touch Joi's streak at all.
- **Personal best, confirmed:** even when the active streak resets to 0 after an imperfect shift, the caregiver's highest-ever Perfect Shift Streak for that resident is retained permanently and displayed alongside the current number (e.g. "MK: 3 — Personal best: 200") — the achievement doesn't vanish just because the active count did.
- **Worked example Sam confirmed as correct:** Night 1, Joi's shift with MK is perfect → streak 1. Night 2, a different caregiver covers MK and has a late check → doesn't touch Joi's number at all, she wasn't on shift. Night 3, Joi's back — first check is the gimme, rest of shift on time → perfect shift → streak 2. Night 4, Joi has one late check with MK → not perfect → her streak resets to 0, but her personal best (2) stays recorded.

**Explicitly open, not yet decided — flag for the build session:**
- Whether Perfect Shift Streak gets its own celebration treatment, or stays a quieter, background-tracked number (only "tonight's streak" has a confirmed celebration design so far)
- Exact evaluation trigger for "was the last shift perfect" — end-of-shift (sign-out) vs. start-of-next-shift (login) vs. continuous
- Data model for persistence — this needs a new place to live in `Code.gs`/the Sheet (e.g. a new tab keyed by staff + resident, holding current streak, personal best, and whether the most recent shift was perfect), since nothing like this exists yet. This is a real backend build, not a quick addition — flagged as deserving focused, unhurried attention in its own session rather than being rushed.

**Celebration visual design — confirmed for "tonight's streak"**
- Text: "[Resident initials] On-Time Streak! [N] in a row!"
- **Confetti explicitly ruled out** — already the visual signature of the existing check-in completion celebration (`spawnConfetti()`); the streak celebration needed its own distinct identity.
- **Escalating intensity, confirmed:** the celebration itself scales with the streak count — a fresh "2 in a row" gets a modest treatment, a long streak gets bigger text, bigger character, faster/more prominent motion.
- **Each companion gets their own signature "multiplied" element**, one instance of the element per point in the streak (so streak 5 shows 5 of the element), rather than a single shared treatment for all three:
  - Bea — hearts floating outward, ties directly to her existing "your care ❤️ inspires your actions" language
  - Will — balloons rising
  - Remi — bubbles rising
  - All three share a common backdrop: **northern lights sweeping**, intensity/vividness scaling with streak count — chosen specifically because Cara only runs at night, unlike the flat color-pulse background used in earlier drafts.
- Base motion: a boogie-style side-to-side sway, each companion in their own established brand color (Bea blue, Will amber, Remi teal), consistent with the Companion Moments visual language from Session 10.
- **Ideas considered and set aside** for tonal reasons — genuinely fun, but felt like a different app's voice than Cara's warmth-plus-clinical-seriousness balance: unicorn galloping, UFO beaming a checkmark, cartoon trumpet fanfare, celebration train, Mardi Gras marching band. Penguins/otters/dolphins/birds were considered but ranked below the house/nature-themed options as "generic fun mascot" rather than something distinctly Cara's own.
- **A strong idea raised but not yet built into any mockup:** a small house graphic where each on-time check-in in a streak lights up one more window — flagged as thematically ideal (Cara is literally a house checked room by room, overnight) and a natural fit for the multiply-by-count mechanic, worth prototyping in the actual build session even though it wasn't demoed live this session.

**Repositioning "last check-in" note — mockup approved, not yet built**
New note above the position-selection buttons on Step 3 of the check-in flow, showing the resident's most recently recorded position plus when it was recorded. Two layout options were mocked up as a standalone HTML file (styled with the app's real fonts/colors, not a generic demo) after the Visualizer tool became unresponsive mid-session:
- **Option A** (inline, inside the existing position card) — considered, not chosen.
- **Option B** (separated light gray chip above the card) — **confirmed direction.** Originally included a ↻ icon intended to suggest repositioning; Sam correctly flagged it read as a confusing "refresh" symbol instead, since it wasn't clickable and did nothing. Removed — final approved version is text-only: "Last check-in: [Position] · [time] · [relative time ago]."
- **Not yet wired to real data.** Position history isn't currently fetched back from the Sheet anywhere in the app. Building this for real needs a small backend read, following the same pattern as the existing `fetchCrossShiftTiming()` cross-shift fetch at login — flagged as a small lift, not a blocker, just not yet done.

**Deploy workflow note — supersedes the Session 11 Netlify-batching guidance**
With hosting now on GitHub Pages, the credit-conservation reason for batching `index.html` deploys no longer applies — GitHub Pages has no metered credit pool to protect. Still good practice to batch related changes into clean, deliberate uploads rather than many small ones, but it's now a tidiness preference, not a hard constraint. `Code.gs` deploys remain completely free and separate regardless of front-end host, as established in Session 11.

**Still pending, carried forward and updated:**
- Confirm the Session 12 `Code.gs` report threshold fixes are actually deployed
- Build the `apiToken` shared-secret check and resident-ID whitelist in `Code.gs`
- Confirm Google Drive photo folder sharing settings directly
- Confirm one full overnight shift has run cleanly on the new GitHub Pages link before deleting the Netlify sites
- Build the on-time streak feature (both mechanics) — index.html celebration screens + new Code.gs persistent data model for Perfect Shift Streak
- Build the repositioning "last check-in" note (Option B, confirmed) — needs the small backend read for position history
- Native speaker (Leriy) review of Haitian Creole clinical content
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE

### July 3, 2026 (Session 11 — data integrity investigation, de-dup guard, delayed-sync detection, FSC two-photo requirement)

**Background: the MK/AM/CJ data investigation**
Sam brought raw sheet data showing MK missing a midnight round, near-duplicate rows for MK (15–30s apart) and CJ (five rows in 15 seconds), and a discrepancy between staff self-report ("no rounds after 4am") and 6am-hour timestamps. Working theory: WiFi dead zone at the back of the house (MK/AM's rooms) and marginal signal in CJ's room. Diagnosis, reasoning from the actual retry/queue code (not speculation):
- Near-duplicate rows = the app's 3-attempt retry logic resending an already-successful submission because the *response* got lost on a weak connection, not because the request failed.
- Photos present on some duplicate rows but missing on others (CJ) = expected behavior, since photos are only carried in the 20-minute in-memory retry layer, never the persistent localStorage queue — a connection gap longer than 20 minutes permanently loses the photo but the text-only data still eventually syncs.
- The 6am timestamps likely reflect delayed writes of check-ins actually performed earlier — **the sheet's Timestamp column reflects server receipt time, not the moment the staff member tapped submit** — meaning Joi's account and the data may not actually conflict.
- MK's fully-missing round is the one pattern that isn't just "delayed" — either the queued retry never got a connectivity window before the localStorage aged out, or it genuinely didn't happen. Recommended follow-up: check the Debug Log tab for a doPost attempt at that time, and ask Joi whether she recalls seeing the on-screen sync-failure banner that night.

Sam tested WiFi signal directly and confirmed: weak-to-nonexistent in MK/AM's rooms, marginal in CJ's — validating the connectivity theory. Also confirmed staff use a mix of WiFi and cellular, with cellular coverage in the area being unreliable regardless of location — reinforcing that a WiFi fix (mesh/extenders) is the right target rather than a cellular booster. Xfinity Pro plan ($15/mo, up to 2 extenders + 4G/LTE ISP-outage backup) confirmed as a reasonable fix for the WiFi dead-zone piece specifically; the LTE-backup feature addresses a different problem (whole-home ISP outage) and doesn't affect a single dead zone.

**De-duplication + delayed-sync detection — built and deployed**
Two coordinated changes across `index.html` and `Code.gs`:

*index.html:* `submitCheckin()` now generates a `checkinId` (`residentId-timestamp-random`) once, at the moment of capture — before any send attempt. `capturedAt` (ISO timestamp of the tap) is captured alongside it. Both are added to the payload object, and because every retry path (immediate 3x retry, 20-min background retry, persistent localStorage queue) reuses the same payload object rather than regenerating it, both fields automatically survive every retry attempt with no additional wiring needed.

*Code.gs (Sam has already deployed this):*
- New `ensureLogColumns(sheet)` helper migrates any pre-existing resident sheet (all 8 existed before this change) by adding "Checkin ID" and "Captured At" columns at the far right the next time a check-in lands — safe no-op once they exist.
- `doPost` checks the last 30 rows for a matching `checkinId` before writing. Match found → skip the write, return `{status:'ok', note:'duplicate skipped'}`, log `DUPLICATE_SKIPPED` to Debug Log. This is what actually stops retry-generated duplicate rows from being written at all.
- `sendShiftReport` reads the two new columns, uses `checkinId` as a safety-net dedup during row collection (in case anything ever slips past the `doPost` check), and flags any row where `Timestamp` is more than 3 minutes later than `Captured At` as a "delayed sync" — meaning the care happened on time but the write was late, most likely due to a weak connection.
- New **Connectivity Note** report section (styled like the existing Pace Note, blue/informational, not a clinical alert) lists delayed-sync entries — resident, staff, captured time, logged time, gap in minutes. Per Sam's direction, this section only renders when `delayedSync.length > 0` — no visual noise on clean nights.
- Shift Overview table gained a "Delayed sync (captured earlier than logged)" count row.

**Live-tested and confirmed working.** Built a standalone local test tool (`dedup-test.html` — not part of the deployed app, just a one-off HTML file Sam opens locally, no Netlify involved) that POSTs an identical fake check-in twice with the same `checkinId` directly to the Apps Script endpoint, targeting a disposable `ZZ_TEST_DEDUP` sheet tab so no real resident data is touched. Result: first POST wrote one row and logged `OK`; second POST 14 seconds later was caught and logged `DUPLICATE_SKIPPED` with the matching `checkinId` in the detail column. Confirmed in production — this is a real, working fix, not just a theoretical one.

**Deploy note established this session:** `Code.gs` deploys through Google Apps Script — free, no Netlify credits involved, safe to push anytime. `index.html` deploys through Netlify, which is running low on credits — so `index.html` changes get batched across a session and deployed once, rather than after every change. Claude tracks a running "pending changes" list through the session so nothing gets lost before the final deploy.

**Full Skin Care two-photo requirement**
For CJ, MK, BH, KG only: the single required photo prompt is now two required photo prompts, rendered as stacked zones on the same photo step. Badge language is deliberately large/bold/high-contrast — a big step up from the previous gray subtext treatment — because these are the core clinical documentation shots for this care level:
- **Photo 1** — red badge, two lines: "PHOTO 1:" / "SENSITIVE SKIN AREA". Subtext below the camera icon: generic "Tap to take required photo" (not resident-specific, since this is the skin check itself).
- **Photo 2** — blue badge, single line: "PHOTO 2: REPOSITIONING (New Position)". Subtext below the camera icon: the resident-specific photo note (e.g. "Bottom — shift end and each repositioning round." for MK) — carried over from what used to be the single photo prompt's subtext.

Submit stays disabled (`checkReady()`) until both `ck.photo` and `ck.photo2` are true. The "+ Add another photo" button only appears once both required photos are captured. Optional-photo numbering (`addExtraPhoto()`) accounts for the extra required slot — FSC residents' first optional photo is labeled "Photo 3," everyone else's is "Photo 2," matching actual column position.

**No Code.gs changes needed for this feature.** The repositioning photo (`ck.photo2Data`) is prepended to the `extraPhotos` array at submit time, so it lands in the sheet's existing "Photo 2" column automatically — Photo 1 = skin check, Photo 2 = repositioning, Photo 3+ = optional, with zero backend changes. All labels and subtext fully translated EN/ES/HT.

**Still pending (carried forward):**
- Native speaker (Leriy) review of Haitian Creole clinical content
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE
- Confirm MK's fully-missing midnight round via Debug Log tab + ask Joi about the sync-failure banner
- Decide on Xfinity Pro extender purchase pending final confirmation of WiFi vs. cellular usage patterns (largely resolved this session — WiFi confirmed as primary and the right target)

### July 2, 2026 (Session 10 — Companion Moments)

**New screen: `s-greeting` (Companion Moment)**
A full-screen character appearance that fires mid-shift — not at login. Character zooms in with spring easing (`cubic-bezier(0.34,1.56,0.64,1)`), speech bubble fades up, "Let's go →" returns to dashboard. Background gradient is character-specific: Bea on deep blue (`#1E3A8A→#4A6CF7`), Will on warm amber (`#7A4500→#E8A838`), Remi on teal (`#0B4D44→#2A9D8F`). Blink loop fires every 2–6s using skin-tone ellipse lids (`ry` animated 0→full→0): Bea `#FFD700`, Remi `#C8845A`, Will `#8B5E3C`. Lid coordinates derived from actual eye white ellipse data in each SVG constant.

**Five moments per shift:**
- First greeting: fires after check-in 1, 2, or 3 (randomized at login via `session.greetAfter`)
- Milestones at check-ins 10, 22, 34, 40 — each threshold offset ±3 (`session.milestoneThresholds`) so timing never feels algorithmic
- Total check-ins counted cumulatively across rounds: `(session.round - 1) × RESIDENTS.length + session.done.length`
- All fire via `celebrationBack()` — the "Back to rounds" button on the celebration screen

**Character selection:** random each milestone, never the same character back to back (`session._lastCompanionChar` excludes last shown). Message repeat prevention also in place per character (`session._lastMsg_[charId]`) — one message per character per milestone currently, guard activates when second messages are added.

**Session state added at login:** `session.greetAfter`, `session.greetShown`, `session.milestonesFired`, `session.milestoneThresholds`, `session._lastCompanionChar`

**"Other" staff** greets as "caregiver"

**Messages (`COMPANION_MOMENTS` in `index.html`):**

| Milestone | Bea | Will | Remi |
|---|---|---|---|
| First / 10 | "I see you, {name}." | "{name}, going strong." | "{name}, this is consistent care." |
| 22 | "{name}, your care ❤️ inspires your actions." | "{name}, you got this." | "{name}, you're still showing up." |
| 34 | "You've held this house all night, {name}." | "Few can do what you are doing, {name}." | "{name}, you are protecting skin health." |
| 40 | "Almost done, {name}. Amazing work." | "{name}: Persistence — that's you." | "Morning's coming. Well done, {name}." |

**Pending / natural next step:** add a 2nd message per character per milestone so the message-level repeat guard activates and moments feel fresh across shifts.

**Still pending (carried forward):**
- Native speaker (Leriy) review of Haitian Creole clinical content
- In-app sync queue reset button
- Dynamic resident config from Sheet (deferred until Workspace restored)
- Workspace migration back to ABLE

### July 1, 2026 (Session 9 — translation completion, care timing logic, Remi post-change reminder)

**Translation completion (`index.html` only — no Code.gs changes):**
Full ES/HT translation pass covering every user-facing string in the app. Architecture: `tr(obj)` resolves `{en,es,ht}` objects to the current language with English fallback; `trPick(arr)` picks randomly then resolves; `trTemplate(obj, vars)` handles strings with embedded dynamic values (streak counts, resident IDs) using `{placeholder}` tokens so each language can independently author its word order. Canonical English always sent to the backend — `careLevel`, position values, and concern chip values are stored and transmitted in English regardless of UI language, exactly as breathing/distress/incontinence already worked. Everything translated: chip labels (11), all 8 residents' Remi tips/tasks/photo notes across all timing tiers, position labels, care level labels, all character message pools (Bea/Remi/Will dashboard/opening/celebration/round-end, rushed alerts, skin streak celebration tags and messages), and all remaining UI strings including login screen, dashboard progress labels, round labels, and the sync failure banner. Note: Spanish is a confident first pass; Haitian Creole is good-faith first pass — needs native speaker (Leriy) review before fully trusting clinical content.

**Care timing logic (`index.html` only):**
Three-tier task system already architected (helpers `isFirstCheck()`, `isAfterThree()`, `residentTasks()`, `residentRemi()`, `residentPhotoNote()` were present from a prior state); what was missing was the actual content. Updated `RESIDENTS` array to trim core task lists to every-check-in minimum and added `firstCheckTasks`, `afterThreeTasks`, `firstCheckPhotoNote` per affected resident. `RESIDENT_TEXT` already contained complete translated content for all tiers. Result: YC adds heels/ankles on first check and toes/breasts/tabs after 3am; MK/CJ/BH/KG add skin inspection on first check and a second pass after 3am; SR adds breasts/lotion after 3am; AM unchanged (nocTasks already handles overnight). AM nocTasks note: the lotion/scalp tasks show during the day (correct) and are suppressed overnight via the existing `nocTasks` override — `isNocHours()` is 10pm–6am, so testing during the day will show daytime tasks as expected.

**Remi post-change reminder (`index.html` only):**
New `REMI_CEL_CHANGE` pool (4 messages, all translated) fires on the celebration screen whenever a check-in included "Needs change." Remi is guaranteed the slot — not left to the random 25% chance of the normal pool split. Messages focus on using the change as a skin inspection and photo opportunity. `wasNeedsChange` is now threaded through `showStandardCelebration(r, pts, wasNeedsChange)` and `showSkinStreakCelebration(r, pts, wasNeedsChange)`.

**Still pending:**
- Native speaker review of ES/HT translations (especially Haitian Creole clinical content)
- In-app sync reset button (low priority — escalates from sync banner, not yet built)
- Migrate back to ABLE Google Workspace once restored — **still a workaround, waiting on approval**
- All Phase 2 items (daytime shift, dynamic resident config, PWA, etc.)

### June 30, 2026 (Session 8 — photo loss incident: missing Drive folder + concurrency bug)

**Trigger:** Sam reported no photos had been captured since 6/29 7am, including testing done the afternoon of 6/29. Recalled he had cleared out old photos from Drive to free up storage space the same afternoon.

**Investigation — two issues found, in the order they were discovered:**
1. A burst of ~14 simultaneous `doPost` calls was found in Apps Script Executions at 4:32pm on 6/29, mixed Completed/Failed, all ~9-11s duration. Diagnosed as `flushPendingRetries()`/`flushPendingQueue()` in `index.html` firing every backlogged check-in at once instead of one at a time — overwhelming `doPost`'s single script lock (10s wait), causing most to time out with an uncaught, unlogged exception. This was a genuine bug and was fixed, but turned out not to be the actual root cause of the photo loss — it was a secondary, real problem surfaced during the same investigation.
2. Sam confirmed Sheet rows *were* appearing with full data but blank photo columns — ruling out a total backend failure and pointing specifically at the photo-save step. Apps Script's Executions panel proved too hard to navigate on mobile to get a clear error message, so a Debug Log sheet tab was added instead (see below) to surface errors directly in the spreadsheet.
3. First Debug Log entries immediately revealed the real cause: `DriveApp.getFolderById(PHOTO_FOLDER_ID)` throwing `"No item with the given ID could be found"` on every photo save — the photo folder itself had been deleted (not just photos inside it) during Sam's storage cleanup. Storage usage confirmed: was at 81%, dropped to 66% after cleanup; Trash was empty (ruling out a Trash-retention quota theory considered earlier).

**Fixed and confirmed live:**
- ✅ New `PHOTO_FOLDER_ID` set to a live folder (`1w4tl67nnrVL0YoPIkgVUTElEpsbEQgK1`), confirmed owned by rsamyates@gmail.com (same account running Apps Script)
- ✅ `flushPendingRetries()` and `flushPendingQueue()` converted from parallel to sequential (`for...of` + `await`) — never more than one outstanding request to the backend at a time
- ✅ New "Debug Log" sheet tab, auto-created, logs every `doPost` call's outcome (OK / PHOTO_SAVE_ERROR / LOCK_TIMEOUT / ERROR) with resident, staff, and detail — self-trims to 500 rows
- ✅ Lock-wait timeout now caught explicitly (previously uncaught) and logged before returning a clean error response
- ✅ Sam confirmed via live test: photo captured, check-in submitted, Debug Log showed `OK` / `Photo Saved? = Yes`, photo verified present in the new Drive folder
- ✅ Sam separately confirmed it's safe to consolidate old/loose Cara care photos back into the new folder — `savePhoto()` only ever adds files, never reads or counts existing contents

**Still pending:**
- Build an in-app reset button for the persistent retry queue (`localStorage`), so a stuck backlog can be cleared without navigating to Safari's Website Data settings — discussed as a nice-to-have but not yet built since Sam was able to clear it manually this time
- Same Phase 2 items as before (translations, time-aware care prompts, dynamic resident config from Sheet, etc.)

### June 26–27, 2026 (Session 7 — staff report, rushed flag overhaul, Will affirmation fix, Belem E.)

**Staff performance report (Joi G. — Week One):**
Built as a static HTML report (print CSS for PDF export) using the same design principle as NOC reports — no JS rendering dependency. Features: character portraits (celebration poses for praise, standard for improvement section), Sam's Memoji + Joi's AI avatar embedded as base64, three-tier check-in time reference card, data from `Workaround_Sheet_-_Cara_Data.xlsx` (updated twice mid-session as new nights landed). Character copy workshopped with Sam: Remi (slow careful look / aging skin), Bea (care not love / presence), Will (tortoise and hare / sustainable pace). Delivery: HTML file via email or Netlify Drop — not session-bound.

**Rushed care overhaul — live testing and corrections:**
- SR, TH, AM removed from all rush detection (app alert + report flag + staff rushed count)
- First implementation: consecutive pattern tracked per-resident (same resident twice in a row). Live test revealed this never fired — Sam tested five different residents back-to-back, none triggered, because no single resident had two rushed check-ins in a row.
- Corrected to shift-wide consecutive tracking: any two consecutive rush-eligible check-ins (any resident) by the same staff member trigger the alert. `session.lastRushed` is a single boolean persisting across the whole shift (not a per-resident object), resetting only on sign-out. `Code.gs` updated to match — groups by staff, walks chronologically, skips exempt residents without breaking the pattern.
- Additional fix: `session.lastRushed` was also being reset at the start of each new round (`nextRound()`), which prevented cross-round detection. Removed that reset.
- Live-tested confirmed: alert fires on the second consecutive rushed eligible check-in after at least one prior check-in has established a baseline elapsed time. First check-in of a session always has null elapsed time and cannot be the "first" of a pair — expected behavior.
- Both `index.html` and `Code.gs` require deploy/redeploy for changes to take effect.

**Will round-end affirmation fix:**
- Two message pools: `STRONG_END_MSGS` (full round + zero rush triggers) and `GENERAL_END_MSGS` (all other completions)
- Will's "more work in one round than most people do in a day" line moved to `STRONG_END_MSGS` — it only fires when genuinely earned
- `session.roundHadRush` flag tracks whether any rush alert fired during the round; resets on `nextRound()`

**Staff added:** Belem E. (pin 0280)

**Items scoped but not yet built:**
- ~~Translation completion~~ — completed Session 9
- ~~Time-aware care prompts~~ — completed Session 9
- Dynamic resident config from Google Sheet (deferred until Workspace restored)
- `Cara_Resident_Setup_v2.xlsx` Day tab captured for Phase 2

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
