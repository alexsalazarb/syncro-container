# Status — task-03-play-store-validation

| Field | Value |
|-------|-------|
| Status | blocked |
| Agent | claude-sonnet-4-6 |
| Branch | plan/android-r8-minification/task-03-prep |
| PR | — |
| Started | 2026-06-17 |
| Completed | — |

## Notes

**Kill criterion #1 met** — R8 (`minifyEnabled true`) causa crash inmediato en Play Store builds.

### Root cause
`java.lang.IllegalArgumentException: No view found for id 0x1 (unknown) for FlutterFragment`

R8 incompatibilidad con `FlutterFragmentActivity`. El crash ocurre en `FragmentActivity.onStart()` al intentar resolver el container view de FlutterFragment. No reproducible en builds locales — solo en Play Store.

### Intentos fallidos
| Intento | Cambio | Resultado |
|---------|--------|-----------|
| 1 | `shrinkResources false` | Crash idéntico |
| 2 | `proguard-android.txt` (sin optimization passes) | Crash idéntico |
| 3 | Keep rules extensas para `io.flutter.**`, `androidx.fragment.**` | Crash idéntico |

### Estado actual
Revertido a `minifyEnabled false`. `mappingFileUploadEnabled true` queda activo (no genera mapping útil sin minification, pero no rompe nada).

### Para desbloquear en el futuro
Investigar si Flutter 3.x tiene una configuración específica de R8 compatible con `FlutterFragmentActivity`. Ver: https://docs.flutter.dev/deployment/android#obfuscating-dart-code
