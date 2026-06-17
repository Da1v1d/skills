# JavaScript Patterns Architect Reference

## Table Of Contents

1. Core Principles
2. Module Boundaries
3. Plain Functions And Functional Composition
4. Object Modules And Closures
5. Classes And Prototypes
6. Factory Pattern
7. Strategy Pattern
8. Adapter And Facade Patterns
9. Dependency Injection
10. Repository / Gateway Pattern
11. Observer / Pub-Sub Pattern
12. Command Pattern
13. Pipeline And Middleware Patterns
14. Async Architecture
15. Error Handling
16. Runtime Validation
17. Data Mapping
18. Configuration Design
19. Performance And Memory
20. JavaScript Anti-patterns
21. Pattern Selection Guide
22. Code Examples
23. Testing Guidance

## 1. Core Principles

- Prefer plain functions and modules until there is a real need for objects, classes, or pattern machinery.
- Keep side effects at the edges of the system.
- Keep pure transformations easy to test.
- Make data flow explicit.
- Hide third-party libraries behind small local contracts when vendor churn or testability matters.
- Do not turn every variation into a framework. A small `if` statement is often better than a strategy registry.
- Avoid global mutable state unless it is infrastructure-level and intentionally scoped.
- Separate I/O, validation, mapping, business rules, and formatting.

## 2. Module Boundaries

Good modules have one reason to change. Prefer modules that expose a small set of named functions or a small service object.

Good boundaries:

- `api-client.js`: request transport.
- `users-api.js`: user endpoints.
- `user-mapper.js`: map remote payloads into app models.
- `user-validation.js`: validate user input.
- `billing-service.js`: coordinate billing rules and dependencies.

Bad boundaries:

- `helpers.js`: vague dumping ground.
- `utils.js`: acceptable only inside a narrow scope, not as a cross-app junk drawer.
- `manager.js`: often hides too many responsibilities.
- Modules that import from unrelated features.

## 3. Plain Functions And Functional Composition

Use plain functions for deterministic transformations, formatting, filtering, scoring, validation helpers, and small policy decisions.

Best for:

- Reusable pure logic.
- Mapping data.
- Composing behavior in tests.
- Keeping business rules independent from infrastructure.

Bad for:

- Stateful workflows where object identity matters.
- Long parameter lists that should become an options object or service contract.
- Logic that requires lifecycle management.

Example:

```js
export const normalizeEmail = (email) => {
  return email.trim().toLowerCase();
};

export const isCompanyEmail = (email, domain) => {
  return normalizeEmail(email).endsWith(`@${domain}`);
};
```

## 4. Object Modules And Closures

Use closures when you need private state without exposing a class.

Best for:

- Small clients.
- Memoized services.
- Counters, queues, caches.
- Dependency-injected modules.

Bad for:

- Complex object graphs.
- APIs where lifecycle should be explicit.
- Hiding state that tests need to reset.

```js
export const createTokenStore = () => {
  let token = null;

  const getToken = () => {
    return token;
  };

  const setToken = (nextToken) => {
    token = nextToken;
  };

  return { getToken, setToken };
};
```

## 5. Classes And Prototypes

Use classes when object identity, polymorphism, lifecycle, or encapsulated mutable state is central to the design.

Good use:

- SDK clients.
- Stateful parsers.
- Long-lived connections.
- Domain objects with meaningful invariants.

Bad use:

- Static utility namespaces.
- One-method wrappers.
- Recreating Java-style architecture in JavaScript.

Prefer composition over inheritance. If inheritance is mostly for code reuse, use functions or object composition instead.

## 6. Factory Pattern

### Purpose

Use factories to create configured functions, services, clients, or strategies.

Best for:

- Injecting dependencies.
- Creating environment-specific clients.
- Avoiding module-level global state.
- Building testable services.

Bad for:

- Simple object literals.
- One-off construction.
- Abstract factories with no actual variation.

