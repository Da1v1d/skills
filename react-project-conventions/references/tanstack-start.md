# TanStack Start Reference

Use this reference for TanStack Start projects.

## Stack

- TanStack Start
- TanStack Router
- React
- TypeScript
- Tailwind CSS or the project styling system
- Server functions when applicable

## Routing

Use the TanStack Start route tree convention already present in the repository.

Common layouts:

```txt
src/routes/
  __root.tsx
  index.tsx
  dashboard.tsx
```

or, if the repository keeps route-level composition under `src/app/`:

```txt
src/app/
  router.tsx
  routes/
```

Follow the existing project route location. Do not move an established TanStack Start project to `src/app/` only for consistency with other React stacks.

Route files should compose features, loaders, and route metadata. Keep reusable feature logic inside `src/features/<feature>/`.

Use `src/features/<feature>/ui/pages/` for feature page-level views that are composed by TanStack Start route files.

## Loaders And Server Functions

Keep route loaders thin.

Good:

```tsx
export const Route = createFileRoute("/accounts")({
  loader: () => accountsQueryOptions(),
  component: AccountsPage,
});
```

Put feature query options, mutations, and server functions inside the feature when they are feature-specific:

```txt
src/features/accounts/api/accounts.service.ts
src/features/accounts/model/queries/accounts.queries.ts
src/features/accounts/model/actions/create-account.action.ts
```

Put cross-feature infrastructure in `src/shared/services/`.

## UI

Use web primitives and project-approved web UI components.

Do not use React Native-only primitives unless the project is explicitly configured for React Native Web.

## Styling

Prefer Tailwind CSS or the project-approved styling approach.

Good:

```tsx
<main className="min-h-screen bg-white px-4">
  <h1 className="text-lg font-semibold text-black">Accounts</h1>
</main>
```

Avoid mixing multiple styling systems without a clear project reason.
