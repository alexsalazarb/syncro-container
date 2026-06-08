# Task 03 — Exclude RUNNING_TIMER_ENTRY from Android Auto-Backup

## JIRA

[SE-12509](https://syncrotech.atlassian.net/browse/SE-12509)

## Objective

Android Auto-Backup (enabled by default on Android 6.0+) backs up SharedPreferences to Google
Drive and restores them on reinstall. This is why Derrick's stuck timer survived uninstall +
reinstall — the stale `RUNNING_TIMER_ENTRY` was restored from backup.

Exclude the `RUNNING_TIMER_ENTRY` SharedPreferences key from backup by adding Android backup rules.
This is a preventive fix that ensures future reinstalls start with a clean timer state.

> **Note**: task-02 (startup validation) is still needed — it handles existing stuck states already
> present in SP. This task prevents new ones from surviving reinstalls going forward.

## Dependencies

None — task-03 is independent.

## File Ownership

**Create:**
- `syncro-flutter/android/app/src/main/res/xml/backup_rules.xml`

**Modify:**
- `syncro-flutter/android/app/src/main/AndroidManifest.xml`

**Do NOT modify:**
- Any Dart/Flutter source files

---

## Implementation Steps

### Step 1 — Read existing AndroidManifest.xml

Read `syncro-flutter/android/app/src/main/AndroidManifest.xml` to check:
- Current `android:allowBackup` value
- Whether `android:fullBackupContent` or `android:dataExtractionRules` already exist
- The `<application>` tag attributes

### Step 2 — Determine the right backup rules approach

Flutter's `shared_preferences` plugin stores data in a SharedPreferences XML file named after
the app package. The file is typically `{package_name}_preferences.xml`.

**For Android 12+ (API 31+)**: use `android:dataExtractionRules`
**For Android 6–11 (API 23–30)**: use `android:fullBackupContent`

To support both, provide both attributes pointing to the same or different XML files.

Prefer providing both for maximum coverage:

```xml
<!-- In <application>: -->
android:fullBackupContent="@xml/backup_rules"
android:dataExtractionRules="@xml/data_extraction_rules"
```

### Step 3 — Create backup_rules.xml (API 23–30)

Create `syncro-flutter/android/app/src/main/res/xml/backup_rules.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <!--
        Exclude the running timer entry from backup/restore.
        A stale RUNNING_TIMER_ENTRY restored after reinstall causes a stuck timer
        to appear across all tickets. See SE-12509.
    -->
    <exclude domain="sharedpref" path="RUNNING_TIMER_ENTRY" />
</full-backup-content>
```

> **Note on `domain="sharedpref"`**: This excludes the named key from the default SharedPreferences
> file. If the app uses a named SharedPreferences file (check the `shared_preferences` plugin), the
> path may need to be the full filename instead.

### Step 4 — Create data_extraction_rules.xml (API 31+)

Create `syncro-flutter/android/app/src/main/res/xml/data_extraction_rules.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<data-extraction-rules>
    <cloud-backup>
        <exclude domain="sharedpref" path="RUNNING_TIMER_ENTRY" />
    </cloud-backup>
    <device-transfer>
        <exclude domain="sharedpref" path="RUNNING_TIMER_ENTRY" />
    </device-transfer>
</data-extraction-rules>
```

### Step 5 — Update AndroidManifest.xml

In the `<application>` tag, add (or update) the backup attributes:

```xml
android:fullBackupContent="@xml/backup_rules"
android:dataExtractionRules="@xml/data_extraction_rules"
```

If `android:allowBackup="false"` already exists — that disables backup entirely which already
achieves our goal. In that case, verify and document but skip creating the XML files.

---

## Acceptance Criteria

- `backup_rules.xml` exists with `<exclude domain="sharedpref" path="RUNNING_TIMER_ENTRY" />`
- `data_extraction_rules.xml` exists with cloud-backup and device-transfer exclusions for `RUNNING_TIMER_ENTRY`
- `AndroidManifest.xml` references both XML files in the `<application>` tag
- `fvm flutter build apk` completes without errors related to the backup config
- After uninstall + reinstall on Android 6+, `RUNNING_TIMER_ENTRY` is NOT restored from backup

## Testing

No unit tests for this task — it's a configuration change.

**Manual verification** (QA or dev):
1. Set a running timer in the app (start a timer)
2. Force-kill and uninstall the app
3. Reinstall
4. Open the app and navigate to any ticket
5. Verify no stuck timer appears

Run build validation: `fvm flutter build apk`

## Relevant KB

- Android developer docs: `developer.android.com/identity/data/autobackup`
