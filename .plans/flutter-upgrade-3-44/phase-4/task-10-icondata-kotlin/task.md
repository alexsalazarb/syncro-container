# Task: IconData final + Kotlin AGP Migration

**Plan**: Flutter Upgrade 3.32.4 → 3.44.1
**Phase**: 4 — Routing + Yellow Packages
**Task ID**: task-10
**Task Path**: phase-4/task-10-icondata-kotlin
**Depends On**: task-01
**JIRA**: SE-12530

## Objective

Fix two breaking changes from Flutter 3.44: (1) `IconData` is now `final` — any class extending/implementing it causes a compile error; (2) migrate Android build from `id "kotlin-android"` to the built-in Kotlin plugin (required before AGP 9).

## Context

### Breaking Change 1: IconData final (Flutter 3.44)

`IconData` is marked `final`. Classes using `extends IconData` or `implements IconData` fail with:
`'IconData' is 'final' and can't be extended or implemented outside of its library`

**5 files need inspection:**
- `lib/features/chat/presentation/widget/chat_item.dart`
- `lib/features/alerts/presentation/widgets/alert_item_widget.dart`
- `lib/core/global_widgets/molecules/icon_text_button_in_row.dart`
- `lib/core/global_widgets/custom_radio_button.dart`
- `lib/core/global_widgets/atoms/custom_icon_widget.dart`

### Breaking Change 2: Kotlin AGP 9 migration (Flutter 3.44 forward)

AGP 9 removes `id "kotlin-android"` plugin support. Flutter 3.44 adds a temporary shim, but it must be done before bumping to AGP 9.

**Files to update:**
- `android/app/build.gradle` — has `id "kotlin-android"` and `kotlinOptions { jvmTarget = '17' }`
- `android/settings.gradle` — has `id "org.jetbrains.kotlin.android" version "2.0.21" apply false`

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/chat/presentation/widget/chat_item.dart` | review/modify | Check IconData usage |
| `syncro-flutter/lib/features/alerts/presentation/widgets/alert_item_widget.dart` | review/modify | Check IconData usage |
| `syncro-flutter/lib/core/global_widgets/molecules/icon_text_button_in_row.dart` | review/modify | Check IconData usage |
| `syncro-flutter/lib/core/global_widgets/custom_radio_button.dart` | review/modify | Check IconData usage |
| `syncro-flutter/lib/core/global_widgets/atoms/custom_icon_widget.dart` | review/modify | Check IconData usage |
| `syncro-flutter/android/app/build.gradle` | modify | Remove id "kotlin-android", update kotlinOptions |
| `syncro-flutter/android/settings.gradle` | review | Verify kotlin plugin reference |

### Do NOT Modify
- `pubspec.yaml` — already updated in task-01
- Files owned by other tasks

## Implementation Steps

### Step 1: Check IconData usage in 5 files

For each file, search for `extends IconData` or `implements IconData`:
```bash
rg "extends IconData|implements IconData" lib/features/chat/presentation/widget/chat_item.dart lib/features/alerts/presentation/widgets/alert_item_widget.dart lib/core/global_widgets/ --type dart
```

If found, replace with a wrapper class:
```dart
// BEFORE (won't compile in Flutter 3.44):
class AppIcon implements IconData { ... }

// AFTER:
class AppIcon {
  final IconData icon;
  const AppIcon(this.icon);
  // expose needed properties via delegation
}
```

### Step 2: Migrate Kotlin AGP 9

In `android/app/build.gradle`:
```groovy
// BEFORE:
plugins {
    id "com.android.application"
    id "kotlin-android"   // REMOVE THIS
    id "dev.flutter.flutter-gradle-plugin"
}
// ...
kotlinOptions {
    jvmTarget = '17'    // REMOVE THIS BLOCK
}

// AFTER:
plugins {
    id "com.android.application"
    id "dev.flutter.flutter-gradle-plugin"
}
// Add at top level:
kotlin {
    compilerOptions {
        jvmTarget = JvmTarget.JVM_17
    }
}
```

In `android/settings.gradle`, remove or comment the kotlin-android plugin line if it's redundant after the above change.

### Step 3: Verify

```bash
cd syncro-flutter
fvm flutter analyze lib/features/chat/presentation/widget/chat_item.dart lib/features/alerts/presentation/widgets/alert_item_widget.dart lib/core/global_widgets/
fvm flutter build apk --debug
```

## Testing

- [ ] No `extends IconData` or `implements IconData` errors in flutter analyze
- [ ] Android APK debug build succeeds

## Completion Criteria

- [ ] All 5 IconData files inspected; any violations fixed
- [ ] `id "kotlin-android"` removed from android/app/build.gradle
- [ ] Android build succeeds
- [ ] Status updated in status.md
