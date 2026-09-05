# Architecture: StudyQuest

**Project:** StudyQuest (Tech-Titans)
**Type:** Static multi-page web application
**Stack:** HTML5, CSS3, Vanilla JavaScript (ES6), Chart.js (CDN)
**Persistence:** Browser `localStorage`
**Backend:** None (fully client-side)

---

## 1. High-Level Architecture

StudyQuest is a **static, client-only multi-page application (MPA)** — not a single-page app, and not framework-based. There is no build step, bundler, or transpilation; every HTML page loads its own set of `<script>` tags directly, and every JS file is a plain script (no ES modules, no `import`/`export`).

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser (Client)                      │
│                                                                │
│  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌─────────────┐ │
│  │ index    │  │ dashboard │  │ planner  │  │ tasks       │ │
│  │ .html    │─▶│ .html     │◀▶│ .html    │◀▶│ .html       │ │
│  └──────────┘  └─────┬─────┘  └────┬─────┘  └──────┬──────┘ │
│                       │             │               │        │
│                       └─────────┬───┴───────────────┘        │
│                                 ▼                             │
│                       ┌──────────────────┐                   │
│                       │  analytics.html  │                   │
│                       └──────────────────┘                   │
│                                 │                             │
│                                 ▼                             │
│                     ┌────────────────────┐                   │
│                     │  Shared JS Layer    │                  │
│                     │  (app.js,           │                  │
│                     │   gamification.js,  │                  │
│                     │   confetti.js,      │                  │
│                     │   timer-widget.js)  │                  │
│                     └──────────┬──────────┘                  │
│                                 ▼                             │
│                     ┌────────────────────┐                   │
│                     │  localStorage       │                  │
│                     │  (single source of  │                  │
│                     │   truth, JSON keys) │                  │
│                     └────────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

There is no server round-trip for app data. The only external network dependency is the Chart.js library, loaded from a CDN on `analytics.html`.

---

## 2. Page Inventory & Responsibilities

| Page | Loads (in order) | Responsibility |
|---|---|---|
| `index.html` | inline `<script>` only | Marketing/landing page, CTA into the app. No shared modules loaded — fully standalone. |
| `dashboard.html` | `confetti.js` → `app.js` → `gamification.js` → inline script → `timer-widget.js` | Daily summary view: stats, today's schedule snapshot, badges, quick actions. |
| `planner.html` | `confetti.js` → `app.js` → `gamification.js` → `planner.js` → `timer-widget.js` | 3-step wizard: upload/parse syllabus → review → generate schedule. |
| `tasks.html` | `confetti.js` → `app.js` → `gamification.js` → `tasks.js` → `timer-widget.js` | Task CRUD, filtering, proof-based completion flow. |
| `analytics.html` | Chart.js (CDN) → `confetti.js` → `app.js` → `gamification.js` → `pomodoro.js` → `analytics.js` → inline script → `timer-widget.js` | Charts, summary metrics, full-page Pomodoro timer. |

**Note:** `gamification.js` is always loaded before `timer-widget.js` on every page that includes both, so `awardXP()` and its dependencies (`KEYS`, `getData`, `saveData`) are guaranteed to exist by the time `timer-widget.js` runs.

---

## 3. JavaScript Module Layer

### 3.1 Shared / Cross-Page Modules
These are included on every "app" page (all except `index.html`) and provide functionality the whole app depends on.

- **`app.js`** — Foundational layer, loaded first among the custom modules.
  - Defines `KEYS` (the canonical map of all localStorage key names)
  - `getData(key, fallback)` / `saveData(key, value)` — the only sanctioned read/write path to localStorage (JSON-serialized)
  - Date utilities: `formatDate`, `formatTime`, `getToday`, `isToday`, `isPast`, `daysUntil`
  - `generateId()` — ID generation for tasks/courses
  - `showToast(message, type)` — global toast notification system
  - `checkDailyLogin()` — streak logic, run once per page load
  - `renderHeader()` — re-renders XP bar, level badge, streak badge in the header
  - `initSidebar()` / `initMobileMenu()` — navigation behavior
  - `initApp()` — orchestrates the above, bound to `DOMContentLoaded`

- **`gamification.js`** — Depends on `app.js` (uses `KEYS`, `getData`, `saveData`, `showToast`).
  - `calculateLevel(xp)` — derives level from XP against fixed thresholds
  - `awardXP(amount, reason)` — **the single authoritative XP-award function.** Updates XP, recalculates and saves level, fires toast, triggers level-up overlay if leveled up, checks badge unlocks, re-renders header.
  - Badge definitions and `checkBadges()`

- **`confetti.js`** — Standalone canvas-based confetti animation, invoked by other modules on celebration moments (e.g. level-up, badge unlock).

