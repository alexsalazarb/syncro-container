# Create Git Branch

Run from the code root (`PROJECT_CODE_ROOT` — container subfolder or single-layout root).

**If `PR_INTEGRATION=false`** (branch from main):
```bash
git fetch origin
if git show-ref --verify --quiet refs/heads/plan/{plan-slug}/{task-path}; then
  git switch plan/{plan-slug}/{task-path}
else
  git switch -c plan/{plan-slug}/{task-path}
fi
```

**If `PR_INTEGRATION=true`** (branch from integration branch):
```bash
git fetch origin
if git show-ref --verify --quiet refs/heads/plan/{plan-slug}/{task-path}; then
  git switch plan/{plan-slug}/{task-path}
else
  git switch -c plan/{plan-slug}/{task-path} origin/plan/{plan-slug}
fi
```

**Mark in-progress**: Update `{PLANS_DIR}/{plan-slug}/{task-path}/status.md` to `in-progress` before writing any code.
