# TypeScript Patterns Architect Reference

## Table Of Contents

1. Core Principles
2. Public API Types
3. Type Aliases vs Interfaces
4. Unknown, Never, And Any
5. Narrowing
6. Discriminated Unions
7. Result Types
8. Generics
9. Utility Types
10. Branded Types
11. Template Literal Types
12. Conditional And Mapped Types
13. Type Guards And Assertion Functions
14. Runtime Validation Boundaries
15. API Client Types
16. Domain Modeling
17. Configuration Types
18. Error Types
19. Module Boundaries
20. Declaration Files And Library Types
21. TypeScript Anti-patterns
22. Pattern Selection Guide
23. Code Examples
24. Testing Guidance

## 1. Core Principles

- Use TypeScript to describe real contracts, not to perform clever puzzles.
- Prefer simple explicit types over generic abstractions.
- Use `unknown` for untrusted data and narrow it.
- Avoid `any`; if it is unavoidable, isolate it at a boundary.
- Make impossible states impossible with discriminated unions.
- Keep runtime validation separate from compile-time types.
- Export public types intentionally. Do not leak internal helper types unless consumers need them.
- Prefer inference inside function bodies and explicit types at public boundaries.
- Use advanced type features only when they reduce consumer mistakes.

## 2. Public API Types

Public APIs should be boring and stable:

- Explicit parameter types.
- Explicit return types for exported functions.
- Named option types for complex parameters.
- Narrow literal unions for modes, variants, and statuses.
- JSDoc only when the type alone does not explain behavior.

```ts
export type CreateUserInput = {
  email: string;
  displayName: string;
};

export type User = {
  id: string;
  email: string;
  displayName: string;
};

export const createUser = async (input: CreateUserInput): Promise<User> => {
  // ...
};
```

## 3. Type Aliases vs Interfaces

Use either consistently, but prefer:

- `type` for unions, intersections, utility types, function types, and aliases.
- `interface` when consumers are expected to augment a shape or when project conventions prefer it for object contracts.

Avoid debating this when consistency matters more than the distinction.

## 4. Unknown, Never, And Any

### `unknown`

Use for data whose shape is not trusted yet:

- JSON parsing.
- External API payloads.
- Local storage.
- Message events.
- Catch variables.

### `never`

Use for exhaustive checks and impossible states.

### `any`

Avoid. If required for third-party interop, isolate it and immediately convert to a safe local type.

## 5. Narrowing

Narrow values before use:

- `typeof`
- `Array.isArray`
- `in`
- equality checks
- custom type guards
- runtime schema validation

Do not use broad type assertions to silence the compiler.

## 6. Discriminated Unions

Use discriminated unions for states with distinct variants.

Best for:

- Async states.
- Domain workflows.
- Payment statuses.
- Validation results.
- Command/event payloads.

```ts
export type LoadState<TData> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: TData }
  | { status: "error"; error: Error };
```

This is better than optional fields like `{ data?: TData; error?: Error; loading?: boolean }` because it prevents impossible combinations.

## 7. Result Types

Use result types for expected failures that callers should handle.

```ts
export type Result<TValue, TError> =
  | { ok: true; value: TValue }
  | { ok: false; error: TError };
```

Best for:

- Validation.
- Parsing.
- Domain rules.
- Recoverable operations.

Use exceptions for unexpected failures or infrastructure errors if that matches the codebase.

## 8. Generics

Use generics when the caller provides a type relationship the function preserves.

Good:

```ts
export const mapValues = <TInput, TOutput>(
  values: TInput[],
  mapper: (value: TInput) => TOutput,
): TOutput[] => {
  return values.map(mapper);
};
```

Bad:

```ts
export const getValue = <TValue>(value: TValue): TValue => {
  return value;
};
```

The second example adds no meaningful abstraction.

Rules:

- Name generics descriptively for public APIs, such as `TUser` or `TResponse`.
- Avoid more than two or three generic parameters unless the abstraction is truly library-level.
- Prefer overloads or separate functions when generic inference becomes hostile.
- Do not force consumers to pass type parameters when inference can work.