- **`timer-widget.js`** — Floating, draggable Pomodoro widget injected into the DOM on every app page except `index.html`.
  - Self-contained state machine (`state.running`, `state.mode`, `state.timeLeft`, etc.), persisted under `studyquest_timer_widget`
  - Injects its own DOM (bubble + card) at runtime rather than relying on markup already in the HTML
  - **Known issue:** currently writes XP directly to `localStorage` instead of calling `awardXP()` — see Section 6.

### 3.2 Page-Specific Modules
Each loads only on its own page and self-initializes via its own `DOMContentLoaded` listener.

- **`planner.js`** (`planner.html`) — Syllabus upload/parse (local heuristic parser, not a real AI call), course/topic review editing, multi-course schedule generation, weekly/daily schedule views, slot completion tracking.
- **`tasks.js`** (`tasks.html`) — Task CRUD, filtering, the proof-verification modal (URL / media / skip), XP awarding on completion (via `awardXP()`).
- **`analytics.js`** (`analytics.html`) — Computes summary metrics, renders all Chart.js charts (weekly hours, tasks by subject, streak trend, daily completions).
- **`pomodoro.js`** (`analytics.html`) — Full-page Pomodoro timer lifecycle, session history logging, XP awarding via `awardXP()` (the correct reference implementation compared to `timer-widget.js`).

### 3.3 Module Dependency Graph

```
app.js  (defines KEYS, getData, saveData, showToast, renderHeader)
  │
  ├──▶ gamification.js  (defines awardXP, calculateLevel, checkBadges)
  │        │
  │        ├──▶ tasks.js        (calls awardXP on task completion)
  │        ├──▶ planner.js      (calls awardXP on schedule gen / block done)
  │        ├──▶ pomodoro.js     (calls awardXP on focus session — CORRECT)
  │        └──▶ timer-widget.js (writes localStorage directly — BUG, see §6)
  │
  └──▶ confetti.js  (invoked by gamification.js / others on celebrations)
```

---

## 4. Data Architecture

### 4.1 Storage Mechanism
All persistent state lives in `localStorage`, accessed exclusively (by convention) through `getData()` / `saveData()` in `app.js`, which handle JSON serialization/deserialization. There is no IndexedDB, no cookies, and no server-side storage.

### 4.2 Storage Keys

| Key | Shape | Written by |
|---|---|---|
| `studyquest_xp` | `number` | `gamification.js` (`awardXP`), `timer-widget.js` (bug — bypasses `awardXP`) |
| `studyquest_level` | `number` | `gamification.js` (`awardXP`) only |
| `studyquest_streak` | `number` | `app.js` (`checkDailyLogin`) |
| `studyquest_tasks` | `Task[]` | `tasks.js` |
| `studyquest_subjects` | `Subject[]` | `tasks.js`, `planner.js` |
| `studyquest_schedule` | `ScheduleDay[]` | `planner.js` |
| `studyquest_sessions` | `Session[]` | `pomodoro.js`, `timer-widget.js` |
| `studyquest_badges` | `string[]` | `gamification.js` (`checkBadges`) |
| `studyquest_last_active` | `string` (date) | `app.js` (`checkDailyLogin`) |
| `studyquest_schedules_generated` | `number` | `planner.js` |
| `studyquest_total_study_hours` | `number` | `planner.js`, `pomodoro.js`, `timer-widget.js` |
| `studyquest_courses` | `Course[]` | `planner.js` |
| `studyquest_timer_widget` | `object` (widget state) | `timer-widget.js` |

### 4.3 Core Data Shapes

**Task:**
```json
{
  "id": "abc123",
  "name": "Review Chapter 5",
  "subject": "Mathematics",
  "dueDate": "2026-03-20",
  "priority": "High",
  "completed": false,
  "completedAt": null,
  "notes": "Focus on integrals",
  "proof": null
}
```

**Schedule Day:**
```json
{
  "date": "2026-03-15",
  "slots": [
    {
      "topicId": "topic_1",
      "topicName": "Integration Basics",
      "subjectName": "Mathematics",
      "subjectColor": "#4F8EF7",
      "difficulty": "Medium",
      "duration": 90,
      "startTime": "09:00",
      "endTime": "10:30",
      "completed": false,
      "skipped": false
    }
  ]
}
```

**Pomodoro Session:**
```json
{
  "date": "2026-03-12",
  "type": "focus",
  "duration": 25,
  "completedAt": "2026-03-12T10:15:00.000Z"
}
```

### 4.4 Data Flow Pattern
The app follows a simple, repeated pattern with no central state manager or reactive framework:

