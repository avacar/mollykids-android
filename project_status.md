# MollyKids Project Status

**Last updated:** 2026-04-19  
**Current phase:** Phase 7 — Parent Settings UI (done)

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
- [x] Molly upstream cloned into ForkOfMolly directory
- [x] `kids` flavor added to `app/build.gradle.kts` (build validated ✅ 2026-04-19)
- [ ] CI/build system (if any) works with both `kids` and non-kids flavors
- [ ] `README.md` explains that this is a parental-controls fork of Molly
- [x] All Phase 0 files committed with clear commit message

**Status:** [x] Done (core setup complete; README still pending)

---

### Phase 1 — Data Model (ParentalControlValues)
**Goal:** Create `ParentalControlValues.kt` to store parental control state (enabled/disabled, PIN
hash, allowed thread IDs). Register it in `SignalStore.kt` alongside other domain value classes.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/keyvalue/ParentalControlValues.kt` (new)
- `app/src/main/java/org/thoughtcrime/securesms/keyvalue/SignalStore.kt` (register new values class)
- `app/src/test/java/org/thoughtcrime/securesms/keyvalue/ParentalControlValuesTest.kt` (new)

**Acceptance criteria:**
- [x] `ParentalControlValues` class created with properties: `parentalModeEnabled`, `parentPinHash`, `allowedThreadIds`
- [x] PIN stored as SHA-256(random salt + userPin); salt auto-generated on first call
- [x] Class follows existing pattern (extends `SignalStoreValues`, registered in `SignalStore`)
- [x] Unit tests verify get/set behavior and PIN hashing
- [x] Tests confirm fresh install defaults: `parentalModeEnabled = true`, `parentPinHash = ""`, `allowedThreadIds = empty`
- [x] All Phase 1 files committed

**Status:** [x] Done

---

### Phase 2 — Conversation List Filtering
**Goal:** Filter conversation list to show only allowed threads when parental mode is enabled.
Implement in `ConversationListViewModel` as a filter step on the data stream.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/conversationlist/ConversationListViewModel.kt`

**Acceptance criteria:**
- [x] Conversation list filtered via `.map { list -> if (parentalModeEnabled) filter(list) else list }`
- [x] Unit/integration test: parental mode ON, two threads (allowed + disallowed) → only allowed thread appears
- [x] Test: parental mode OFF → all threads appear
- [x] Test: dynamically toggling parental mode updates visible list
- [x] No changes to database queries (filter only in ViewModel)
- [x] All Phase 2 commits made

**Status:** [x] Done

---

### Phase 3 — Block New Conversations & Calls
**Goal:** Hide new-conversation FAB and disable call initiation when parental mode is enabled.
Block outbound and suppress incoming calls from non-allowed threads.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/conversationlist/ConversationListFragment.java`
- `app/src/main/java/org/thoughtcrime/securesms/conversation/NewConversationActivity.kt`
- `app/src/main/java/org/thoughtcrime/securesms/conversation/ConversationFragment.kt` (or call entry point)

**Key clarification:** Call button in conversation header is intentionally **unchanged** — children can still call within allowed threads. What is blocked is starting *new* calls/conversations with non-allowed contacts.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/main/MainFloatingActionButtons.kt` (FAB hiding)
- `app/src/main/java/org/thoughtcrime/securesms/conversation/NewConversationActivity.kt` (safety-net guard)
- `app/src/main/java/org/thoughtcrime/securesms/calls/new/NewCallActivity.kt` (safety-net guard)
- `app/src/main/java/org/thoughtcrime/securesms/keyvalue/ParentalControlValues.kt` (isThreadCallAllowed helper)
- `app/src/main/java/org/thoughtcrime/securesms/service/webrtc/IncomingCallActionProcessor.java` (1:1 call block)
- `app/src/main/java/org/thoughtcrime/securesms/service/webrtc/IncomingGroupCallActionProcessor.java` (group ring block)
- `app/src/test/java/org/thoughtcrime/securesms/keyvalue/ParentalCallGuardTest.kt` (new, 7 tests)

