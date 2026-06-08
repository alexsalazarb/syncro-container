# Task: Verification + Test Suite

**Plan**: Flutter Upgrade 3.32.4 → 3.38.7
**Phase**: 5 — Verification
**Task ID**: task-09
**Task Path**: phase-5/task-09-verification
**Depends On**: task-01, task-02, task-03, task-04, task-05, task-06, task-07, task-08
**JIRA**: SE-12530

## Objective

Verificar la compatibilidad de los packages que necesitan prueba manual (`hive`, `scribble`, `modal_bottom_sheet`, `overlay_support`), correr la suite de tests completa, y confirmar que la app funciona end-to-end en device físico.

## Context

**Packages a verificar manualmente:**

| Package | Riesgo | Por qué necesita verificación |
|---------|--------|------------------------------|
| hive 2.2.3 | MEDIUM | Constraint viejo pero pub debería re-interpretar; verificar con pub get |
| hive_flutter 1.1.0 | MEDIUM | Mismo concern que hive |
| scribble 0.10.0+1 | HIGH | Unmaintained 2 años, `<3.0.0` bound — puede fallar o tener errores de análisis |
| modal_bottom_sheet 3.0.0 | MEDIUM | 2 años sin updates, targets Flutter 3.19 — API interna puede haber cambiado |
| overlay_support 2.1.0 | MEDIUM | 3 años sin updates — mismo concern |

**Si scribble falla**: el feature de dibujo/firma en la app necesita un reemplazo. Opciones a evaluar: `perfect_freehand`, `flutter_drawing_board`, o fork propio. Estimar esfuerzo y documentar en status.

## File Ownership

Este task es principalmente de verificación — no modifica código de features. Solo modifica:

| File | Action | Notes |
|------|--------|-------|
| `phase-5/task-09-verification/findings.md` | create | Resultados de verificación |
| Código de scribble/modal_bottom_sheet si hay quick fixes | modify | Solo si hay fixes de 1-2 líneas |

### Do NOT Modify
- Archivos owned por otras tasks
- Código de features (no es scope de esta task reparar features complejas)

## Implementation Steps

### Step 1: flutter pub get + analyze final

Con todo el código de las tasks anteriores integrado:

```bash
cd syncro-flutter
fvm flutter pub get
fvm flutter analyze 2>&1 | tee /tmp/analyze_final.txt
grep -c "error\|warning" /tmp/analyze_final.txt
```

Documentar resultado en `findings.md`.

### Step 2: Verificar hive + hive_flutter

```bash
cd syncro-flutter
# Verificar que pub resolvió hive correctamente:
cat pubspec.lock | grep "hive"
# Debería mostrar hive 2.2.3 resuelto

# Buscar uso de hive:
rg "Hive\.|HiveBox\|openBox\|lazyOpenBox" lib/ --type dart | head -10
```

Si `hive` generó warnings de analyze, documentarlos. Si generó errores, escalar.

### Step 3: Verificar scribble

```bash
# Verificar que pub resolvió scribble:
cat pubspec.lock | grep "scribble"

# Buscar uso de scribble:
rg "Scribble\b|ScribblePointerMode\|ScribbleState" lib/ --type dart -l
```

Correr analyze en los archivos que usan scribble:
```bash
fvm flutter analyze [archivos con scribble] 2>&1
```

**Si hay errores de compilación**: documentar en `findings.md` con el error exacto y estimar esfuerzo de reemplazo. Aplicar kill criteria del overview.

### Step 4: Verificar modal_bottom_sheet

```bash
rg "showMaterialModalBottomSheet\|showCupertinoModalBottomSheet\|modal_bottom_sheet" lib/ --type dart -l
```

Si hay errores de analyze en archivos que lo usan, documentar. Si la API cambió internamente, hacer quick fix o documentar como deuda.

### Step 5: Verificar overlay_support

```bash
rg "overlay_support\|OverlaySupport\|showSimpleNotification\|overlay_\|OverlayEntry" lib/ --type dart -l
```

Mismo proceso: verify pub resolution, analyze errors.

### Step 6: Correr test suite completa

```bash
cd syncro-flutter
fvm flutter test 2>&1 | tee /tmp/test_results.txt
# Contar failures:
grep -E "FAILED|ERROR" /tmp/test_results.txt | wc -l
```

Documentar:
- Total tests
- Tests pasando
- Tests fallando (con nombre y error)
- Si hay regresiones, identificar cuál task causó cada falla

### Step 7: Smoke test en device físico

Probar el golden path en device físico (iOS + Android si posible):
- [ ] Login (OAuth webview — verificar `webview_flutter_wkwebview` upgrade)
- [ ] Dashboard carga correctamente
- [ ] Lista de tickets con paginación (infinite_scroll_pagination migrado)
- [ ] @ mentions en chat (flutter_mentions — el fix del fork)
- [ ] Biometric unlock (local_auth)
- [ ] Notificaciones push recibidas
- [ ] Timer de ticket inicia y para
- [ ] Background/foreground: screen-blur aparece/desaparece (UISceneDelegate)

### Step 8: Documentar hallazgos

Crear `task-09-verification/findings.md` con:
- Resultado de `flutter analyze` (# errores/warnings)
- Resultado de `flutter test` (# pass/fail)
- Estado de hive, scribble, modal_bottom_sheet, overlay_support
- Items pendientes / deuda técnica identificada
- Smoke test results por feature

## Testing

- [ ] `fvm flutter test` corre sin regresiones
- [ ] `fvm flutter analyze` cero errores
- [ ] Smoke test en iOS device físico completo
- [ ] Smoke test en Android emulator/device

## Kill Criteria

- scribble no resuelve bajo Dart 3.10 y el feature de dibujo/firma no tiene reemplazo viable en el sprint → documentar en findings.md, agregar como deuda técnica, proceder igual con el upgrade.

## Completion Criteria

- [ ] `findings.md` creado con resultados de todos los packages verificados
- [ ] `fvm flutter test` sin nuevas regresiones
- [ ] `fvm flutter analyze` cero errores
- [ ] Smoke test golden path completado en iOS
- [ ] Smoke test golden path completado en Android
- [ ] Status actualizado en `status.md`
