# Rules.md — StudyQuest Development Rules

This document defines the rules every contributor — human or AI agent (Antigravity, Claude Code, Cursor, etc.) — must follow when working on this codebase. It exists because this is a **no-build, vanilla HTML/CSS/JS project**: there is no compiler, no linter enforced at build time, and no type system to catch mistakes. These rules are the substitute for that safety net. Follow them exactly; do not "improve" the project by breaking them silently.

---

## 1. Golden Rules (never violate these)

1. **Never write directly to `localStorage`.** Always go through `getData(key, fallback)` / `saveData(key, value)` from `app.js`. Direct `localStorage.getItem`/`setItem` calls elsewhere in the codebase are bugs, not precedent to copy — see the `timer-widget.js` XP desync issue in `Architecture.md` §6 for what happens when this rule is broken.
2. **Never award XP by hand.** Any code path that grants the player XP must call `awardXP(amount, reason)` from `gamification.js`. Never do `xp += amount` and write it back manually — that skips level recalculation, badge checks, and the header re-render.
3. **Never rename or remove an existing HTML `id` or class name** without first searching every `.js` file for `getElementById`, `querySelector`, `classList`, and `dataset` references to it. JS is tightly coupled to specific IDs/classes across this codebase (see `Architecture.md` §3 for the dependency map); a silent rename breaks functionality with no compiler to catch it.
4. **Never introduce a build step, bundler, framework, or package manager dependency** (React, Vue, webpack, npm scripts, etc.) without an explicit decision to do so. This project is deliberately zero-build so it can be opened directly in a browser or served as static files.
5. **Never touch `.git/` contents directly.** Use git commands, not manual file edits inside `.git/`.

---

## 2. File & Folder Structure Rules

- All application code lives under `studyquest/`. Do not create a second top-level app folder.
- HTML pages stay flat in `studyquest/` (`index.html`, `dashboard.html`, `planner.html`, `tasks.html`, `analytics.html`). Do not nest pages into subfolders — every relative path (`css/...`, `js/...`) assumes this flat structure.
- CSS stays split three ways and each file keeps its current purpose:
  - `css/style.css` — design tokens, reset, layout, sidebar/header/nav, responsive rules, utility classes
  - `css/components.css` — reusable UI components (cards, buttons, forms, modals, badges, etc.)
  - `css/animations.css` — `@keyframes` and animation utility classes only
  - Do not put component styles in `style.css` or layout styles in `components.css`. If a new style doesn't clearly belong in one of the three, default to `components.css`.
- JS stays one file per responsibility, matching the existing module boundaries in `Architecture.md` §3. Do not merge multiple modules into one file, and do not split one module's logic across multiple files.
- New shared utilities go in `app.js`. New gamification logic (XP, levels, badges) goes in `gamification.js`. Do not duplicate a helper that already exists in one of these files.

---

## 3. Naming & Style Conventions

Match what's already in the codebase — do not introduce a second style:

- **JavaScript:**
  - `const` + arrow functions for top-level declarations (`const doThing = () => { ... }`), matching every existing file
  - camelCase for variables and functions (`getSubjectColor`, `checkDailyLogin`)
  - UPPER_SNAKE_CASE for constant lookup objects/maps (`KEYS`, `LEVEL_THRESHOLDS`)
  - 4-space indentation (no tabs)
  - No semicolon omission — existing code always terminates statements with `;`
  - No ES modules (`import`/`export`) — this project uses plain global-scope scripts loaded via `<script src="...">` tags in a specific order; keep it that way
- **CSS:**
  - kebab-case class names (`.stat-card`, `.xp-bar-fill`, `.levelup-overlay`)
  - All themeable values (colors, radius, shadow, spacing where reasonable) must reference a `var(--token)` from `:root` in `style.css` — never hardcode a hex color or px shadow value in a component rule if an equivalent token exists
  - New design tokens go in the `:root` block in `style.css`, nowhere else
- **HTML:**
  - kebab-case for `id` attributes (`add-task-btn`, `pomo-widget-card`)
  - Every new interactive element that JS needs to find must have a unique, descriptive `id` — don't rely on tag/position selectors for anything JS touches
- **Line endings:** the existing codebase uses CRLF (`\r\n`). Match the existing file's line-ending style when editing it; don't mix LF into a CRLF file.
- **Comments:** existing files use a consistent banner-comment style for section headers, e.g.:
  ```js
  /* ── Section Name ── */
  ```
  Use this style for new top-level sections in JS files rather than plain `//` block dividers.

