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
3. **At the end of each session**, update both `project_status.md` and `CLAUDE.md` to reflect progress:
   - **`project_status.md`:**
     - Mark the current phase status ([ ] Not started, [~] In progress, [x] Done)
     - Update "Current Phase" at the top
     - Add any discovered issues to "Known Issues / Deferred Work"
     - Add any new feature ideas to the "Feature Backlog"
     - Add a Session Notes entry with the date and summary of work completed
   - **`CLAUDE.md`** (this file):
     - Update any development principles that turned out to be wrong or need refinement
     - Add new guidance based on lessons learned in this session (e.g. "We found it's better to X rather than Y")
4. Commit both documentation updates so the next session has fresh context.

**Important:** At the end of every session, the assistant will ask for permission to commit these documentation updates. Example: "I've finished Phase X. May I update project_status.md and CLAUDE.md, then commit?" — the user should explicitly approve before committing.

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

## SSL Certificate Issue (Java/Gradle) — Blocking Build Validation

**Problem:** `./gradlew` commands fail with `SSLHandshakeException: PKIX path building failed`. This
occurs because the JVM's default certificate store (`cacerts`) is missing or outdated root CA
certificates trusted by `services.gradle.org` and Maven Central.

**Why it happens:** Java installations include a snapshot of root certificates. If Windows or system
certificates have been updated but Java hasn't, the JVM can't validate modern certificate chains.
This is common on older Java installations (Java 8/11 on Windows 11).

**Root cause on this machine:** Likely Java runtime has outdated or incomplete root CA certificates.

**How to fix:**

1. **Update Java first** (simplest, recommended):
   - Download latest LTS JDK: https://adoptium.net (Eclipse Adoptium) or https://www.oracle.com/java/
   - Install Java 17+ (newer versions include current root certificates)
   - Verify: `java -version` shows 17+
   - This often resolves the issue automatically

2. **If updating Java doesn't work, manually update cacerts:**
   ```bash
   # Find your JDK (e.g., C:\Program Files\Java\jdk-17)
   # Locate keytool: <JDK>\bin\keytool.exe
   
   # Download certificate chain from services.gradle.org
   # (Use browser or: openssl s_client -connect services.gradle.org:443)
   
   # Import the certificate:
   keytool -import -alias gradle-ca -file <downloaded-cert> \
     -keystore "C:\Program Files\Java\jdk-17\lib\security\cacerts" \
     -storepass changeit -noprompt
   ```

3. **Verify Gradle works:**
   ```bash
   cd c:\Users\Alex Ghosh\Documents\Visual Studio 2017\Projects\ForkOfMolly
   ./gradlew --version
   ```

**When to address:** Before Phase 2 or 3 — we need successful builds to validate code. The
issue is documented in `project_status.md` under "Known Issues / Deferred Work" with the label
`[BLOCKING]`.

**Priority:** High — all future phases depend on this.

## License

Molly and this fork are licensed under AGPLv3. For personal family use with no public distribution,
no additional obligations. If distributed publicly, the fork must remain open-source under AGPLv3.

## Getting Help

Refer to the plan file (`/plans/there-exists-a-fork-ticklish-treasure.md`) for detailed
architectural decisions and code location references. If stuck on a phase, check "Known Issues"
in `project_status.md` — the problem may be documented from a prior session.
