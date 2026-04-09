# Stack Discovery Commands

Per-stack commands for Phase 2.1 of the `audit-and-plan` skill.
Substitute `{PROJECT_CODE_ROOT}` with the resolved code root.

---

## Swift / iOS

```bash
# Project structure
find {PROJECT_CODE_ROOT} -name "*.swift" \
  -not -path "*/.git/*" -not -path "*/Pods/*" -not -path "*/.build/*" | head -60
cat {PROJECT_CODE_ROOT}/Podfile 2>/dev/null | head -60
cat {PROJECT_CODE_ROOT}/Package.swift 2>/dev/null | head -40

# Entry points
find {PROJECT_CODE_ROOT} \( -name "AppDelegate.swift" -o -name "SceneDelegate.swift" \
  -o -name "*App.swift" \) -not -path "*/Pods/*"

# Architecture signals
find {PROJECT_CODE_ROOT} \( -name "*Coordinator*" -o -name "*ViewModel*" \
  -o -name "*Manager*" -o -name "*Router*" -o -name "*Interactor*" \) \
  -name "*.swift" -not -path "*/Pods/*" | head -30

# Largest files
find {PROJECT_CODE_ROOT} -name "*.swift" -not -path "*/Pods/*" \
  -exec wc -l {} + 2>/dev/null | sort -rn | head -20
```

## Rails

```bash
cat {PROJECT_CODE_ROOT}/Gemfile | head -60
cat {PROJECT_CODE_ROOT}/config/routes.rb
ls {PROJECT_CODE_ROOT}/app/models/ {PROJECT_CODE_ROOT}/app/controllers/ \
   {PROJECT_CODE_ROOT}/app/services/ 2>/dev/null
find {PROJECT_CODE_ROOT}/app -name "*.rb" -exec wc -l {} + | sort -rn | head -20
```

## React / Next.js

```bash
cat {PROJECT_CODE_ROOT}/package.json
find {PROJECT_CODE_ROOT} -not -path "*/node_modules/*" -not -path "*/.next/*" \
  \( -name "*.tsx" -o -name "*.ts" \) | head -60
find {PROJECT_CODE_ROOT}/pages {PROJECT_CODE_ROOT}/app {PROJECT_CODE_ROOT}/src \
  -type f 2>/dev/null | head -40
```

## Python

```bash
cat {PROJECT_CODE_ROOT}/requirements.txt 2>/dev/null || \
  cat {PROJECT_CODE_ROOT}/pyproject.toml 2>/dev/null | head -40
find {PROJECT_CODE_ROOT} -name "*.py" -not -path "*/.git/*" \
  -exec wc -l {} + | sort -rn | head -20
```

## Generic

```bash
find {PROJECT_CODE_ROOT} -not -path "*/.git/*" -type f | sort | head -60
```
