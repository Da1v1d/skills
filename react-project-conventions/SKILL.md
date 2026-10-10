---
name: react-project-conventions
description: Core project conventions for React, React Native or Expo, Next.js, and TanStack Start projects. Use when creating, editing, reviewing, or organizing React projects — deciding where a file or folder goes, structuring feature slices, naming files, writing imports and exports, placing queries, mutations, and API code, or choosing styling, TypeScript, and package-manager/runtime conventions.
---

# React Project Conventions

## How To Use This Skill

Apply the shared conventions in this file to every React-family project, then read the stack reference that matches the repository:

- React SPA or Vite: `references/react.md`
- React Native or Expo: `references/react-native.md`
- Next.js: `references/next-js.md`
- TanStack Start: `references/tanstack-start.md`

When changing implementation details such as imports, component style, handler naming, TypeScript, hooks, API modules, assets, or file naming, read `references/code-style.md`.

If a project mixes stacks, read each relevant reference and prefer the more specific stack rule when there is overlap.

## Precedence

Apply these conventions fully to new projects and new code.

In an existing repository that consistently follows a different convention, follow the repository for that area and keep the change consistent with its surroundings. Do not mass-migrate existing code to these conventions unless the user asks.

## Package Manager / Runtime

For new projects, prefer **Bun**.

For existing projects, follow the detected package manager and runtime from `package.json`, the `packageManager` field, lockfiles, scripts, and existing documentation.

- Use `bun run` and `bun install` when the project uses Bun or has no established package manager.
- Use the existing tool when the repository clearly uses `npm`, `pnpm`, or `yarn`.
- Match the repository's current command style in examples, scripts, and documentation.

## Architecture

The project follows a **Feature-Sliced Design inspired** structure under `src/`.

The main rule:

> Keep feature-specific code inside the feature. Move code to `shared/` only when it is reused by multiple features or is truly platform/project-wide.

## Top-Level Structure

| Path                       | Purpose                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------- |
| `src/app/`                 | App routing, layouts, pages/screens, and route-level composition                      |
| `src/processes/`           | App-wide providers, contexts, initialization, and cross-cutting wiring                |
| `src/features/<feature>/`  | Feature slices with UI, model, API, lib, config, and tests                            |
| `src/shared/components/`   | Reusable shared UI components                                                         |
| `src/shared/hooks/`        | Cross-feature reusable hooks                                                          |
| `src/shared/utils/`        | Cross-feature helpers and pure utilities                                              |
| `src/shared/constants/`    | Shared constants                                                                      |
| `src/shared/types/`        | Shared TypeScript types                                                               |
| `src/shared/services/`     | Shared infrastructure such as HTTP clients, storage, logging, analytics, i18n helpers |
| `src/shared/configs/`      | Shared configuration modules                                                          |
| `src/shared/tests/`        | Cross-feature test utilities, render helpers, and shared mocks                        |
| `src/shared/translations/` | i18n resources                                                                        |
| `src/assets/images/`       | Static images                                                                         |
| `src/assets/icons/`        | Static icons                                                                          |
| `src/assets/videos/`       | Static videos                                                                         |

Route files should mainly compose feature pages or screens. Avoid putting large business logic directly inside route files.

Good:

```tsx
import LoginScreen from "@/features/auth/ui/screens/login-screen";

const LoginRoute = () => {
  return <LoginScreen />;
};

export default LoginRoute;
```

## Feature Slice Structure

Each feature lives in:

```txt
src/features/<feature>/
```

Recommended structure:

```txt
src/features/<feature>/
  api/
  config/
  lib/
  model/
    context/
    hooks/
    stores/
  ui/
    pages/ or screens/
    forms/
    wrappers/
    blocks/
    elements/
  tests/
    fixtures/
    mocks/
```

## Feature Segments

### `ui/`

Feature-specific components.

Use this folder for components used only by this feature.

Organize feature UI by composition level:

- `pages/` for React web, Next.js, and TanStack Start page-level feature views
- `screens/` for React Native / Expo screen-level feature views
- `forms/` for feature-specific form components, field groups, and validation UI
- `wrappers/` for feature-specific layout shells, providers, guards, and composition wrappers
- `blocks/` for larger feature sections composed from elements and shared components
- `elements/` for small feature-only UI pieces

Use either `pages/` or `screens/` based on the stack. Do not use both unless the same feature is intentionally shared across web and native.

Start flat. When a feature has only a handful of UI files (about five or fewer), keep them all directly in `ui/` and do not split them into `pages/`, `screens/`, `forms/`, `wrappers/`, `blocks/`, or `elements/`. A small feature with a screen, a form, and a wrapper stays flat.

Flat, for a small feature:

```txt
src/features/settings/ui/settings-screen.tsx
src/features/settings/ui/settings-form.tsx
src/features/settings/ui/settings-wrapper.tsx
src/features/settings/ui/settings-row.tsx
```

Introduce the subfolders only when the flat `ui/` folder grows past that and becomes hard to scan. When you do:

- Move all UI files into subfolders at once. Do not mix flat files with subfolders.
- Use exactly this set (`pages/` or `screens/`, `forms/`, `wrappers/`, `blocks/`, `elements/`) rather than inventing new ones.
- Create only the subfolders that will hold files. Do not add empty ones for completeness.

Split, for a larger feature:

```txt
src/features/auth/ui/screens/login-screen.tsx
src/features/auth/ui/screens/sign-up-screen.tsx
src/features/auth/ui/forms/login-form.tsx
src/features/auth/ui/forms/sign-up-form.tsx
src/features/auth/ui/wrappers/auth-form-wrapper.tsx
src/features/auth/ui/blocks/social-login-block.tsx
src/features/auth/ui/elements/password-strength-meter.tsx
```

