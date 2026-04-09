# Audit Checklist

Evaluation criteria for Phases 3–5 of the `audit-and-plan` skill.
Adapt depth based on the codebase Class (A/B/C) from Phase 1.

---

## Phase 3: Architecture & Patterns

For each significant area, read the code and answer:

**Architecture**
- What pattern is used? (MVVM, MVC, Clean, Coordinator, Redux, etc.)
- How is the app / service structured at the top level?
- What is the dependency injection strategy?
- What is the navigation / routing pattern?
- What is the data flow? (reactive, callbacks, delegates, async/await)

**Patterns in use**
- What libraries / frameworks are doing the heavy lifting?
- What recurring patterns do you see? (e.g. every ViewModel has a DisposeBag, every coordinator pushes via a closure)
- What naming conventions are used? (e.g. `FooViewController`, `FooViewModel`, `FooCoordinator`)
- Are there base classes or protocols everything inherits from?

**Conventions to follow**
- File organization (by feature, by layer, flat?)
- How are errors handled?
- How are async operations written?
- How is state managed?
- What does a "standard" unit look like? (Find an example of a well-written screen / endpoint / model)

These conventions become the rules in `{AGENTS_MD}`. Write them down as you go.

---

## Phase 4: Business Logic & Product Domain

For each project being audited, identify two layers:

### 4a. Domain features (shared → `CONTAINER_KB_DIR/product/domain/`)

These are the *business concepts* that any developer on any stack needs to understand.
Ask: "If I described this to an Android dev, would they care?" → yes = domain.

- **What does this product do?** Plain-language summary.
- **What are the core domain concepts?** (e.g. "Article", "Subscription", "User", "Localization Key")
- **What are the business rules?** (validation, eligibility, state machines, access control, pricing logic)
- **What data models exist?** (canonical shape — look across all stacks for the source of truth → `CONTAINER_KB_DIR/technical/data-models/`)
- **What are the cross-platform flows?** (e.g. the auth flow involves iOS screen + backend token endpoint + refresh logic)
- **What external services are shared?** (REST API → `CONTAINER_KB_DIR/technical/api/`; BLE protocol → `CONTAINER_KB_DIR/technical/ble/`; used by multiple stacks)

If doing a full container audit, synthesize findings **across all projects** before writing these docs.
Note per-stack implementation differences as callouts inside the shared doc rather than separate files.

### 4b. Platform features (per-project → `KB_DIR/product/features/`)

These are features or capabilities that only exist on one platform, or whose presentation
is so platform-specific that agents on other stacks wouldn't need the detail.

Ask: "Does this feature exist (or matter) on other platforms?" → no = platform-specific.

- **What screens / views are unique to this platform?** (e.g. admin dashboard only exists on web)
- **What capabilities are platform-only?** (e.g. iOS widgets, Android notifications, web file upload)
- **How does this platform present shared domain features?** (e.g. how the iOS login screen implements the auth domain concept — biometrics, keyboard handling, the specific VC flow)
- **What business rules are enforced only on this platform?** (e.g. some validation only happens client-side on iOS)

**Container audit rule**: After listing platform features for all projects, check for
shared features that exist on multiple platforms (e.g. BLE pairing on iOS and Android,
therapy session on iOS and Android). Each platform that implements a shared feature with
meaningful platform-specific details (permissions, libraries, UI patterns) should get
its own `product/features/` doc — even though the domain concept is shared.

**KB routing for shared resources:**
- Shared REST API contracts → `CONTAINER_KB_DIR/technical/api/`
- BLE protocol specs → `CONTAINER_KB_DIR/technical/ble/`
- Canonical data shapes → `CONTAINER_KB_DIR/technical/data-models/`
- Cross-stack engineering standards → `CONTAINER_KB_DIR/engineering/best-practices/`
- Security/compliance rules → `CONTAINER_KB_DIR/engineering/security/`
- Platform-specific integrations (StoreKit, Google Pay, etc.) → `KB_DIR/technical/integrations/`

---

## Phase 5: Identify Problems

Classify each problem found:

| Severity | Meaning |
|----------|---------|
| 🔴 Critical | Security risk, data loss, crashes in production |
| 🟠 High | Broken functionality, blocks future work |
| 🟡 Medium | Technical debt, makes maintenance hard |
| 🟢 Low | Style, naming, minor cleanup |

**Depth by class:**
- **Class A** → Keep this section brief — note debt but don't overweight it.
- **Class B** → Balance — document both what's solid and where it breaks down.
- **Class C** → This is the primary output.