**Acceptance criteria:**
- [x] Compose/new-conversation FAB hidden when `parentalModeEnabled = true` (CHATS/ARCHIVE)
- [x] `NewConversationActivity` finishes immediately if parental mode is on
- [x] New-call FAB hidden when `parentalModeEnabled = true` (CALLS tab)
- [x] `NewCallActivity` finishes immediately if parental mode is on
- [x] Call button in ConversationFragment toolbar unchanged — children can call within allowed threads
- [x] Incoming 1:1 call from non-allowed thread → silently rejected (delegates to handleDenyCall)
- [x] Incoming 1:1 call from allowed thread → shown normally
- [x] Incoming group call ring from non-allowed thread → silently cancelled (cancelGroupRing)
- [x] Incoming group call ring from allowed thread → shown normally
- [x] Unit tests: 7 new tests in ParentalCallGuardTest.kt + 5 new tests in ParentalControlValuesTest.kt
- [x] `assembleProdKidsDebug` BUILD SUCCESSFUL; all 27 tests green
- [ ] All Phase 3 commits made

**Status:** [x] Done

---

### Phase 4 — Group Invite PIN Gate
**Goal (revised):** Invites are completely hidden from children. A parent uses a PIN-gated
"Pending group invites" overflow menu item to view and accept/decline invites on the child's behalf.
Accepting an invite automatically adds the thread to `allowedThreadIds` so it appears in the child's list.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/keyvalue/ParentalControlValues.kt` (added `verifyPin`, `addAllowedThreadId`)
- `app/src/main/java/org/thoughtcrime/securesms/conversation/v2/ConversationFragment.kt` (guard `GROUP_V2_INVITE` display)
- `app/src/main/java/org/thoughtcrime/securesms/util/CommunicationActions.java` (block group-link join in parental mode)
- `app/src/main/java/org/thoughtcrime/securesms/main/MainToolbar.kt` (add "Pending group invites" menu item + callback)
- `app/src/main/java/org/thoughtcrime/securesms/MainActivity.kt` (implement `onPendingGroupInvitesClick`)
- `app/src/main/java/org/thoughtcrime/securesms/parental/ParentalPinDialog.kt` (new — reusable PIN entry dialog)
- `app/src/main/java/org/thoughtcrime/securesms/parental/PendingGroupInvitesViewModel.kt` (new)
- `app/src/main/java/org/thoughtcrime/securesms/parental/PendingGroupInvitesFragment.kt` (new — BottomSheet)
- `app/src/test/java/org/thoughtcrime/securesms/keyvalue/ParentalInviteGuardTest.kt` (new — 7 tests)

**Acceptance criteria:**
- [x] Invite acceptance flow traced; entry point identified (`DisabledInputView.showAsMessageRequest` in `ConversationFragment`)
- [x] `GROUP_V2_INVITE` accept UI hidden in `ConversationFragment` when parental mode is on
- [x] Group-link join (`CommunicationActions.handleGroupLinkUrl`) blocked in parental mode (toast shown)
- [x] `verifyPin(pin)` and `addAllowedThreadId(threadId)` added to `ParentalControlValues`
- [x] Overflow menu shows "Pending group invites" only when `parentalModeEnabled = true`
- [x] Tapping menu item → PIN dialog; wrong PIN → toast, no action; correct PIN → `PendingGroupInvitesFragment`
- [x] Parent accepts invite → `acceptMessageRequest` called + thread added to `allowedThreadIds` → appears in child's list
- [x] Parent declines invite → `deleteMessageRequest` called, removed from pending list
- [x] "No pending invites" shown when list is empty
- [x] `assembleProdKidsDebug` BUILD SUCCESSFUL
- [x] 25 tests in `ParentalControlValuesTest` + 7 in `ParentalInviteGuardTest` — all green (32 total)
- [ ] All Phase 4 commits made

**Status:** [x] Done

---

