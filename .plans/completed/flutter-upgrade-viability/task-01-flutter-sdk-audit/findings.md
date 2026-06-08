# Flutter SDK Upgrade Path Audit — Findings

**Audit Date**: 2026-06-08
**From**: Flutter 3.32.4 / Dart 3.8.1
**To**: Flutter 3.38.7 / Dart 3.10.7

---

## Summary

The upgrade path from Flutter 3.32.4 to 3.38.7 crosses two major stable releases — Flutter 3.35 and Flutter 3.38 — with no intermediate dedicated stable release notes for 3.33, 3.34, 3.36, or 3.37 (Flutter's versioning skips those; bug-fix patches appear as 3.32.x, 3.35.x, and 3.38.x on the stable channel). The most impactful change for this project is the **UISceneDelegate adoption** on iOS, which requires manual migration because the `AppDelegate.swift` is customized (Pendo integration + lifecycle hooks prevent auto-migration). The Android build already uses Java 17, satisfying Flutter 3.38's new minimum. Dart 3.10 adds dot-shorthand syntax and a few SDK-level breaking changes, none of which affect this codebase directly.

---

## Flutter Breaking Changes (3.32 → 3.38)

> **Note on version numbering**: The Flutter stable channel releases documentation for 3.32.0, 3.35.0, and 3.38.0 only. Versions 3.33, 3.34, 3.36, and 3.37 do not have dedicated stable release note pages; intermediate fixes ship as patch releases (e.g., 3.32.1, 3.32.2). Breaking changes are grouped under the stable minor where they were introduced.

---

### Flutter 3.35

#### 1. iOS minimum version raised to 13.0
- **Change**: iOS 13.0 `@available` checks dropped; iOS 12 no longer supported.
- **Impact**: LOW — iOS 12 market share is negligible. Syncro likely already targets iOS 13+.
- **Affected files**: `ios/Podfile` (deployment target), `ios/Runner.xcodeproj`

#### 2. macOS minimum version raised to 10.15
- **Change**: macOS 10.14 (Mojave) no longer supported.
- **Impact**: LOW — project is iOS/Android only; macOS target not active.

#### 3. `AppBarTheme.color` deprecated → `backgroundColor`
- **Change**: `AppBarTheme` and `AppBarThemeData` `color` parameter deprecated in favour of `backgroundColor`.
- **Impact**: MEDIUM — generates deprecation warnings. Not a runtime break; must be resolved before a future removal.
- **Affected files**: Any file using `AppBarTheme(color: ...)` — check theme files.

#### 4. `DropdownButtonFormField.value` deprecated → `initialValue`
- **Change**: The `value` constructor parameter renamed to `initialValue`.
- **Impact**: MEDIUM — deprecation warning. Old parameter still works until removed.
- **Affected files**: Any screen using `DropdownButtonFormField(value: ...)`.

#### 5. `Slider.year2023` deprecated
- **Change**: The `year2023` flag on `Slider` is deprecated; Material 2024 design is now the default.
- **Impact**: LOW — only affects apps that explicitly set `Slider.year2023 = true`.

#### 6. `Switch.activeColor` deprecated → `activeThumbColor`
- **Change**: The `activeColor` property on `Switch` renamed to `activeThumbColor` for clarity.
- **Impact**: MEDIUM — deprecation warning. Not a runtime break.

#### 7. `SemanticsProperties.elevation` and `.thickness` removed
- **Change**: These two accessibility properties were removed from `SemanticsProperties`.
- **Impact**: LOW — these were rarely used; removal causes a compile error only if referenced.
- **Affected files**: Any custom `Semantics(...)` call using `elevation:` or `thickness:`.

#### 8. `Form` widget can no longer be used as a sliver
- **Change**: Using `Form` directly inside a `CustomScrollView` as a sliver now throws.
- **Impact**: LOW — uncommon pattern; only affects code that wraps `Form` as a sliver.

#### 9. Android default `abiFilters` now set by Flutter
- **Change**: Flutter sets default ABI filters automatically. Manual `abiFilters` in `build.gradle` may conflict.
- **Impact**: MEDIUM — if the project's `app/build.gradle` defines custom `abiFilters`, they may conflict with Flutter's defaults and cause build errors or unexpected behavior.
- **Affected files**: `android/app/build.gradle`

#### 10. `Visibility` widget no longer focusable by default when `maintainState: true`
- **Change**: `Visibility(maintainState: true)` no longer keeps the widget focusable by default. Use `maintainFocusability: true` to restore old behavior.
- **Impact**: LOW — only relevant if accessibility/focus trees are explicitly tested.

#### 11. `$FLUTTER_ROOT/version` file removed
- **Change**: The plain text `$FLUTTER_ROOT/version` file replaced by `$FLUTTER_ROOT/bin/cache/flutter.version.json`.
- **Impact**: LOW — only affects custom CI/CD scripts that parse that file directly.
- **Affected files**: Any CI scripts reading `$FLUTTER_ROOT/version`.

#### 12. `showCupertinoSheet` parameter renamed: `pageBuilder` → `builder`
- **Change**: The `pageBuilder` parameter in `showCupertinoSheet` renamed to `builder`.
- **Impact**: MEDIUM — compile error if this API is used. Codebase search needed to confirm usage.

#### 13. Android: `SystemChrome.setPreferredOrientations` will not work on Android 16+ (API 36)
- **Change**: This API no longer controls orientation on Android 16+. A compile-time warning is added.
- **Impact**: MEDIUM — if Syncro locks orientation programmatically (likely for certain views), this will silently fail on Android 16 devices.
- **Affected files**: Any code calling `SystemChrome.setPreferredOrientations(...)`.

---

### Flutter 3.38

#### 1. UISceneDelegate adoption — iOS (HIGH IMPACT)
- **Change**: Flutter 3.38 introduces `UISceneDelegate`-based lifecycle. Apps with an **unmodified** `AppDelegate` are auto-migrated. Apps with customized `AppDelegate` require **manual migration**.
- **Impact**: HIGH — `AppDelegate.swift` in this project is customized: it imports Pendo, registers plugins manually, implements `applicationWillResignActive`/`applicationDidBecomeActive` (screen-blur pattern), and handles Pendo URL scheme. Auto-migration will NOT run. Manual steps required: move `GeneratedPluginRegistrant.register` to `didInitializeImplicitFlutterEngine`, create a `SceneDelegate`, update `Info.plist` with `UIApplicationSceneManifest`. The screen-blur lifecycle hooks may need rethinking under UIScene semantics (`sceneWillResignActive`/`sceneDidBecomeActive`).
- **Affected files**: `ios/Runner/AppDelegate.swift`, `ios/Runner/Info.plist` (new `UIApplicationSceneManifest` key needed), potentially a new `ios/Runner/SceneDelegate.swift`.
- **Deadline**: Required before iOS 26 SDK builds; auto-required in Flutter 3.41.

#### 2. Android minimum Java version raised to 17
- **Change**: Java 11 support removed; Java 17 is now the minimum for Android toolchain.
- **Impact**: LOW — `android/app/build.gradle` already specifies `JavaVersion.VERSION_17` and `jvmTarget = '17'`. `android/build.gradle` uses `JavaVersion.VERSION_21`. No change needed.
- **Affected files**: None (already compliant).

#### 3. Android default page transition is now `PredictiveBackPageTransitionBuilder`
- **Change**: Android page transitions now default to predictive-back gesture animations. Previously the default was `ZoomPageTransitionsBuilder`.
- **Impact**: MEDIUM — visual change on Android. If the app uses `go_router` (it does) the transition animations will change on Android 14+ devices. No code break, but QA team should validate UI/UX on Android 14+.
- **Affected files**: Possibly `main.dart` / `app_theme.dart` if explicit `PageTransitionsTheme` is configured.

#### 4. SnackBar with action no longer auto-dismisses
- **Change**: Any `SnackBar` with an action button is now persistent by default (`persist: true`). Previously it auto-dismissed after the duration. Set `persist: false` to restore old behavior.
- **Impact**: MEDIUM — any `SnackBar` with an `action:` parameter will no longer auto-dismiss, potentially blocking the UI until the user taps. Codebase-wide search for `SnackBar(` with `action:` needed.
- **Affected files**: Multiple screens; search for `SnackBar(` + `action:`.

#### 5. `CupertinoDynamicColor` — wide gamut deprecations
- **Change**: `.red`, `.green`, `.blue`, `.opacity`, and `.withOpacity()` deprecated on `CupertinoDynamicColor`. Use `.r`, `.g`, `.b`, `.a`, and `.withValues()` instead.
- **Impact**: LOW — deprecation warnings only; old accessors still work. Only affects code using `CupertinoDynamicColor` color channel access directly (uncommon in most business apps).

#### 6. `OverlayPortal.targetsRootOverlay` deprecated
- **Change**: The named constructor `OverlayPortal.targetsRootOverlay(...)` deprecated. Use `OverlayPortal(overlayLocation: OverlayChildLocation.rootOverlay, ...)` instead.
- **Impact**: LOW — only affects code directly using `OverlayPortal.targetsRootOverlay`. Unlikely in this codebase.

#### 7. `SemanticsProperties.focusable` and `SemanticsConfiguration.isFocusable` deprecated
- **Change**: These accessibility properties deprecated in favour of `isFocused` (tri-state model).
- **Impact**: LOW — only affects custom `Semantics(...)` widgets using these properties.

#### 8. `AssetManifest.json` file removed
- **Change**: The legacy `AssetManifest.json` file (deprecated in Flutter 3.8) has been fully removed. Asset loading now uses `AssetManifest.bin` only.
- **Impact**: LOW — only affects code using `AssetBundle.loadString('AssetManifest.json')` directly. Standard `AssetImage` and `rootBundle` usage is unaffected.

#### 9. Jetifier removed from Android build
- **Change**: `jetifier` support removed from Flutter's Android build tooling.
- **Impact**: MEDIUM — if any third-party plugin still requires Jetifier to convert Java library bytecode, the build will break. All modern plugins (firebase_*, flutter_bloc, go_router) already support AndroidX natively. Check with `./gradlew dependencyInsight` if build errors appear.

#### 10. `js_util` removed from web (dart2js/dart2wasm)
- **Change**: All usages of the deprecated `js_util` package removed from Flutter framework's web targets.
- **Impact**: LOW — Syncro is iOS/Android only; this is a web-only change.

#### 11. iOS debugging: LLDB replaces previous method for iOS 17+ / Xcode 26+
- **Change**: LLDB is now the default debugging method for iOS 17+ with Xcode 26+. The old debug bridge option is removed.
- **Impact**: LOW — transparent to application code; affects local dev tooling only.

---

## Dart Breaking Changes (3.9 → 3.10)

### Dart 3.9

#### 1. Type promotion assumes null safety always active
- **Change**: Type promotion, reachability, and definite assignment analysis now always assumes null safety (no longer checks for pre-null-safety opt-out). This may produce new `dead_code` warnings.
- **Impact**: LOW — this project is already fully null-safe (`sdk: ">=3.8.0 <4.0.0"`). May surface dead code warnings in older utility code. No runtime impact.

#### 2. Flutter SDK constraint upper bound now enforced
- **Change**: The `flutter` SDK upper bound in `pubspec.yaml`'s `environment` is now enforced during `pub get`. If the installed Flutter version exceeds the upper bound, pub will fail.
- **Impact**: LOW — `pubspec.yaml` specifies `sdk: ">=3.8.0 <4.0.0"` (Dart SDK constraint only). Flutter SDK constraint is not explicitly pinned in this project, so this change has no effect.

---

### Dart 3.10

#### 1. Generator return type inference changed (`sync*` / `async*`)
- **Change**: `sync*` and `async*` functions without an explicit return type now infer non-nullable yields when no `null` is yielded. This may produce new warnings about unnecessary null-aware operators (`?.`).
- **Impact**: LOW — generates analyzer warnings; no runtime change. Review generator functions if any produce lint noise.

#### 2. IPv4 address parsing stricter (leading zeros rejected)
- **Change**: `Uri.parseIPv4Address` and `Uri.parseIPv6Address` no longer allow leading zeros in octets (e.g., `"192.168.001.001"` now throws).
- **Impact**: LOW — only affects code constructing IPv4 URIs with zero-padded octets. Unlikely in this codebase.

#### 3. `RegExp` and `RegExpMatch` can no longer be implemented
- **Change**: Implementing (not extending) `RegExp` or `RegExpMatch` is deprecated and will be removed. Use built-in implementations.
- **Impact**: LOW — only affects custom `RegExp` implementations. Highly unlikely in this codebase.

#### 4. `IOOverrides` can no longer be implemented — only extended
- **Change**: `dart:io IOOverrides` must be extended, not implemented. Any class using `implements IOOverrides` will cause a compile error.
- **Impact**: LOW — only relevant in test infrastructure overriding IO. Check test helpers if any.

#### 5. `dart:js_util` and `package:js` removed from Wasm compilation
- **Impact**: LOW — Syncro is iOS/Android only.

#### 6. Dart CLI split from VM (`dart` vs `dartvm`)
- **Change**: The `dart` executable is now purely for CLI dispatch; the VM binary is `dartvm`. CI scripts running `dart` for app execution are unaffected; custom scripts calling the VM directly may need updating.
- **Impact**: LOW — transparent for standard `fvm flutter` workflows.

#### 7. `@required` annotation from `package:meta` no longer supported
- **Change**: The pre-null-safety `@required` annotation is removed. Only the native Dart `required` keyword is supported.
- **Impact**: LOW — this project uses Dart null-safety keywords. The `meta` package's `@required` is a pre-2.12 artifact. Any occurrence in this codebase would already be a stale remnant.

#### 8. New dot shorthand syntax (language feature)
- **Change**: Dart 3.10 introduces `.enumValue` / `.StaticMember` shorthand when the type can be inferred. This is additive and backwards-compatible; no existing code breaks.
- **Impact**: LOW (positive) — no migration required; optional new syntax.

---

## FVM Configuration Change

**Current `.fvmrc` value**:
```json
{ "flutter": "3.32.4" }
```

**Required change**: Update `"flutter"` to `"3.38.7"`.

```json
{ "flutter": "3.38.7" }
```

**Note**: The version `3.38.7` is already installed globally on this machine (reported as "needs setup" — meaning it is installed but `.fvmrc` does not yet point to it). After updating `.fvmrc`, run `fvm use 3.38.7` from the `syncro-flutter/` directory to re-link the SDK symlink. No other FVM configuration changes are needed.

---

## Conclusion

The SDK upgrade is **mostly mechanical with one HIGH-priority manual task**:

1. **HIGH — UISceneDelegate iOS migration (manual)**: `AppDelegate.swift` is customized (Pendo SDK, screen-blur pattern, URL scheme). Auto-migration will not run. This requires creating `SceneDelegate.swift`, updating `AppDelegate.swift` to use `FlutterImplicitEngineDelegate`, and updating `Info.plist`. Estimate: ~2–4h including testing Pendo deep links and screen-blur behavior on device. Required before iOS 26 SDK; mandatory in Flutter 3.41.

2. **MEDIUM — SnackBar persistence audit**: All `SnackBar` widgets with `action:` now persist until user interaction. Search the codebase for `SnackBar(` + `action:` and add `persist: false` where auto-dismiss is the intended UX.

3. **MEDIUM — Android predictive-back transitions**: QA must validate navigation animations on Android 14+ devices. No code change required unless explicit `PageTransitionsTheme` override is desired.

4. **MEDIUM — Deprecation sweep** (non-blocking, but accumulates): `AppBarTheme.color`, `DropdownButtonFormField.value`, `Switch.activeColor`, `showCupertinoSheet pageBuilder` — these generate warnings but do not break builds. Should be cleaned up in the same branch.

5. **MEDIUM — `SystemChrome.setPreferredOrientations` on Android 16+**: If orientation locking is used (check codebase), it will silently fail on Android 16 devices. Needs investigation.

6. **LOW — Android Jetifier removal**: Verify build succeeds with no Jetifier dependency; all current plugins appear AndroidX-native.

The Dart 3.9 and 3.10 changes have no significant impact on this codebase. The Flutter version in `.fvmrc` is a one-line change.

---

## Sources

- [Flutter Breaking Changes and Migration Guides](https://docs.flutter.dev/release/breaking-changes) — official master list of all breaking changes with migration guides
- [Flutter 3.35.0 Release Notes](https://docs.flutter.dev/release/release-notes/release-notes-3.35.0) — full 3.35 changelog
- [Flutter 3.38.0 Release Notes](https://docs.flutter.dev/release/release-notes/release-notes-3.38.0) — full 3.38 changelog
- [What's New in Flutter 3.38 (Flutter Blog)](https://blog.flutter.dev/whats-new-in-flutter-3-38-3f7b258f7228) — feature narrative for 3.38
- [UISceneDelegate Adoption Migration Guide](https://docs.flutter.dev/release/breaking-changes/uiscenedelegate) — step-by-step iOS migration
- [Android Java Gradle Migration Guide](https://docs.flutter.dev/release/breaking-changes/android-java-gradle-migration-guide) — Java 17 toolchain migration
- [Dart Breaking Changes](https://dart.dev/resources/breaking-changes) — official Dart breaking-changes list
- [Dart Changelog](https://dart.dev/changelog) — full Dart SDK changelog
- [Announcing Dart 3.10](https://dart.dev/blog/announcing-dart-3-10) — Dart 3.10 feature announcement
- [Dart What's New](https://dart.dev/resources/whats-new) — release history for Dart 3.9 and 3.10
- [Flutter 3.38 Breaking Changes — Migration Guide (vagary.tech)](https://vagary.tech/blog/flutter-3-38-release-notes-breaking-changes-migration-guide) — community summary
