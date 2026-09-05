# StudyQuest (code brakers)

StudyQuest is a multi-page, gamified study planner built with HTML, CSS, and vanilla JavaScript.
It helps students plan syllabus topics, manage tasks, run Pomodoro sessions, and track learning progress with analytics and XP-based progression.

## Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Pages and User Flow](#pages-and-user-flow)
4. [Project Structure](#project-structure)
5. [Tech Stack](#tech-stack)
6. [Data Model (localStorage)](#data-model-localstorage)
7. [JavaScript Module Responsibilities](#javascript-module-responsibilities)
8. [How to Run Locally](#how-to-run-locally)
9. [Gamification Rules](#gamification-rules)
10. [Known Limitations](#known-limitations)
11. [Contributing](#contributing)
12. [License](#license)

## Overview

StudyQuest is designed as a client-side web app with no backend dependency for core usage.
All app state is persisted in the browser using `localStorage`, which enables quick startup and offline-friendly behavior for most interactions.

The app includes:

- A landing page (`index.html`)
- A dashboard (`dashboard.html`)
- An AI-style planner workflow (`planner.html`)
- A task manager with proof-based completion (`tasks.html`)
- An analytics and Pomodoro page (`analytics.html`)

## Key Features

- AI-style syllabus parsing (local heuristic parser, no external LLM call)
- 3-step planner wizard: Upload -> Review -> Generate
- Multi-course schedule generation with intensity modes
- Task management with priority, due dates, filters, and edit/delete
- Task verification flow using URL or media proof before full XP rewards
- XP, level system, streaks, badges, and level-up overlays
- Dashboard cards for daily study progress and motivation
- Analytics charts powered by Chart.js
- Pomodoro timer (main analytics timer + global floating widget)
- Confetti celebration animations for milestone moments
- Responsive navigation with sidebar and mobile bottom tab bar

## Pages and User Flow

### 1) Landing (`studyquest/index.html`)

- Product intro and CTA to open the app (`dashboard.html`)
- Animated hero, feature highlights, and stats counter

### 2) Dashboard (`studyquest/dashboard.html`)

- Shows daily stats:
  - Study hours today
  - Tasks completed today
  - Current streak
  - Total XP
- Displays today's generated schedule snapshot
- Shows badges and next badge indicator
- Provides motivational quotes and quick-action shortcuts

### 3) Planner (`studyquest/planner.html`)

- Step 1: Upload syllabus file or paste text
- Step 2: Review parsed output (course name, exam date, topics, difficulty, hours)
- Step 3: Generate and view schedule (weekly and daily views)
- Supports marking slots as done/skipped and shows topic completion tracking

### 4) Tasks (`studyquest/tasks.html`)

- Create, edit, delete, and filter tasks
- Complete task flow opens verification modal:
  - URL proof (valid URL required)
  - Media upload proof (file type and size checks)
  - Optional skip verification with reduced XP reward

### 5) Analytics (`studyquest/analytics.html`)

- Summary cards for total hours, completed tasks, streak, and XP
- Charts:
  - Weekly study hours
  - Tasks by subject
  - Streak trend
  - Tasks completed per day
- Pomodoro focus/break timer with session history

## Project Structure

```text
Tech-Titans/
	README.md
	studyquest/
		analytics.html
		dashboard.html
		index.html
		planner.html
		tasks.html
		css/
			animations.css
			components.css
			style.css
		js/
			analytics.js
			app.js
			confetti.js
			gamification.js
			planner.js
			pomodoro.js
			tasks.js
			timer-widget.js
```

## Tech Stack

- HTML5 (multi-page app)
- CSS3 (modular stylesheets: base, components, animations)
- Vanilla JavaScript (ES6 style modules via script files)
- Browser `localStorage` for persistence
- Chart.js (loaded from CDN in `analytics.html`)

## Data Model (localStorage)

App state is persisted using these keys (defined in `js/app.js`):

- `studyquest_xp`
- `studyquest_level`
- `studyquest_streak`
- `studyquest_tasks`
- `studyquest_subjects`
- `studyquest_schedule`
- `studyquest_sessions`
- `studyquest_badges`
- `studyquest_last_active`
- `studyquest_schedules_generated`
- `studyquest_total_study_hours`
- `studyquest_courses` (planner-specific)
- `studyquest_timer_widget` (floating timer widget state)

### Example data shapes

Task object:

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

Schedule day object:

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

Pomodoro session object:

```json
{
  "date": "2026-03-12",
  "type": "focus",
  "duration": 25,
  "completedAt": "2026-03-12T10:15:00.000Z"
}
```

## JavaScript Module Responsibilities

- `js/app.js`
  - Shared utilities, storage helpers, date helpers
  - Sidebar and mobile menu behavior
  - Header rendering and daily login/streak checks
  - Toast notifications

- `js/gamification.js`
  - XP awards and level calculation
  - Level-up overlays
  - Badge definitions and unlock checks

- `js/planner.js`
  - Syllabus upload flow and parser logic
  - Course/topic review editing
  - Schedule generation and schedule interactions

- `js/tasks.js`
  - Task CRUD and filters
  - Verification modal logic and proof handling
  - Task completion XP workflow

- `js/analytics.js`
  - Analytics summary metrics
  - Chart.js chart rendering

- `js/pomodoro.js`
  - Analytics page Pomodoro timer lifecycle
  - Session recording and XP awards

- `js/timer-widget.js`
  - Floating global timer widget injected on pages
  - Draggable widget with persistent timer state

- `js/confetti.js`
  - Canvas-based confetti effects for celebrations

## How to Run Locally

This is a static frontend project.

### Option A: Open directly

1. Navigate to `studyquest/`.
2. Open `index.html` in your browser.

### Option B: Serve with a local static server (recommended)

Using Python:

```bash
cd studyquest
python -m http.server 5500
```

Then visit:

```text
http://localhost:5500
```

Using VS Code Live Server extension is also supported.

## Gamification Rules

Current implemented logic includes:

- Daily login bonus: +10 XP
- Streak maintain bonus (when consecutive): +15 XP
- Planner schedule generation: +30 XP
- Planner study block completion: +20 XP
- Task completion with URL proof: +100 XP
- Task completion with media proof: +150 XP
- Task completion with skipped verification: +25 XP
- Pomodoro focus completion: +25 XP
- Floating widget break completion: +5 XP

Level thresholds:

- Lv1: 0
- Lv2: 200
- Lv3: 500
- Lv4: 1000
- Lv5: 2000
- Lv6: 4000
- Lv7: 8000

Level names:

- Beginner, Student, Learner, Scholar, Expert, Master, Legend

## Known Limitations

- No backend/database; data is browser-local only
- Data is not synced across browsers/devices
- Clearing browser storage resets progress
- Syllabus "AI" parsing is heuristic/local and may require manual corrections
- Browser notification and audio behavior depends on user permissions and browser policy
- Chart.js depends on CDN availability in `analytics.html`

## Contributing

1. Fork the repository.
2. Create a feature branch.
3. Commit your changes with clear messages.
4. Open a pull request describing:
   - Problem
   - Approach
   - Validation steps

## License

No explicit license file is currently present in the repository.
Add a `LICENSE` file (for example MIT) if you want to define reuse permissions clearly.