### Phase 5 — Notification Suppression
**Goal:** Suppress notifications for non-allowed threads when parental mode is enabled. Allowed
threads show notifications normally.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/notifications/v2/DefaultMessageNotifier.kt`
- Extension method (or new utility): `NotificationState.filterThreads(allowedIds: Set<Long>)`

**Acceptance criteria:**
- [x] `NotificationState.filterThreads()` added; filters pending notifications to allowed threads only
- [x] In `DefaultMessageNotifier.updateNotification()`, apply filter if `parentalModeEnabled = true`
- [x] Unit test: parental mode ON, messages from allowed + disallowed threads → only allowed notify
- [x] Unit test: parental mode OFF → all threads notify (guard in DefaultMessageNotifier skips filter call; verified structurally)
- [x] 5 unit tests in `NotificationStateParentalFilterTest.kt` — all green (37 total parental tests)
- [ ] All Phase 5 commits made

**Status:** [x] Done

---

### Phase 6 — Hide Stories & Lock Settings
**Goal:** Hide Stories tab when parental mode is enabled. Hide sensitive settings (Account,
Privacy/Advanced, Payments) from the child's Settings screen.

**Key files:**
- `app/src/main/java/org/thoughtcrime/securesms/main/MainNavigation.kt` (Stories tab filter)
- `app/src/main/java/org/thoughtcrime/securesms/components/settings/app/AppSettingsFragment.kt` (Account/Linked Devices/Donate rows)
- `app/src/main/java/org/thoughtcrime/securesms/components/settings/app/privacy/PrivacySettingsFragment.kt` (Advanced sub-setting)
- `app/src/test/java/org/thoughtcrime/securesms/keyvalue/ParentalNavigationFilterTest.kt` (new)

**Acceptance criteria:**
- [x] Stories tab/menu item visibility gated on `!parentalModeEnabled`
- [x] Settings categories hidden:
  - Account (phone number, username, linked devices, device transfer)
  - Privacy > Advanced
  - Payments (mapped to "Donate to Signal" — no standalone Payments in Molly)
- [x] Benign settings remain visible: Notifications, Appearance, Chat settings, Stories settings row
- [x] Unit/integration tests: 4 new tests in `ParentalNavigationFilterTest.kt` — all green (41 total parental tests)
- [ ] All Phase 6 commits made

**Status:** [x] Done

---

### Phase 7 — Parent Settings UI
**Goal:** Create PIN-gated "Parent Controls" activity accessible from main menu. Allows parent to
set/change PIN, toggle parental mode, select allowed chats.

**Key files:**
- New file: `app/src/main/java/org/thoughtcrime/securesms/parental/ParentalControlActivity.kt`
- New file: `app/src/main/java/org/thoughtcrime/securesms/parental/ParentalControlViewModel.kt`
- New file: `app/src/main/res/layout/activity_parental_control.xml`
- New file: `app/src/main/res/layout/item_parental_thread.xml`
- `app/src/main/java/org/thoughtcrime/securesms/main/MainToolbar.kt` (menu entry point + callback)
- `app/src/main/java/org/thoughtcrime/securesms/MainActivity.kt` (callback implementation)
- `app/src/main/AndroidManifest.xml` (activity registration)
- `app/src/main/res/values/strings.xml` (11 new strings)
- New test: `app/src/test/java/org/thoughtcrime/securesms/keyvalue/ParentalControlViewModelTest.kt`

**Acceptance criteria:**
- [x] "Parent Controls" menu item in overflow (⋮) menu (alongside "Pending group invites", visible when `parentalModeEnabled`)
- [x] Tapping opens PIN entry dialog (required every time); if no PIN set → goes straight to activity with setup flag
- [x] Correct PIN → shows control panel; wrong PIN → toast, stays on menu
- [x] Control panel has three sections:
  - Master toggle to enable/disable parental mode
  - Full list of all threads with toggle switches (add to allowed, remove from allowed)
  - Change PIN button (opens two-field new-PIN + confirm dialog; validates length ≥ 4; requires current PIN first unless no PIN is set)
- [x] Fresh install: no PIN set → shows PIN-setup screen before rendering control panel
- [x] Toggling a thread → `allowedThreadIds` updated in real time; `settingsChanges` fires → Phase 2 reactive pipeline refreshes conversation list
- [x] 6 unit tests in `ParentalControlViewModelTest.kt` — all green (62 total parental tests)
- [ ] Integration test: parent sets PIN → all subsequent access requires PIN (deferred; covered by verifyPin unit tests)
- [x] All Phase 7 commits made

**Status:** [x] Done

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
- ✅ Currently on branch: `parental-controls` (all work isolated from `main`)
- ✅ Added `kids` flavor to `app/build.gradle.kts` distribution dimension with PARENTAL_CONTROLS_ENABLED flag
- ✅ Updated selectableVariants to include all kids flavor combinations (prodKidsDebug/Release, stagingKidsDebug/Release)
- ✅ Committed Phase 0 work (2 commits)

**Lessons learned:**
- Gradle wrapper requires internet for dependency download
- Phase 0 is functionally complete (repo initialized, flavor added, documented)
- Next session should: (1) optionally validate Gradle build if internet is available, (2) create README.md for MollyKids overview, (3) start Phase 1 (ParentalControlValues data model)

**Deferred to next session:**
- Gradle build validation (`./gradlew assemble`)
- README.md creation with project overview and setup instructions

**2026-04-18 (Phase 1 — Data Model):**
- ✅ Completed ParentalControlValues.kt with all required properties and methods
- ✅ PIN hashing using random 16-byte salt (auto-generated on first call, device-local)
- ✅ Registered ParentalControlValues in SignalStore.kt (init property, onFirstEverAppLaunch, companion accessor)
- ✅ Created comprehensive unit tests (12 test cases covering defaults, PIN hashing, salt generation, thread ID storage)
- ✅ Committed Phase 1 work (1 commit)
- ✅ Gradle build validation passed (2026-04-19) — `assembleProdKidsDebug` BUILD SUCCESSFUL
- ✅ Unit tests passed (2026-04-19) — all 12 ParentalControlValuesTest cases green
- Fixed test compilation errors: KeyValueStore constructor API change (now takes KeyValuePersistentStorage), assertk Set assertion (containsExactly → isEqualTo)

**Lessons learned:**
- Random salt approach (device-local) chosen over ACI-based salt for better security and clarity on multi-device behavior
- PIN salt is generated lazily (not during app launch) to keep onFirstEverAppLaunch() simple
- ParentalControlValues deliberately omitted from backup inclusion — parental config must be set fresh on any new device
- Existing test pattern (BackupDownloadNotifierUtilTest.kt) uses assertk; replicated for consistency

**2026-04-18 (Housekeeping — SSL resolved):**
- ✅ SSL certificate issue resolved — Gradle builds now unblocked
- ✅ Removed SSL blocking issue from Known Issues and CLAUDE.md

**2026-04-19 (Phase 2 — Conversation List Filtering):**
- ✅ Added `settingsChanges: PublishSubject<Unit>` to `ParentalControlValues`; changed `parentalModeEnabled` to explicit getter/setter; `setAllowedThreadIds` also emits on write
- ✅ Added `applyParentalFilter(list, enabled, allowedIds)` companion function to `ConversationListViewModel`
- ✅ Chained `.map` on `conversationsState` to apply parental filter on every list emission
- ✅ Subscribed to `settingsChanges` in ViewModel `init` to call `controller.onDataInvalidated()` when settings change (reactive update without DB query changes)
- ✅ Created `ConversationListParentalFilterTest.kt` (8 unit tests) and added 2 `settingsChanges` tests to `ParentalControlValuesTest.kt` (14 tests total now)
- ✅ All tests pass; `assembleProdKidsDebug` BUILD SUCCESSFUL

**Lessons learned:**
- Non-THREAD `Conversation` items (headers, footers) have `threadId < 0`; the filter must pass them through unconditionally or the list structure breaks
- Extracted filter as `internal` companion function enables clean unit testing without full ViewModel scaffolding
- `PublishSubject.toFlowable(BackpressureStrategy.LATEST)` is the correct bridge when subscribing to a Subject inside a Flowable pipeline in the ViewModel

**2026-04-19 (Phase 3 — Block New Conversations & Calls):**
- ✅ Hidden compose FAB and new-call FAB in `MainFloatingActionButtons.kt` via parental-mode early return in `PrimaryActionButton()` composable (CHATS/ARCHIVE/CALLS destinations only; STORIES untouched for Phase 6)
- ✅ Added safety-net `finish()` guards to `NewConversationActivity.onCreate()` and `NewCallActivity.onCreate()` to block deep-link bypass
- ✅ Added `isThreadCallAllowed(threadId: Long): Boolean` to `ParentalControlValues` — central helper used by both call processors and tests
- ✅ Suppressed incoming 1:1 calls from non-allowed threads in `IncomingCallActionProcessor.handleLocalRinging()` via `handleDenyCall()` delegation
- ✅ Suppressed incoming group call rings from non-allowed threads in `IncomingGroupCallActionProcessor.handleGroupCallRingUpdate()` via `cancelGroupRing(DeclinedByUser)`
- ✅ Created `ParentalCallGuardTest.kt` (7 tests) + added 5 tests to `ParentalControlValuesTest.kt` (now 20 tests)
- ✅ `assembleProdKidsDebug` BUILD SUCCESSFUL; all 27 parental-control tests green

**Key clarification on calls:** Children CAN call contacts within allowed threads (call button in ConversationFragment untouched). Only NEW calls to new contacts and incoming calls from non-allowed threads are blocked.

**Lessons learned:**
- `SignalStore.parentalControl` is a Kotlin property (no parens); Java callers use `SignalStore.parentalControl()` via JvmName annotation — easy to confuse
- Test files in `org.thoughtcrime.securesms.parental` can't access package-private APIs in `org.thoughtcrime.securesms.keyvalue.KeyValueDataSet`; keep parental-control tests in the `keyvalue` package
- Delegating to `handleDenyCall()` in `IncomingCallActionProcessor` reuses the existing reject-and-terminate logic cleanly; requires `activePeer.localRinging()` to have been called first (which it is at that point in `handleLocalRinging()`)

**2026-04-19 (Phase 0 + Phase 1 build validation):**
- ✅ Installed JDK 21 LTS (Adoptium Temurin) + JDK 17 LTS (Adoptium Temurin); configured Gradle via `org.gradle.java.home` and `org.gradle.java.installations.paths`
- ✅ Installed Android Studio + Android SDK (platform 36, build-tools 35.0.0 auto-downloaded by Gradle)
- ✅ Phase 0 validated: `./gradlew :app:assembleProdKidsDebug` BUILD SUCCESSFUL (8m first run)
- ✅ Phase 1 validated: all 12 ParentalControlValuesTest cases pass
- ✅ Fixed 2 test compilation errors discovered during validation (API drift from upstream + assertk Set assertion)

**Lessons learned:**
- Gradle requires JDK 17 toolchain for compilation (via `kotlinJvmTarget`); JDK 21 is used as the daemon JVM only
- `org.gradle.java.installations.paths` must be in `~/.gradle/gradle.properties` (not project-level) to work for included builds like `build-logic`
- `KeyValueStore` constructor now takes `KeyValuePersistentStorage` (not `KeyValueDataSet` directly); tests need an in-memory wrapper
- assertk has no `containsExactly` for `Set<T>`; use `isEqualTo(setOf(...))` instead

**2026-04-19 (Phase 4 — Group Invite PIN Gate):**
- ✅ Revised goal: invites are completely hidden from children; parents manage invites via PIN-gated overflow menu item
- ✅ Added `verifyPin(pin)` and `addAllowedThreadId(threadId)` to `ParentalControlValues`
- ✅ Guarded `GROUP_V2_INVITE` display in `ConversationFragment.presentInputReadyState()` — accept UI suppressed in parental mode
- ✅ Blocked group-link joins in `CommunicationActions.handleGroupLinkUrl()` — shows toast in parental mode
- ✅ Created `ParentalPinDialog.kt` — reusable PIN entry dialog (object, not Fragment; callable from any context)
- ✅ Created `PendingGroupInvitesViewModel.kt` — queries groups where self is `PENDING_MEMBER`, accept/decline via `MessageRequestRepository`
- ✅ Created `PendingGroupInvitesFragment.kt` — `BottomSheetDialogFragment` listing pending invites with Accept/Decline buttons
- ✅ Added "Pending group invites" item to `MainToolbar.kt` `ChatDropdownItems()` (only visible when `parentalModeEnabled`); wired callback in `MainActivity.ToolbarCallback`
- ✅ Accepting an invite calls `addAllowedThreadId` → `settingsChanges` fires → Phase 2 reactive pipeline auto-shows thread in child's list
- ✅ Added 5 tests to `ParentalControlValuesTest.kt` (now 25 total) + new `ParentalInviteGuardTest.kt` (7 tests)
- ✅ `assembleProdKidsDebug` BUILD SUCCESSFUL; all 32 parental-control tests green

**Lessons learned:**
- `SignalDatabase.groups` and `SignalDatabase.threads` are Kotlin properties (no parens); Java callers use `()` via `@get:JvmName` annotation — distinct from `SignalStore.parentalControl` which is the same pattern
- `GroupTable.Reader.getNext()` returns `GroupRecord?`; assign to a non-var local inside the loop to avoid smart-cast issues
- Duplicate import of `SignalStore` causes a compile error ("conflicting import: imported name is ambiguous") — check existing imports before adding
- `GroupChangeFailureReason` is a plain Java enum with no `toDisplayString()` method; use `reason.name` for a simple error label

**2026-04-19 (Phase 5 — Notification Suppression):**
- ✅ Added `filterThreads(allowedThreadIds: Set<Long>): NotificationState` to `NotificationState` using `data class copy()` — preserves mute/profile filtered messages
- ✅ Applied filter in `DefaultMessageNotifier.updateNotification()` immediately after `constructNotificationState()`; early-exit on `state.isEmpty` naturally silences all notifications when allowed set is empty
- ✅ Created `NotificationStateParentalFilterTest.kt` (5 unit tests): allowed retained, disallowed removed, mixed, empty set, side-lists preserved
- ✅ All 37 parental-control tests green; `assembleProdKidsDebug` BUILD SUCCESSFUL

**Lessons learned:**
- `NotificationState` is a pure Kotlin `data class` with no Android dependencies — pure unit tests (no Robolectric) work cleanly; use `mockk(relaxed = true)` for `Recipient` and `NotificationItem` fields
- Phase 3 and 4 work was committed in this session (previous session left them uncommitted)

**2026-04-19 (Phase 6 — Hide Stories & Lock Settings):**
- ✅ Extracted `buildNavEntries(isStoriesEnabled, parentalModeEnabled)` helper from `MainNavigation.kt` — applied to both `MainNavigationBar` and `MainNavigationRail`
- ✅ Stories tab filtered out when `parentalModeEnabled = true`, independently of the Stories feature flag
- ✅ Account, Linked Devices, Donate to Signal rows + their trailing divider wrapped in `if (!parentalModeEnabled)` in `AppSettingsFragment.kt`
- ✅ Privacy > Advanced `clickPref` (+ preceding divider) wrapped in `if (!SignalStore.parentalControl.parentalModeEnabled)` in `PrivacySettingsFragment.kt`
- ✅ Benign settings (Appearance, Chats, Stories privacy settings, Notifications, Privacy top-level, Backups, Network, Help) remain visible
- ✅ Created `ParentalNavigationFilterTest.kt` (4 unit tests) — 41 total parental tests green
- ✅ `assembleProdKidsDebug` BUILD SUCCESSFUL

**Lessons learned:**
- Molly has no standalone Payments feature; "Donate to Signal" (external browser link) is the closest analog and was hidden to match the acceptance criterion
- `PrivacySettingsFragment` uses the legacy DSL settings system (not Compose) — wrapping a `clickPref` in an `if` block works exactly like Compose conditional items
- Extracting the nav entries filter as an `internal fun` in the same file keeps the helper co-located with its call sites and testable without any mocking

**2026-04-19 (Phase 7 — Parent Settings UI):**
- ✅ Added `onParentalControlsClick()` to `MainToolbarCallback` interface + `Empty` stub; added "Parent Controls" dropdown item in `ChatDropdownItems()` alongside "Pending group invites" (both gated on `parentalModeEnabled`)
- ✅ Implemented `onParentalControlsClick()` in `MainActivity.ToolbarCallback` — no-PIN-set path skips dialog and passes `EXTRA_SETUP_PIN=true`; otherwise uses existing `ParentalPinDialog` then starts activity
- ✅ Created `ParentalControlViewModel` (AndroidViewModel) — loads thread list via `ThreadTable.getRecentConversationList` + `readerFor`, exposes `parentalEnabled` and `threads` as LiveData, delegates all writes to `SignalStore.parentalControl`
- ✅ Created `ParentalControlActivity` — `AppCompatActivity` with `activity_parental_control.xml`; three sections: master switch, thread list (inflated dynamically from `item_parental_thread.xml`), Change PIN button with two-field confirmation dialog
- ✅ Registered `ParentalControlActivity` in `AndroidManifest.xml` (`exported=false`, `adjustResize`)
- ✅ Created `ParentalControlViewModelTest.kt` (6 tests) — all green; 62 total parental tests
- ✅ `assembleProdKidsDebug` BUILD SUCCESSFUL

**Lessons learned:**
- Used a standalone `AppCompatActivity` rather than plugging into `AppSettingsActivity`'s NavGraph — keeps parental UI self-contained and reduces upstream merge conflict surface
- `ThreadTable.getRecentConversationList(limit, includeInactiveGroups, hideV1Groups)` + `readerFor(cursor).use { }` is the correct pattern for iterating all threads in memory; cursor must be closed via `use`
- ViewModel tests live in `org.thoughtcrime.securesms.keyvalue` package (not `parental`) to access `KeyValueDataSet` package-private APIs — consistent with lessons from Phase 3
