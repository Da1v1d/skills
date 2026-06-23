# Mutations Reference

Write-side server state: `useMutation`, optimistic updates, invalidation, and direct cache writes via the `QueryClient`.

All mutation functions call named feature request functions (`createTodo`, `updateTodo`, `deleteTodo`, …) and reference query-key constants (`TODOS_KEY`, `todoKey`, …). See the main `SKILL.md` for the [API Layer](../SKILL.md#api-layer) and [Query Keys](../SKILL.md#query-keys) conventions.

## Mutations (useMutation)

### Basic Usage

The `mutationFn` is just the feature request function — `createTodo` owns the URL, method, headers, and JSON handling:

```tsx
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { createTodo } from "@/features/todos/api/todos.api";
import { TODOS_KEY } from "@/features/todos/config/todos.constants";

const AddTodo = () => {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: createTodo, // (payload: CreateTodoPayload) => Promise<Todo>
    // Lifecycle callbacks
    onMutate: async (variables) => {
      // Called before mutationFn
      // Good for optimistic updates
      return { previousTodos }; // context for onError
    },
    onSuccess: (data, variables, context) => {
      // Invalidate related queries
      queryClient.invalidateQueries({ queryKey: TODOS_KEY });
    },
    onError: (error, variables, context) => {
      // Rollback optimistic updates
      queryClient.setQueryData(TODOS_KEY, context.previousTodos);
    },
    onSettled: (data, error, variables, context) => {
      // Always runs (success or error)
      queryClient.invalidateQueries({ queryKey: TODOS_KEY });
    },
  });

  return (
    <button
      onClick={() => mutation.mutate({ title: "New Todo" })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? "Adding..." : "Add Todo"}
    </button>
  );
};
```

When a mutation needs more than the raw request call (e.g. an id plus payload), wrap it inline:

```tsx
useMutation({
  mutationFn: ({ id, ...payload }: UpdateTodoArgs) => updateTodo(id, payload),
});

useMutation({ mutationFn: deleteTodo }); // (id: string) => Promise<void>
```

### Mutation State

```tsx
const {
  mutate,         // Fire-and-forget
  mutateAsync,    // Returns promise
  isPending,      // Mutation in progress
  isError,
  isSuccess,
  isIdle,         // Not yet fired
  data,           // Success response
  error,          // Error object
  reset,          // Reset state to idle
  variables,      // Variables passed to mutate
  status,         // 'idle' | 'pending' | 'error' | 'success'
} = useMutation({ ... })
```

## Optimistic Updates

```tsx
const mutation = useMutation({
  mutationFn: ({ id, ...payload }: UpdateTodoArgs) => updateTodo(id, payload),
  onMutate: async (newTodo) => {
    // 1. Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: todoKey(newTodo.id) });

    // 2. Snapshot previous value
    const previousTodo = queryClient.getQueryData(todoKey(newTodo.id));

    // 3. Optimistically update
    queryClient.setQueryData(todoKey(newTodo.id), newTodo);

    // 4. Return context for rollback
    return { previousTodo };
  },
  onError: (err, newTodo, context) => {
    // Rollback on error
    queryClient.setQueryData(todoKey(newTodo.id), context.previousTodo);
  },
  onSettled: () => {
    // Always refetch to sync with server
    queryClient.invalidateQueries({ queryKey: TODOS_KEY });
  },
});
```

### Optimistic Updates on Lists

```tsx
onMutate: async (newTodo) => {
  await queryClient.cancelQueries({ queryKey: TODOS_KEY })
  const previousTodos = queryClient.getQueryData(TODOS_KEY)

  queryClient.setQueryData(TODOS_KEY, (old) => [...old, newTodo])

  return { previousTodos }
},
onError: (err, newTodo, context) => {
  queryClient.setQueryData(TODOS_KEY, context.previousTodos)
},
```

## Query Invalidation

Invalidation is the standard way to sync the cache after a mutation — mark queries stale so active ones refetch:

```tsx
const queryClient = useQueryClient();

// Invalidate all queries
queryClient.invalidateQueries();

// Invalidate by prefix
queryClient.invalidateQueries({ queryKey: TODOS_KEY });

// Invalidate exact match
queryClient.invalidateQueries({ queryKey: todoKey(1), exact: true });

// Invalidate with predicate
queryClient.invalidateQueries({
  predicate: (query) =>
    query.queryKey[0] === TODOS_KEY[0] && query.queryKey[1]?.status === "done",
});

// Invalidate and refetch immediately
queryClient.refetchQueries({ queryKey: TODOS_KEY });

// Remove from cache entirely
queryClient.removeQueries({ queryKey: todoKey(1) });

// Reset to initial state
queryClient.resetQueries({ queryKey: TODOS_KEY });
```

## QueryClient Cache API

Direct cache reads and writes — used inside mutation callbacks for optimistic updates and rollback:

```tsx
const queryClient = useQueryClient();

// Get cached data
queryClient.getQueryData(TODOS_KEY);

// Set cached data
queryClient.setQueryData(TODOS_KEY, updatedTodos);
queryClient.setQueryData(TODOS_KEY, (old) => [...old, newTodo]);

// Get query state
queryClient.getQueryState(TODOS_KEY);

// Check if fetching / mutating
queryClient.isFetching({ queryKey: TODOS_KEY });
queryClient.isMutating();

// Cancel queries (before optimistic updates)
queryClient.cancelQueries({ queryKey: TODOS_KEY });

// Invalidate (marks stale, refetches active)
queryClient.invalidateQueries({ queryKey: TODOS_KEY });

// Refetch (force refetch even if fresh)
queryClient.refetchQueries({ queryKey: TODOS_KEY });

// Remove from cache
queryClient.removeQueries({ queryKey: TODOS_KEY });

// Reset to initial state
queryClient.resetQueries({ queryKey: TODOS_KEY });

// Clear entire cache
queryClient.clear();

// Mutation defaults — register the request function as the default mutationFn
queryClient.setMutationDefaults(ADD_TODO_KEY, { mutationFn: createTodo });
```

## Testing Mutations

```tsx
import { renderHook, waitFor } from "@testing-library/react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      mutations: { retry: false },
      queries: { retry: false, gcTime: Infinity },
    },
  });
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};

test("creates a todo", async () => {
  const { result } = renderHook(() => useMutation({ mutationFn: createTodo }), {
    wrapper: createWrapper(),
  });

  result.current.mutate({ title: "New Todo" });

  await waitFor(() => expect(result.current.isSuccess).toBe(true));
  expect(result.current.data).toMatchObject({ title: "New Todo" });
});
```

Mock the feature request function (`createTodo`) rather than `fetch`, since transport lives in the shared HTTP client.
