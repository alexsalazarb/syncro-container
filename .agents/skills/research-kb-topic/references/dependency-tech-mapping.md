# Dependency → Technology Mapping

Per-stack tables for resolving dependency file entries to technology names.
Used by `research-kb-topic` Step 2 (scan mode).

---

## Dependency Files by Stack

| Stack | Dependency file(s) | Parse method |
|-------|-------------------|--------------|
| Swift | `Package.swift`, `Podfile` | Extract package/pod names |
| Android | `build.gradle(.kts)`, `libs.versions.toml` | Extract from dependencies block |
| Flutter | `pubspec.yaml` | Extract from `dependencies:` and `dev_dependencies:` |
| Rails | `Gemfile` | Extract gem names |
| React | `package.json` | Extract from `dependencies` and `devDependencies` |
| Python | `requirements.txt`, `pyproject.toml`, `Pipfile` | Extract package names |
| Generic | Any manifest with dependencies section | Best-effort extraction |

---

## Name Normalization Rules

1. **Case**: Lowercase for matching (`RxSwift` → `rxswift`)
2. **Strip decorators**: Remove `pod`, `gem`, `import`, version specifiers (`~> 5.0`, `^18.0`)
3. **Scoped packages**: Use the package name (`@reduxjs/toolkit` → `Redux Toolkit`)
4. **Multi-package libraries**: Group related packages (`RxSwift` + `RxCocoa` → `RxSwift`)

---

## Common Aliases

When checking KB coverage, match against **both** the dependency entry name and the
normalized technology name. A single KB doc may cover multiple related dependencies.

| Dependency Entries | Technology Name | Likely KB filename |
|-------------------|----------------|---------------------|
| `RxSwift`, `RxCocoa`, `RxRelay` | RxSwift | `rxswift-patterns` |
| `RealmSwift`, `Realm` | Realm | `realm-patterns` |
| `Alamofire` | Alamofire | `alamofire-patterns` |
| `Kingfisher`, `SDWebImage`, `Nuke` | Image Loading | `image-loading-patterns` |
| `SnapKit`, `TinyConstraints` | Auto Layout Libraries | `autolayout-patterns` |
| `CoreData`, `SwiftData` | Persistence | `persistence-patterns` |
| `StoreKit` | StoreKit | `storekit-patterns` |
| `sidekiq` | Sidekiq | `sidekiq-patterns` |
| `devise`, `omniauth` | Authentication | `auth-patterns` |
| `pundit`, `cancancan` | Authorization | `authorization-patterns` |
| `redux`, `@reduxjs/toolkit` | Redux | `redux-patterns` |
| `zustand` | Zustand | `zustand-patterns` |
| `tanstack/react-query`, `swr` | Data Fetching | `data-fetching-patterns` |
| `sqlalchemy` | SQLAlchemy | `sqlalchemy-patterns` |
| `celery` | Celery | `celery-patterns` |
| `pytest` | Pytest | `pytest-patterns` |
| `fastapi` | FastAPI | `fastapi-patterns` |
| `provider`, `riverpod`, `bloc` | State Management | `state-management-patterns` |
| `dio` | Dio (HTTP) | `dio-patterns` |

> This table is **not exhaustive** — it provides common examples. For unlisted dependencies,
> use the dependency name directly as the technology name and construct the KB filename
> as `{kebab-case-name}-patterns.md`.

---

## Standard Library — Core Platform (skip these)

Technologies too basic for a dedicated KB doc. **Only skip items in this list** — anything
not listed here is a candidate even if it ships with the platform.

| Stack | Core (skip) |
|-------|-------------|
| Swift | Foundation, UIKit (basic), SwiftUI (basic view lifecycle), XCTest (basic assertions) |
| Android | Android SDK core, Kotlin stdlib, JUnit (basic) |
| Flutter | Flutter SDK, Dart core libraries, `material`, `cupertino` |
| Rails | ActionController (basic), ActionView (basic), Rake |
| React | React core, ReactDOM, JSX |
| Python | stdlib modules (os, sys, json, pathlib, typing, logging, unittest) |

**Important**: Complex platform frameworks that have non-trivial patterns SHOULD get KB docs
even if they ship with the platform. Examples: CoreData, StoreKit, CoreBluetooth,
Combine, Room, WorkManager, Jetpack Navigation, ActiveRecord (advanced patterns).
Use judgment — if developers commonly struggle with it, it deserves a doc.
