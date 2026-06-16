# React Reference

Use this reference for plain React web apps, including Vite and React Router projects.

## Stack

- React
- TypeScript
- Vite when applicable
- React Router when applicable
- Tailwind CSS or the project styling system
- Browser-native APIs and project shared UI

## Routing

Use `src/app/` for route-level files.

For Vite / React Router:

```txt
src/app/
  router.tsx
  providers.tsx
  routes/
```

Route files should compose feature UI and route-level providers. Keep large business logic inside `src/features/<feature>/`.

## UI

Use web primitives and project-approved web UI components.

Do not use React Native-only primitives unless the project is explicitly configured for React Native Web.

Good:

```tsx
const LoginPage = () => {
  return (
    <main className="min-h-screen bg-white px-4">
      <LoginForm />
    </main>
  );
};

export default LoginPage;
```

## Styling

Prefer Tailwind CSS or the project-approved styling approach.

Good:

```tsx
<div className="flex min-h-screen bg-white px-4">
  <h1 className="text-lg font-semibold text-black">Title</h1>
</div>
```

Avoid mixing multiple styling systems without a clear project reason.

## Browser Boundaries

Keep direct browser API usage close to the component or hook that needs it.

Use explicit guards when code may run during SSR, tests, or prerendering:

```ts
if (typeof window === "undefined") {
  return;
}
```