### `model/`

Feature client state and state logic.

Use exactly these subfolders:

- `hooks/` for feature hooks and screen-level state logic
- `stores/` for feature stores
- `context/` for feature-scoped React context and its provider

Do not put queries, mutations, server actions, or server functions in `model/`. They belong in `api/`.

```txt
src/features/auth/model/hooks/use-login.ts
src/features/auth/model/stores/auth.store.ts
src/features/onboarding/model/context/onboarding.context.tsx
```

### `api/`

Feature data access: endpoint calls, queries, mutations, and server-side entry points.

Keep files flat in `api/` and name them by kind:

| File                     | Contains                                                  |
| ------------------------ | --------------------------------------------------------- |
| `<feature>.service.ts`   | Endpoint calls built on the shared HTTP client            |
| `<feature>.queries.ts`   | Query keys and `queryOptions` factories                   |
| `<feature>.mutations.ts` | `mutationOptions` factories or mutation functions         |
| `<feature>.actions.ts`   | Next.js Server Actions or TanStack Start server functions |

Create only the files the feature needs. Split one kind into several files only when it grows large, keeping the suffix, for example `create-account.actions.ts`.

```txt
src/features/accounts/api/accounts.service.ts
src/features/accounts/api/accounts.queries.ts
src/features/accounts/api/accounts.mutations.ts
src/features/accounts/api/accounts.actions.ts
```

Feature API modules compose shared infrastructure from `src/shared/services`. Do not create duplicate raw HTTP clients inside features if a shared client already exists.

### `lib/`

Feature-only pure code.

Use this folder for:

- feature-specific utilities
- feature-specific types
- validations
- formatters
- mappers

```txt
src/features/accounts/lib/accounts.types.ts
src/features/accounts/lib/accounts.utils.ts
src/features/auth/lib/auth.validation.ts
```

Name `lib/` files `<feature>.<kind>.ts`. Group related helpers in one file per kind instead of creating one file per function.

### `config/`

Feature-specific constants, configs, feature flags, or permission definitions.

Name files `<feature>.<kind>.ts`.

```txt
src/features/onboarding/config/onboarding.config.ts
src/features/auth/config/auth.constants.ts
src/features/accounts/config/accounts.permissions.ts
```

### `tests/`

Feature-specific tests, fixtures, and mocks.

Use this folder for:

- unit and integration tests for the feature's `ui/`, `model/`, `api/`, and `lib/` code
- test fixtures such as sample entities and API responses
- feature-only mocks such as mocked services, stores, or request handlers

Name each test file after the module it covers, using the `.test.ts` or `.test.tsx` suffix. Keep tests directly in `tests/` and add `fixtures/` and `mocks/` only when the feature has enough of them to justify the split.

```txt
src/features/auth/tests/use-login.test.ts
src/features/auth/tests/login-form.test.tsx
src/features/accounts/tests/accounts.utils.test.ts
src/features/accounts/tests/fixtures/accounts.fixtures.ts
src/features/accounts/tests/mocks/accounts.service.mock.ts
```

Tests import from the feature's own segments and from `shared/`. They should not import from other features' `tests/` folders. Move render helpers, mock factories, or fixtures to `src/shared/tests/` only when two or more features use them.

## Feature Naming

Use plural feature folder names when the feature represents a domain aggregate.

Good:

```txt
src/features/accounts/
src/features/users/
src/features/orders/
src/features/notifications/
```

Accept singular names only when the existing repository already uses them consistently.

## Placement Rules

### Keep Inside A Feature When Only One Feature Uses It

```txt
src/features/auth/lib/auth.validation.ts
src/features/accounts/model/hooks/use-account-filters.ts
src/features/onboarding/ui/elements/step-indicator.tsx
src/features/auth/tests/mocks/auth.service.mock.ts
```

### Move To Shared When Two Or More Features Use It

```txt
src/shared/components/button.tsx
src/shared/hooks/use-debounce.ts
src/shared/utils/format-date.ts
src/shared/types/api.ts
src/shared/tests/render-with-providers.tsx
```

## Implementation Style

For detailed import, component, function, handler, TypeScript, hook, API, asset, and file naming conventions, read `references/code-style.md`.

## State Management

Keep client state inside the feature's `model/` and server state in TanStack Query through the feature's `api/<feature>.queries.ts`.

Put app-wide providers and global wiring in `src/processes/`:

```txt
src/processes/providers/app-provider.tsx
src/processes/providers/ui-provider.tsx
```

## Public Reuse Rule

Before moving something to `shared/`, ask:

1. Is this used by two or more features?
2. Is this independent from one specific feature?
3. Can it be reused without importing feature-specific code?

If the answer is no, keep it inside the feature.

## Cross-Feature Imports

Avoid importing one feature directly into another unrelated feature.

Bad:

```ts
import { useAuthStore } from "@/features/auth/model/stores/auth.store";
```

inside another unrelated feature.

Prefer moving the reusable part to `shared/` or exposing the needed behavior through a process/provider when it is app-wide.

Allowed exceptions:

- route/screen composition inside `src/app/`
- app-wide orchestration inside `src/processes/`

Do not create barrel `index.ts` files for features or segments. Import each module from its own file path, for example `@/features/auth/ui/forms/login-form`.

## Default Decision Rule

When unsure where to place code:

1. Start inside the feature.
2. Keep it close to where it is used.
3. Move it to `shared/` only after real reuse appears.
4. Keep `shared/` clean, generic, and feature-independent.
