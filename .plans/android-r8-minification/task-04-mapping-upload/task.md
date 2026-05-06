# Task 04 — Mapping File Upload Integration

## Objective

Integrar el upload del `mapping.txt` al proceso de release (`/syncro-create-release`) para que no sea un paso manual a olvidar.

## Pre-condition

task-03 completado: R8 validado en Play Store sin crashes.

## Steps

### 1. Actualizar skill syncro-create-release

Agregar al Step 2 del skill el reporte de la ruta del mapping file:

```
Artifact resultante:
- AAB: syncro-flutter/build/app/outputs/bundle/productionRelease/app-production-release.aab
- Mapping: syncro-flutter/build/app/outputs/mapping/productionRelease/mapping.txt
```

Agregar al Step 5 (Report):
```
Android AAB:     build/.../app-production-release.aab (XX MB)
Android Mapping: build/.../mapping/productionRelease/mapping.txt
iOS IPA:         build/ios/export/syncro.ipa (XX MB)

Próximos pasos:
- Android: subir el .aab Y el mapping.txt en Play Console → Internal Testing
```

### 2. Actualizar build.gradle (documentar estado final)

Asegurarse de que el `build.gradle` quede documentado correctamente:
```groovy
release {
    signingConfig keystorePropertiesFile.exists() ? signingConfigs.release : signingConfigs.debug
    minifyEnabled true   // R8 habilitado — ver proguard-rules.pro para reglas
    shrinkResources true
    proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
}
```

### 3. Actualizar nota en build.gradle

Reemplazar el comentario actual:
```groovy
// minification disabled: ProGuard/R8 caused crashes on Play Store internal testing
```
Por:
```groovy
// R8 enabled with custom rules in proguard-rules.pro — tested on Play Store internal testing
```

### 4. Actualizar engram memory

Actualizar la memoria del proyecto con el estado final del signing y minificación.

## File Ownership

**Modifica:**
- `syncro-flutter/android/app/build.gradle`
- `.claude/skills/syncro-create-release/SKILL.md`

## Success Criteria

- Skill muestra ruta del mapping.txt en el reporte final
- Build.gradle refleja el estado real con comentario actualizado
- Developer sabe que debe subir AMBOS archivos a Play Console
