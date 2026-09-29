# iPad Adaptive Layout Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Give Elychron a readable iPad layout with a sidebar on wide windows and the existing bottom tabs on narrow windows.

**Architecture:** Keep `HomePage` and its `PageController` as the only mobile navigation owner. A small adaptive shell measures its allocated width and keeps the page content in a stable widget slot while swapping only navigation chrome. Shared width widgets constrain common sheets and task forms on wide iOS windows.

**Tech Stack:** Flutter/Dart Cupertino UI, existing `DesktopNavRail`, Flutter widget tests, iOS simulators.

**Spec:** `docs/superpowers/specs/2026-09-29-ipad-adaptive-layout-design.md`

## Global Constraints

- One Flutter app and one iOS target; no data-model, native entitlement, alarm, share, signing, or dependency changes.
- Use current window width, never iPad model or a global orientation lock. Use a 900 logical-pixel wide-layout breakpoint and verify it on simulator.
- Keep the five destinations ordered 日程 / 待办 / 专注 / 学业 / 设置; detail routes keep their existing full-page navigation.
- Main content maximum width: 920 logical pixels; shared sheet: 560; task form body: 720. iPhone, Android, and desktop layout remain unchanged.
- Keep changes focused in the fork so upstream Elychron can merge cleanly.

## Review Focus

1. Wide-to-narrow resize on a selected non-default tab retains that tab and its page state (Task 1 test).
2. A wide Android window still uses the mobile bottom tabs (Task 1 test).
3. A popup with many options remains scrollable and its cancel action stays visible (Task 2 test).
4. A popup opened with the keyboard visible keeps its action above the keyboard (Task 2 test).
5. A task form retains entered text when its available width changes (Task 3 test).

---

### Task 1: Adaptive main-page navigation

**Files:**
- Create: `lib/page/tablet/adaptive_home_body.dart` — width branch, stable content slot, readable-width wrapper.
- Modify: `lib/page/home_page.dart` — pass the existing PageView, tab bar, index, and selection callback to the shell.
- Modify: `lib/page/desktop/desktop_nav_rail.dart` — add `showDropHint` (default `true`) so the same rail can omit desktop copy on iPad.
- Test: `test/adaptive_home_body_test.dart`.

**Interfaces:**
- Produces: `AdaptiveHomeBody({required Widget content, required CupertinoTabBar bottomBar, required int selectedIndex, required ValueChanged<int> onSelect})`.
- Produces: `AdaptiveHomeBody.wideBreakpoint = 900`, `AdaptiveHomeBody.maxContentWidth = 920`, `AdaptiveHomeBody.railWidth = 208`.
- Consumes: `DesktopNavRail(index: ..., onSelect: ..., showDropHint: false)`; only `HomePage` owns the `PageController` and `HomeModHooks`.

- [ ] **Step 1: Write failing widget tests.** In `test/adaptive_home_body_test.dart`, cover these named cases and assertions; use a `pumpShell(width, platform, selectedIndex)` helper with a keyed stateful fake content page:

  ```dart
  expect(find.byType(DesktopNavRail), findsOneWidget); // iOS width 900
  expect(find.byType(CupertinoTabBar), findsNothing);
  expect(contentRect.width, lessThanOrEqualTo(920)); // iOS width 1200
  expect(find.byType(CupertinoTabBar), findsOneWidget); // iOS width 899, Android width 1200
  expect(fakePageState.id, originalStateId); // after iOS resize 1100 → 700
  expect(selectedIndex, 2); // after resize; rail callback also returns index 2
  ```
- [ ] **Step 2: Run `flutter test test/adaptive_home_body_test.dart` and confirm the new tests fail before implementation.**
- [ ] **Step 3: Implement `AdaptiveHomeBody` and wire `HomePage`.** Keep content as the first child in a stable `Stack` slot across width changes; position the rail or bottom bar around it. Preserve the existing keyboard-inset and translucent-tab behavior on narrow windows. Add `showDropHint` to `DesktopNavRail` without changing its desktop default.
- [ ] **Step 4: Run `flutter test test/adaptive_home_body_test.dart test/desktop_nav_rail_test.dart test/desktop_frame_test.dart`; confirm all pass.**
- [ ] **Step 5: Commit the three product files and test as `feat(ipad): adapt main navigation to window width`.**

