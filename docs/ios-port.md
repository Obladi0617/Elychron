# Elychron iOS port

## Supported behavior

| Feature | iOS behavior |
| --- | --- |
| Tasks, calendar, focus, existing widgets | Existing Flutter and WidgetKit implementations; compile and launch on iOS 27 simulator |
| Share to Elychron | Share Extension saves text, image, or file in an App Group inbox. Separate shares are processed one at a time and retained until handled. Open Elychron after saving to choose a new or existing task. Runtime transfer still needs a signed App Group test. |
| Task alarm mode | iOS 26+ schedules an app-owned AlarmKit alarm. iOS 15–25 and denied AlarmKit authorization fall back to local notifications with snooze/dismiss actions. In-app alarm pages loop the bundled synthetic chime. |
| Manual native alarm | iOS 26+ creates an independent one-time Elychron alarm. It is not added to Apple Clock and does not follow task edits. |
| Background scholarly refresh | Existing BGAppRefresh/Workmanager integration; iOS decides whether and when to run it. The 15-minute interval is only an earliest request. |
| Shortcuts quick add | Use Shortcuts’ “Open URLs” action with `elychron://todo/create`. The simulator recognizes the URL and displays Elychron as its target; opening the task editor still needs an unlocked simulator confirmation. |

## Build and signing

- Minimum iOS version: 15.0. Use Xcode 27 and the repository's Flutter SDK/dependencies.
- App IDs: `com.obladi0617.elychron` (Release) and `com.obladi0617.elychron.debug` (Debug), with corresponding widget and share extensions. App Groups are `group.com.obladi0617.elychron` and `group.com.obladi0617.elychron.debug`.
- Xcode signing is Automatic. In Xcode, sign in to your Apple account and select the same team for Runner, WidgetExtensions, and ShareExtension. Keep App Groups enabled for all three targets. The original author's team and provisioning profile were removed from this fork.
- For simulator: `flutter build ios --simulator --no-codesign`. For a compile-only device build: `flutter build ios --release --no-codesign`. The latter produces an **unsigned** `build/ios/iphoneos/Runner.app`; it is not an installable IPA.
- A signed IPA can be exported only after Xcode has a valid team, development certificate, provisioning profile, and target device. A free Personal Team can test on its own registered device for a limited period; broad distribution needs the paid Apple Developer Program. Never share an Apple password or verification code in chat.

## Merge strategy

The iOS native implementations live in `ios/ShareSupport`, `ios/ShareExtension`, and `ios/Runner/AppDelegate.swift`. Dart changes reuse existing share/alarm channels and touch only the task reminder route and platform wording. The iOS Xcode project necessarily changes to embed ShareExtension and use fork-owned signing identifiers. Rebase this branch onto the fork's `main`, or merge the latest Elychron into the fork first, then merge this branch. Expect to inspect `ios/Runner.xcodeproj/project.pbxproj` if future Elychron releases add Xcode targets.

## Verification

- `flutter test`: 672 tests pass.
- `swiftc -module-cache-path /private/tmp/elychron-swift-module-cache ios/ShareSupport/ShareInbox.swift test_native/share_inbox/main.swift -o /private/tmp/elychron-share-inbox-test && /private/tmp/elychron-share-inbox-test`: passes.
- iOS 27 simulator build and launch: passes. The simulator build was unsigned, so App Group delivery and AlarmKit authorization were not exercised end to end.
- `elychron://todo/create`: recognized by iOS in the simulator; first-open confirmation was not tapped because the remote Mac is locked.
- iOS device Release build without code signing: passes.
