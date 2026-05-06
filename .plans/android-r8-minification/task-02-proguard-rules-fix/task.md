# Task 02 — ProGuard Rules Fix

## Objective

Agregar las reglas ProGuard faltantes identificadas en task-01 y verificar que el build release local no crashea.

## Pre-condition

task-01 completado con lista de clases/métodos problemáticos documentados en su `status.md`.

## Steps

### 1. Leer findings de task-01

Revisar `task-01-local-r8-investigation/status.md` para obtener la lista de crashes y sus causas.

### 2. Agregar reglas específicas a proguard-rules.pro

Para cada crash identificado, agregar la regla correspondiente. Ejemplos comunes:

```proguard
# Si crash es ClassNotFoundException en com.ejemplo.Clase:
-keep class com.ejemplo.Clase { *; }

# Si crash es NoSuchMethodException en método específico:
-keepclassmembers class com.ejemplo.Clase {
    public void metodoEspecifico(...);
}

# Si crash es en reflection:
-keepclassmembers class * {
    @com.ejemplo.Annotation *;
}
```

### 3. Habilitar `shrinkResources true` (si task-01 pasó sin crash de recursos)

```groovy
release {
    minifyEnabled true
    shrinkResources true
    proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
}
```

### 4. Rebuild y re-testear en device

```bash
fvm flutter build apk --flavor production --release
adb install -r build/app/outputs/flutter-apk/app-production-release.apk
```

Repetir todos los flows de task-01. Si hay nuevos crashes → iterar agregando reglas.

### 5. Verificar tamaño del AAB

```bash
fvm flutter build appbundle --flavor production --release
ls -lh build/app/outputs/bundle/productionRelease/app-production-release.aab
```

Comparar con el tamaño actual (67MB sin R8). Si kill criteria #2 se activa → evaluar.

### 6. Remover flags temporales de debugging

Quitar del `proguard-rules.pro`:
```
-printmapping ...  (remover)
-verbose           (remover)
```

## File Ownership

**Modifica:**
- `syncro-flutter/android/app/proguard-rules.pro`
- `syncro-flutter/android/app/build.gradle`

## Testing

- Build release local con R8 activo
- Instalación vía adb en device físico
- Todos los flows principales sin crashes
- Verificar tamaño AAB dentro de rango aceptable
