# Task 01 — Local R8 Investigation

## Objective

Habilitar R8 en modo release LOCAL (no Play Store) e identificar exactamente qué causa los crashes. El objetivo NO es fixear — es diagnosticar.

## Context

El `proguard-rules.pro` actual tiene reglas para Flutter, Pendo, Firebase, Fragments, threading, etc. Aun así hubo crashes. Hay que saber EXACTAMENTE cuál clase o método R8 está removiendo/obfuscando que causa el problema.

## Steps

### 1. Habilitar R8 temporalmente en build.gradle

```groovy
release {
    minifyEnabled true
    shrinkResources false  // mantener false por ahora — solo minificación
    proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
}
```

> `shrinkResources false` mantiene todos los recursos para aislar si el problema es minificación de código vs recursos.

### 2. Agregar flag de debugging a proguard-rules.pro

Agregar temporalmente al inicio del archivo:
```
-printmapping build/outputs/mapping/productionRelease/mapping.txt
-verbose
```

### 3. Build local en release mode

```bash
cd syncro-flutter
fvm flutter build apk --flavor production --release
```

> APK (no AAB) para instalación directa en device — no pasar por Play Store.

### 4. Instalar y probar en device físico

```bash
adb install -r build/app/outputs/flutter-apk/app-production-release.apk
```

Probar:
- [ ] Login y navegación básica
- [ ] Tickets list
- [ ] Asset detail
- [ ] Chat
- [ ] Appointments
- [ ] Pendo (verificar que trackea eventos)

### 5. Si hay crash — capturar stack trace

```bash
adb logcat | grep -E "FATAL|AndroidRuntime|flutter"
```

Anotar:
- Clase y método que falla
- Si es `ClassNotFoundException` o `NoSuchMethodException` → regla ProGuard faltante
- Si es un crash de lógica → problema diferente

### 6. Documentar findings

Actualizar `status.md` con:
- Lista de clases/métodos identificados como causa
- Stack traces relevantes
- Decisión: continuar con task-02 o kill criteria activado

## File Ownership

**Modifica:**
- `syncro-flutter/android/app/build.gradle` (cambio temporal — revertir al terminar si kill criteria)
- `syncro-flutter/android/app/proguard-rules.pro` (solo agregar `-verbose` y `-printmapping` temporalmente)

**No tocar:**
- Ningún archivo Dart/Flutter

## Testing

- Instalación directa en device físico con `adb`
- Correr todos los flows principales de la app
- Verificar Pendo tracking

## Kill Criteria Activation

Si los crashes son en código de terceros sin solución ProGuard conocida (e.g. crash dentro de Pendo SDK nativo), activar kill criteria #1 y cerrar el plan.
