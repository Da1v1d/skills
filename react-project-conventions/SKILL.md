---
name: react-project-conventions
description: Core project conventions for React web and React Native or Expo projects. Use when creating, editing, reviewing, or organizing React projects that should follow the documented architecture, naming, styling, TypeScript, and Bun conventions.
---

# React Project Conventions

## Package Manager / Runtime

Use **Bun** for this project.

- Run scripts with `bun run`
  - Example: `bun run start`
  - Example: `bun run build`

- Install dependencies with `bun install`
- Prefer `bun` over `npm` or `yarn` in commands, scripts, examples, and documentation.

## Stack

This convention file applies to both **React web** and **React Native / Expo** projects.

### React Web

- React
- TypeScript
- Vite / Next.js when applicable
- Tailwind CSS / project styling system
- Browser-native APIs and project shared UI

### React Native / Expo

- React Native
- Expo
- Expo Router when applicable
- TypeScript
- Uniwind / Tailwind
- React Native and Expo primitives
- Shared app components from `src/shared/components`

Do not use platform-wrong UI kits.

- In React Native, do not use web UI kits such as MUI, Ant Design Web, Chakra UI, or browser-only components.
- In React web, do not use React Native-only primitives unless the project is explicitly configured for React Native Web.

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

## Routing

Use `src/app/` for route-level files.

### React Web

For Next.js:

```txt
src/app/
  layout.tsx
  page.tsx
  dashboard/
    page.tsx
```

For Vite / React Router:

```txt
src/app/
  router.tsx
  providers.tsx
  routes/
```

### React Native / Expo

For Expo Router:

```txt
src/app/
  _layout.tsx
  index.tsx
  (tabs)/
  (auth)/
```

Route files should mainly compose features and screens. Avoid putting large business logic directly inside route files.

Good:

```tsx
import LoginForm from "@/features/auth/ui/login-form";

const LoginScreen = () => {
  return <LoginForm />;
};

export default LoginScreen;
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
```

## Feature Segments

### `ui/`

Feature-specific components.

Use this folder for components used only by this feature.

```txt
src/features/auth/ui/login-form.tsx
src/features/onboarding/ui/onboarding-step.tsx
src/features/accounts/ui/accounts-list.tsx
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
src/features/auth/model/hooks/useLogin.ts
src/features/auth/model/stores/auth.store.ts
src/features/onboarding/model/context/onboarding.context.tsx
```

### `api/`

Feature API modules.

Use this folder when a feature calls backend endpoints. Feature API modules should compose shared infrastructure from `src/shared/services`.

```txt
src/features/accounts/api/accounts.api.ts
src/features/auth/api/auth.api.ts
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
src/features/auth/lib/accounts.validation.ts
```

### `config/`

Feature-specific constants, configs, or feature flags.

```txt
src/features/onboarding/config/onboarding.configs.ts
src/features/auth/config/auth.constants.ts
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

### Keep inside a feature when only one feature uses it

```txt
src/features/auth/lib/validation.ts
src/features/accounts/model/hooks/useAccounts.ts
src/features/onboarding/ui/StepIndicator.tsx
```

### Move to shared when two or more features use it

```txt
src/shared/components/Button.tsx
src/shared/hooks/useDebounce.ts
src/shared/utils/formatDate.ts
src/shared/types/api.ts
```

## Imports

Prefer stable public import paths when aliases are configured.

Good:

```ts
import LoginForm from "@/features/auth/ui/LoginForm";
import Button from "@/shared/components/Button";
import { formatDate } from "@/shared/utils/formatDate";
```

Avoid long relative imports across slices.

Bad:

```ts
import Button from "../../../../shared/components/Button";
```

Relative imports are acceptable only for files inside the same local module or folder.

Good:

```ts
import { validateLoginForm } from "../lib/validation";
```

## Components

Use `const` declarations for components.

Good:

```tsx
type LoginScreenProps = {
  title?: string;
};

const LoginScreen = ({ title }: LoginScreenProps) => {
  return <LoginForm title={title} />;
};

