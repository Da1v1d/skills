# Code Style

Use this reference when changing implementation details such as imports, component style, handler naming, TypeScript, hooks, API modules, assets, or file naming.

## Contents

- Imports
- Exports
- Components
- Functions
- Handlers
- TypeScript
- Hooks
- API Layer
- Assets
- File Naming
- Code Style

## Imports

Use aliases for project imports.

Project code under `src/` must be imported through the configured alias, usually `@/`.

Good:

```ts
import LoginForm from "@/features/auth/ui/forms/login-form";
import Button from "@/shared/components/button";
import { validateLoginForm } from "@/features/auth/lib/auth.validation";
import { formatDate } from "@/shared/utils/format-date";
```

Do not use relative imports for project files, even when the relative path is short.

Bad:

```ts
import Button from "../../../../shared/components/button";
import { validateLoginForm } from "../lib/auth.validation";
import { formatDate } from "../../shared/utils/format-date";
```

Relative imports are allowed only for package-local files outside the app source tree, such as test fixtures or config files, when no alias is configured for that area.

Import each module from its own file. Do not add or import through barrel `index.ts` files.

## Exports

Use `export default` for components, including pages, screens, and route files.

Use named exports for everything else: hooks, utilities, types, constants, services, query and mutation factories, actions, and stores.

```ts
export const useLogin = () => {};
export const accountsQueryOptions = () => {};
export type Account = { id: string };
```

## Components

Always use arrow functions assigned to `const` for components.

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

Do not use function declarations for components.

Bad:

```tsx
export default function LoginScreen() {
  return <LoginForm />;
}
```

## Functions

Prefer arrow functions assigned to `const` instead of function declarations for project code.

The one exception is API services, which are classes with static methods (see API Layer).

Good:

```ts
const formatUserName = (firstName: string, lastName: string) => {
  return `${firstName} ${lastName}`;
};

const AccountsList = () => {
  const { data } = useQuery(accountsQueryOptions());
  return null;
};
```

Bad:

```ts
function formatUserName(firstName: string, lastName: string) {
  return `${firstName} ${lastName}`;
}

function AccountsList() {
  const { data } = useQuery(accountsQueryOptions());
  return null;
}
```

## Handlers

Use `on...` names for parent/main callbacks and prop-facing functions that are passed into components.

Good:

```tsx
const onPress = () => {};
const onSubmit = () => {};
const onCloseModal = () => {};
const onChangeText = (value: string) => {};

type Props = {
  onPress: () => void;
  onChangeText: (value: string) => void;
};
```

Inside child components, when a local function wraps a callback or adds component-specific logic, use a descriptive `handler` suffix and call the `on...` callback from there.

Good:

```tsx
type Props = {
  onPress: () => void;
  onChangeText: (value: string) => void;
};

const InnerButton = ({ onPress, onChangeText }: Props) => {
  const pressHandler = () => {
    onPress();
  };

  const changeTextHandler = (value: string) => {
    onChangeText(value);
  };

  return null;
};
```

Use the `handler` suffix only when the local function wraps an `on...` callback received from props. If there is no prop callback to call, keep the `on...` name for the local function.

Good:

```tsx
const SearchInput = () => {
  const [value, setValue] = useState("");

  const onChangeText = (text: string) => {
    setValue(text);
  };

  return <TextInput value={value} onChangeText={onChangeText} />;
};
```

Bad:

```tsx
// No onChangeText prop exists, so the suffix has nothing to wrap.
const SearchInput = () => {
  const [value, setValue] = useState("");

  const changeTextHandler = (text: string) => {
    setValue(text);
  };

  return <TextInput value={value} onChangeText={changeTextHandler} />;
};
```

Bad:

```tsx
const handlePress = () => {};
const onSubmit = () => {
  validateForm();
  props.onSubmit();
};
const click = () => {};
```

Use descriptive names for variables, functions, and handlers.

## TypeScript

Use TypeScript everywhere.

Use `type` for object shapes, props, and unions. Use `interface` only when declaration merging is required, such as augmenting a library type.

Name a component's props type `Props` when it is used only in that file. Name it `<ComponentName>Props` and export it when other files need it.

Prefer explicit types for:

- component props
- API responses
- store state
- public utility function arguments
- shared reusable modules

Good:

```ts
// Used only inside this file.
type Props = {
  className: string;
  children: ReactNode;
};

// Imported by other files.
export type LoginFormProps = {
  isLoading: boolean;
  onSubmit: (values: LoginFormValues) => void;
};

type User = {
  name: string;
  surname: string;
};

type AdminUser = User & {
  isAdmin: true;
};
```

Avoid `any`.

Use `unknown` when the type is truly unknown and narrow it before usage.

## Hooks

Place hooks as described in `SKILL.md`: feature hooks in `model/hooks/`, cross-feature hooks in `src/shared/hooks/`.

Hook names must start with `use`.

Good:

```ts
useLogin;
useAccountFilters;
useDebounce;
useKeyboardInsets;
```

Define query options in the feature's `api/<feature>.queries.ts`. Do not create a hook that only forwards to `useQuery(xxxQueryOptions())`. Call `useQuery(accountsQueryOptions())` directly in each component. Create a custom hook only when it accepts props/params or adds logic (`select`, `enabled`, derived state, combining several queries).

