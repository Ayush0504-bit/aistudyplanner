# Phases.md — StudyQuest Development Roadmap

This document breaks StudyQuest's development into phases: what's already built, what's in progress, and what's planned. It exists to give the team (and hackathon judges, if relevant) a clear picture of project maturity and sequencing. Read alongside `PRD.md` (what the product should do), `Architecture.md` (how it's built), and `Rules.md` (how to change it safely).

---

## Phase 0 — MVP (Completed)

**Status: ✅ Done — this is the current shipped state of the codebase.**

The core product as described in `PRD.md` is fully implemented:

- 5-page multi-page app: Landing, Dashboard, Planner, Tasks, Analytics
- Local heuristic syllabus parser (3-step Planner wizard: upload → review → generate)
- Multi-course schedule generation with weekly/daily views and slot completion tracking
- Task management with CRUD, filters, and a proof-based verification flow (URL / media / skip)
- Full gamification layer: XP, levels, streaks, badges, level-up overlays
- Analytics page with Chart.js visualizations and a full-page Pomodoro timer
- Floating, draggable global Pomodoro widget across app pages
- Confetti celebration animations
- Responsive layout (sidebar on desktop, bottom tab bar on mobile)
- Complete localStorage-based persistence layer, no backend required

**Known debt carried out of this phase:** the `timer-widget.js` XP desync bug (documented in `Architecture.md` §6) — the floating widget awards XP by writing to `localStorage` directly instead of calling `awardXP()`, so level/badge/header state can fall out of sync when a session is completed via the widget.

---

## Phase 1 — Stabilization (Next, Highest Priority)

**Goal:** fix known bugs and harden the existing MVP before adding new surface area or changing its look.

| Task | Priority | Reference |
|---|---|---|
| Fix `timer-widget.js` XP desync — route XP awards through `awardXP()` instead of manual `localStorage` writes | High | `Architecture.md` §6, `Rules.md` §6 |
| Remove dead `KEYS_LEVEL` / redundant `KEYS_XP` local constants left over from the buggy implementation | Low | — |
| Verify no duplicate toast notifications after the XP fix (widget's mini-toast vs. `awardXP()`'s global toast) | Medium | — |
| Add a `LICENSE` file (README currently notes none exists) | Low | `PRD.md` §9 |
| Manually regression-test all 5 pages after the fix (no automated test suite exists yet) | High | `Rules.md` §6 |

**Exit criteria:** completing a Pomodoro session via the floating widget produces identical XP/level/badge/header behavior to completing one via the full Analytics-page timer.

---

## Phase 2 — Visual Redesign

**Goal:** full UI reskin, preserving all existing functionality and DOM/JS contracts.

| Task | Priority | Reference |
|---|---|---|
| Choose one committed visual direction (dark deep-work / warm editorial / bold RPG / neubrutalism, or a custom direction) | High | Prior redesign prompt work |
| Update design tokens in `css/style.css` `:root` (colors, typography, radius, shadow) | High | `Architecture.md` §5, `Rules.md` §3/§5 |
| Restyle high-visibility gamification components with extra care: XP bar, level badge, level-up overlay, badge grid | High | `Rules.md` §5 |
| Retune (not rename) animations in `css/animations.css` to match new theme | Medium | `Rules.md` §5 |
| Update Chart.js color scheme in `analytics.js` if new theme clashes with default chart colors | Medium | Flagged as follow-up in redesign prompt |
| Verify responsive breakpoints (sidebar ↔ bottom tab bar) still work post-restyle | High | `Architecture.md` §5 |
| Accessibility pass: confirm WCAG AA contrast on new palette | Medium | `Rules.md` §5 |

**Exit criteria:** all 5 pages render the new visual direction consistently; zero HTML/JS files changed; all existing interactions (modals, timer, XP bar, charts) work identically to Phase 1.

**Note:** This phase should only start after Phase 1 is complete — redesigning on top of a known-buggy interaction (the widget XP bug) makes it harder to tell whether a future issue is visual or functional.

---

## Phase 3 — Documentation & Process Hardening

**Goal:** make the project maintainable beyond the original team / hackathon window.

| Task | Priority |
|---|---|
| Keep `Architecture.md`, `Rules.md`, and `PRD.md` in sync as features change | Ongoing |
| Add inline code comments for any non-obvious logic introduced in Phases 1–2 | Medium |
| Document the badge-unlock conditions currently defined in `gamification.js` (not fully detailed in current docs) | Medium |
| Establish a lightweight manual QA checklist (5 pages × key interactions) to run before any merge, given there's no automated test suite | Medium |

---

## Phase 4 — Feature Expansion (Future, Not Yet Scheduled)

These map to the "Future Considerations" section of `PRD.md` §12. None are started; sequencing below is suggested, not committed.

| Feature | Depends On | Notes |
|---|---|---|
| Backend + user accounts for cross-device sync | Architectural shift (see `Rules.md` §8 — currently out of scope) | Would require introducing a server, a database, and auth — a significant departure from the current zero-backend model |
| Real LLM-based syllabus parsing (replacing the heuristic parser) | Backend or client-side API integration | Could initially be added client-side via a direct API call before a full backend exists |
| Browser/push notifications for scheduled study blocks | None (client-only) | Feasible within current architecture using the Notifications API |
| Data export/import (CSV/JSON) | None (client-only) | Feasible within current architecture; addresses the "clearing storage resets everything" limitation from `PRD.md` §10 |
| Social features (leaderboards, study groups) | Backend + accounts | Long-term; not feasible in the current client-only model |

**Sequencing rationale:** notifications and data export/import are achievable without breaking the current zero-backend architecture and should be considered before backend-dependent features (accounts, sync, social) that require a fundamental architectural shift documented as out-of-scope in `Rules.md` §8.

---

## Phase Summary Timeline

```
Phase 0: MVP                    ████████████████████ COMPLETE
Phase 1: Stabilization          ░░░░░░░░░░░░░░░░░░░░ NEXT (bug fixes)
Phase 2: Visual Redesign        ░░░░░░░░░░░░░░░░░░░░ PLANNED (after Phase 1)
Phase 3: Docs & Process         ░░░░░░░░░░░░░░░░░░░░ ONGOING (parallel to 1–2)
Phase 4: Feature Expansion      ░░░░░░░░░░░░░░░░░░░░ FUTURE (not scheduled)
```

---

## Change Log Convention

As phases complete, update this file rather than deleting completed sections — change a phase's status line (e.g. `Status: ✅ Done`) and keep the task table as a historical record of what was done and why. This keeps `Phases.md` useful as both a forward-looking roadmap and a lightweight changelog, given the project has no other structured release history.

---

*This roadmap reflects the current understanding of project priorities as of this document's creation. It should be revised by the team as priorities shift — treat it as a living document, not a fixed contract.*
