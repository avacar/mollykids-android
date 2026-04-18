# MollyKids Project Status

**Last updated:** 2026-04-18  
**Current phase:** Phase 1 — Data Model (ParentalControlValues)

---

## Phase Breakdown

### Phase 0 — Documentation & Repo Setup
**Goal:** Initialize repo, establish CLAUDE.md and project_status.md, clone Molly upstream,
create `kids` Gradle flavor.

**Key files:**
- `CLAUDE.md` (created)
- `project_status.md` (this file)
- `app/build.gradle.kts` (add `kids` flavor to `distribution` dimension)
- `.gitignore`, `README.md` (update to reference MollyKids fork)

**Acceptance criteria:**
- [ ] Molly upstream cloned into ForkOfMolly directory
- [ ] `kids` flavor added to `app/build.gradle.kts` and builds successfully (`./gradlew assemble`)
- [ ] CI/build system (if any) works with both `kids` and non-kids flavors
- [ ] `README.md` explains that this is a parental-controls fork of Molly
- [ ] All Phase 0 files committed with clear commit message

**Status:** [x] Done

---

### Phase 1 — Data Model (ParentalControlValues)
**Goal:** Create `ParentalControlValues.kt` to store parental control state (enabled/disabled, PIN
hash, allowed thread IDs). Register it in `SignalStore.kt` alongside other domain value classes.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/keyvalue/ParentalControlValues.kt` (new)
- `app/src/main/java/org/thoughtcrime/securesms/SignalStore.kt` (register new values class)

**Acceptance criteria:**
- [ ] `ParentalControlValues` class created with properties: `parentalModeEnabled`, `parentPinHash`, `allowedThreadIds`
- [ ] PIN stored as SHA-256(userPin + Application.getInstallId())
- [ ] Class follows existing pattern (extends `SignalStoreValues`, registered in `SignalStore`)
- [ ] Unit tests verify get/set behavior and PIN hashing
- [ ] Tests confirm fresh install defaults: `parentalModeEnabled = true`, `parentPinHash = ""`, `allowedThreadIds = empty`
- [ ] All Phase 1 files committed

**Status:** [ ] Not started

---

### Phase 2 — Conversation List Filtering
**Goal:** Filter conversation list to show only allowed threads when parental mode is enabled.
Implement in `ConversationListViewModel` as a filter step on the data stream.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/conversationlist/ConversationListViewModel.kt`

**Acceptance criteria:**
- [ ] Conversation list filtered via `.map { list -> if (parentalModeEnabled) filter(list) else list }`
- [ ] Unit/integration test: parental mode ON, two threads (allowed + disallowed) → only allowed thread appears
- [ ] Test: parental mode OFF → all threads appear
- [ ] Test: dynamically toggling parental mode updates visible list
- [ ] No changes to database queries (filter only in ViewModel)
- [ ] All Phase 2 commits made

**Status:** [ ] Not started

---

### Phase 3 — Block New Conversations & Calls
**Goal:** Hide new-conversation FAB and disable call initiation when parental mode is enabled.
Block outbound and suppress incoming calls from non-allowed threads.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/conversationlist/ConversationListFragment.java`
- `app/src/main/java/org/thoughtcrime/securesms/conversation/NewConversationActivity.kt`
- `app/src/main/java/org/thoughtcrime/securesms/conversation/ConversationFragment.kt` (or call entry point)

**Acceptance criteria:**
- [ ] Compose/new-conversation FAB hidden when `parentalModeEnabled = true`
- [ ] Clicking through intent to `NewConversationActivity` when parental mode ON → activity finishes immediately
- [ ] Call initiation button hidden in conversation header
- [ ] Incoming call from non-allowed thread → silently rejected (no incoming-call UI)
- [ ] Incoming call from allowed thread → shown normally
- [ ] Unit/integration tests for each guard
- [ ] All Phase 3 commits made

**Status:** [ ] Not started

---

### Phase 4 — Group Invite PIN Gate
**Goal:** When a child attempts to accept a group invite, PIN dialog is required. Only parents
(who know the PIN) can accept invites on behalf of the child.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/groups/ui/invitesandrequests/` (likely fragment/ViewModel)
- New file: `app/src/main/java/org/thoughtcrime/securesms/parental/ParentalPinDialog.kt` (reusable PIN entry dialog)