### Task 2: Shared iPad sheet width and keyboard safety

**Files:**
- Create: `lib/design/adaptive_sheet_frame.dart` — shared popup frame for wide iOS windows.
- Modify: `lib/design/dingtalk_sheet.dart` — use the frame for `showDingTalkSheet` and `showDingTalkPanel`.
- Test: `test/adaptive_sheet_frame_test.dart`.

**Interfaces:**
- Produces: `AdaptiveSheetFrame({required Widget child})`, with `maxWidth = 560`; narrow or non-iOS windows keep the existing bottom presentation.
- Consumes: the existing `DingTalkSheetShell`; selected values, cancel returns, and panel callbacks are unchanged.

- [ ] **Step 1: Write failing widget tests.** In `test/adaptive_sheet_frame_test.dart`, pump `showDingTalkSheet` at iOS widths 1100 and 500, then assert:

  ```dart
  expect(cardRect.width, lessThanOrEqualTo(560)); // width 1100
  expect(cardRect.width, closeTo(500, 1)); // width 500
  expect(find.text('取消'), findsOneWidget); // 30 options; list scrolls
  expect(cancelRect.bottom, lessThanOrEqualTo(availableHeight - 320)); // keyboard inset
  ```
- [ ] **Step 2: Run `flutter test test/adaptive_sheet_frame_test.dart` and confirm the new tests fail.**
- [ ] **Step 3: Implement the frame and use it in both shared sheet entry points.** Center the wide card with full corner rounding, bound its height by the safe area and keyboard inset, and leave narrow behavior unchanged. Keep `DingTalkSheetShell`'s option/result semantics intact.
- [ ] **Step 4: Run `flutter test test/adaptive_sheet_frame_test.dart`; confirm all pass.**
- [ ] **Step 5: Commit as `feat(ipad): constrain shared sheets`.**

### Task 3: Readable task forms and end-to-end verification

**Files:**
- Create: `lib/design/adaptive_form_body.dart` — constrain form bodies while preserving full-height scrolling.
- Modify: `lib/page/task/task_create_page.dart` and `lib/page/task/task_edit_page.dart` — wrap only their `ListView` bodies, retaining existing navigation bars and save/cancel methods.
- Modify: `docs/ios-port.md` — record tested iPad behavior and limitations.
- Test: `test/adaptive_form_body_test.dart`.

**Interfaces:**
- Produces: `AdaptiveFormBody({required Widget child})`, with `maxWidth = 720`; the wrapper uses available width and preserves full height.
- Consumes: the existing form `ListView` widgets; no Task model or persistence changes.

- [ ] **Step 1: Write failing widget tests.** In `test/adaptive_form_body_test.dart`, use a keyed text field inside `AdaptiveFormBody`:

  ```dart
  expect(formRect.width, lessThanOrEqualTo(720)); // iOS width 1100
  expect(formRect.width, closeTo(500, 1)); // iOS width 500
  expect(find.text('保留这段输入'), findsOneWidget); // after resize 1100 → 500
  expect(tester.takeException(), isNull); // keyboard inset and focused field
  ```
- [ ] **Step 2: Run `flutter test test/adaptive_form_body_test.dart` and confirm the new tests fail.**
- [ ] **Step 3: Implement the wrapper and apply it to both task forms.** Keep the current `CupertinoPageScaffold`/navigation bars full-page and preserve child form state across width changes.
- [ ] **Step 4: Run `flutter test test/adaptive_form_body_test.dart` and the complete `flutter test` suite; confirm all pass.** Run `dart analyze` on changed Dart files with no errors.
- [ ] **Step 5: Build with `flutter build ios --simulator --no-codesign`; install and inspect on iPad mini and iPad Pro simulators at portrait and landscape sizes, and verify iPhone still shows bottom tabs.** Exercise all five tabs, task create/edit, sheets, and keyboard entry where simulator control is available; document any UI-control or signing limit without claiming that case passed.
- [ ] **Step 6: Run `git diff --check`, fetch `upstream/main`, and run `git merge-tree --write-tree HEAD refs/remotes/upstream/main`; update `docs/ios-port.md`, commit as `feat(ipad): constrain task forms and verify tablet layout`, then push the existing fork PR branch.**
