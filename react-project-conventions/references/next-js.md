# Next.js Reference

Use this reference for Next.js projects.

## Stack

- Next.js App Router
- React
- TypeScript
- Tailwind CSS or the project styling system
- Server Components by default
- Server Actions and Route Handlers when applicable

## Routing

Use `src/app/` for Next.js App Router files.

```txt
src/app/
  layout.tsx
  page.tsx
  dashboard/
    page.tsx
  api/
    health/
      route.ts
```

Route files should compose feature UI and route-level data loading. Keep reusable feature logic inside `src/features/<feature>/`.

Use `src/features/<feature>/ui/pages/` for feature page-level views that are composed by Next.js route files.

## Server And Client Components

Prefer Server Components by default.

Use `"use client"` only when a component needs:

- React state or effects
- browser APIs
- event handlers
- client-only libraries

Keep client components small and push non-interactive rendering back to server components.

Good:

```tsx
import LoginForm from "@/features/auth/ui/login-form";

const LoginPage = () => {
  return <LoginForm />;
};

export default LoginPage;
```

## API And Server Logic

Use Route Handlers for HTTP endpoints:

```txt
src/app/api/accounts/route.ts
```

Put shared backend clients, auth helpers, and infrastructure in `src/shared/services/`.

Feature-specific server logic should stay in the feature:

```txt
src/features/accounts/api/accounts.api.ts
src/features/accounts/model/actions/create-account.action.ts
```

## Styling

Prefer Tailwind CSS or the project-approved styling approach.

Good:

```tsx
<main className="min-h-screen bg-white px-4">
  <h1 className="text-lg font-semibold text-black">Dashboard</h1>
</main>
```

Avoid mixing multiple styling systems without a clear project reason.