**Acceptance criteria:**
- [ ] Invite acceptance flow traced; entry point identified
- [ ] Before `GroupManager.acceptInvite(...)` call, show PIN dialog if parental mode is enabled
- [ ] PIN dialog accepts user input, verifies against `parentPinHash`
- [ ] Correct PIN → proceed with acceptance; wrong PIN → dismiss and stay on invite screen
- [ ] No invite acceptance UI available to child (no "accept" button; accept only via parent PIN)
- [ ] Integration test: parental mode ON, child taps invite → PIN dialog appears; parent enters PIN → invite accepted
- [ ] All Phase 4 commits made

**Status:** [ ] Not started

---

### Phase 5 — Notification Suppression
**Goal:** Suppress notifications for non-allowed threads when parental mode is enabled. Allowed
threads show notifications normally.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/notifications/v2/DefaultMessageNotifier.kt`
- Extension method (or new utility): `NotificationState.filterThreads(allowedIds: Set<Long>)`

**Acceptance criteria:**
- [ ] `NotificationState.filterThreads()` added; filters pending notifications to allowed threads only
- [ ] In `DefaultMessageNotifier.updateNotification()`, apply filter if `parentalModeEnabled = true`
- [ ] Unit test: parental mode ON, messages from allowed + disallowed threads → only allowed notify
- [ ] Unit test: parental mode OFF → all threads notify
- [ ] Integration test (or high-level): receive message in allowed group → notification shown; receive in disallowed group → silent
- [ ] All Phase 5 commits made

**Status:** [ ] Not started

---

### Phase 6 — Hide Stories & Lock Settings
**Goal:** Hide Stories tab when parental mode is enabled. Hide sensitive settings (Account,
Privacy/Advanced, Payments) from the child's Settings screen.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/main/MainActivity.kt` (or bottom-nav host)
- `app/src/main/java/org/thoughtcrime/securesms/preferences/ApplicationPreferencesActivity.java` (or Compose equivalent)

**Acceptance criteria:**
- [ ] Stories tab/menu item visibility gated on `!parentalModeEnabled`
- [ ] Settings categories hidden:
  - Account (phone number, username, linked devices, device transfer)
  - Privacy > Advanced
  - Payments
- [ ] Benign settings remain visible: Notifications, Appearance, Chat settings
- [ ] Unit/integration tests: parental mode ON → Stories tab absent, Account settings not accessible
- [ ] All Phase 6 commits made

**Status:** [ ] Not started

---

### Phase 7 — Parent Settings UI
**Goal:** Create PIN-gated "Parent Controls" activity accessible from main menu. Allows parent to
set/change PIN, toggle parental mode, select allowed chats.

**Key files:**
- New file: `app/src/main/java/org/thoughtcrime/securesms/parental/ParentalControlActivity.kt`
- New file: `app/src/main/java/org/thoughtcrime/securesms/parental/ParentalControlViewModel.kt`
- `app/src/main/java/org/thoughtcrime/securesms/conversationlist/ConversationListFragment.java` (menu entry point)
- XML: `app/src/main/res/menu/conversation_list_menu.xml` (add menu item)

**Acceptance criteria:**
- [ ] "Parent Controls" menu item in overflow (⋮) menu
- [ ] Tapping opens PIN entry dialog (required every time)
- [ ] Correct PIN → shows control panel; wrong PIN → stays on menu
- [ ] Control panel has three tabs/sections:
  - Master toggle to enable/disable parental mode
  - Full list of all threads with toggle switches (add to allowed, remove from allowed)
  - Change PIN (requires current PIN, then new PIN entry + confirmation)