## 9. Utility Types

Use utility types to express local transformations:

- `Pick`
- `Omit`
- `Partial`
- `Required`
- `Record`
- `Extract`
- `Exclude`
- `ReturnType`
- `Parameters`

Avoid stacking utility types until the result is unreadable. Name the resulting type when it becomes part of the public API.

## 10. Branded Types

Use branded types to prevent mixing primitive values with different meanings.

Best for:

- IDs.
- Currencies.
- Tokens.
- User-provided identifiers.

Bad for:

- Every string in the app.
- Values that never cross important boundaries.

```ts
export type Brand<TValue, TBrand extends string> = TValue & { readonly __brand: TBrand };

export type UserId = Brand<string, "UserId">;
export type OrganizationId = Brand<string, "OrganizationId">;
```

Use constructors or validators to create branded values safely.

## 11. Template Literal Types

Use template literal types for constrained string formats when they improve correctness.

Good:

```ts
export type TranslationKey = `settings.${string}` | `profile.${string}`;
```

Bad:

- Modeling complex regex-like formats that TypeScript cannot truly validate.
- Huge unions that slow the compiler.

## 12. Conditional And Mapped Types

Use conditional and mapped types for library-level transformations, not ordinary app code.

Good use:

- Typed SDKs.
- Form libraries.
- API route clients.
- Reusable schema helpers.

Bad use:

- Replacing a simple explicit type.
- Type-level logic that the team cannot maintain.
- Nested conditional types with unclear compiler errors.

## 13. Type Guards And Assertion Functions

Use type guards to narrow unknown data:

```ts
export const isUser = (value: unknown): value is User => {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  return "id" in value && "email" in value;
};
```

Use assertion functions when invalid data should throw:

```ts
export const assertUser = (value: unknown): asserts value is User => {
  if (!isUser(value)) {
    throw new Error("Invalid user.");
  }
};
```

Prefer schema libraries for complex validation.

## 14. Runtime Validation Boundaries

TypeScript does not validate runtime data.

Validate:

- HTTP responses.
- Environment variables.
- Local storage.
- URL/search params.
- Webhook payloads.
- Message events.

After validation, convert external data into internal domain types.

## 15. API Client Types

Keep external DTOs separate from internal models when API payloads are unstable or vendor-owned.

```ts
type UserDto = {
  user_id: string;
  display_name: string;
};

export type User = {
  id: string;
  displayName: string;
};

export const mapUserDto = (dto: UserDto): User => {
  return {
    id: dto.user_id,
    displayName: dto.display_name,
  };
};
```

Do not duplicate DTO/model layers when the shapes are identical and stable.

## 16. Domain Modeling

Model domain concepts directly:

- Use literal unions for finite states.
- Use discriminated unions for workflows.
- Use branded IDs for important identifiers.
- Use readonly arrays and objects when mutation is not part of the contract.
- Use domain-specific names instead of generic bags like `Data` or `Payload`.

## 17. Configuration Types

Prefer named option types:

```ts
export type CreateHttpClientOptions = {
  baseUrl: string;
  timeoutMs: number;
  getToken?: () => string | null;
};
```

Avoid boolean traps:

```ts
sendEmail(user, true, false);
```

Prefer explicit options:

```ts
sendEmail(user, { includeReceipt: true, previewOnly: false });
```

## 18. Error Types

Use typed errors or result errors when callers need to branch.

```ts
export type PaymentError =
  | { code: "card-declined"; message: string }
  | { code: "insufficient-funds"; message: string }
  | { code: "network-error"; retryable: true };
```

Avoid checking error message strings.

## 19. Module Boundaries

- Keep exported types close to the module that owns them.
- Export only what other modules need.
- Avoid barrel files that hide import ownership or create circular dependencies.
- Keep shared types generic and feature-independent.
- Do not put every type in `types.ts` if ownership is clearer in a specific module.

Good:

```txt
src/features/billing/lib/billing.types.ts
src/features/billing/api/billing.api.ts
src/shared/services/http-client.ts
```

## 20. Declaration Files And Library Types

