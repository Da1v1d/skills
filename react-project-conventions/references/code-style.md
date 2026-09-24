# Code Style

Use this reference when changing implementation details such as imports, component style, handler naming, TypeScript, hooks, API modules, assets, or file naming.

## Contents

- Imports
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
import { validateLoginForm } from "@/features/auth/lib/validation";
import { formatDate } from "@/shared/utils/format-date";
```

Do not use relative imports for project files, even when the relative path is short.

Bad:

```ts
import Button from "../../../../shared/components/button";
import { validateLoginForm } from "../lib/validation";
import { formatDate } from "../../shared/utils/format-date";
```

Relative imports are allowed only for package-local files outside the app source tree, such as test fixtures or config files, when no alias is configured for that area.

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

Use `export default` for screen and component files.

## Functions

Prefer arrow functions assigned to `const` instead of function declarations for project code.

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

Prefer explicit types for:

- component props
- API responses
- store state
- public utility function arguments
- shared reusable modules

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
useAccountFilters;
useDebounce;
useKeyboardInsets;
```

Do not create a hook that only forwards to `useQuery(xxxQueryOptions())`. Call `useQuery(accountsQueryOptions())` directly in each component. Create a custom hook only when it accepts props/params or adds logic (`select`, `enabled`, derived state, combining several queries).

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
src/shared/services/http-client.ts
src/features/auth/api/auth.service.ts
src/features/accounts/api/accounts.service.ts
```

Feature APIs should not duplicate base request logic.

Good:

```ts
// src/shared/services/http-client.ts
export class HttpClient extends ApiRequestService {
  constructor() {
    super();
  }
}

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
```

Do not unwrap responses with `.then((res) => res.data)` in feature services. Response unwrapping lives in the shared `ApiRequestService` (`HttpClient`) when it is available; only unwrap in the feature module if the shared client returns the raw response.

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
use-login.tsx
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

Recommended:

```txt
login-screen.tsx
login-form.tsx
auth.service.ts
auth.store.ts
use-login.ts
validation.ts or schemas.ts
types.ts
utils.ts
constants.ts
use-login.test.ts
login-form.test.tsx
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
