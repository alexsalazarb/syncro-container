---
name: syncro-smoke-test
description: >
  Runs a smoke test for syncro-flutter on an Android emulator in release mode (AOT compilation).
  Cleans the build environment first, then launches the app on the emulator and prompts the user
  for manual verification. Can be run standalone or as a pre-step before generating release artifacts.
  Trigger: /syncro-smoke-test or "smoke test", "correr smoke test", "probar en emulador".
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## When to Use

- User explicitly says `/syncro-smoke-test`, "smoke test", "correr smoke test", "probar en emulador"
- Also invoked by `/syncro-create-release` before generating artifacts
- Do NOT trigger automatically

---

## Step 0 — Clean Build Environment (MANDATORY)

> **Por qué es obligatorio**: Flutter compila Dart de forma incremental. Si el build cache quedó de una versión anterior o un branch diferente, `flutter build` puede reusar output viejo aunque el código en disco haya cambiado. Esto causó que el AAB de 1.7.0+438 no incluyera el fix de SE-12791 a pesar de estar en el código fuente.

Desde `syncro-flutter/`:
```bash
fvm flutter clean
fvm flutter pub get
```

---

## Step 1 — Smoke Test en Android Emulator

> **Por qué es obligatorio antes de un release**: El smoke test en release mode detecta problemas que solo aparecen en AOT compilation (release) y nunca en debug mode. Corre ANTES de generar el AAB para no subir un build roto a Play Store.

### 1a. Verificar emulador disponible

```bash
# Ver emuladores disponibles
fvm flutter emulators

# Ver dispositivos/emuladores corriendo
fvm flutter devices
```

Si no hay ningún emulador Android corriendo, lanzar uno:
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

¿Todo OK? Confirmá para continuar.
Si algo falla, terminar el proceso con Ctrl+C y corregir antes de continuar.
```

**NO continuar hasta que el usuario confirme.**

Una vez confirmado, matar el proceso de flutter run antes de seguir:
```bash
pkill -f "flutter run"
```

---

## Error Handling

| Error | Causa probable | Solución |
|-------|---------------|---------|
| `Flutter SDK not found` | fvm no configurado | Correr `fvm use` en `syncro-flutter/` primero |
| App no arranca en emulador | Cache corrupto aún | Correr `flutter clean` + `flutter pub get` de nuevo |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | Firma diferente a la instalada | ADB desinstala automáticamente y reintenta — ignorar |

---

## Notes

- **`flutter clean` es OBLIGATORIO** — previene que el build cache reutilice output compilado de una versión anterior.
- **El smoke test corre en release mode (AOT)** — no debug. Algunos bugs solo aparecen en compilación AOT.
- Si se corre desde `/syncro-create-release`, el proceso de `flutter run` debe terminarse antes de continuar con los builds de artifacts.