Use `.d.ts` files for ambient declarations, third-party module shims, or global types. Do not use declaration files for ordinary app types that can live in `.ts` files.

For libraries:

- Keep exported types stable.
- Avoid exposing internal helper types.
- Test type behavior with compile-time type tests when the public type API is complex.

## 21. TypeScript Anti-patterns

- `any` spread through application code.
- Type assertions instead of validation.
- Optional fields representing mutually exclusive states.
- Overly generic helper types.
- Deep conditional types in app code.
- Enums used where literal unions are simpler.
- Public APIs that require manual generic parameters.
- Duplicated DTO and domain types with no meaningful difference.
- Huge global type files.
- Barrel files that cause circular dependencies.
- Branded types for every primitive.
- `Partial<T>` accepted in places that actually require specific fields.
- Ignoring `undefined` instead of modeling absence intentionally.

## 22. Pattern Selection Guide

| Need | Recommended TypeScript Pattern |
| --- | --- |
| Untrusted input | `unknown` + runtime validation |
| Finite state variants | Discriminated union |
| Expected recoverable failure | Result type |
| Preserve caller-provided type relation | Generic |
| Prevent ID mixups | Branded type |
| Constrain string families | Template literal type |
| Shape transformation | Utility type |
| Complex reusable type transformation | Conditional / mapped type |
| Branch on error type | Error union or custom error class |
| External payload differs from internal model | DTO + mapper |
| Simple app object | Explicit type, no advanced pattern |

## 23. Code Examples

### Exhaustive Check

```ts
export type SubscriptionStatus =
  | { type: "trial"; endsAt: Date }
  | { type: "active"; renewsAt: Date }
  | { type: "cancelled"; cancelledAt: Date };

const assertNever = (value: never): never => {
  throw new Error(`Unhandled case: ${JSON.stringify(value)}`);
};

export const getSubscriptionLabel = (status: SubscriptionStatus): string => {
  switch (status.type) {
    case "trial":
      return `Trial ends ${status.endsAt.toISOString()}`;
    case "active":
      return `Renews ${status.renewsAt.toISOString()}`;
    case "cancelled":
      return `Cancelled ${status.cancelledAt.toISOString()}`;
    default:
      return assertNever(status);
  }
};
```

### Runtime Validation Boundary

```ts
export type User = {
  id: string;
  email: string;
};

export const isUser = (value: unknown): value is User => {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  const candidate = value as Record<string, unknown>;

  return typeof candidate.id === "string" && typeof candidate.email === "string";
};

export const parseUser = (value: unknown): User => {
  if (!isUser(value)) {
    throw new Error("Invalid user payload.");
  }

  return value;
};
```

### Typed Strategy

```ts
export type ShippingMethod = "standard" | "express";

export type ShippingInput = {
  subtotal: number;
  weightKg: number;
};

export type ShippingStrategy = {
  calculate: (input: ShippingInput) => number;
};

const shippingStrategies: Record<ShippingMethod, ShippingStrategy> = {
  standard: {
    calculate: (input) => input.weightKg * 2,
  },
  express: {
    calculate: (input) => input.weightKg * 4 + 10,
  },
};

export const calculateShipping = (
  method: ShippingMethod,
  input: ShippingInput,
): number => {
  return shippingStrategies[method].calculate(input);
};
```

### Branded IDs

```ts
type Brand<TValue, TBrand extends string> = TValue & { readonly __brand: TBrand };

export type UserId = Brand<string, "UserId">;

export const createUserId = (value: string): UserId => {
  if (!value.startsWith("user_")) {
    throw new Error("Invalid user id.");
  }

  return value as UserId;
};
```

## 24. Testing Guidance

- Type-level behavior should usually be tested through normal compile checks.
- Add type tests for exported library utilities with complex generics.
- Test runtime validators with valid, invalid, missing, and extra-field payloads.
- Test discriminated-union behavior through public functions.
- Test API mappers with real-looking DTOs.
- Avoid testing private helper types.
- Do not rely on TypeScript types as a substitute for runtime tests at I/O boundaries.
