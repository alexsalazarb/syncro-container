# Task 03 — Play Store Internal Testing Validation

## Objective

Subir el AAB con R8 activo a Play Store internal testing y confirmar que no hay crashes en producción. Este es el paso que falló históricamente — hay que hacerlo con cuidado.

## Pre-condition

task-02 completado: R8 activo, sin crashes en device local.

## Steps

### 1. Generar AAB de producción con R8

```bash
cd syncro-flutter
fvm flutter build appbundle --flavor production --release
```

Verificar que `mapping.txt` fue generado:
```bash
ls build/app/outputs/mapping/productionRelease/mapping.txt
```

### 2. Subir a Play Console → Internal Testing

- Subir el `.aab` normalmente
- En el mismo upload, subir el `mapping.txt` como "Deobfuscation file" (sección Native Debugging Symbols o R8/ProGuard mapping)

### 3. Instalar desde Play Store (NO via adb)

Instalar la versión de internal testing desde Play Store en al menos 2 devices físicos. Este paso es crítico — los crashes anteriores solo ocurrían en builds de Play Store, no en adb local.

### 4. Probar todos los flows

- [ ] Login
- [ ] Tickets list + detail
- [ ] Asset management
- [ ] Real-time chat
- [ ] Appointments (create/edit)
- [ ] Time tracking
- [ ] Push notifications
- [ ] Pendo events (verificar en Pendo dashboard)
- [ ] Firebase Crashlytics (verificar que no hay nuevos crashes)

### 5. Monitorear Crashlytics 24-48hs

Revisar Firebase Crashlytics buscando crashes nuevos. Si aparecen crash-free rate < 99% → kill criteria #1.

## File Ownership

No modifica código — solo valida.

## Success Criteria

- App instala desde Play Store sin crashes
- Todos los flows principales funcionan
- Crashlytics no reporta issues nuevos en 48hs
- `mapping.txt` subido correctamente a Play Console
