# Elychron iOS native integration

## Scope

Port the existing Flutter app in the user's Elychron fork to iOS 15+. Preserve the Android implementation and keep shared Dart changes small so the fork can merge future Elychron changes. The first deliverable is a buildable iOS app and, after the user's Apple account is configured on this Mac, an installable development IPA.

## Behaviors

- Receive text, images and files from the iOS Share sheet. A Share Extension saves payloads in an App Group container and tells the user to return to Elychron. iOS does not guarantee that Share Extensions may open their containing app. The host drains each payload once through the existing `celechron/share` channels when opened or resumed. Failed attachments surface as unreadable items. The extension never invokes Flutter directly.
- iOS 26+ uses AlarmKit for app-owned alarms. Existing task reminders stay synced to task edits, completion and deletion. iOS 15–25 uses actionable local notifications. A manual “system alarm” entry on iOS 26+ creates an Elychron AlarmKit alarm; the UI never claims to write to Apple Clock. On older iOS it explains the limitation.
- Background scholarly refresh uses the existing Workmanager/BGTaskScheduler integration. The app presents it as opportunistic: the system chooses execution time; it is not a 15-minute guarantee.
- Add a small iOS Shortcuts “create task” action only after the core paths work. Lock Screen widgets and focus Live Activity remain separate follow-up work to reduce merge conflicts.

## Boundaries and validation

- App Group, URL scheme, bundle IDs and entitlements must agree across host and extension. Production signing values remain configurable; never commit an Apple account or credential.
- Keep iOS-specific code in `ios/` and expose a narrow MethodChannel interface to Dart. Avoid changing upstream data models.
- Run Flutter tests, iOS simulator build and smoke test, and an unsigned device archive. Real-device installation and signed IPA require the user's Apple login in Xcode and a connected device or signing certificate.
- Check `git merge-tree` or a dry-run merge against the fork's latest main, and document any remaining conflict.