1. Page loads → `DOMContentLoaded` fires
2. `app.js`'s `initApp()` runs first (shared across all app pages): daily login check, header render, nav init
3. Page-specific module's own `DOMContentLoaded` handler runs: reads relevant keys via `getData()`, renders the page
4. User interacts (completes a task, finishes a Pomodoro session, generates a schedule)
5. The relevant module calls `awardXP()` (or, in the case of `timer-widget.js`'s bug, writes localStorage directly) and `saveData()` for its own domain data
6. `awardXP()` internally calls `renderHeader()` to reflect new XP/level immediately — but only when `awardXP()` is actually used

This means UI freshness after a state change depends entirely on the acting module remembering to call the right shared functions — there is no automatic re-render or observer pattern tying localStorage changes to the DOM.

---

## 5. Styling Architecture

Three CSS files, loaded in the same order on every app page: `style.css` → `components.css` → `animations.css`.

| File | Contents |
|---|---|
| `css/style.css` | Design tokens (`:root` CSS variables), reset, base typography, sidebar, header, main layout, bottom tab bar, responsive breakpoints, utility classes (`.text-muted`, `.flex-center`, spacing helpers) |
| `css/components.css` | Reusable UI components: cards, buttons, forms, modals, toasts, badges, task cards, filter pills, tab switcher, weekly grid, Pomodoro display, level-up overlay |
| `css/animations.css` | `@keyframes` definitions and animation utility classes (fade/slide-in, floating shapes, pulse effects, toast in/out, level-up bounce/spin, confetti fall, shimmer/skeleton loading, badge unlock) |

**Design tokens** (`css/style.css` `:root`): `--bg-primary`, `--bg-card`, `--accent-blue/green/yellow/purple/red/orange/pink`, `--text-primary`, `--text-muted`, `--border`, `--shadow`, `--shadow-hover`, `--radius`, `--radius-sm`, `--radius-full`, `--sidebar-width`, `--header-height`, `--transition`.

Nearly all component styles reference these variables rather than hardcoding values, which makes global re-theming (see the earlier full-redesign work) tractable without touching HTML or JS.

---

## 6. Known Architectural Issue

**XP state can desync from Level/Badge state.**

`awardXP()` in `gamification.js` is meant to be the single write path for XP because it atomically keeps XP, level, badges, and the header UI in sync. `timer-widget.js` does not go through this function — it manually reads/writes the `studyquest_xp` key in `onTimerComplete()`, meaning completing a Pomodoro session via the floating widget increases XP without recalculating level, checking badges, or refreshing the header. This is an architectural gap between the "correct" data flow (module → `awardXP()` → all dependent state updates) and the "shortcut" flow that `timer-widget.js` currently takes. (Full fix already specified separately — see prior conversation / bug ticket.)

**Architectural takeaway:** any new module that awards XP must call `awardXP(amount, reason)` rather than touching `studyquest_xp` directly, or it will silently reproduce this same class of bug.

---

## 7. External Dependencies

| Dependency | Used in | Loaded via |
|---|---|---|
| Chart.js | `analytics.html` only | CDN (`cdn.jsdelivr.net`) |
| Google Fonts (Plus Jakarta Sans, DM Sans) | All pages | `@import` in `css/style.css` |

No package manager, no `node_modules`, no build tooling is part of the runtime — these are the only two network dependencies, both optional in the sense that the app degrades (but doesn't crash) without them.

---

## 8. Deployment Model

Because there is no backend and no build step, deployment is just static file hosting:

- **Local:** open `studyquest/index.html` directly, or serve via `python -m http.server` / VS Code Live Server
- **Production:** any static host (GitHub Pages, Netlify, Vercel static, S3 + CloudFront, etc.) — copy the `studyquest/` directory as-is

There is no environment configuration, no API keys (beyond the optional Chart.js CDN), and no server process to manage.

---

## 9. Architectural Constraints & Trade-offs

| Constraint | Implication |
|---|---|
| No backend | Zero infrastructure cost and instant deploy, but no cross-device sync, no data backup, and progress is lost if browser storage is cleared |
| No build step | Fast to iterate on for a hackathon timeline; but no minification, no module bundling, no TypeScript safety net — bugs like the `timer-widget.js` XP desync are easy to introduce and hard to catch without manual review |
| No reactive framework | Simple mental model (imperative render-on-load, render-after-action), but every new feature that changes shared state must remember to manually call the right shared functions (`awardXP`, `renderHeader`, `saveData`) — there's no single source of truth enforcing this |
| localStorage only | No conflict resolution needed (single browser, single tab assumption); breaks down if the app is ever used in multiple tabs simultaneously (no storage event listeners currently wire tabs together) |

---

*This document describes the architecture as implemented in the current codebase (Tech-Titans / StudyQuest), reverse-engineered from source for hackathon documentation purposes.*