---

## 4. Data & State Rules

- Every localStorage key must be registered in the `KEYS` object in `app.js` before use anywhere else. Do not invent a raw string key inline in a feature file.
- Any new persisted data shape (a new kind of object stored in localStorage) must be documented in `Architecture.md` §4.3 (Core Data Shapes) when added.
- State-changing UI actions (completing a task, generating a schedule, finishing a Pomodoro session) must, in this order:
  1. Update the relevant domain data via `saveData()`
  2. Call `awardXP()` if the action earns XP
  3. Rely on `awardXP()` (or, if no XP is involved, call `renderHeader()` directly) to refresh shared UI — don't manually patch header DOM elements from a page-specific module
- Don't assume single-tab usage without saying so. If a feature could break under multiple open tabs (no `storage` event listeners currently exist), note it as a known limitation rather than silently ignoring it.

---

## 5. UI / Styling Change Rules

(Relevant given prior full-redesign work on this project.)

- A visual/theme change must be achieved by editing CSS variable values and component CSS rules — **never** by changing HTML structure, IDs, or class names, and **never** by editing `.js` files, unless a class needs to be *added* (never renamed or removed) for a specific new visual state.
- Before changing any class's styling, confirm whether that class is referenced in `.js` (via `classList.add/remove/toggle` or `querySelector`) so you know if it carries functional meaning (e.g. `.completed`, `.open`, `.active`) versus being purely decorative.
- Any new animation must be added to `animations.css` as a named `@keyframes` + utility class, following the existing pattern (see `.fade-in-up`, `.pulse-glow`, `.badge-unlock` for reference), not as an inline `<style>` block in an HTML file.
- Maintain WCAG AA contrast (~4.5:1) between text and background tokens when changing the color palette.

---

## 6. Bug-Fix & Refactor Rules

- When fixing a bug, prefer reusing an existing correct implementation elsewhere in the codebase over writing new logic. (Example: `timer-widget.js`'s XP bug should be fixed by calling the same `awardXP()` pattern already used correctly in `pomodoro.js`, not by writing a new XP-handling function.)
- Do not silently fix unrelated issues while fixing a targeted bug. Flag anything else you notice as a separate item (in `Architecture.md` or a follow-up note) rather than bundling unrelated changes into one fix.
- Any fix that touches shared modules (`app.js`, `gamification.js`) must be manually verified against **every** page that loads that module, not just the page where the bug was noticed.
- Run `node --check <file>.js` on any JS file you edit before considering the change complete — there is no other automated syntax check in this project.

---

## 7. Rules for AI Coding Agents Specifically

If you are an AI agent (Antigravity, Claude Code, Cursor, etc.) operating on this repository:

1. **Read `Architecture.md` and this file before making changes.** Do not infer conventions purely from a single file you happen to open first.
2. **Do not add a package manager, build tool, or framework** unless the user explicitly asks for a stack migration. Treat "redesign the UI" or "fix this bug" requests as scoped to the existing vanilla stack.
3. **State your intended file changes before making them** when a task could plausibly touch shared modules (`app.js`, `gamification.js`) or more than 2 files — this project has no CI/tests to catch a wrong assumption, so a quick confirmation step substitutes for that.
4. **Never regenerate a whole file from scratch** when a targeted edit will do. Full-file rewrites make it hard for a human reviewer to see what actually changed in a no-diff-tooling, no-build project.
5. **After any change, explicitly list which of the 5 HTML pages are affected** and confirm (or ask the human to confirm) that each one still functions, since there's no automated test suite.
6. **When in doubt about a naming/style choice, match the nearest existing example in the same file**, not a general best-practice default — consistency with the existing 1,800+ lines of CSS and JS matters more than adopting a "more correct" pattern the rest of the codebase doesn't use.

---

## 8. What NOT to Add (Scope Guardrails)

Unless explicitly requested by the project owner:

- No backend server, database, or API layer
- No user authentication system
- No real AI/LLM integration replacing the current heuristic syllabus parser
- No TypeScript conversion
- No CSS framework (Tailwind, Bootstrap) — the project has its own hand-built design system
- No test framework setup — out of scope for this hackathon-stage project unless asked

---

*These rules exist to keep a no-build, no-linter, no-type-system codebase maintainable by both humans and AI agents. When a rule and a "better practice" conflict, follow the rule — consistency with the rest of this codebase is the priority over individual code quality in isolation.*