- [ ] Fresh install: no PIN set → shows PIN-setup screen before enabling parental mode
- [ ] Toggling a thread → `allowedThreadIds` updated in real time; conversation list refreshes
- [ ] Unit tests: PIN validation, allowlist toggles, storage persistence
- [ ] Integration test: parent sets PIN → all subsequent access requires PIN
- [ ] All Phase 7 commits made

**Status:** [ ] Not started

---

### Phase 8 — Registration Guide & First-Run Setup Flow
**Goal:** Add first-run setup that explains the Google Voice registration process and enforces
PIN setup before the child can use the app.

**Key files:**
- New file: `app/src/main/java/org/thoughtcrime/securesms/parental/ParentalFirstRunActivity.kt`
- Possibly: modify existing registration/onboarding flow to inject parental setup

**Acceptance criteria:**
- [ ] On fresh install of `kids` flavor, after account is registered, first-run setup prompts parent to:
  - Read brief explanation of Google Voice registration (not required by app, but documented for parent's reference)
  - Set a PIN for parental controls
  - Enable parental mode
- [ ] Setup can be skipped (for testing), but parental mode cannot be used without a PIN
- [ ] Once setup is complete, main conversation list opens
- [ ] Unit/integration tests: fresh install flow tested end-to-end
- [ ] All Phase 8 commits made

**Status:** [ ] Not started

---

## Known Issues / Deferred Work

- **PIN brute-force:** Current SHA-256 hash offers no rate-limiting. On a rooted device, attacker
  could brute-force the PIN. For family use this is acceptable, but should be revisited if
  security requirements change (consider adding rate-limiting or key-stretching in the future).

- **Linked devices:** Signal supports 5 linked devices per account. If a child tries to link a
  device, parental controls don't automatically apply (each device would need independent PIN).
  Needs design decision: block device linking entirely, or propagate settings across linked devices?

- **Call notifications:** Incoming-call suppression is discussed but not detailed in the plan.
  May require deeper integration with Molly's call stack; implementation will clarify feasibility.

- **Upstream merge conflicts:** As Molly evolves, the `kids` flavor and new files will need
  periodic rebasing. No automated merge strategy defined yet.

---

## Feature Backlog

- **Per-chat send/receive toggle:** Allow parent to set a chat as read-only for the child (child
  can see messages but not send). Current plan is full send/receive for allowed chats only.

- **Time-of-day restrictions:** Restrict messaging to certain hours (e.g. no messaging after 9pm).

- **Remote management companion app:** A parent app that can push configuration to children's
  devices without needing physical access to the PIN. Requires backend/sync infrastructure.

- **Activity logging:** Log which chats the child accessed, when, and who sent messages (without
  decrypting content). Useful for parental oversight without breaking encryption.

- **Child-to-parent SOS:** Special button/message that alerts parent if child is distressed or
  needs help.

- **Conversation request blocking:** Currently, group invites require PIN acceptance. Could extend
  to direct conversation requests (if Molly supports them in Signal's newer versions).

- **Multi-profile support:** Separate "kid mode" and "parent mode" profiles on the same device,
  each with their own PIN/settings. Useful if parent wants to test the app.

---

## Session Notes

**2026-04-18 (Phase 0 — Repo Setup):**
- ✅ Created CLAUDE.md with project governance, development principles, and session workflow
- ✅ Created project_status.md as living project tracker (read at start of every session)
- ✅ Cloned Molly upstream; initialized git repo with `parental-controls` feature branch
- ✅ Added `kids` flavor to `app/build.gradle.kts` distribution dimension with PARENTAL_CONTROLS_ENABLED flag
- ✅ Updated selectableVariants to include all kids flavor combinations (prodKidsDebug/Release, stagingKidsDebug/Release)
- ✅ Committed Phase 0 work

**Next:** Build Gradle build test requires internet access; skipping validation for now. Proceeding to Phase 1 (ParentalControlValues data model).
