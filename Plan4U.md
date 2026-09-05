# Product Requirements Document: StudyQuest

**Version:** 1.0
**Team:** Tech-Titans
**Document Type:** PRD (reverse-engineered from existing codebase)
**Status:** Draft for review

---

## 1. Overview

### 1.1 Product Summary
StudyQuest is a gamified, multi-page study planner for students. It combines syllabus-based schedule generation, task management with proof-based verification, a Pomodoro focus timer, and an XP/level/badge progression system, all wrapped in a visually engaging, animated UI.

### 1.2 Problem Statement
Students struggle to convert a syllabus into an actionable daily study plan, stay consistent with study habits, and stay motivated over long exam-preparation periods. Most planning tools are either too rigid (static calendars) or too disconnected from actual study behavior (no feedback loop, no accountability).

### 1.3 Target Users
- School and college students preparing for exams
- Self-learners following a structured syllabus or curriculum
- Students who respond well to gamified motivation (XP, streaks, badges)

### 1.4 Product Goals
- Turn a syllabus into a realistic, day-by-day study schedule with minimal manual effort
- Increase study consistency through streaks, reminders, and visible progress
- Make task completion accountable via lightweight proof-of-work verification
- Provide clear analytics so students can see where their time actually goes
- Make the process of studying feel rewarding through game-like feedback (XP, levels, badges, confetti)

---

## 2. Scope

### 2.1 In Scope (Current Implementation)
- Client-side, browser-only web application (HTML/CSS/vanilla JS)
- Local heuristic syllabus parsing (no external AI/LLM call despite "AI-style" framing)
- Schedule generation, task management, Pomodoro timer, analytics dashboard
- Full gamification layer (XP, levels, streaks, badges)
- Data persistence via `localStorage` only

### 2.2 Out of Scope (Current Implementation)
- User accounts, authentication, or multi-device sync
- Backend server or database
- Real AI/LLM-based syllabus understanding
- Push notifications outside the browser session
- Data export/import or backup functionality

---

## 3. Tech Stack

| Layer | Technology |
|---|---|
| Structure | HTML5 (5-page multi-page app) |
| Styling | CSS3 — modular: `style.css`, `components.css`, `animations.css` |
| Logic | Vanilla JavaScript (ES6, script-based modules, no build step) |
| Charts | Chart.js (via CDN) |
| Persistence | Browser `localStorage` (JSON-serialized) |
| Hosting | Static — can run via direct file open or any static file server |

---

## 4. Information Architecture / Pages

| Page | File | Purpose |
|---|---|---|
| Landing | `index.html` | Product intro, hero section, CTA into the app |
| Dashboard | `dashboard.html` | Daily snapshot: hours studied, tasks done, streak, XP, today's schedule, badges |
| Planner | `planner.html` | 3-step wizard: upload syllabus → review parsed data → generate schedule |
| Tasks | `tasks.html` | Task CRUD, filtering, and proof-based completion flow |
| Analytics | `analytics.html` | Charts (weekly hours, tasks by subject, streak trend, daily completions) + Pomodoro timer |

Navigation: persistent sidebar (desktop) and bottom tab bar (mobile), present on all app pages except the landing page.

---

## 5. Functional Requirements

### 5.1 Landing Page
- FR-1.1: Display product introduction, animated hero, and feature highlights
- FR-1.2: Provide a primary CTA that routes to the Dashboard

### 5.2 Dashboard
- FR-2.1: Display today's study hours, tasks completed today, current streak, and total XP
- FR-2.2: Show a snapshot of today's generated schedule
- FR-2.3: Display earned badges and the next badge in progress
- FR-2.4: Show a motivational quote and quick-action shortcuts to Planner/Tasks

### 5.3 Planner
- FR-3.1: Accept syllabus input via file upload or pasted text
- FR-3.2: Parse input locally (heuristic parser) into course name, exam date, topics, difficulty, and estimated hours
- FR-3.3: Allow the user to review and manually edit all parsed fields before generation
- FR-3.4: Generate a multi-course schedule with a configurable intensity mode and daily study hours
- FR-3.5: Present schedule in both weekly and daily views
- FR-3.6: Allow marking individual schedule slots as done or skipped, with topic-level completion tracking
- FR-3.7: Award XP for schedule generation and for completed study blocks

### 5.4 Tasks
- FR-4.1: Support create, edit, delete, and filter operations on tasks (by subject, priority, status, due date)
- FR-4.2: On task completion, open a verification modal requiring one of:
  - A valid URL as proof
  - A media file upload (with file type/size validation)
  - An explicit "skip verification" option
- FR-4.3: Award different XP amounts depending on verification method (see Section 7)

### 5.5 Analytics
- FR-5.1: Display summary cards for total hours, completed tasks, current streak, and total XP
- FR-5.2: Render charts: weekly study hours, tasks by subject, streak trend, tasks completed per day (via Chart.js)
- FR-5.3: Provide a full Pomodoro timer (focus/break cycle) with session history logging

### 5.6 Global / Cross-Page
- FR-6.1: Persistent floating Pomodoro widget available on Dashboard, Planner, Tasks, and Analytics pages, independent of the full Analytics-page timer
- FR-6.2: Toast notification system for XP gains, confirmations, and errors
- FR-6.3: Daily login/streak check on app load (increments or resets streak based on last-active date)
- FR-6.4: Confetti celebration animation on milestone events (e.g., level-up)

