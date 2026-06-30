---
name: tanstack-query
description: Powerful asynchronous state management, server-state utilities, and data fetching for TS/JS, React, Vue, Solid, Svelte & Angular.
---

## Overview

TanStack Query (formerly React Query) manages server state - data that lives on the server and needs to be fetched, cached, synchronized, and updated. It provides automatic caching, background refetching, stale-while-revalidate patterns, pagination, infinite scrolling, and optimistic updates out of the box.

**Package:** `@tanstack/react-query`
**Devtools:** `@tanstack/react-query-devtools`
**Current Version:** v5

## How To Use This Skill

Apply the shared conventions in this file (setup, query keys, the API layer, and `queryOptions`), then read the reference that matches what you are building:

- Reading server state (queries, suspense, pagination, infinite, prefetching, SSR): `references/queries.md`
- Writing server state (mutations, optimistic updates, invalidation, cache writes): `references/mutations.md`

## API Layer

**Never call `fetch` inline inside a `queryFn` or `mutationFn`.** Inline `fetch("/api/todos")` duplicates base URL, headers, error handling, and JSON parsing across every hook, and couples your cache layer to transport details. Instead, point query/mutation functions at named request functions (`getTodos`, `getTodo`, `createTodo`, or semi `TodosApi.getAll`, `TodosApi.getById`, `TodosApi.create` depends on project architecture/structure, just remember that you need to keep project conventions) defined in your project's API layer.

```tsx
useQuery({
  queryKey: TODOS_KEY,
  queryFn: ({ signal }) => getTodos(signal),
});

useQuery({
  queryKey: todoKey(todoId),
  queryFn: ({ signal }) => getTodo(todoId, signal),
});

useMutation({ mutationFn: createTodo });
```

Throughout `references/queries.md` and `references/mutations.md`, `getTodos`, `getTodo`, `createTodo`, etc. always refer to these feature API request functions. Reads forward the `signal` (`getTodos(signal)`) so TanStack Query can cancel them.

## Core Concepts

### Query Keys

Query keys uniquely identify cached data. They must be serializable arrays:

Keys are serializable arrays, defined once as constants (see the next section) and referenced everywhere — never as inline string literals. The comment after each line shows the shape the constant resolves to:

```tsx
// Simple key → ["todos"]
useQuery({ queryKey: TODOS_KEY, queryFn: ({ signal }) => getTodos(signal) });

// With variables (dependency array pattern) → ["todos", { status, page }]
useQuery({ queryKey: todoListKey({ status, page }), queryFn: getTodos });

// Hierarchical keys for invalidation → ["todos", todoId]
useQuery({ queryKey: todoKey(todoId), queryFn: () => getTodo(todoId) });
// → ["todos", todoId, "comments"]
useQuery({
  queryKey: todoCommentsKey(todoId),
  queryFn: () => getTodoComments(todoId),
});

// Invalidation matches prefixes:
// queryClient.invalidateQueries({ queryKey: TODOS_KEY })
// ^ Invalidates ALL queries starting with 'todos'
```

### Query Keys Live in a Constants File (No Raw Strings)

**Never use raw string literals as query keys inline.** Scattering `["todos"]` across files invites typos, makes refactors error-prone, and leaves no single source of truth. Define keys once as exported constants in a dedicated `*.constants.ts` file and reference them everywhere.

```tsx
// <feature>.constants.ts, constants.ts, query-constants.tsx — single source of truth
export const TODOS_KEY = ["todos"] as const;
export const todoKey = (id: string) => [...TODOS_KEY, id] as const;
export const todoCommentsKey = (id: string) =>
  [...todoKey(id), "comments"] as const;
export const todoListKey = (filters: { status: string; page: number }) =>
  [...TODOS_KEY, filters] as const;
export const todoPageKey = (page: number) => [...TODOS_KEY, page] as const;

// Every other feature follows the same pattern in its own constants file
export const PROJECTS_KEY = ["projects"] as const;
export const USERS_KEY = ["users"] as const;
export const userKey = (id: string) => [...USERS_KEY, id] as const;
export const postsKey = (userId: string) => ["posts", userId] as const;
export const ADD_TODO_KEY = ["addTodo"] as const; // mutation key
```

```tsx
// Usage — always reference the constant, never a string literal
import { TODOS_KEY, todoKey } from '<feature>.constants' or  'constants', or 'query-constants'

useQuery({ queryKey: TODOS_KEY, queryFn: ({ signal }) => getTodos(signal) })
useQuery({ queryKey: todoKey(todoId), queryFn: () => getTodo(todoId) })

// Invalidation stays consistent and refactor-safe
queryClient.invalidateQueries({ queryKey: TODOS_KEY })       // all todos
queryClient.removeQueries({ queryKey: todoKey(todoId) })     // one todo
```

The `as const` assertions preserve literal tuple types for full inference, and composing from `TODOS_KEY` keeps prefix-based invalidation working. Pair this with the `queryOptions` helper below so the key and `queryFn` live together.

### queryOptions Helper

Wrap each query's key, `queryFn`, and options in a `queryOptions` factory, colocated with the feature's request functions in `src/features/<feature>/api/`. This yields one reusable, fully-typed config shared across `useQuery`, `useSuspenseQuery`, `prefetchQuery`, and `ensureQueryData`. See [Query Options Pattern](references/queries.md#query-options-pattern-recommended) in the queries reference for the full example.

## Best Practices

1. **Define request functions in the feature API module** (`getTodos`, `createTodo`, …) that compose a shared HTTP client — never call `fetch` inline in `queryFn`/`mutationFn`
2. **Use `queryOptions` helper** for type-safe, reusable query configurations
3. **Never inline raw string query keys** - define them as `as const` constants in a `*.constants.ts` file and import them
4. **Structure query keys hierarchically** for granular invalidation
5. **Set appropriate `staleTime`** - 0 means always refetch on mount (default), increase for less dynamic data
6. **Use `placeholderData`** (not `initialData`) for keeping previous page data during pagination
7. **Prefer `useSuspenseQuery`** when using Suspense boundaries for cleaner component code
8. **Use `enabled`** for dependent queries, not conditional hook calls
9. **Always invalidate after mutations** - don't rely solely on optimistic updates
10. **Cancel queries in `onMutate`** before optimistic updates to prevent race conditions
11. **Use `ensureQueryData`** in route loaders instead of `prefetchQuery` for immediate access
12. **Set `retry: false` in tests** to avoid timeout issues
13. **Don't destructure the query result** if you need to pass it around (breaks reactivity)
14. **Use `select`** for derived data instead of transforming in the component
15. **Keep query functions pure** - they should only fetch, not cause side effects
16. **Use `gcTime: Infinity`** in tests to prevent cache cleanup during assertions

## Common Pitfalls

- Calling `fetch` inline instead of a named request function (duplicates base URL, headers, error handling)
- Using `initialData` when you mean `placeholderData` (initialData counts as "fresh" data)
- Not providing `initialPageParam` for infinite queries (required in v5)
- Calling hooks conditionally (violates React rules)
- Not cancelling queries before optimistic updates (race conditions)
- Setting `staleTime` higher than `gcTime` (data gets garbage collected while "fresh")
- Forgetting to wrap tests with `QueryClientProvider`
- Using same `QueryClient` instance across tests (shared state)
- Not awaiting `invalidateQueries` in mutation callbacks when order matters
