# Status — task-03-play-store-validation

| Field | Value |
|-------|-------|
| Status | blocked |
| Agent | — |
| Branch | plan/android-r8-minification/task-03 |
| PR | — |
| Started | — |
| Completed | — |

## Notes

Requiere acción manual del developer:
1. Subir el AAB a Play Console → Internal Testing (path: `syncro-flutter/build/app/outputs/bundle/productionRelease/app-production-release.aab`)
2. En el mismo upload, adjuntar el mapping.txt → sección "Deobfuscation file" (`build/app/outputs/mapping/productionRelease/mapping.txt`)
3. Instalar desde Play Store en al menos 2 devices físicos (NO vía adb — los crashes anteriores solo aparecen en builds de Play Store)
4. Probar todos los flows: login, tickets, assets, chat, appointments, time tracking, push notifications, Pendo
5. Monitorear Crashlytics 24-48hs — si crash-free rate < 99% → kill criteria #1
6. Marcar este status como complete cuando pase 48hs sin crashes nuevos
