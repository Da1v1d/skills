# Queries Reference

Read-side server state: `useQuery`, suspense, parallel/dependent/paginated/infinite queries, prefetching, and SSR.

All query functions call named feature request functions (`getTodos`, `getTodo`, …) and reference query-key constants (`TODOS_KEY`, `todoKey`, …). See the main `SKILL.md` for the [API Layer](../SKILL.md#api-layer) and [Query Keys](../SKILL.md#query-keys) conventions.

## Query Functions

The `queryFn` receives a `QueryFunctionContext`. Forward its `signal` to the request function so TanStack Query can cancel in-flight requests on unmount or key change:

```tsx
import { getTodo, getTodos } from "@/features/todos/api/todos.api";
import { TODOS_KEY, todoKey } from "@/features/todos/config/todos.constants";

// Pass the id through, forward the signal for automatic cancellation
useQuery({
  queryKey: todoKey(todoId),
  queryFn: ({ signal }) => getTodo(todoId, signal),
});

useQuery({
  queryKey: TODOS_KEY,
  queryFn: ({ signal }) => getTodos(signal),
});
```

The request functions own the URL, headers, and error handling — the `queryFn` only wires the cache key to the service call.

## Queries (useQuery)

### Basic Usage

```tsx
import { useQuery } from "@tanstack/react-query";
import { getTodos } from "@/features/todos/api/todos.api";
import { TODOS_KEY } from "@/features/todos/config/todos.constants";

const Todos = () => {
  const {
    data,
    error,
    isLoading, // First load, no data yet
    isFetching, // Any fetch in progress (including background)
    isError,
    isSuccess,
    isPending, // No data yet (same as isLoading in most cases)
    status, // 'pending' | 'error' | 'success'
    fetchStatus, // 'fetching' | 'paused' | 'idle'
    refetch,
    isStale,
    isPlaceholderData,
    dataUpdatedAt,
    errorUpdatedAt,
  } = useQuery({
    queryKey: TODOS_KEY,
    queryFn: ({ signal }) => getTodos(signal),
  });

  if (isLoading) return <Spinner />;
  if (isError) return <Error message={error.message} />;
  return <TodoList todos={data} />;
};
```

### Query Options

```tsx
useQuery({
  queryKey: TODOS_KEY,
  queryFn: ({ signal }) => getTodos(signal),

  // Freshness
  staleTime: 5000, // ms data stays fresh (default: 0)
  gcTime: 300000, // ms unused data stays in cache (default: 5 min)

  // Refetching
  refetchInterval: 10000, // Poll every 10s
  refetchIntervalInBackground: false, // Don't poll when tab hidden
  refetchOnMount: true, // Refetch on component mount if stale
  refetchOnWindowFocus: true, // Refetch on window focus if stale
  refetchOnReconnect: true, // Refetch on network reconnect

  // Retry
  retry: 3, // Number of retries (or function)
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),

  // Conditional
  enabled: !!userId, // Only run when truthy

  // Initial/placeholder data
  initialData: () => cachedData,
  initialDataUpdatedAt: Date.now() - 10000,
  placeholderData: (previousData) => previousData, // keepPreviousData pattern
  placeholderData: initialTodos,

  // Transform
  select: (data) => data.filter((todo) => !todo.done),

  // Structural sharing (default: true)
  structuralSharing: true,

  // Network mode
  networkMode: "online", // 'online' | 'always' | 'offlineFirst'

  // Meta (accessible in query function context)
  meta: { purpose: "user-facing" },
});
```

## Infinite Queries

The request function takes the cursor; the `queryFn` forwards `pageParam` to it:

```tsx
import { useInfiniteQuery } from "@tanstack/react-query";
import { getProjectsPage } from "@/features/projects/api/projects.api";
import { PROJECTS_KEY } from "@/features/projects/config/projects.constants";

const InfiniteList = () => {
  const {
    data,
    fetchNextPage,
    fetchPreviousPage,
    hasNextPage,
    hasPreviousPage,
    isFetchingNextPage,
    isFetchingPreviousPage,
  } = useInfiniteQuery({
    queryKey: PROJECTS_KEY,
    queryFn: ({ pageParam, signal }) => getProjectsPage(pageParam, signal),
    initialPageParam: 0,
    getNextPageParam: (lastPage, allPages, lastPageParam) => {
      return lastPage.nextCursor ?? undefined; // undefined = no more pages
    },
    getPreviousPageParam: (firstPage, allPages, firstPageParam) => {
      return firstPage.prevCursor ?? undefined;
    },
    maxPages: 3, // Keep max 3 pages in cache (for performance)
  });

  return (
    <div>
      {data.pages.map((page) =>
        page.items.map((item) => <Item key={item.id} item={item} />),
      )}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? "Loading..."
          : hasNextPage
            ? "Load More"
            : "No more"}
      </button>
    </div>
  );
};
```

```ts
// src/features/projects/api/projects.api.ts
export const getProjectsPage = (cursor: number, signal?: AbortSignal) =>
  httpClient.get<ProjectsPage>(`/projects?cursor=${cursor}`, { signal });
```

## Parallel Queries

```tsx
// Multiple independent queries run in parallel automatically
const Dashboard = () => {
  const usersQuery = useQuery({
    queryKey: USERS_KEY,
    queryFn: ({ signal }) => getUsers(signal),
  });
  const projectsQuery = useQuery({
    queryKey: PROJECTS_KEY,
    queryFn: ({ signal }) => getProjects(signal),
  });

  // Both fetch simultaneously
};

// Dynamic parallel queries with useQueries
const UserProjects = ({ userIds }) => {
  const queries = useQueries({
    queries: userIds.map((id) => ({
      queryKey: userKey(id),
      queryFn: () => getUser(id),
    })),
    combine: (results) => ({
      data: results.map((r) => r.data),
      pending: results.some((r) => r.isPending),
    }),
  });
};
```

## Dependent Queries

```tsx
// Sequential queries using enabled
const UserPosts = ({ userId }) => {
  const userQuery = useQuery({
    queryKey: userKey(userId),
    queryFn: () => getUser(userId),
  });

  const postsQuery = useQuery({
    queryKey: postsKey(userId),
    queryFn: () => getPostsByUser(userId),
    enabled: !!userQuery.data, // Only run when user is loaded
  });
};
```

## Paginated Queries

```tsx
const PaginatedList = () => {
  const [page, setPage] = useState(1);

  const { data, isPlaceholderData } = useQuery({
    queryKey: todoPageKey(page),
    queryFn: () => getTodosByPage(page),
    placeholderData: (previousData) => previousData, // Keep showing old data
  });

  return (
    <div style={{ opacity: isPlaceholderData ? 0.5 : 1 }}>
      {data.items.map((item) => (
        <Item key={item.id} item={item} />
      ))}
      <button
        onClick={() => setPage((p) => p + 1)}
        disabled={isPlaceholderData || !data.hasMore}
      >
        Next
      </button>
    </div>
  );
};
```

## Suspense Integration

```tsx
import {
  useSuspenseQuery,
  useSuspenseInfiniteQuery,
} from "@tanstack/react-query";

// Component will suspend until data is loaded
const TodoList = () => {
  const { data } = useSuspenseQuery({
    queryKey: TODOS_KEY,
    queryFn: ({ signal }) => getTodos(signal),
  });
  // data is guaranteed to be defined here
  return (
    <ul>
      {data.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
};

// Wrap with Suspense boundary
const App = () => {
  return (
    <ErrorBoundary fallback={<Error />}>
      <Suspense fallback={<Loading />}>
        <TodoList />
      </Suspense>
    </ErrorBoundary>
  );
};

// Multiple suspense queries (fetch in parallel)
const Dashboard = () => {
  const [{ data: users }, { data: projects }] = useSuspenseQueries({
    queries: [
      { queryKey: USERS_KEY, queryFn: ({ signal }) => getUsers(signal) },
      { queryKey: PROJECTS_KEY, queryFn: ({ signal }) => getProjects(signal) },
    ],
  });
};
```

## Prefetching

```tsx
const queryClient = useQueryClient();

// Prefetch on hover
const TodoLink = ({ todoId }) => {
  const prefetch = () => {
    queryClient.prefetchQuery({
      queryKey: todoKey(todoId),
      queryFn: ({ signal }) => getTodo(todoId, signal),
      staleTime: 5000, // Only prefetch if data older than 5s
    });
  };

  return (
    <Link to={`/todos/${todoId}`} onMouseEnter={prefetch}>
      Todo {todoId}
    </Link>
  );
};

// Prefetch in route loader (TanStack Router integration)
export const Route = createFileRoute("/todos/$todoId")({
  loader: ({ context: { queryClient }, params: { todoId } }) =>
    queryClient.ensureQueryData(todoQueryOptions(todoId)),
});

// Prefetch infinite queries
queryClient.prefetchInfiniteQuery({
  queryKey: PROJECTS_KEY,
  queryFn: ({ pageParam, signal }) => getProjectsPage(pageParam, signal),
  initialPageParam: 0,
  pages: 3, // Prefetch first 3 pages
});
```

## SSR & Hydration

### Server-Side Prefetching

```tsx
// Server component or loader
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from "@tanstack/react-query";

const getServerSideProps = async () => {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery({
    queryKey: TODOS_KEY,
    queryFn: ({ signal }) => getTodos(signal),
  });

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
};

const Page = ({ dehydratedState }) => {
  return (
    <HydrationBoundary state={dehydratedState}>
      <Todos />
    </HydrationBoundary>
  );
};
```

### Streaming SSR (React Server Components)

```tsx
import { dehydrate, HydrationBoundary } from "@tanstack/react-query";
import { makeQueryClient } from "./query-client";

const Page = async () => {
  const queryClient = makeQueryClient();

  // Prefetch on server
  await queryClient.prefetchQuery({
    queryKey: TODOS_KEY,
    queryFn: ({ signal }) => getTodos(signal),
  });

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <TodoList />
    </HydrationBoundary>
  );
};

export default Page;
```

## Query Cancellation

The `signal` flows from `queryFn` into the request function and onward to `fetch`, so unmount or key changes abort the request automatically:

```tsx
useQuery({
  queryKey: TODOS_KEY,
  // getTodos forwards the signal to httpClient.get → fetch
  queryFn: ({ signal }) => getTodos(signal),
});

// Manual cancellation
queryClient.cancelQueries({ queryKey: TODOS_KEY });
```

## Network Mode

```tsx
useQuery({
  queryKey: TODOS_KEY,
  queryFn: ({ signal }) => getTodos(signal),
  // 'online' (default): only fetch when online
  // 'always': always fetch (useful for local-first)
  // 'offlineFirst': try fetch, use cache if offline
  networkMode: "offlineFirst",
});
```

## Window Focus Refetching

```tsx
// Disable globally
const queryClient = new QueryClient({
  defaultOptions: {
    queries: { refetchOnWindowFocus: false },
  },
});

// Custom focus manager
import { focusManager } from "@tanstack/react-query";

// For React Native
focusManager.setEventListener((handleFocus) => {
  const subscription = AppState.addEventListener("change", (state) => {
    handleFocus(state === "active");
  });
  return () => subscription.remove();
});
```

## TypeScript Patterns

### Typing Query Functions

Type flows from the request function's return type, so `useQuery` infers `data` with no extra annotations:

```tsx
// src/features/todos/lib/todos.types.ts
export interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

// src/features/todos/api/todos.api.ts
export const getTodos = (signal?: AbortSignal) =>
  httpClient.get<Todo[]>("/todos", { signal });

// Component — data: Todo[] | undefined, fully inferred
const { data } = useQuery({
  queryKey: TODOS_KEY,
  queryFn: ({ signal }) => getTodos(signal),
});

// With select
const { data } = useQuery({
  queryKey: TODOS_KEY,
  queryFn: ({ signal }) => getTodos(signal),
  select: (data): string[] => data.map((t) => t.title),
});
// data: string[] | undefined
```

### Typing Errors

```tsx
// Default error type is Error
const { error } = useQuery<Todo[], AxiosError>({
  queryKey: TODOS_KEY,
  queryFn: ({ signal }) => getTodos(signal),
});

// Or register globally
declare module "@tanstack/react-query" {
  interface Register {
    defaultError: AxiosError;
  }
}
```

### Query Options Pattern (Recommended)

Define these in the feature API module so the key, request function, and options live together:

```tsx
import { queryOptions, infiniteQueryOptions } from "@tanstack/react-query";
import { getTodos, getTodo } from "@/features/todos/api/todos.api";
import { TODOS_KEY, todoKey } from "@/features/todos/config/todos.constants";

export const todosOptions = queryOptions({
  queryKey: TODOS_KEY,
  queryFn: ({ signal }) => getTodos(signal),
  staleTime: 5000,
});

export const todoOptions = (id: string) =>
  queryOptions({
    queryKey: todoKey(id),
    queryFn: ({ signal }) => getTodo(id, signal),
    enabled: !!id,
  });

// Full type inference everywhere
const { data } = useQuery(todosOptions);
const { data } = useSuspenseQuery(todoOptions("123"));
await queryClient.ensureQueryData(todosOptions);
queryClient.invalidateQueries({ queryKey: todosOptions.queryKey });
```

## Persistence

```tsx
import { persistQueryClient } from "@tanstack/react-query-persist-client";
import { createSyncStoragePersister } from "@tanstack/query-sync-storage-persister";

const persister = createSyncStoragePersister({
  storage: window.localStorage,
});

persistQueryClient({
  queryClient,
  persister,
  maxAge: 1000 * 60 * 60 * 24, // 24 hours
});
```

## Testing Queries

```tsx
import { renderHook, waitFor } from "@testing-library/react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false, // Don't retry in tests
        gcTime: Infinity, // Prevent garbage collection during tests
      },
    },
  });
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};

test("fetches todos", async () => {
  const { result } = renderHook(
    () =>
      useQuery({
        queryKey: TODOS_KEY,
        queryFn: ({ signal }) => getTodos(signal),
      }),
    { wrapper: createWrapper() },
  );

  await waitFor(() => expect(result.current.isSuccess).toBe(true));
  expect(result.current.data).toEqual(expectedTodos);
});

// Mock with setQueryData for component tests
test("renders todos", () => {
  const queryClient = new QueryClient();
  queryClient.setQueryData(TODOS_KEY, mockTodos);

  render(
    <QueryClientProvider client={queryClient}>
      <TodoList />
    </QueryClientProvider>,
  );

  expect(screen.getByText("Todo 1")).toBeInTheDocument();
});
```
