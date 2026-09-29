# Elychron iPad adaptive layout

## Intent and scope

Make the existing universal iOS app comfortable on iPad mini through iPad Pro in portrait, landscape, and narrower multitasking windows. Keep one Flutter app and one iOS target. Preserve task, calendar, focus, school, settings, share, and alarm behavior. Keep the iPhone and desktop layouts intact and keep the fork easy to merge with Elychron upstream.

The user chose a tablet-style sidebar on wide windows and the existing bottom tab bar on narrow windows. The sidebar appears on the five main pages; pushed detail and editor pages retain their existing full-page navigation and back behavior. This is an iPad layout pass, not a rewrite of the page hierarchy or a new iPad-only feature set.

## Layout behavior

- Choose the layout from the **current Flutter window width**, not the physical device model, so iPad split view and resizing can switch modes. On iOS, use a breakpoint around 900 logical pixels: below it, keep the current bottom tab bar; at or above it, show a left sidebar. The exact breakpoint may be adjusted against simulator screenshots to ensure the remaining content width stays usable.
- Keep the existing `HomePage` page list, `PageController`, selected index, and `HomeModHooks`. Switching width changes only the navigation shell, without rebuilding business pages or losing the selected tab and scroll position.
- Reuse the existing desktop rail's five destinations and styling where practical, but omit desktop-only drag-and-drop copy on iPad. The sidebar is part of the main page layout, so pushed routes can continue to occupy the full app window and show their normal back controls.
- On wide windows, center the main page content and cap its readable width near the desktop app's 920 logical pixel limit. Backgrounds continue across the available area. On narrow windows, retain the current full-width layout and bottom bar.
- Give shared choice and information sheets a bounded width near 560 logical pixels on wide iPad windows, with safe-area and keyboard insets respected. Keep their current phone presentation at narrow widths. Limit the body width of the common task create/edit forms to roughly 720 logical pixels on wide windows while retaining the existing full-page navigation bars and save/cancel behavior.

## Alternatives considered

1. **Adapt `HomePage` in place (chosen).** It already owns all five pages and tab state. A width branch and shared width wrappers make the smallest change and avoid a second iPad navigation state.
2. Reuse `DesktopFrame` and `DesktopHome` as the iPad root. This would keep the sidebar on every pushed route, but changes root navigation, share hooks, and route behavior. It also carries desktop drag-and-drop semantics into iPad.
3. Build a separate iPad shell and duplicate the five page routes. This creates divergent behavior and increases upstream merge work.

Flutter's [adaptive layout guidance](https://docs.flutter.dev/ui/adaptive-responsive/general) recommends measuring available window space and switching navigation patterns at a breakpoint. The [Flutter samples navigation implementation](https://github.com/flutter/samples/blob/8a4cf1db16d52741f0e59e1bfe818723430c35bc/material_3_demo/lib/src/home.dart) demonstrates the same width-driven rail/bar pattern. The sample is Material; Elychron keeps its Cupertino styling and existing page state.

## Boundaries and error cases

- Do not change data models, native iOS entitlements, Bundle IDs, share queue semantics, alarms, or signing settings.
- Avoid device-model checks and global orientation locks. The iPad target already allows portrait and landscape.
- When a window becomes narrow while a route or sheet is open, content must remain reachable; the user must be able to cancel/save without an overflow or hidden button.
- Do not move `HomeModHooks` or create a second listener when the layout branch changes. The sidebar and bottom bar must call the same page-selection path.
- Keep new adaptive logic in a small shared layout helper or focused widgets, and avoid formatting or restructuring unrelated upstream files.

## Verification

- Widget tests cover both sides of the breakpoint, destination order, tab selection, and preservation of the selected page after a width change. Add focused tests for sheet/form width and keyboard-safe behavior only where a concrete layout regression is identified.
- Run the full Flutter test suite and static analysis of changed files. Build the iOS simulator app.
- Install and inspect on iPad mini and a larger iPad simulator in portrait and landscape. Check a narrow window where the simulator permits it. Smoke-test all five main tabs, task create/edit, shared choice sheets, and keyboard entry. Verify iPhone layout remains the bottom-tab version.
- iPad layout verification does not require Apple signing. Signed IPA and native App Group/AlarmKit runtime verification remain part of the separate signing work.
