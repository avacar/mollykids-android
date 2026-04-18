# MollyKids Development Guide

## Project Purpose

MollyKids is a fork of Molly (a Signal hardening) designed to enable children to communicate with
pre-approved family members via Signal-compatible messaging on semi-locked-down tablets. The app
provides parental controls via PIN-gated settings that restrict children's visibility and access
to only parent-approved group chats, preventing outbound messaging to unauthorized contacts.

## Core Objectives

1. **Zero unauthorized contacts** — Children can only see and message group chats that parents
   explicitly allow. No new conversations, no direct contacts outside approved groups.
2. **Family-first communication** — Leverage Signal's username feature so children interact via
   parent-set names, not phone numbers. Registration uses free Google Voice numbers, not child
   phone numbers.
3. **Parent control layer** — PIN-gated settings screen on the child's device that parents use to
   configure which threads are accessible. No remote management infrastructure initially.
4. **Upstream compatibility** — Parental-control additions must be isolated to the `kids` Gradle
   flavor and new files, enabling clean merges with upstream Molly releases.

## Before Every Session: Read `project_status.md`

**This is critical.** At the start of each session, read `project_status.md` to understand:
- Which phase is active
- What work remains in the current phase
- Known issues and deferred work (not in scope yet)
- The feature backlog (future ideas)

`project_status.md` is the authoritative source of project state. Do not rely on Git history or
conversation memory to understand where we are.

## Development Principles

### 1. Test-Driven Development (TDD)
- Write unit or integration tests **before or alongside** implementation.
- A phase is **not done** until tests cover the new behavior.
- Example: Before implementing conversation filtering, write a test that verifies non-allowed
  threads do not appear in the conversation list.
- If a feature is hard to test, that's often a sign the design needs rethinking — ask before
  proceeding.

### 2. Maximize Upstream Molly Compatibility
- **Isolate parental additions to the `kids` Gradle flavor** wherever possible. Shared Signal
  protocol, network, crypto, and database code must be untouched.
- **New files only** for parental features (e.g. `ParentalControlValues.kt`, `ParentalControlActivity.kt`).
- **Minimize modifications to existing files.** If a file must be edited, use guards (e.g.
  `if (SignalStore.parental().isEnabled)`) rather than rewriting logic.
- When Molly releases a new version, our parental changes should rebase/merge cleanly with zero
  or minimal conflict resolution.

### 3. Push Back on Incorrect Suggestions
- If the user proposes something architecturally unsound, incompatible with Molly's design, or
  likely to break upstream merges, **raise the concern immediately** and ask clarifying questions.
- Example: If the user asks to "add a second user account," ask whether we're actually creating a
  separate Signal account or a UI-level restriction. The former breaks Molly's assumptions; the
  latter is what the plan describes.
- When in doubt, check Molly's codebase or Signal's protocol documentation before implementing.

## Session Workflow

1. Read and update `project_status.md` with the current date and phase.
2. Work on the active phase per the acceptance criteria listed in `project_status.md`.
3. At the end of each session, update `project_status.md` to reflect progress:
   - Mark the current phase status ([ ] Not started, [~] In progress, [x] Done)
   - Add any discovered issues to "Known Issues / Deferred Work"
   - Add any new feature ideas to the "Feature Backlog"
4. Commit these documentation updates so the next session has fresh context.

## Key Files

- **`CLAUDE.md`** (this file) — Development guidelines and project principles
- **`project_status.md`** — Living status document; read at the start of every session
- **`app/build.gradle.kts`** — Gradle build config; add `kids` flavor here
- **`app/src/main/java/org/thoughtcrime/securesms/keyvalue/`** — Add `ParentalControlValues.kt` here
- **Plan file:** `/plans/there-exists-a-fork-ticklish-treasure.md` — Detailed implementation roadmap

## Registration Approach (Reference)

For context: Children use **free Google Voice numbers** (one per child), registered by parents.
After account setup, they communicate via **Signal usernames**, not phone numbers. This is fully
supported by Molly/Signal today and requires no app changes.

## License

Molly and this fork are licensed under AGPLv3. For personal family use with no public distribution,
no additional obligations. If distributed publicly, the fork must remain open-source under AGPLv3.

## Getting Help

Refer to the plan file (`/plans/there-exists-a-fork-ticklish-treasure.md`) for detailed
architectural decisions and code location references. If stuck on a phase, check "Known Issues"
in `project_status.md` — the problem may be documented from a prior session.
