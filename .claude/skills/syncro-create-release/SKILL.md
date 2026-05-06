---
name: syncro-create-release
description: >
  Builds a production release for both iOS and Android.
  Android: generates a signed AAB via Gradle. iOS: archives via xcodebuild + exports for App Store.
  Trigger: /syncro-create-release or "crear release", "build produccion", "build release".
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## When to Use

- User explicitly says `/syncro-create-release`, "crear release", "build produccion", "build de produccion", "release build"
- Do NOT trigger automatically

---

## Prerequisites Check

Before building, verify:

### Android
```bash
ls syncro-flutter/android/key.properties
ls syncro-flutter/android/app/syncro-mobile-key.keystore
```
Si alguno falta → abort con:
> "Falta `key.properties` o `syncro-mobile-key.keystore`. Copiarlos desde `syncro-temp/android/`. Ambos están gitignoreados — no se commitean."

### iOS
```bash
xcrun xcodebuild -version
```
If fails → abort with:
> "Xcode command line tools no están disponibles. Instalá con `xcode-select --install`."

> **Android signing**: usa un keystore de producción (`syncro-mobile-key.keystore`) referenciado en `android/key.properties`.
> Ambos archivos están gitignoreados y deben estar presentes en la máquina que hace el release.
> Si faltan, copiarlos desde `syncro-temp/android/` (keystore) y `syncro-temp/android/key.properties`.
> SHA1 esperado por Play Store: `FD:29:10:C2:A8:83:C1:A4:EE:33:05:F9:46:18:AB:F1:08:8C:C6:BE`

---

## Step 1 — Flutter Build (required before platform-specific steps)

Desde `syncro-flutter/`:
```bash
fvm flutter build appbundle --flavor production --release
fvm flutter build ios --flavor production --release --no-codesign
```

> El `--no-codesign` en iOS es intencional: el signing lo hace `xcodebuild` en el paso de archive.

---

## Step 2 — Android Bundle

Desde `syncro-flutter/android/`:
```bash
./gradlew app:bundleProductionRelease
```

Artifact resultante: `syncro-flutter/build/app/outputs/bundle/productionRelease/app-production-release.aab`

Reportar ruta exacta al usuario.

---

## Step 3 — iOS Archive

Desde `syncro-flutter/`:

Calcular la ruta del archive en el directorio default de Xcode (para que aparezca automáticamente en Window → Organizer):

```bash
ARCHIVE_DATE=$(date +"%Y-%m-%d")
ARCHIVE_TIME=$(date +"%Y-%m-%d at %H.%M.%S")
ARCHIVE_PATH="$HOME/Library/Developer/Xcode/Archives/$ARCHIVE_DATE/Syncro $ARCHIVE_TIME.xcarchive"
```

```bash
xcodebuild archive \
  -workspace ios/Runner.xcworkspace \
  -scheme production \
  -configuration Release-production \
  -archivePath "$ARCHIVE_PATH" \
  -allowProvisioningUpdates \
  CODE_SIGN_STYLE=Automatic \
  DEVELOPMENT_TEAM=Y3LYQ633EF 2>&1 | tail -5
```

> El archive queda en `~/Library/Developer/Xcode/Archives/` y aparece automáticamente en Xcode → Window → Organizer.

---

## Step 4 — iOS Export (App Store)

```bash
xcodebuild -exportArchive \
  -archivePath "$ARCHIVE_PATH" \
  -exportPath build/ios/export \
  -exportOptionsPlist ios/ExportOptions-production.plist \
  -allowProvisioningUpdates 2>&1 | tail -5
```

Artifact resultante: `syncro-flutter/build/ios/export/syncro.ipa`

Reportar ruta exacta al usuario.

---

## Step 5 — Report

Al finalizar, reportar:

```
✅ Release builds completados

Android AAB: syncro-flutter/build/app/outputs/bundle/productionRelease/app-production-release.aab
iOS IPA:     syncro-flutter/build/ios/export/syncro.ipa

Próximos pasos:
- Android: subir el .aab en Google Play Console → Internal Testing
- iOS: subir el .ipa con Transporter o desde Xcode → Organizer → Distribute
```

---

## Error Handling

| Error | Causa probable | Solución |
|-------|---------------|---------|
| `No signing certificate` | Provisioning profile vencido o ausente | Abrir Xcode → Preferences → Accounts → Download Manual Profiles |
| `Flutter SDK not found` | fvm no configurado | Correr `fvm use` en `syncro-flutter/` primero |
| `bundleProductionRelease not found` | Flavor mal especificado | Verificar flavor `production` en `build.gradle` |

---

## Notes

- El signing de Android usa `syncro-mobile-key.keystore` — es el upload key registrado en Play Console. Requiere `android/key.properties` y `android/app/syncro-mobile-key.keystore` presentes en la máquina (gitignoreados). Backup en `syncro-temp/android/`.
- El signing de iOS usa `Automatic` — Xcode gestiona los provisioning profiles.
- NO modificar `build.gradle` ni hacer switch de branches — ese flujo fue eliminado.
- El branch `gradleForProd` está deprecado desde que se introdujo el signing condicional.
