# Task: SDK Deprecation Sweep

**Plan**: Flutter Upgrade 3.32.4 → 3.44.1
**Phase**: 4 — Routing + Yellow Packages
**Task ID**: task-08
**Task Path**: phase-4/task-08-sdk-deprecation-sweep
**Depends On**: task-01
**JIRA**: SE-12530

## Objective

Resolver los breaking changes y deprecaciones del Flutter SDK 3.35/3.38 que afectan código existente: el cambio de comportamiento de `SnackBar` con `action`, las deprecaciones de `AppBarTheme.color`, `Switch.activeColor`, `DropdownButtonFormField.value`, y una auditoría de `SystemChrome.setPreferredOrientations`.

## Context

**Cambios a resolver:**

| Cambio | Severidad | Descripción |
|--------|-----------|-------------|
| `SnackBar(action:)` ahora persiste | MEDIUM | Ya no se auto-dismiss — agregar `persist: false` |
| `AppBarTheme.color` → `backgroundColor` | MEDIUM | Deprecation warning |
| `Switch.activeColor` → `activeThumbColor` | MEDIUM | Deprecation warning |
| `DropdownButtonFormField.value` → `initialValue` | MEDIUM | Deprecation warning |
| `SystemChrome.setPreferredOrientations` en Android 16+ | MEDIUM | Auditar uso — silenciosamente falla en API 36 |

**Archivos afectados (por hallazgos del Explore):**
- `SnackBar`: `lib/core/utils/toast.dart` (centralizado) + 20 archivos de presentación
- `AppBarTheme`: `lib/core/delegates/capitalize_search_delegate.dart`
- `Switch`, `DropdownButtonFormField`, `SystemChrome`: buscar en el codebase

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/utils/toast.dart` | modify | SnackBar central — agregar persist: false |
| `syncro-flutter/lib/core/delegates/capitalize_search_delegate.dart` | modify | AppBarTheme.color → backgroundColor |
| Archivos con SnackBar + action fuera de toast.dart | review/modify | Verificar si usan action: |
| Archivos con Switch.activeColor | modify | → activeThumbColor |
| Archivos con DropdownButtonFormField.value | modify | → initialValue |

### Do NOT Modify
- `pubspec.yaml` — ya actualizado en task-01
- Archivos owned por otras tasks

## Implementation Steps

### Step 1: SnackBar — buscar todos los usos con action

```bash
cd syncro-flutter
rg "SnackBar\(" lib/ --type dart -l
rg -A5 "SnackBar\(" lib/ --type dart | rg "action:"
```

Para cada `SnackBar(action: ...)` encontrado, agregar `persist: false` si el comportamiento esperado es auto-dismiss:

```dart
// ANTES (v14 — se auto-dismissía):
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: Text('Mensaje'),
    action: SnackBarAction(label: 'OK', onPressed: () {}),
  ),
);

// DESPUÉS (v3.38 — ahora persiste hasta tap):
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: Text('Mensaje'),
    action: SnackBarAction(label: 'OK', onPressed: () {}),
    duration: const Duration(seconds: 4),  // si se quiere que desaparezca
    // O: persist: false  (si el parámetro está disponible)
  ),
);
```

**Nota**: verificar en el changelog de Flutter 3.38 si el parámetro se llama exactamente `showCloseIcon: false` o si simplemente hay que remover `action:` y usar un timer.

### Step 2: AppBarTheme.color → backgroundColor

En `capitalize_search_delegate.dart` y cualquier otro archivo:

```dart
// ANTES:
AppBarTheme(color: Colors.white)

// DESPUÉS:
AppBarTheme(backgroundColor: Colors.white)
```

```bash
rg "AppBarTheme.*color:" lib/ --type dart
rg "AppBarThemeData.*color:" lib/ --type dart
```

### Step 3: Switch.activeColor → activeThumbColor

```bash
rg "activeColor:" lib/ --type dart | rg -v "// "
```

```dart
// ANTES:
Switch(activeColor: AppColors.primary, ...)

// DESPUÉS:
Switch(activeThumbColor: AppColors.primary, ...)
```

### Step 4: DropdownButtonFormField.value → initialValue

```bash
rg "DropdownButtonFormField" lib/ --type dart -l
```

En cada uso, renombrar `value:` → `initialValue:` si el parámetro se usaba posicionalmente o nominalmente como `value:`.

### Step 5: SystemChrome.setPreferredOrientations — auditar

```bash
rg "setPreferredOrientations" lib/ --type dart
```

Si existe, documentar en el status.md que este API silenciosamente fallará en Android 16 (API 36). No hay un fix directo disponible aún — es un issue conocido del SDK. Registrar como deuda técnica.

### Step 6: Material color token algorithm audit

Check theme files for `ColorScheme.fromSeed()` usage:
```bash
rg "ColorScheme\.fromSeed|ColorScheme\.fromImageProvider" lib/ --type dart -l
```

If found, visually audit the 4 affected tokens (`onPrimaryContainer`, `onSecondaryContainer`, `onTertiaryContainer`, `onErrorContainer`) after upgrade. Use `.copyWith()` to override if brand accuracy is required.

### Step 7: FontWeight + Lexend variable font audit

```bash
rg "FontVariation\('wght'" lib/ --type dart
```

If any `TextStyle` combines `fontWeight:` AND `FontVariation('wght', ...)`, verify the rendered output hasn't changed.

### Step 8: Verificar compilación y warnings

```bash
cd syncro-flutter
fvm flutter analyze lib/ 2>&1 | rg "warning|error" | head -30
```

El objetivo es cero errores y reducción significativa de warnings.

## Testing

- [ ] `fvm flutter analyze` sin errores, mínimo de warnings
- [ ] SnackBars con action testear manualmente que se comportan como antes (se dismissan)
- [ ] No hay warnings de `AppBarTheme.color`, `Switch.activeColor`, `DropdownButtonFormField.value`

## Completion Criteria

- [ ] Todos los `SnackBar(action:)` revisados y con comportamiento de dismiss correcto
- [ ] `AppBarTheme.color` → `backgroundColor` en todos los usos
- [ ] `Switch.activeColor` → `activeThumbColor` en todos los usos
- [ ] `DropdownButtonFormField.value` → `initialValue` en todos los usos
- [ ] `SystemChrome.setPreferredOrientations` auditado y documentado en status
- [ ] Material color tokens auditados (ColorScheme.fromSeed usage checked)
- [ ] FontWeight + Lexend variable font combination auditada
- [ ] `fvm flutter analyze` sin errores
- [ ] Status actualizado en `status.md`
