# FORM. Personal Fitness Tracker — Full App Documentation

> Use this file with Claude chat to generate a formatted PDF, Notion doc, or Google Doc.
> Prompt suggestion: *"Turn this markdown into a clean, well-formatted document with a table of contents, sections, and clear headings. Make it readable as a reference guide."*

---

## Table of Contents

1. [Overview](#1-overview)
2. [Navigation](#2-navigation)
3. [Tab: Today](#3-tab-today)
4. [Tab: Dashboard](#4-tab-dashboard)
5. [Tab: Log](#5-tab-log)
6. [More Sheet — Secondary Pages](#6-more-sheet--secondary-pages)
7. [Data Storage](#7-data-storage)
8. [Data Flow — What Updates What](#8-data-flow--what-updates-what)
9. [Plan Structure](#9-plan-structure)

---

## 1. Overview

**FORM. Personal Fitness Tracker** is a personal training companion app built as a Progressive Web App (PWA). It is designed around a single 30-week training plan spanning June 7, 2026 through January 3, 2027, combining calisthenics, kettlebell training, and a structured running progression toward marathon readiness.

### Tech Stack
- **Single file:** `index.html` (~4,800 lines) — no framework, no build step
- **Storage:** Browser `localStorage` (all data stays on device)
- **Charts:** Chart.js (loaded via CDN)
- **PWA:** Installable on iPhone via Safari → Add to Home Screen
- **Deploy:** Netlify (static, no backend)

### What the App Does
- Shows today's scheduled workout session based on the current week and phase
- Tracks daily training logs, weekly check-ins, and monthly benchmark assessments
- Displays a live dashboard with performance stats, progress charts, and a training activity grid
- Manages phase milestones, training goals, and full plan reference
- Provides quick-log for parkrun times on Saturdays

### Important Note on Data Isolation
The PWA (home screen icon) and Safari browser have **separate localStorage**. Data logged in Safari does not appear in the PWA and vice versa. Use the Export/Import feature to transfer data between the two.

---

## 2. Navigation

### Bottom Nav Bar

The app has 4 navigation buttons fixed to the bottom of the screen:

| Button | Icon | Tab ID | What It Does |
|--------|------|--------|--------------|
| Today | 📋 | `today` | Shows today's training session |
| Dashboard | 📊 | `stats` | Stats, charts, history |
| Log | ✏️ | `log` | Daily log and catch-up log |
| More | ··· | — | Opens the More sheet overlay |

Tapping a tab calls `switchTab(tabId)`, which:
1. Closes any open secondary page
2. Activates the correct tab page and highlights the nav button
3. Scrolls to the top
4. Calls the appropriate render function for that tab

### Sub-Tabs

Two tabs have sub-tabs (displayed as a row of pills below the header):

**Dashboard tab:**
- Dashboard (default)
- History

**Log tab:**
- Daily (default)
- Catch-Up

Tapping a sub-tab calls `switchSubTab(parent, subId)`, which shows the correct content panel and triggers its render function.

### More Sheet

Tapping ··· opens a bottom sheet overlay with 5 cards in a 2-column grid:

| Card | Icon | Opens |
|------|------|-------|
| Weekly Check-in | 🏃 | Sunday/weekly log form |
| Monthly | 📅 | Full monthly assessment form |
| Goals | 🎯 | Year goal + monthly goal tracker |
| Profile | 👤 | Profile setup and baseline benchmarks |
| Plan Browser | 🗓️ | Full 30-week plan reference |

**Opening:** Tap ··· button → `openMoreSheet()`

**Closing (three ways):**
- Tap the dark overlay area outside the sheet
- Swipe the handle pill downward 60px or more
- Press Escape (keyboard shortcut)

### Secondary Pages

Each More card opens a full-screen page that slides in from the right. Tapping "← Back" closes it and returns to the previous tab.

---

## 3. Tab: Today

**Rendered by:** `renderTodayPlan()` — called every time the Today tab is opened.

**What it reads:** Profile (plan start date), current week/phase position, today's day of the week, and the phase's weekly schedule.

### Sections (in display order)

#### 1. Header
- Shows the current day name (e.g., "Saturday")
- **"Log Today" button** — tapping it pre-fills the Daily Log form with today's date and sessions, then switches to the Log tab. Uses sessionStorage to pass the pre-fill data.

#### 2. Setup Prompt *(conditional)*
Only shown if no plan start date has been set in Profile. Contains a button to open Profile.

#### 3. Today's Sessions
The main section. Determines what type of day it is and renders accordingly:

**Rest Day** — shown when today's schedule has no AM session or the session is marked "rest"
- Displays a rest card with recovery advice
- Monday rest days also show a weekly summary (Monday Summary) above the rest card

**Run Day** — shown when the AM session text contains keywords like "run", "parkrun", "lsd", "zone 2", "tempo"
- Session title, description, duration
- Week target box showing the specific distance or time goal for this week's run
- **Saturday only:** Shows the Parkrun Quick Log (see Log section)

**Gym/Calisthenics Day** — shown for all other AM sessions
- Session title, description, duration
- Exercise table with sets, reps, rest, and coaching notes pulled from `phase.workouts[dayKey]`

**PM Session** *(if scheduled)*
- Shown below the AM card in a blue-tinted card
- Thursday: includes PM run decision note based on recovery status
- PM run days also show a note about easy pace guidelines

**Recovery Alert** *(Thursday and Friday only)*
- Displayed after sessions on Thu/Fri
- Shows the Recovery Load Table: a guide mapping resting HR signals to training decisions

#### 4. Phase + Week Progress
Shows where you are in the plan:
- "Week X of 30" — overall plan progress bar
- Phase name and week-within-phase pips (dots):
  - Conditioning phase: 2 pips
  - All other phases: 4 pips
  - Completed weeks: dimmed green; current week: bright green with glow; future weeks: grey

#### 5. Milestone Checklist
Phase-specific achievement targets (e.g., "Hold a 60s plank", "Complete a 10km LSD run"). Each milestone has a circular checkbox:
- Tapping a checkbox calls `toggleMilestone(id)`
- State saved to `ft_milestones` in localStorage
- Persists across sessions and re-renders immediately
- When all milestones are checked: shows a "✓ Ready to advance" completion badge

#### 6. Quick Reference Accordions
Five collapsible reference panels (tap to expand/collapse):

| Accordion | Content |
|-----------|---------|
| 🔥 Universal Warm-Up | Dynamic mobility routine, exercise list with reps |
| ❄️ Universal Cool-Down | Static stretches with durations |
| 💓 Recovery Load Table | Resting HR thresholds mapped to training decisions |
| ⚠️ Injury Prevention | Common risks, causes, and prevention strategies |
| 📅 Full Weekly Template | Schedule grid: all 8 phases × all 7 days |

---

## 4. Tab: Dashboard

### Sub-Tab: Dashboard

**Rendered by:** `renderDashboard()` — called when the Dashboard tab or sub-tab is opened, and again when Profile is saved.

#### Sections (in display order)

**1. Header**
- Your name (from Profile) in large type
- Phase badge: "Wk X — Phase Name"
- Today's date
- "Week X of phase · Day Y of plan"

**2. Year Goal Banner**
- Displays your year-end goal text (set in Goals)
- If not set: shows a prompt link to add it

**3. Monthly Goal Progress**
- Shows the first active (unachieved) monthly goal
- Progress bar: `((current value − baseline) / (target − baseline)) × 100%`
- Current value is read from the most recent weekly check-in or monthly assessment for the selected metric

**4. Recovery Widget**
Compares your most recently logged resting HR (from any daily log) to your baseline HR (set in Profile):

| HR Elevation | Status | Recommendation |
|-------------|--------|----------------|
| Less than +5 bpm | 💚 Green | Good to train as planned |
| +5 to +6 bpm | 🟡 Amber | Reduce Thursday PM volume |
| +7 bpm or more | 🔴 Red | Skip Thursday PM — rest only |

If no resting HR has been logged, or no baseline is set in Profile, the widget shows no data.

**5. Today Preview**
- Shows today's AM session, PM session (if any), and duration
- Pulled from the current phase's weekly schedule
- "View full session →" button switches to the Today tab

**6. Training Activity Grid**
A visual log of training days since the plan start date:
- Each column = one week (Monday to Sunday)
- Each row = one day
- **Green dot** = at least one non-rest session logged that day
- **Grey** = no session logged
- **Transparent** = future date (not yet reached)
- Scrolls horizontally as weeks accumulate

**7. Performance Stat Cards**
Eight cards showing current values, recent change, and improvement from your starting baseline:

| Stat Card | Data Source | Updates When |
|-----------|-------------|--------------|
| Bodyweight | Weekly check-ins + monthly assessments | Weekly Check-in or Monthly Assessment saved |
| Parkrun 5km | Daily logs + weekly check-ins + monthly assessments | Parkrun Quick Log, Weekly Check-in, or Monthly Assessment |
| Push-Ups | Weekly check-ins + monthly assessments | Weekly Check-in or Monthly Assessment saved |
| Pull-Ups | Weekly check-ins + monthly assessments | Weekly Check-in or Monthly Assessment saved |
| Plank Hold | Weekly check-ins + monthly assessments | Weekly Check-in or Monthly Assessment saved |
| Dead Hang | Weekly check-ins + monthly assessments | Weekly Check-in or Monthly Assessment saved |
| Waist | Weekly check-ins + monthly assessments | Weekly Check-in or Monthly Assessment saved |
| Weekly Volume | Weekly check-ins only | Weekly Check-in saved |

Each card shows:
- Current value + unit
- **Week-to-week delta:** ↑ or ↓ from the previous logged entry (green = improvement, red = regression)
- **Baseline delta:** total improvement from your starting value set in Profile (shown in smaller text below)

For time-based metrics (Parkrun), lower is better — the delta arrows flip accordingly.

**8. Progress Charts**
Eight Chart.js charts plotting your historical data over time. All use a dark theme with acid-green lines/bars.

| Chart | Type | Data Source |
|-------|------|-------------|
| Bodyweight (kg) | Line | Monthly + weekly: `weight` |
| Parkrun Time | Line (inverted Y) | Daily + weekly + monthly: `parkrunTotalSec` |
| Push-Ups (reps) | Line | Monthly + weekly: `pushups` |
| Pull-Ups (reps) | Line | Monthly + weekly: `pullups` |
| Weekly Running Volume (km) | Bar | Weekly: `totalVolume` |
| Waist Circumference (cm) | Line | Monthly + weekly: `waist` |
| Plank Hold (seconds) | Line | Monthly + weekly: `plank` |
| Dead Hang (seconds) | Line | Monthly + weekly: `deadHang` |

Charts populate as you complete weekly check-ins and monthly assessments. They will be empty until at least one entry is saved.

---

### Sub-Tab: History

**Rendered by:** `renderHistory()`

Shows all logged entries (daily logs, weekly check-ins, monthly assessments) in a single merged list, sorted newest first.

**Filter buttons:** All · Daily · Weekly · Monthly

Each entry is a collapsible card. Tapping the card header expands it to show full detail.

**Daily card detail:** Sessions logged, resting HR, energy AM/PM, push-ups/pull-ups/plank, notes
**Weekly card detail:** Volume, parkrun time, LSD, run distances, weight, waist, benchmarks, sessions count
**Monthly card detail:** All benchmarks, gym lifts, energy, sleep, keto adherence, notes

**Delete button:** Available on expanded daily log cards only. Tapping it shows a confirmation prompt, then permanently removes the entry from storage and re-renders the history list.

---

## 5. Tab: Log

### Sub-Tab: Daily

**Rendered by:** `renderDailyLog()` | **Saved by:** `saveDailyLog()`

Use this every day to log what you did. The date defaults to today.

#### Fields

| Field | Type | Notes |
|-------|------|-------|
| Date | Date picker | Defaults to today |
| Sessions | Checkboxes | am_calisthenics, kb_plyo, hard_run, parkrun, lsd, pm_easy_run, gym, rest |
| Resting HR | Number input | In bpm; feeds the Dashboard recovery widget |
| Energy AM | Slider 1–10 | Morning energy level |
| Energy PM | Slider 1–10 | Only shown if a PM session is ticked |
| Push-Ups | Number input | Optional quick rep count |
| Pull-Ups | Number input | Optional quick rep count |
| Plank | Number input | In seconds, optional |
| Notes | Textarea | Free text |

#### Save Logic
- If no entry exists for this date: creates a new entry
- If an entry already exists for this date: **merges** sessions (union, no duplicates), keeps existing numeric values if the new form left them blank, updates notes if provided
- Shows "Daily log saved!" or "Daily log updated!" toast

#### Pre-fill from Today Tab
When you tap "Log Today" on the Today tab, it pre-fills the Daily Log form with today's date and checks the appropriate session boxes based on your phase schedule, then switches to this tab automatically.

---

### Sub-Tab: Catch-Up

**Rendered by:** `renderCatchUpLog()` | **Saved by:** `saveCatchUpLog()`

For logging a day you forgot to record. Defaults to yesterday's date; you can change it to any past date.

Fields are identical to Daily Log (except no PM Energy slider). Uses different HTML element IDs (`cu-*`) so both forms can coexist in the DOM without conflict. Saves to the same `ft_daily_logs` array with the same same-date merge logic.

---

### Parkrun Quick Log *(in Today tab — Saturdays only)*

A fast time-entry widget shown on Saturday's Today page. Enter minutes and seconds, tap "Save time":
- Saves `parkrunTotalSec` (total seconds) to today's daily log
- Adds `parkrun` to today's sessions
- Stashes the time in sessionStorage so the Weekly Check-in form auto-fills the parkrun fields when you open it
- Parkrun time immediately appears in the Dashboard Parkrun stat card

---

## 6. More Sheet — Secondary Pages

### Weekly Check-in

**Accessed via:** ··· → Weekly Check-in
**Rendered by:** `renderWeeklyCheckin()` | **Saved by:** `saveWeeklyCheckin()`
**Complete:** Every Sunday or Monday morning

This is the primary data source for most Dashboard stat cards and progress charts. Fill it in consistently for meaningful chart data.

#### Fields

| Section | Field | Notes |
|---------|-------|-------|
| Week | Week Ending (Sunday) | Date picker, defaults to most recent Sunday |
| Running | Parkrun time (min + sec) | Auto-filled if Parkrun Quick Log was used Saturday |
| Running | Wednesday run distance (km) | |
| Running | Wednesday tempo/intervals? | Yes / No radio |
| Running | Tuesday PM distance (km) | |
| Running | Tuesday PM status | Completed / Skipped / Walked |
| Running | Thursday PM distance (km) | |
| Running | Thursday PM status | Completed / Skipped / Walked |
| Running | Sunday LSD distance (km) | |
| Running | LSD perceived effort | Slider 1–10 |
| Calisthenics | Push-ups, Pull-ups, Dips | Max reps best set |
| Calisthenics | Plank, Dead Hang | Seconds, best hold |
| Body | Weight (kg), Waist (cm) | |
| Summary | Sessions completed (0–9) | |
| Notes | Free text | |

**Total Volume** is auto-calculated: `5km (parkrun) + Wednesday + Tuesday PM + Thursday PM + LSD`

Saved to `ft_weekly_checkins` array. Every save adds a new entry (no deduplication).

---

### Monthly Assessment

**Accessed via:** ··· → Monthly
**Rendered by:** `renderMonthlyAssessment()` | **Saved by:** `saveMonthlyAssessment()`

A full benchmark snapshot. Fill in at the end of each phase or at meaningful milestones.

#### Fields

| Section | Fields |
|---------|--------|
| Plan Position | Plan week (1–30), date |
| Body | Weight (kg), Waist (cm) |
| Calisthenics | Push-ups, Pull-ups, Chin-ups, Dips, Pistol squats/leg, Air squats in 60s, Toes-to-bar, L-sit (s), Plank (s), Dead Hang (s), Handstand (s) |
| Running | Parkrun time, Longest run (km), Wednesday peak distance (km), Monthly total volume (km) |
| Gym Lifts | Deadlift (kg), Weighted Pull-up added (kg), OHP (kg), KB Swing weight (kg) |
| Subjective | Avg energy (1–10), Avg sleep (hours), Keto adherence (Strict / Mostly / Off), Notes |

Saved to `ft_monthly_assessments` array. Every save appends a new record.

---

### Goals

**Accessed via:** ··· → Goals
**Rendered by:** `renderGoals()`

Two types of goals:

**Year-End Goal** — a free-text vision statement (e.g., "Run a sub-25 parkrun and complete a marathon"). Displayed as a banner on the Dashboard.

**Monthly Goals** — trackable targets with a metric, target value, and baseline:

| Field | Description |
|-------|-------------|
| Plan Week | Which week this goal targets (1–30) |
| Metric | The stat being tracked (see below) |
| Description | Human-readable goal label |
| Target | Numeric target value |
| Baseline | Starting value (used to calculate % progress) |

**Available metrics:** Bodyweight, Waist, Push-Ups, Pull-Ups, Plank, Dead Hang, Parkrun time, Weekly Volume, Deadlift, L-Sit, Handstand

The Dashboard progress bar always shows the **first unachieved goal** from the list. Tap "Mark Achieved" to retire a goal and advance to the next.

---

### Profile

**Accessed via:** ··· → Profile
**Pre-rendered** as a static HTML form (not dynamically rendered on open).

The most important field is **Plan Start Date** — set this to June 7, 2026 (or whenever you actually started). Every week number, phase position, progress bar, and milestone is calculated from this date.

#### Fields

| Section | Fields |
|---------|--------|
| Personal | Name, Age, Height (cm) |
| Body | Starting Weight (kg), Waist (cm), Resting HR (bpm) |
| Plan Setup | Plan Start Date (YYYY-MM-DD) |
| Baseline — Calisthenics | Push-ups, Pull-ups, Chin-ups, Dips, Air squats (60s), Pistols/leg, Plank (s), Dead Hang (s), Toes-to-bar, L-Sit (s), Handstand (s) |
| Baseline — Running | Parkrun time (min + sec) |
| Baseline — Gym | Deadlift (kg), Weighted Pull-up (kg), OHP (kg), KB Swing (kg) |
| Data | Export / Import buttons |

Baseline values appear as "from baseline" deltas on every Dashboard stat card.

**Export:** Downloads `fitness-tracker-backup-{date}.json` containing all stored data.
**Import:** Restores all data from a previously exported file. Use this to sync data between Safari and the PWA.

---

### Plan Browser

**Accessed via:** ··· → Plan Browser
**Rendered by:** `renderPlanBrowserPage()`

A read-only reference showing all 8 phases as expandable accordions. Each phase accordion contains:
- Theme description
- Full 7-day weekly schedule table (AM/PM/duration for every day)
- Workout details for each training day (exercises, sets, reps, rest, coaching notes)
- Running plan by week (parkrun target, LSD distance, Wednesday run, PM runs)
- Optional nutrition guidance

---

## 7. Data Storage

All data is stored in the browser's `localStorage`. There is no server or cloud backup. Data is isolated to the origin (URL) and PWA context.

### Storage Keys

| App Key | localStorage String | Type | What It Holds |
|---------|---------------------|------|---------------|
| `profile` | `ft_profile` | Object | User profile + all baseline benchmarks |
| `daily` | `ft_daily_logs` | Array | All daily log entries (including catch-up and parkrun quick-log) |
| `weekly` | `ft_weekly_checkins` | Array | All weekly check-in entries |
| `monthly` | `ft_monthly_assessments` | Array | All monthly assessment entries |
| `goals` | `ft_goals` | Object | Year goal string + monthly goals array |
| `milestones` | `ft_milestones` | Array | IDs of completed phase milestones |
| `lastExport` | `ft_last_export` | String | Timestamp of last data export |

### Daily Log Entry Schema

```json
{
  "id": 1749295200000,
  "date": "2026-06-07",
  "sessions": ["parkrun", "am_calisthenics"],
  "restingHR": 52,
  "energyAM": 8,
  "energyPM": 7,
  "pushups": 25,
  "pullups": 8,
  "plank": 90,
  "parkrunTotalSec": 1714,
  "notes": "Parkrun: 28:34. Felt strong."
}
```

### Weekly Check-in Entry Schema

```json
{
  "id": 1749295200000,
  "weekEnding": "2026-06-07",
  "parkrunMin": 28,
  "parkrunSec": 34,
  "parkrunTotalSec": 1714,
  "wedDist": 5.2,
  "wedTempo": false,
  "tueDist": 4.0,
  "tueStatus": "completed",
  "thuDist": 5.0,
  "thuStatus": "completed",
  "lsdDist": 8.0,
  "lsdRPE": 6,
  "totalVolume": 27.2,
  "pushups": 27,
  "pullups": 9,
  "dips": 12,
  "plank": 95,
  "deadHang": 40,
  "weight": 82.5,
  "waist": 88.0,
  "totalSessions": 7,
  "notes": "Good week overall."
}
```

### Profile Schema

```json
{
  "name": "Raeez",
  "age": 30,
  "height": 178,
  "startingWeight": 85,
  "waistCm": 92,
  "restingHR": 54,
  "startDate": "2026-06-07",
  "savedAt": "2026-06-07T08:00:00.000Z",
  "baseline": {
    "pushups": 20,
    "pullups": 5,
    "plank": 60,
    "deadHang": 25,
    "parkrunMin": 30,
    "parkrunSec": 45,
    "deadlift": 100
  }
}
```

### Storage Error Handling
If localStorage is full, `storageSet()` catches the error and shows a toast: *"Storage full — export your data to free space."* No data is lost on read errors — `storageGet()` returns `null` silently.

---

## 8. Data Flow — What Updates What

This table shows every user action, where it writes, and what parts of the app it affects on next render.

| User Action | Written To | What It Drives |
|-------------|-----------|----------------|
| Parkrun Quick Log (Today tab, Saturday) | `ft_daily_logs` — today's entry | Activity grid green dot; Parkrun stat card (immediately); Weekly Check-in parkrun field auto-fill |
| Daily Log saved | `ft_daily_logs` | Activity grid; History list; Recovery widget (if resting HR logged) |
| Catch-Up Log saved | `ft_daily_logs` (past date) | History list; Activity grid for that past date |
| Weekly Check-in saved | `ft_weekly_checkins` | All 8 stat cards; all 8 progress charts; Weekly Volume stat |
| Monthly Assessment saved | `ft_monthly_assessments` | All 8 stat cards; all 8 progress charts; baseline delta comparisons |
| Profile saved (including start date) | `ft_profile` | Week number + phase position everywhere; baseline deltas on stat cards; recovery widget baseline HR; dashboard name display |
| Milestone checkbox toggled | `ft_milestones` | Milestone checklist state; completion badge on Today tab |
| Year goal saved | `ft_goals.yearGoal` | Dashboard year goal banner |
| Monthly goal added/achieved/deleted | `ft_goals.monthlyGoals` | Dashboard monthly goal progress bar |
| History card deleted (daily only) | `ft_daily_logs` (removes entry by ID) | History list re-renders; activity grid updates |

### Key Dependency: Plan Start Date

The plan start date (set in Profile) is the single most important piece of data. Every week number, phase, session, milestone set, and progress bar in the entire app is calculated from it:

```
daysElapsed = today − startDate
weekNo      = min(floor(daysElapsed / 7) + 1, 30)
phase       = first phase where startWeek ≤ weekNo ≤ endWeek
weekInPhase = weekNo − phase.startWeek + 1
```

If the start date is not set, the app defaults to Week 1, Phase 0 (Conditioning).

---

## 9. Plan Structure

### 30-Week Overview

The plan covers June 7, 2026 through January 3, 2027. It is divided into 8 phases:

| Phase | Weeks | Dates | Training Days |
|-------|-------|-------|---------------|
| Conditioning | 1–2 | Jun 7 – Jun 20 | 2 gym days/week |
| Phase 1: Foundation | 3–6 | Jun 21 – Jul 18 | 3 gym days/week |
| Phase 2: Base Build | 7–10 | Jul 19 – Aug 15 | 3 gym + Sunday mobility |
| Phase 3: Strength Foundations | 11–14 | Aug 16 – Sep 12 | 3 gym + active Sunday |
| Phase 4: Strength & Endurance | 15–18 | Sep 13 – Oct 10 | 4 full training days |
| Phase 5: Intensity | 19–22 | Oct 11 – Nov 7 | 4 full days + Monday mobility |
| Phase 6: Peak Conditioning | 23–26 | Nov 8 – Dec 5 | 4 full training days |
| Phase 7: Performance | 27–30 | Dec 6 – Jan 3 2027 | 4 full days, intervals, 30km LSD |

### Deload Weeks
Weeks 6, 10, 14, 18, 22, and 26 are deload weeks. The running plan marks these with reduced targets. Tuesday and Thursday PM runs are dropped in deload weeks.

### Phase Data Structure
Each phase contains:
- **Weekly Schedule** — AM/PM sessions and duration for all 7 days
- **Workouts** — exercise tables per training day (sets, reps, rest, coaching notes)
- **Milestones** — 2–4 achievement targets to complete before advancing
- **Running Plan** — week-by-week targets for parkrun, LSD, Wednesday, and PM runs
- **Theme** — narrative description of the phase's focus

### Running Plan Cadence (Phase 1 example)
Each week in the running plan specifies:
- `satParkrun` — Saturday parkrun goal (e.g., "5km easy, no time pressure")
- `sunLSD` — Sunday long slow distance (e.g., "30 min easy zone 2")
- `wed` — Wednesday run (e.g., "25 min zone 2")
- `tuePM` / `thuPM` — evening run targets (added in later phases)
- `deload: true` — optional flag marking a recovery week

---

*Generated from source code — `index.html`, FORM. Personal Fitness Tracker PWA*
*Plan: Jun 7, 2026 → Jan 3, 2027 · Hosted at https://formfitnesstracker.netlify.app*
