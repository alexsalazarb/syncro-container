---
name: syncro-create-release
description: >
  Builds a production release for both iOS and Android.
  Includes mandatory clean + smoke test on Android emulator before generating artifacts.
  Android: generates a signed AAB via Gradle. iOS: archives via xcodebuild + exports for App Store.
  Trigger: /syncro-create-release or "crear release", "build produccion", "build release".
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.1"
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

## Step 0 — Clean Build Environment (MANDATORY)

> **Por qué es obligatorio**: Flutter compila Dart de forma incremental. Si el build cache quedó de una versión anterior o un branch diferente, `flutter build` puede reusar output viejo aunque el código en disco haya cambiado. Esto causó que el AAB de 1.7.0+438 no incluyera el fix de SE-12791 a pesar de estar en el código fuente.

Desde `syncro-flutter/`:
```bash
fvm flutter clean
fvm flutter pub get
```

---

## Step 1 — Smoke Test en Android Emulator (MANDATORY — antes de generar artifacts)

> **Por qué es obligatorio**: El smoke test en release mode detecta problemas que solo aparecen en AOT compilation (release) y nunca en debug mode. Corre ANTES de generar el AAB para no subir un build roto a Play Store.

### 1a. Verificar emulador disponible

```bash
# Ver emuladores disponibles
fvm flutter emulators

# Ver dispositivos/emuladores corriendo
fvm flutter devices
```

Si no hay ningún emulador corriendo, lanzar uno:
```bash
fvm flutter emulators --launch <emulator_id>
# Esperar ~30s a que bootee, luego verificar con `fvm flutter devices`
```

### 1b. Correr en release mode sobre el emulador

```bash
# Reemplazar <device_id> con el id del emulador Android de `flutter devices`
fvm flutter run --flavor production --release -d <device_id>
```

> La app corre en modo release (AOT compilation) — idéntico a lo que se sube a Play Store.

### 1c. Verificación manual — STOP

**DETENER aquí y pedirle al usuario que verifique en el emulador:**

```
⏸️  Smoke test en progreso. Verificá en el emulador Android:

1. ✅ La app arranca correctamente
2. ✅ Create Ticket → dejar campos vacíos → Save → aparecen mensajes de error (no solo borde rojo)
3. ✅ Create Appointment → verificar que funciona
4. ✅ Cualquier flow afectado por los cambios de este release

¿Todo OK? Confirmá para continuar con la generación de artifacts.
Si algo falla, terminar el proceso con Ctrl+C y corregir antes de continuar.
```

**NO continuar al Step 2 hasta que el usuario confirme.**

---

## Step 2 — Flutter Build (required before platform-specific steps)

Desde `syncro-flutter/`:
```bash
fvm flutter build appbundle --flavor production --release
fvm flutter build ios --flavor production --release --no-codesign
```

> El `--no-codesign` en iOS es intencional: el signing lo hace `xcodebuild` en el paso de archive.

---

## Step 3 — Android Bundle

Desde `syncro-flutter/android/`:
```bash
./gradlew app:bundleProductionRelease
```

Artifacts resultantes:
- AAB: `syncro-flutter/build/app/outputs/bundle/productionRelease/app-production-release.aab`
- Mapping: `syncro-flutter/build/app/outputs/mapping/productionRelease/mapping.txt`

Reportar ambas rutas al usuario.

---

## Step 4 — iOS Archive

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

## Step 5 — iOS Export (App Store)

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

## Step 6 — Report

Al finalizar, reportar:

```
✅ Release builds completados

Android AAB:     syncro-flutter/build/app/outputs/bundle/productionRelease/app-production-release.aab
Android Mapping: syncro-flutter/build/app/outputs/mapping/productionRelease/mapping.txt
iOS IPA:         syncro-flutter/build/ios/export/syncro.ipa

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

- **`flutter clean` es OBLIGATORIO** — previene que el build cache reutilice output compilado de una versión anterior. Sin este paso, el AAB puede contener código viejo aunque el fuente esté actualizado (ver SE-12791).
- **El smoke test corre en release mode (AOT)** — no debug. Esto es crítico porque algunos bugs solo aparecen en compilación AOT y no son detectables con `flutter run` sin `--release`.
- El signing de Android usa `syncro-mobile-key.keystore` — es el upload key registrado en Play Console. Requiere `android/key.properties` y `android/app/syncro-mobile-key.keystore` presentes en la máquina (gitignoreados). Backup en `syncro-temp/android/`.
- El signing de iOS usa `Automatic` — Xcode gestiona los provisioning profiles.
- NO modificar `build.gradle` ni hacer switch de branches — ese flujo fue eliminado.
- El branch `gradleForProd` está deprecado desde que se introdujo el signing condicional.