Good:

```tsx
const AccountsList = () => {
  const { data } = useQuery(accountsQueryOptions());
  return null;
};
```

Bad:

```ts
// One-line wrapper with no params and no logic.
const useAccounts = () => {
  return useQuery(accountsQueryOptions());
};
```

## API Layer

Place API files as described in `SKILL.md`: the shared HTTP client in `src/shared/services/`, and feature services, queries, mutations, and actions flat in `src/features/<feature>/api/`.

Feature APIs should not duplicate base request logic. Services are classes with static methods that call the shared client instance.

Good:

```ts
// src/shared/services/http-client.ts
export const HttpClient = new ApiRequestService({
  baseURL: import.meta.env.VITE_API_URL,
});

// src/features/accounts/api/accounts.service.ts
import { HttpClient } from "@/shared/services/http-client";

export class AccountsService {
  public static getAll() {
    return HttpClient.get<Account[]>("/accounts");
  }

  public static getById(id: string) {
    return HttpClient.get<Account>(`/accounts/${id}`);
  }
}

// src/features/accounts/api/accounts.queries.ts
import { queryOptions } from "@tanstack/react-query";
import { AccountsService } from "@/features/accounts/api/accounts.service";

export const accountsKeys = {
  all: ["accounts"] as const,
  detail: (id: string) => ["accounts", id] as const,
};

export const accountsQueryOptions = () =>
  queryOptions({
    queryKey: accountsKeys.all,
    queryFn: () => AccountsService.getAll(),
  });

// src/features/accounts/api/accounts.mutations.ts
import { mutationOptions } from "@tanstack/react-query";
import { AccountsService } from "@/features/accounts/api/accounts.service";

export const createAccountMutationOptions = () =>
  mutationOptions({
    mutationFn: (values: CreateAccountValues) => AccountsService.create(values),
  });
```

Do not unwrap responses with `.then((res) => res.data)` in feature services. Response unwrapping lives in the shared client when it is available; only unwrap in the feature module if the shared client returns the raw response.

Bad:

```ts
// src/features/accounts/api/accounts.service.ts
// Re-creates base URL and headers that already live in the shared client,
// and unwraps the response that the shared client already handles.
export class AccountsService {
  public static getAll() {
    return axios
      .get<Account[]>(`${import.meta.env.VITE_API_URL}/accounts`, {
        headers: { Authorization: `Bearer ${getToken()}` },
      })
      .then((res) => res.data);
  }
}
```

## Assets

Use `src/assets` for static media.

```txt
src/assets/images/
src/assets/icons/
src/assets/videos/
```

Feature-specific assets may stay inside a feature only if they are strongly tied to that feature and not reused elsewhere.

## File Naming

Use clear, descriptive kebab-case file and folder names.

Use `-` between words. Do not use camelCase or PascalCase for file or folder names.

Good:

```txt
use-login.ts
login-screen.tsx
account-settings-form.tsx
keyboard-insets.ts
http-client.ts
```

Bad:

```txt
useLogin.tsx
LoginScreen.tsx
AccountSettingsForm.tsx
keyboardInsets.ts
HttpClient.ts
```

This rule applies to filenames and directory names. TypeScript identifiers should still use the normal language conventions, such as `LoginScreen`, `useLogin`, and `formatDate`.

Name files by what they are:

| Kind                              | Pattern                                 | Example                                    |
| --------------------------------- | --------------------------------------- | ------------------------------------------ |
| Components, pages, screens        | `<name>.tsx`                            | `login-form.tsx`, `login-screen.tsx`       |
| Hooks                             | `use-<name>.ts`                         | `use-login.ts`                             |
| Feature `api/`, `lib/`, `config/` | `<feature>.<kind>.ts`                   | `auth.service.ts`, `auth.validation.ts`    |
| Stores and contexts               | `<name>.store.ts`, `<name>.context.tsx` | `auth.store.ts`, `onboarding.context.tsx`  |
| Shared utilities and services     | `<name>.ts`                             | `format-date.ts`, `http-client.ts`         |
| Tests                             | `<module>.test.ts(x)`                   | `use-login.test.ts`, `login-form.test.tsx` |

Avoid vague names:

```txt
helpers.ts
data.ts
stuff.ts
common.ts
```

Inside a feature, always prefix generic kinds with the feature name so the file is identifiable in search results and editor tabs.

Good:

```txt
src/features/auth/lib/auth.utils.ts
src/features/auth/lib/auth.types.ts
src/features/auth/config/auth.config.ts
```

Bad:

```txt
# Missing the feature prefix.
src/features/auth/lib/types.ts
# Icons are static media and belong in assets/.
src/features/auth/lib/auth-icons.ts
# One helper per file. Put it in auth.utils.ts instead.
src/features/auth/lib/format-name.ts
```

## Code Style

Prefer:

- early returns
- small components
- readable names
- colocated feature logic
- simple composition
- arrow functions assigned to `const`
- explicit types where useful
- platform-specific files when platform behavior differs

Avoid:

- deeply nested logic
- large route files
- shared folders becoming dumping grounds
- cross-feature imports between unrelated features
- inline styles for static styling
- platform-wrong UI libraries
- function declarations for project code