export default LoginScreen;
```

Avoid function declarations for components unless there is a strong reason.

Bad:

```tsx
export default function LoginScreen() {
  return <LoginForm />;
}
```

Use `export default` for screen and component files.

## Handlers

Event handlers must use the `handler` suffix.

Good:

```tsx
const pressHandler = () => {};
const submitHandler = () => {};
const closeModalHandler = () => {};
const changeTextHandler = (value: string) => {};
```

Bad:

```tsx
const handlePress = () => {};
const onSubmit = () => {};
const click = () => {};
```

Use descriptive names for variables, functions, and handlers.

## Styling

Use the project styling system consistently.

### React Web

Prefer Tailwind CSS or the project-approved styling approach.

Good:

```tsx
<div className="flex min-h-screen bg-white px-4">
  <h1 className="text-lg font-semibold text-black">Title</h1>
</div>
```

Avoid mixing multiple styling systems without a clear project reason.

### React Native / Expo

Use **Uniwind / Tailwind classes** for styling.

Good:

```tsx
<View className="flex-1 bg-white px-4">
  <Text className="text-lg font-semibold text-black">Title</Text>
</View>
```

Avoid inline styles unless required for dynamic runtime values.

Acceptable:

```tsx
<View style={{ width: progressWidth }} />
```

Bad:

```tsx
<View style={{ flex: 1, backgroundColor: "white", paddingHorizontal: 16 }} />
```

Prefer extracting repeated class groups into reusable components rather than duplicating large class strings.

## Platform-Specific Code

Keep platform-specific code explicit.

Recommended naming:

```txt
button.web.tsx
button.native.tsx
use-keyboard-insets.native.ts
use-viewport-size.web.ts
```

Use platform-specific files when behavior or UI differs between React web and React Native.

Avoid hiding large platform differences inside one messy component.

## TypeScript

Use TypeScript everywhere.

Prefer explicit types for:

- component props
  Good:

```ts
type Props = {
  className: string;
  children: ReactNode;
};

export type LoginFormProps = {
  isLoading: boolean;
  onSubmit: (values: LoginFormValues) => void;
};
```

- API responses
- store state
- public utility function arguments
- shared reusable modules
  Good:

```ts

type User = {
  name: string;
  surname: string;
};

type AdminUser = User && {
  IsAdmin: true;
}
```

Avoid `any`.

Use `unknown` when the type is truly unknown and narrow it before usage.

## Hooks

Feature-specific hooks belong inside the feature.

```txt
src/features/auth/model/hooks/use-login.ts
```

Shared hooks belong in:

```txt
src/shared/hooks/
```

Hook names must start with `use`.

Good:

```ts
useLogin;
useAccounts;
useDebounce;
useKeyboardInsets;
```

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
src/features/onboarding/model/context/onboardin-context.tsx
src/processes/providers/AppProviders.tsx
```

## API Layer

Shared HTTP clients and infrastructure belong in:

```txt
src/shared/services/
```

Feature API modules belong in:

```txt
src/features/<feature>/api
```

Good:

```txt
src/shared/services/httpClient.ts
src/features/auth/api/auth.api.ts
src/features/accounts/api/accounts.api.ts
```

Feature APIs should not duplicate base request logic.

## Assets

Use `src/assets` for static media.

```txt
src/assets/images/
src/assets/icons/
src/assets/videos/
```

Feature-specific assets may stay inside a feature only if they are strongly tied to that feature and not reused elsewhere.

## File Naming

Use clear, descriptive file names.

Recommended:

```txt
LoginScreen.tsx
LoginForm.tsx
auth.api.ts
auth.store.ts
useLogin.ts
validation.ts or schemas.ts
types.ts
utils.ts
constants.ts
```

Avoid vague names:

```txt
helpers.ts
data.ts
stuff.ts
common.ts
```

Use generic proper names like `<feature>.utils.ts` or `<feature>.types.ts` only inside a clear scope.

Good:

```txt
src/features/auth/lib/auth.utils.ts
src/features/auth/lib/auth.types.ts
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

## Code Style

Prefer:

- early returns
- small components
- readable names
- colocated feature logic
- simple composition
- explicit types where useful
- platform-specific files when platform behavior differs

Avoid:

- deeply nested logic
- large route files
- shared folders becoming dumping grounds
- cross-feature imports between unrelated features
- inline styles for static styling
- platform-wrong UI libraries

## Default Decision Rule

When unsure where to place code:

1. Start inside the feature.
2. Keep it close to where it is used.
3. Move it to `shared/` only after real reuse appears.
4. Keep `shared/` clean, generic, and feature-independent no business logic.
