# Engineering Standards

Universal quality standards that apply to every project regardless of stack. Document exceptions in `AGENTS.md`.

---

## Localization

**All user-facing strings must be localized.** Never hardcode display text. How you implement this varies by project — check the project KB for the specific tool and workflow used (e.g. i18n library, translation platform, manual resource files).

---

## Configuration & Secrets

**No hardcoded configuration values** — API keys, base URLs, environment flags, feature flags. Use environment variables, config files, or a secrets manager. Never commit secrets to source control.

---

## Debug Statements

**No debug output in production code** — `print`, `console.log`, `puts`, `debugger`, `pdb`, etc. Use a structured logger. Remove debug statements before committing.

---

## Error Handling

**Never silently swallow errors.** Log or propagate every caught exception. Silent failures make bugs invisible in production.

---

## Testing

**Write tests for business logic.** Every public function in a service or model should have test coverage. See the project KB for test conventions and commands.

---

## Code Review Checklist

Before any commit or PR:
- No hardcoded values (URLs, keys, display strings)
- No debug statements
- Tests pass
- No obvious security issues (SQL injection, XSS, exposed credentials)
