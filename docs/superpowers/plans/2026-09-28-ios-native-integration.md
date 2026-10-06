# iOS native integration implementation plan

Spec: `docs/superpowers/specs/2026-09-28-ios-native-integration-design.md`

1. Establish a clean baseline and repair only build-blocking repository defects. Verify `flutter test` and an iOS simulator build. Keep generated files out of commits.
2. Add the iOS Share Extension and App Group inbox, then connect it to the existing share channels. Verify text, image and file paths in a simulator; test inbox parsing and drain behavior.
3. Add an iOS alarm bridge. Route iOS 26+ task alarms through AlarmKit, and older iOS through local notifications. Update manual alarm UX and cancellation/snooze. Verify with focused Dart tests and simulator build.
4. Verify background registration against the exact Workmanager fork in the project. Fix registration or user-facing wording only where evidence shows a gap. Verify scheduling and run the available background debug hook if possible.
5. Add a minimal Shortcuts quick-add action if the core paths are green. Verify invocation into the task-create flow.
6. Run full Flutter tests, static analysis where useful, iOS simulator smoke tests and unsigned archive. Review mergeability against fork main. Make focused commits and prepare a PR if GitHub authentication permits; request user Apple sign-in only when signing is the remaining blocker.

Decision ledger: execute inline on branch `feat/ios-native-integration` in `/Users/m1/cs/小巧思/Elychron-ios-port`. User authorized execution. No subagents requested.