```js
export const createUsersService = ({ usersApi, logger }) => {
  const getUserProfile = async (userId) => {
    logger.info("Loading user profile", { userId });
    const user = await usersApi.getUser(userId);
    return { id: user.id, displayName: user.name };
  };

  return { getUserProfile };
};
```

## 7. Strategy Pattern

### Purpose

Use strategies when a named behavior varies by runtime condition.

Best for:

- Pricing rules.
- Validation policies.
- Export formats.
- Role-based behavior.
- Tenant-specific workflows.

Bad for:

- Two tiny branches.
- Behavior that is unlikely to grow.
- Strategy maps that hide simple logic.

```js
const validators = {
  email: (value) => value.includes("@"),
  phone: (value) => value.length >= 10,
};

export const validateContact = ({ type, value }) => {
  const validate = validators[type];

  if (!validate) {
    throw new Error(`Unsupported contact type: ${type}`);
  }

  return validate(value);
};
```

## 8. Adapter And Facade Patterns

### Purpose

Use adapters to translate one API into another. Use facades to expose a simpler local API over a complex subsystem.

Best for:

- Analytics.
- Auth.
- Payments.
- Feature flags.
- HTTP clients.
- Storage.
- Notifications.

Bad for:

- Wrapping stable built-ins with no added value.
- Mirroring every vendor method one-to-one.
- Hiding errors so callers cannot make correct decisions.

## 9. Dependency Injection

Dependency injection is passing dependencies in instead of importing hardcoded infrastructure everywhere.

Use it for:

- API clients.
- Clocks and timers.
- Random ID generators.
- Loggers.
- Feature flag clients.
- Vendor SDKs.

Avoid full dependency injection containers unless the codebase is large enough to justify that complexity.

```js
export const createCheckoutService = ({ payments, clock }) => {
  const submitPayment = async (cart) => {
    return payments.charge({
      amount: cart.total,
      submittedAt: clock.now(),
    });
  };

  return { submitPayment };
};
```

## 10. Repository / Gateway Pattern

Use a repository or gateway to isolate persistence or remote I/O from domain logic.

Best for:

- Database access.
- REST/GraphQL API access.
- Local storage.
- External service boundaries.

Bad for:

- Thin wrappers over one fetch call with no mapping, validation, or contract value.

Keep repositories boring: fetch, validate, map, return.

## 11. Observer / Pub-Sub Pattern

Use observer or pub-sub when multiple subscribers need to react to events without tight coupling.

Best for:

- Analytics events.
- Internal event buses in SDKs.
- Cross-module notifications.
- Plugin-style systems.

Bad for:

- Core business flow where explicit function calls are clearer.
- Replacing predictable data flow with invisible events.
- Events with no ownership or lifecycle cleanup.

## 12. Command Pattern

Use commands to represent actions as data or objects.

Best for:

- Undo/redo.
- Queues.
- Job runners.
- Audit trails.
- Retryable actions.

Bad for:

- Ordinary function calls.
- Simple handlers where action objects add ceremony.

## 13. Pipeline And Middleware Patterns

Use pipelines when data should pass through ordered transformations. Use middleware when cross-cutting behavior wraps a request or action.

Best for:

- Request processing.
- Data normalization.
- Validation chains.
- Logging, auth, and tracing around I/O.

Bad for:

- Branch-heavy workflows.
- Steps that secretly depend on external mutable state.
- Pipelines where debugging intermediate values is hard.

## 14. Async Architecture

Rules:

- Use `async` / `await` for readable control flow.
- Use `Promise.all` for independent concurrent work.
- Use `Promise.allSettled` when partial failure is acceptable.
- Accept an `AbortSignal` for cancellable I/O.
- Keep retries, timeouts, and backoff in infrastructure helpers.
- Do not swallow errors.
- Do not mix callback, promise, and event styles without a clear boundary.

```js
export const loadDashboard = async ({ usersApi, reportsApi, signal }) => {
  const [user, reports] = await Promise.all([
    usersApi.getCurrentUser({ signal }),
    reportsApi.listReports({ signal }),
  ]);

  return { user, reports };
};
```

## 15. Error Handling

Design errors so callers can make decisions.

