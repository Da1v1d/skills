---
name: react-project-conventions
description: Core project conventions for React, React Native or Expo, Next.js, and TanStack Start projects. Use when creating, editing, reviewing, or organizing React projects that should follow the documented architecture, naming, styling, TypeScript, and package-manager/runtime conventions.
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
| `src/features/<feature>/`  | Feature slices with UI, model, API, lib, and config                                   |
| `src/shared/components/`   | Reusable shared UI components                                                         |
| `src/shared/hooks/`        | Cross-feature reusable hooks                                                          |
| `src/shared/utils/`        | Cross-feature helpers and pure utilities                                              |
| `src/shared/constants/`    | Shared constants                                                                      |
| `src/shared/types/`        | Shared TypeScript types                                                               |
| `src/shared/services/`     | Shared infrastructure such as HTTP clients, storage, logging, analytics, i18n helpers |
| `src/shared/configs/`      | Shared configuration modules                                                          |
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

```txt
src/features/auth/ui/pages/login-page.tsx
src/features/auth/ui/screens/login-screen.tsx
src/features/auth/ui/forms/login-form.tsx
src/features/auth/ui/wrappers/auth-form-wrapper.tsx
src/features/onboarding/ui/blocks/onboarding-step.tsx
src/features/accounts/ui/elements/account-row.tsx
```

### `model/`

Feature state and data-access logic.

Use this folder for:

- feature hooks
- feature stores
- feature contexts
- state mappers
- screen-level state logic

```txt
src/features/auth/model/hooks/use-login.ts
src/features/auth/model/stores/auth.store.ts
src/features/onboarding/model/context/onboarding.context.tsx
```

### `api/`

Feature API modules.

Use this folder when a feature calls backend endpoints. Feature API modules should compose shared infrastructure from `src/shared/services`.

```txt
src/features/accounts/api/accounts.service.ts
src/features/auth/api/auth.service.ts
```

Do not create duplicate raw HTTP clients inside features if a shared client already exists.

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

### `config/`

Feature-specific constants, configs, feature flags, or permission definitions.

```txt
src/features/onboarding/config/onboarding.configs.ts
src/features/auth/config/auth.constants.ts
src/features/accounts/config/accounts.permissions.tsx
```

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
src/features/auth/lib/validation.ts
src/features/accounts/model/hooks/use-accounts.ts
src/features/onboarding/ui/elements/step-indicator.tsx
```

### Move To Shared When Two Or More Features Use It

```txt
src/shared/components/button.tsx
src/shared/hooks/use-debounce.ts
src/shared/utils/format-date.ts
src/shared/types/api.ts
```

## Implementation Style

For detailed import, component, function, handler, TypeScript, hook, API, asset, and file naming conventions, read `references/code-style.md`.

## State Management

Feature-specific state should stay inside the feature slice.

Shared or global state may live in `src/shared` or `src/processes` depending on purpose.

Use:

- `features/<feature>/model/stores/` for feature stores
- `features/<feature>/model/context/` for feature-scoped context
- `processes/` for app-wide providers and global wiring

Examples:

```txt
src/features/auth/model/stores/auth.store.ts
src/features/onboarding/model/context/onboarding.context.tsx
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

## Default Decision Rule

When unsure where to place code:

1. Start inside the feature.
2. Keep it close to where it is used.
3. Move it to `shared/` only after real reuse appears.
4. Keep `shared/` clean, generic, and feature-independent.
