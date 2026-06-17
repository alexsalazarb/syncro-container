# Task: UISceneDelegate iOS Migration

**Plan**: Flutter Upgrade 3.32.4 → 3.38.7
**Phase**: 3 — Platform + Services
**Task ID**: task-04
**Task Path**: phase-3/task-04-uiscenedelegate-ios
**Depends On**: task-01
**JIRA**: SE-12530

## Objective

Migrar el `AppDelegate.swift` customizado al nuevo modelo `UISceneDelegate` requerido por Flutter 3.38. La migración es manual porque el `AppDelegate` tiene código propio (Pendo SDK, screen-blur pattern, URL scheme handling) que impide la auto-migración.

## Context

**Por qué manual**: Flutter 3.38 auto-migra solo si `AppDelegate` no está customizado. Este proyecto tiene:
1. `import Pendo` + inicialización del SDK de Pendo
2. Screen-blur pattern de seguridad (`applicationWillResignActive` / `applicationDidBecomeActive`) — pone una pantalla en blanco cuando la app va al background
3. URL scheme handling para Pendo deep links

**Archivo actual**: `ios/Runner/AppDelegate.swift` (40 líneas)

**Deadline real**: Requerido antes de Flutter 3.41 (obligatorio). Con iOS 26 SDK también se vuelve obligatorio.

**Guía oficial**: https://docs.flutter.dev/release/breaking-changes/uiscenedelegate

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/ios/Runner/AppDelegate.swift` | modify | Adoptar FlutterImplicitEngineDelegate, mover setup a SceneDelegate |
| `syncro-flutter/ios/Runner/SceneDelegate.swift` | create | Nuevo SceneDelegate para lifecycle hooks |
| `syncro-flutter/ios/Runner/Info.plist` | modify | Agregar UIApplicationSceneManifest |

### Do NOT Modify
- `syncro-flutter/lib/` — esta task es solo iOS nativo
- `pubspec.yaml` — ya actualizado en task-01

## Implementation Steps

### Step 1: Leer el AppDelegate.swift actual

Leer `ios/Runner/AppDelegate.swift` completo para entender el código actual.

### Step 2: Actualizar AppDelegate.swift

El nuevo `AppDelegate` mínimo que adopta `FlutterImplicitEngineDelegate`:

```swift
import UIKit
import Flutter
import Pendo

@main
@objc class AppDelegate: FlutterAppDelegate, FlutterImplicitEngineDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GeneratedPluginRegistrant.register(with: self)
    // Inicialización de Pendo si se hacía aquí
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  // URL scheme para Pendo deep links (se mantiene en AppDelegate)
  override func application(
    _ app: UIApplication,
    open url: URL,
    options: [UIApplication.OpenURLOptionsKey: Any] = [:]
  ) -> Bool {
    // Pendo URL handling
    return super.application(app, open: url, options: options)
  }
}
```

**Nota**: `applicationWillResignActive` / `applicationDidBecomeActive` se mueven a SceneDelegate (ver Step 3).

### Step 3: Crear SceneDelegate.swift

Los lifecycle hooks que antes estaban en AppDelegate ahora van en SceneDelegate. El screen-blur pattern se implementa con `sceneWillResignActive` / `sceneDidBecomeActive`:

```swift
import UIKit
import Flutter

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
  var window: UIWindow?
  private var blurView: UIVisualEffectView?

  func sceneWillResignActive(_ scene: UIScene) {
    // Screen-blur pattern: mostrar blur al ir al background
    guard let window = window else { return }
    let blurEffect = UIBlurEffect(style: .regular)
    let blur = UIVisualEffectView(effect: blurEffect)
    blur.frame = window.bounds
    blur.tag = 999
    window.addSubview(blur)
    blurView = blur
  }

  func sceneDidBecomeActive(_ scene: UIScene) {
    // Remover blur al volver al foreground
    blurView?.removeFromSuperview()
    blurView = nil
    window?.viewWithTag(999)?.removeFromSuperview()
  }
}
```

**Adaptar al pattern exacto existente**: leer primero el AppDelegate actual para replicar el screen-blur exacto.

### Step 4: Actualizar Info.plist

Agregar la key `UIApplicationSceneManifest` en `Info.plist` para registrar el SceneDelegate:

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UIApplicationSupportsMultipleScenes</key>
    <false/>
    <key>UISceneConfigurations</key>
    <dict>
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneConfigurationName</key>
                <string>Default Configuration</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).SceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

### Step 5: Compilar y probar en device físico

```bash
cd syncro-flutter
fvm flutter build ios --debug
# O correr en device:
fvm flutter run -d [device-id]
```

**Probar obligatoriamente en device físico:**
1. App lanza correctamente
2. Screen-blur aparece al presionar Home / cambiar de app
3. Screen-blur desaparece al volver a la app
4. Pendo deep link funciona (abrir URL scheme desde Safari)
5. Push notifications se reciben

## Testing

- [ ] `fvm flutter build ios --debug` sin errores
- [ ] Screen-blur pattern funciona en device físico
- [ ] Pendo deep links funcionan (URL scheme)
- [ ] App no crashea al background/foreground

## Kill Criteria

Si Pendo deep links no funcionan después de la migración y no hay forma de repararlos sin API de Pendo no disponible → revertir y escalar al equipo.

## Completion Criteria

- [ ] `AppDelegate.swift` adopta `FlutterImplicitEngineDelegate`
- [ ] `SceneDelegate.swift` creado con screen-blur pattern
- [ ] `Info.plist` tiene `UIApplicationSceneManifest`
- [ ] Build iOS compila sin errores
- [ ] Screen-blur y Pendo deep links probados en device físico
- [ ] Status actualizado en `status.md`
