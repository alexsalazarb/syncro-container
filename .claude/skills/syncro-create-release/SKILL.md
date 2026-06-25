---
name: syncro-create-release
description: >
  Builds a production release for both iOS and Android.
  Validates the production flavor in profile mode on both Android and iOS before generating artifacts.
  Android: generates a signed AAB via Gradle. iOS: archives via xcodebuild + exports for App Store.
  Trigger: /syncro-create-release or "crear release", "build produccion", "build release".
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.5"
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

## Step 1 — Clean + Dep Refresh

Garantiza que no haya artefactos viejos en caché que puedan contaminar el build. Sin este paso, el compilador puede reusar `.dill` files de builds anteriores y producir un APK/AAB con la versión correcta pero con cambios faltantes.

Desde `syncro-flutter/`:
```bash
fvm flutter clean
fvm flutter pub get
```

---

## Step 2 — Profile Validation (Android + iOS)

Antes de generar cualquier artefacto de producción, validar que el flavor `production` arranca correctamente en ambas plataformas en modo `profile`.

### 2a — Listar dispositivos disponibles

Desde `syncro-flutter/`:
```bash
fvm flutter devices
```

Mostrar la lista al usuario e identificar:
- El dispositivo/emulador Android a usar (ID de la columna `device id`)
- El simulador/dispositivo iOS a usar (ID de la columna `device id`)

### 2b — Validación Android

```bash
fvm flutter run --flavor production --profile -d <android-device-id>
```

Esperar a que la app arranque. Pedir al usuario que la recorra brevemente y presione `q` cuando confirme que funciona.

**NO continuar hasta que el usuario confirme que Android está OK.**

### 2c — Validación iOS

```bash
fvm flutter run --flavor production --profile -d <ios-device-id>
```

Esperar a que la app arranque. Pedir al usuario que la recorra brevemente y presione `q` cuando confirme que funciona.

**NO continuar al Step 3 hasta que el usuario confirme que iOS está OK.**

> Si alguna plataforma falla → abortar. No tiene sentido generar artefactos de release de un build que no arranca.

---

## Step 3 — Flutter Build (required before platform-specific steps)

Desde `syncro-flutter/`:
```bash
fvm flutter build appbundle --flavor production --release --obfuscate --split-debug-info=build/symbols
fvm flutter build ios --flavor production --release --no-codesign --obfuscate --split-debug-info=build/symbols
```

> `--obfuscate --split-debug-info=build/symbols` es OBLIGATORIO — genera los archivos de símbolos Dart necesarios para desencriptar stack traces en Firebase Crashlytics. Sin estos, los crashes muestran `at ... (.)` indescifrables.
> El `--no-codesign` en iOS es intencional: el signing lo hace `xcodebuild` en el paso de archive.

---

## Step 4 — Android Bundle

Desde `syncro-flutter/android/`:
```bash
./gradlew app:bundleProductionRelease
```

Artifacts resultantes:
- AAB: `syncro-flutter/build/app/outputs/bundle/productionRelease/app-production-release.aab`
- Mapping: `syncro-flutter/build/app/outputs/mapping/productionRelease/mapping.txt`

Reportar ambas rutas al usuario.

---

## Step 5 — iOS Archive

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

## Step 6 — iOS Export (App Store)

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

## Step 7 — Upload Dart Symbols to Firebase Crashlytics

Los símbolos Dart de Android deben subirse manualmente. Los de iOS los sube automáticamente el build phase de Xcode durante `xcodebuild archive` — no requieren acción manual.

`build/symbols` contiene símbolos de ambas plataformas mezclados. El CLI de Firebase solo puede procesar los Android (ELF). Separar primero:

```bash
# Separar símbolos Android
mkdir -p build/symbols-android
cp build/symbols/app.android-*.symbols build/symbols-android/

# Subir solo Android
firebase crashlytics:symbols:upload \
  --app=1:920223298498:android:3352304fd0baa59e5b5c5b \
  build/symbols-android
```

> iOS no requiere upload manual — el Xcode build phase `FlutterFire: "flutterfire upload-crashlytics-symbols"` lo ejecuta automáticamente durante el archive.
> Si `firebase` CLI no está disponible: `npm install -g firebase-tools` y luego `firebase login`.

---

## Step 8 — Report

Al finalizar, reportar:

```
✅ Release builds completados

Android AAB:     syncro-flutter/build/app/outputs/bundle/productionRelease/app-production-release.aab
Android Mapping: syncro-flutter/build/app/outputs/mapping/productionRelease/mapping.txt
iOS IPA:         syncro-flutter/build/ios/export/syncro.ipa
Dart Symbols:    syncro-flutter/build/symbols/ (subidos a Firebase Crashlytics)

Próximos pasos:
- Android: subir el .aab Y el mapping.txt en Play Console → Internal Testing
  (Play Console → seleccionar release → "Deobfuscation file" → subir mapping.txt)
- iOS: subir el .ipa con Transporter o desde Xcode → Organizer → Distribute
```

---

## Error Handling

| Error | Causa probable | Solución |
|-------|---------------|---------|
| `No signing certificate` | Provisioning profile vencido o ausente | Abrir Xcode → Preferences → Accounts → Download Manual Profiles |
| `Flutter SDK not found` | fvm no configurado | Correr `fvm use` en `syncro-flutter/` primero |
| `bundleProductionRelease not found` | Flavor mal especificado | Verificar flavor `production` en `build.gradle` |
| App no arranca en emulador | Cache corrupto aún | Correr `flutter clean` + `flutter pub get` de nuevo |

---

## Notes

- **El Step 1 (flutter clean) es obligatorio** — sin él, el compilador puede reusar `.dill` files de builds anteriores y producir un artifact con la versión correcta pero con cambios faltantes.
- **El Step 2 (profile validation) es un gate obligatorio** — si alguna plataforma no arranca en profile/production, no se generan artefactos. Profile mode activa AOT sin ofuscación, lo que facilita detectar crashes antes de commitear al build de release.
- El signing de Android usa `syncro-mobile-key.keystore` — es el upload key registrado en Play Console. Requiere `android/key.properties` y `android/app/syncro-mobile-key.keystore` presentes en la máquina (gitignoreados). Backup en `syncro-temp/android/`.
- El signing de iOS usa `Automatic` — Xcode gestiona los provisioning profiles.
- NO modificar `build.gradle` ni hacer switch de branches — ese flujo fue eliminado.
- El branch `gradleForProd` está deprecado desde que se introdujo el signing condicional.