---

## 6. Gamification System

### 6.1 XP Award Rules

| Action | XP Awarded |
|---|---|
| Daily login bonus | +10 |
| Streak maintained (consecutive day) | +15 |
| Schedule generated (Planner) | +30 |
| Study block completed (Planner) | +20 |
| Task completed with URL proof | +100 |
| Task completed with media proof | +150 |
| Task completed, verification skipped | +25 |
| Pomodoro focus session completed | +25 |
| Pomodoro break completed (floating widget) | +5 |

### 6.2 Level Thresholds

| Level | Name | XP Required |
|---|---|---|
| 1 | Beginner | 0 |
| 2 | Student | 200 |
| 3 | Learner | 500 |
| 4 | Scholar | 1,000 |
| 5 | Expert | 2,000 |
| 6 | Master | 4,000 |
| 7 | Legend | 8,000 |

### 6.3 Badges
- Unlocked based on defined milestone conditions in `gamification.js`
- Checked after every XP-awarding action

---

## 7. Data Model (localStorage)

All state is persisted client-side under the following keys (defined centrally in `js/app.js`):

| Key | Purpose |
|---|---|
| `studyquest_xp` | Total XP |
| `studyquest_level` | Current level |
| `studyquest_streak` | Current daily streak |
| `studyquest_tasks` | Task list |
| `studyquest_subjects` | Subject list |
| `studyquest_schedule` | Generated schedule |
| `studyquest_sessions` | Pomodoro session history |
| `studyquest_badges` | Unlocked badges |
| `studyquest_last_active` | Last active date (streak logic) |
| `studyquest_schedules_generated` | Count of schedules generated |
| `studyquest_total_study_hours` | Cumulative study hours |
| `studyquest_courses` | Planner-specific course data |
| `studyquest_timer_widget` | Floating widget's persistent timer state |

All reads/writes should route through the shared `getData()` / `saveData()` helpers and the `awardXP()` function in `js/app.js` / `js/gamification.js` to keep XP, level, badges, and header UI in sync.

---

## 8. Module Responsibilities

| File | Responsibility |
|---|---|
| `js/app.js` | Storage helpers, shared utilities, sidebar/mobile nav, header rendering, toasts, daily login/streak check |
| `js/gamification.js` | XP awarding (`awardXP`), level calculation, level-up overlay, badge definitions and checks |
| `js/planner.js` | Syllabus upload/parsing, course/topic review editing, schedule generation |
| `js/tasks.js` | Task CRUD, filters, verification modal, proof handling, completion XP |
| `js/analytics.js` | Summary metrics, Chart.js rendering |
| `js/pomodoro.js` | Full-page Pomodoro timer lifecycle, session recording, XP award (correctly via `awardXP()`) |
| `js/timer-widget.js` | Floating global timer widget (draggable, persistent across pages) |
| `js/confetti.js` | Canvas-based confetti celebration effects |

---

## 9. Known Issues / Technical Debt

These were identified during code review and should be tracked as bugs/backlog items:

1. **XP desync bug (High priority):** `js/timer-widget.js` awards XP for completed focus/break sessions by writing directly to `localStorage` instead of calling the shared `awardXP()` function used everywhere else (e.g., `js/pomodoro.js`). This means completing a Pomodoro session via the floating widget increases XP but **does not** recalculate level, trigger level-up overlays, check badge unlocks, or refresh the header UI — causing visible state to fall out of sync with actual XP until another `awardXP()` call happens elsewhere. Fix: replace the manual `localStorage` block in `onTimerComplete()` with `awardXP(25, ...)` / `awardXP(5, ...)` calls.
2. **No license file:** README notes no `LICENSE` is currently present; add one (e.g., MIT) before public distribution.
3. **Heuristic syllabus parser:** The Planner's "AI-style" parsing is local/heuristic, not a real AI model, and may require manual correction — this should be made clear in-product if the current UI implies otherwise.
4. **CDN dependency:** Analytics charts depend on Chart.js loading from a CDN; no offline fallback exists.

---

## 10. Known Limitations (By Design)

- No backend or database — all data is browser-local
- No cross-device or cross-browser sync
- Clearing browser storage/cache resets all progress permanently
- Notification and audio behavior for the Pomodoro timer depends on browser permissions

---

## 11. Non-Functional Requirements

- NFR-1: App must be usable fully offline after initial load (excluding Chart.js CDN dependency on the Analytics page)
- NFR-2: All pages must be responsive across desktop and mobile viewports (sidebar → bottom tab bar)
- NFR-3: State-changing actions (XP, tasks, schedule) must persist immediately to `localStorage`
- NFR-4: No page should have duplicate DOM `id` attributes (validated clean in current codebase)

---

## 12. Future Considerations (Suggested, Not Yet Implemented)

- Backend + accounts for cross-device sync and data backup
- Real LLM-based syllabus parsing for higher accuracy
- Push/browser notifications for scheduled study blocks
- Data export (CSV/JSON) and import
- Collaborative or social features (leaderboards, study groups)

---

*This PRD was reverse-engineered from the existing StudyQuest (Tech-Titans) codebase and README as of the current submission for hackathon review. It should be updated by the team to reflect actual product intent where it differs from current implementation.*