Good options:

- Throw typed/custom errors at infrastructure boundaries.
- Return result objects for expected domain failures.
- Preserve original error causes with `cause`.
- Add context at boundaries, not everywhere.

Avoid:

- Returning `null` for every failure.
- Catching and logging without rethrowing or returning a useful result.
- Stringly typed error checks spread across the app.

## 16. Runtime Validation

Validate untrusted input at system boundaries:

- HTTP responses.
- User input.
- Environment variables.
- Local storage.
- CLI arguments.
- Webhook payloads.

Do not repeatedly validate trusted internal data at every function call unless crossing a boundary.

## 17. Data Mapping

Use mappers to isolate external payload shapes from internal models.

Best for:

- API payloads with unstable names.
- Versioned data.
- Vendor SDKs.
- Database rows.

Bad for:

- Copying identical objects for no reason.
- Mappers that hide business decisions.

## 18. Configuration Design

Prefer small explicit options objects for functions with more than two arguments or optional behavior.

Good:

```js
export const createHttpClient = ({ baseUrl, timeoutMs, getToken }) => {
  // ...
};
```

Bad:

```js
export const createHttpClient = (baseUrl, timeoutMs, token, retry, logger, cache) => {
  // ...
};
```

Do not use giant config objects as a substitute for a clear API.

## 19. Performance And Memory

- Measure before optimizing.
- Avoid repeated work in hot paths.
- Avoid unbounded caches.
- Clean up timers, subscriptions, and event listeners.
- Prefer streaming or iteration for large datasets.
- Avoid accidental sequential awaits when work can be concurrent.

## 20. JavaScript Anti-patterns

- Global mutable state.
- Hidden side effects in utility functions.
- Catching and ignoring errors.
- Overusing classes for namespaces.
- Premature factories and strategy registries.
- Giant `utils.js` files.
- Deep object mutation across module boundaries.
- Boolean parameter traps.
- Long parameter lists.
- Mixing validation, I/O, mapping, and business rules in one function.
- Event buses for core data flow.
- Magic strings with no local constants or validation.
- Date/time and randomness hardcoded into business logic.

## 21. Pattern Selection Guide

| Need | Recommended Pattern |
| --- | --- |
| Small deterministic transformation | Plain function |
| Shared configured behavior | Factory |
| Private simple state | Closure/object module |
| Object identity or lifecycle | Class |
| Runtime behavior variants | Strategy |
| Hide third-party API | Adapter / Facade |
| Isolate persistence or remote I/O | Repository / Gateway |
| Broadcast decoupled events | Observer / Pub-Sub |
| Queue, retry, undo, or audit actions | Command |
| Ordered transformations | Pipeline |
| Cross-cutting request behavior | Middleware |
| Testable infrastructure dependencies | Dependency Injection |
| Simple one-off branch | Plain conditional |

## 22. Code Examples

### Adapter For Analytics

```js
export const createAnalyticsAdapter = ({ vendor }) => {
  const track = ({ name, properties = {} }) => {
    vendor.track(name, properties);
  };

  const identify = (userId) => {
    vendor.identify(userId);
  };

  return { track, identify };
};
```

### Result Object For Expected Failure

```js
export const parseJson = (text) => {
  try {
    return { ok: true, value: JSON.parse(text) };
  } catch (error) {
    return { ok: false, error };
  }
};
```

### Pipeline

```js
export const pipe = (...steps) => {
  return (input) => {
    return steps.reduce((value, step) => step(value), input);
  };
};

const normalizeUser = pipe(trimUserFields, lowerCaseEmail, removeEmptyMetadata);
```

## 23. Testing Guidance

- Test public behavior, not private implementation details.
- Test pure functions with direct inputs and outputs.
- Test services with fake dependencies injected through factories.
- Test adapters against local contracts, not vendor internals.
- Test async failures, cancellation, timeout, and retry behavior.
- Test mappers with realistic external payloads.
- Avoid snapshot-only tests for behavior-heavy modules.
- Keep tests close to the code surface they validate.
