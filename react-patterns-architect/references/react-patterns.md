# React Patterns Architect Reference

## Table Of Contents

1. Core Decision Rules
2. Compound Components
3. Render Props / Children As Function
4. Controlled / Uncontrolled Components
5. State Reducer Pattern
6. Prop Getters
7. Provider Pattern
8. Headless Components
9. Slot Pattern
10. Polymorphic Components
11. Custom Hooks
12. Hook Factories
13. Controller Components
14. Container / Presentational Components
15. Adapter / Facade Patterns
16. Strategy Pattern
17. Higher-Order Components
18. Dependency Injection
19. Declarative APIs, Imperative Handles, And Ref Forwarding
20. Form Architecture
21. State Architecture
22. Context Architecture
23. Design-System Architecture
24. Feature Architecture
25. React Anti-patterns
26. Pattern Selection Guide
27. Code Examples
28. Testing Guidance

## 1. Core Decision Rules

Act like a senior React architect:

- Start with the simplest component design that satisfies the current requirement.
- Add abstraction only when it removes real duplication, stabilizes a public API, improves accessibility, or creates a clear reuse boundary.
- Prefer composition over inheritance.
- Prefer explicit component APIs over hidden dependencies.
- Prefer feature-local code until reuse across features is real.
- Keep reusable UI primitives free of feature-specific business rules.
- Keep server state, client state, form state, URL state, and derived state separate.
- Warn when a pattern is popular but not justified by the problem.

When choosing a pattern, always explain:

- When the pattern is useful.
- When the pattern is harmful.
- The tradeoffs.
- The simplest acceptable solution.
- The scalable solution.
- The concrete code shape.

Use these project conventions in examples:

- Use TypeScript.
- Use Bun in commands, such as `bun run test`.
- Use Feature-Sliced inspired placement under `src/`.
- Use `@/` alias imports for project files.
- Use kebab-case filenames.
- Use arrow functions assigned to `const`.
- Use default exports for component files.
- Use explicit prop types.
- Use event handlers with a `handler` suffix.

## 2. Compound Components

### Purpose

Use compound components when one conceptual widget needs multiple coordinated parts and the consumer should control visual structure:

```tsx
<Tabs defaultValue="profile">
  <Tabs.List>
    <Tabs.Trigger value="profile">Profile</Tabs.Trigger>
    <Tabs.Trigger value="billing">Billing</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Content value="profile">Profile content</Tabs.Content>
  <Tabs.Content value="billing">Billing content</Tabs.Content>
</Tabs>
```

### Best Use Cases

- Tabs, accordion, menus, dropdowns, radio groups, segmented controls.
- Design-system components where layout must be flexible but behavior stays consistent.
- Accessibility-heavy widgets requiring coordinated IDs, ARIA attributes, keyboard behavior, and state.
- APIs where named subcomponents are clearer than large configuration objects.

### Bad Use Cases

- One-off UI with no reuse pressure.
- Static components with no internal coordination.
- Components where children order or implicit context would make behavior hard to understand.
- Cases where a small prop API is enough.

### Context-Based Implementation

Use a scoped context owned by the root component. Keep the context value minimal and stable. Split state and actions if consumers read different parts frequently.

### Static Subcomponents

Attach subcomponents to the root for a coherent API:

```tsx
Tabs.List = TabsList;
Tabs.Trigger = TabsTrigger;
Tabs.Content = TabsContent;
```

This is ergonomic, but it adds typing complexity. If the project favors named exports, use named exports instead.

### TypeScript Typing

Use explicit prop types for each part. For static subcomponents, type the root component as an intersection:

```tsx
type TabsComponent = ((props: TabsProps) => ReactElement) & {
  List: typeof TabsList;
  Trigger: typeof TabsTrigger;
  Content: typeof TabsContent;
};
```

Avoid making every part generic unless the generic directly improves safety for consumers.

### Accessibility Concerns

- Use `role="tablist"`, `role="tab"`, and `role="tabpanel"`.
- Link triggers and panels with stable IDs.
- Manage `aria-selected`, `aria-controls`, and `aria-labelledby`.
- Support keyboard navigation with arrow keys, Home, and End when appropriate.
- Keep focus behavior predictable.

### Tradeoffs

| Benefit | Cost |
| --- | --- |
| Great API flexibility | More internal coordination |
| Good fit for design systems | Context can become too implicit |
| Encourages composition | Static subcomponent typing can be verbose |
| Supports accessible coordination | Harder to inspect than plain props |

### Simplest Acceptable Solution

Use a plain component with props when there are only two or three fixed regions.

### Scalable Solution

Use compound components with scoped context, controlled/uncontrolled state, explicit subcomponent props, and accessibility tests.

## 3. Render Props / Children As Function

### Purpose

Use render props to expose internal state and actions while letting the consumer control rendering.

Render prop:

```tsx
<DataLoader render={({ data, loading }) => <UserList data={data} isLoading={loading} />} />
```

Children as function:

```tsx
<DataLoader>
  {({ data, loading }) => <UserList data={data} isLoading={loading} />}
</DataLoader>
```

The difference is API shape. Render props use a named prop; children as function uses `children`. Both pass state to a consumer-provided render function.

### Best Use Cases

- Exposing internal state.
- Reusable behavior with flexible rendering.
- Legacy code where hooks are not practical.
- Controller-style components that coordinate behavior but do not own visuals.

### Bad Use Cases

- Simple static UI.
- Deep nesting that harms readability.
- Performance-sensitive trees without memoization.
- New code where a custom hook would be simpler and equally flexible.

### Tradeoffs

| Benefit | Cost |
| --- | --- |
| Very flexible rendering | Can reduce readability |
| No styling opinion | Can create nested JSX pyramids |
| Easy to expose state/actions | Render function identity can affect memoization |

### Simplest Acceptable Solution

Use a custom hook if the consumer is already a component and only needs behavior.

### Scalable Solution

Use a custom hook for modern React, or a render-prop controller when JSX ownership must remain outside the behavior component.

## 4. Controlled / Uncontrolled Components

### Purpose

Controlled components let the parent own state. Uncontrolled components let the component own state. Hybrid APIs support both.

Controlled:

- Parent owns `value`.
- Parent responds to `onChange`.
- Best for forms, filters, workflows, URL-synced state, and validation.

Uncontrolled:

- Component owns state.
- Parent may provide `defaultValue`.
- Best for isolated UI where the parent does not care about every transition.

Hybrid:

- Accept `value`.
- Accept `defaultValue`.
- Accept `onChange`.
- Use controlled mode when `value` is provided.

### Best Use Cases

- Design-system inputs, accordions, tabs, selects, modals, drawers.
- Components used in both simple screens and complex forms.
- Components that need internal convenience but external control.

### Bad Use Cases

- Hybrid APIs for components that should always be driven by a form library.
- Controlled mode without a clear owner.
- Uncontrolled mode when state must be persisted, validated, synchronized, or shared.

### Example API Design

```tsx
export type AccordionProps = {
  value?: string;
  defaultValue?: string;
  onValueChange?: (value: string) => void;
  children: ReactNode;
};
```

### Tradeoffs

| Mode | Good For | Risk |
| --- | --- | --- |
| Controlled | Predictable parent-owned workflows | Boilerplate |
| Uncontrolled | Simple isolated UI | Harder to coordinate |
| Hybrid | Design-system flexibility | More implementation paths to test |

## 5. State Reducer Pattern

### Purpose

Use the state reducer pattern when a reusable component needs to let consumers intercept or override state transitions.

The component proposes a state change. The consumer can accept it, modify it, or reject it.

### Best For

- Highly reusable components.
- Design systems.
- Components with complex internal state.
- Components that must support product-specific behavior without hardcoding product rules.

### Bad For

- Simple app-level components.
- Components with one or two straightforward state transitions.
- Teams unfamiliar with reducer-style APIs.

### Example Shape

```tsx
type SelectState = {
  isOpen: boolean;
  selectedValue: string | null;
};

type SelectAction =
  | { type: "open" }
  | { type: "close" }
  | { type: "select"; value: string };

type SelectStateReducer = (state: SelectState, action: SelectAction) => SelectState;
```

### Tradeoffs

| Benefit | Cost |
| --- | --- |
| Powerful consumer override | More API surface |
| Avoids feature rules inside primitives | More tests required |
| Scales design-system behavior | Can confuse app developers |

Use this only when override behavior is a real requirement.

## 6. Prop Getters

### Purpose

Use prop getters to merge internal props with user props safely, especially when a component must preserve accessibility attributes and event handlers.

### Best For

- Headless components.
- Accessibility-heavy components.
- Components with strict event handler requirements.
- Hooks that return props for multiple elements.

### Example Shape

```tsx
type TriggerProps = ButtonHTMLAttributes<HTMLButtonElement>;

type GetTriggerProps = (props?: TriggerProps) => TriggerProps;
```

### Bad Use Cases

- Simple styled components.
- Components where normal prop spreading is enough.
- APIs where prop getters hide too much behavior.

### Tradeoffs

| Benefit | Cost |
| --- | --- |
| Safely merges internal and external props | Extra API concept |
| Protects ARIA/event requirements | Can hide event ordering |
| Great for headless hooks | Needs clear documentation |

## 7. Provider Pattern

### Purpose

Use providers to share scoped state or dependencies across a subtree without manually threading props through every layer.

### Good Use

- Feature-level dependencies.
- Auth/session.
- Theme.
- Compound components.
- Local component trees.
- Context as dependency injection for services.

### Bad Use

- Replacing all props with global state.
- Frequently changing values that rerender large trees.
- Hiding dependencies that should be explicit.
- Creating a provider for every small value.

### Context Splitting Guidance

Split contexts when read/write usage differs:

- `StateContext` for values.
- `ActionsContext` for stable callbacks.
- Separate providers for unrelated concerns.

Use memoization for provider values:

```tsx
const value = useMemo<UserContextValue>(() => {
  return { user, isAuthenticated };
}, [user, isAuthenticated]);
```

For high-frequency updates, prefer an external store with selectors or a dedicated state library instead of raw context.

## 8. Headless Components

### Purpose

Use headless components or hooks when behavior should be shared while styling and markup stay under consumer control.

### Best For

- Design systems.
- Custom UI with shared behavior.
- Accessibility-heavy widgets.
- Components that need multiple visual presentations.

### Bad For

- One-off simple UI.
- Teams that need highly opinionated ready-made components.
- Cases where the headless API is harder to understand than duplicated simple markup.

### Example Shape

```tsx
const { getTriggerProps, getContentProps, isOpen } = useHeadlessModal();
```

### Tradeoffs

| Benefit | Cost |
| --- | --- |
| Styling freedom | More work for consumers |
| Reusable behavior | API must protect accessibility |
| Good design-system foundation | More documentation needed |

## 9. Slot Pattern

### Purpose

Use slots when a component has named replaceable parts but still owns the outer structure.

Slots differ from:

- `children`: one flexible insertion point.
- Compound components: consumer controls structure using coordinated subcomponents.
- Slots: component exposes specific replacement points.

### Best For

- Design systems.
- Customizable cards.
- Layout primitives.
- Modals.
- List items.
- Components with predictable structure and replaceable regions.

### Bad For

- Fully freeform layouts where compound components or plain composition are clearer.
- Tiny components where slots create needless API surface.
- Components where every internal part becomes configurable.

### Example Shape

```tsx
type CardSlots = {
  header?: ReactNode;
  media?: ReactNode;
  footer?: ReactNode;
};

type CardProps = {
  title: string;
  slots?: CardSlots;
  children: ReactNode;
};
```

Prefer slots when named extension points are stable and meaningful.

## 10. Polymorphic Components

### Purpose

Use polymorphic components when the same design-system primitive should render different semantic elements through an `as` prop.

### Concerns

- Correct semantic element.
- Type-safe props.
- Accessibility.
- Ref forwarding.
- Avoiding invalid combinations, such as `href` on a `button`.

### Good Use

- `Button` that can render as `button`, `a`, or router link.
- `Text` that can render as `p`, `span`, `strong`, or headings.
- Layout primitives with semantic control.

### Bad Use

- Every component in a design system.
- Components where semantics should be fixed.
- APIs with complex generic types that are harder to use than separate components.

### Warning

Polymorphism is often overused. Prefer separate components when behavior differs meaningfully:

- `Button`
- `LinkButton`
- `IconButton`

## 11. Custom Hooks

### Purpose

Use custom hooks to reuse stateful logic, side effects, subscriptions, data fetching wrappers, UI behavior, form logic, or URL state.

### Best For

- Data fetching wrappers.
- UI behavior.
- Subscriptions.
- Forms.
- URL state.
- Feature-specific model logic.

### Bad For

- Hiding too much business logic.
- Creating unclear dependencies.
- God hooks that fetch, validate, mutate, navigate, and render decisions all at once.
- Pure formatting helpers that should be utilities.

### Naming And Placement

- Hook names must start with `use`.
- Feature hooks live in `src/features/<feature>/model/hooks/`.
- Shared hooks live in `src/shared/hooks/`.
- Use kebab-case filenames, such as `use-login.ts`.

### Testing Guidance

Test custom hooks through consuming components when possible. Use hook-specific tests for complex state machines, subscriptions, or reusable shared hooks.

## 12. Hook Factories

### Purpose

Use hook factories to generate specialized hooks from shared configuration or injected dependencies.

### Best For

- API modules.
- Feature-specific query hooks.
- Shared behavior with injected dependencies.
- Multi-tenant or variant-based behavior.

### Bad For

- Simple hooks.
- Cases where plain function composition is enough.
- Hiding dependencies in module scope.

### Example Shape

```tsx
type CreateUseFeatureFlagOptions = {
  client: FeatureFlagClient;
};

const createUseFeatureFlag = ({ client }: CreateUseFeatureFlagOptions) => {
  const useFeatureFlag = (key: string) => {
    return client.getBooleanValue(key);
  };

  return useFeatureFlag;
};
```

Prefer explicit dependency injection for testability.

## 13. Controller Components

### Purpose

Use controller components when a component should own behavior and pass state/actions to a rendering layer.

### Best For

- Forms.
- Complex flows.
- Components needing clear separation between behavior and view.
- Legacy class-to-function refactors.

### Compare

| Pattern | Use When |
| --- | --- |
| Controller component | JSX consumer should receive behavior from a component |
| Render props | Consumer needs flexible rendering through a render function |
| Custom hook | Consumer component can call hooks directly |

Prefer custom hooks for new code unless a component boundary is specifically useful.

## 14. Container / Presentational Components

### Purpose

Separate data/state from UI. Containers fetch, subscribe, or coordinate behavior. Presentational components render props.

### Good Use

- Feature screens.
- Data-heavy components.
- Legacy refactors.
- Cases where the UI component should be reusable and easy to test.

### Bad Use

- Artificial separation for tiny components.
- Moving every component into paired `container` and `view` files.
- Presentational components that still import feature stores.

### Modern Alternative

Use feature hooks plus composition:

- `src/features/accounts/model/hooks/use-accounts.ts`
- `src/features/accounts/ui/pages/accounts-page.tsx`
- `src/features/accounts/ui/blocks/accounts-table.tsx`

The page composes hooks and UI. Smaller blocks stay focused.

## 15. Adapter / Facade Patterns

### Purpose

Use adapters or facades to hide third-party library details and protect the app from vendor lock-in.

### Best For

- Analytics.
- Auth.
- Payments.
- Feature flags.
- API clients.
- Notifications.

### Example Interface

```ts
export type AnalyticsEvent = {
  name: string;
  properties?: Record<string, string | number | boolean>;
};

export type AnalyticsService = {
  track: (event: AnalyticsEvent) => void;
  identify: (userId: string) => void;
};
```

Place shared infrastructure in `src/shared/services/`. Feature-specific adapters can live in `src/features/<feature>/api/` or `src/features/<feature>/lib/` when they are not reusable.

## 16. Strategy Pattern

### Purpose

Use the strategy pattern to swap behavior based on runtime conditions while keeping the caller stable.

### Best For

- Different validation rules.
- Different user roles.
- Different app variants.
- Different rendering behavior.
- Tenant-specific flows.

### Example Shape

```ts
type ValidationStrategy<TValues> = {
  validate: (values: TValues) => ValidationResult;
};
```

### Bad Use

- Simple `if` statements.
- Tiny behavior differences that do not deserve named strategies.
- Cases where strategies spread business rules across too many files.

Use a strategy when the behavior set is real, named, and likely to evolve.

## 17. Higher-Order Components

### Purpose

Use a higher-order component when a component must be wrapped with cross-cutting behavior that cannot be expressed cleanly through hooks at the call site.

### Best For

- Legacy React codebases.
- Framework integration boundaries.
- Error boundaries or provider injection wrappers.
- Compatibility layers around third-party components.

### Bad Use

- New feature code where a custom hook is clearer.
- Wrappers that hide required props.
- Multiple nested HOCs that make debugging ownership difficult.
- Styling-only composition that could be a plain component.

### Guidance

Prefer custom hooks and composition for new code. Reach for HOCs only when a wrapper boundary is genuinely useful.

## 18. Dependency Injection

### Purpose

Use dependency injection to pass services, strategies, or environment-specific behavior into React code without hardcoding infrastructure inside components.

### Best For

- Analytics, auth, feature flags, payments, storage, logging, and API clients.
- Tests that need fake services.
- White-label or tenant-specific behavior.
- Feature-scoped service contracts.

### Bad Use

- Injecting simple pure functions that can be imported directly.
- Creating a service container for a small app with no need for runtime substitution.
- Hiding dependencies so deeply that component behavior becomes hard to trace.

### Implementation Options

- Constructor-style factories for hooks and services.
- Provider-scoped services through context.
- Strategy objects for behavior variants.
- Facades in `src/shared/services/` for app-wide infrastructure.

Keep dependency contracts small and typed.

## 19. Declarative APIs, Imperative Handles, And Ref Forwarding

### Declarative APIs

Prefer declarative APIs for ordinary React UI:

```tsx
<Modal isOpen={isOpen} onOpenChange={setIsOpen} />
```

Declarative APIs are easier to test, compose, and reason about because state ownership is explicit.

### Imperative Handles

Use imperative handles only when a parent must trigger a child action that is naturally imperative:

- Focus an input.
- Scroll a list.
- Open a native picker.
- Integrate with a non-React widget.

Avoid imperative handles for state that should be controlled through props.

### Ref Forwarding

Forward refs for reusable primitives when consumers reasonably need DOM access for focus, measurement, or integration. Do not forward refs as a default for every component if there is no use case.

### Tradeoff

| Approach | Benefit | Risk |
| --- | --- | --- |
| Declarative props | Predictable ownership | More parent state |
| Imperative handle | Solves focus/integration needs | Can bypass React data flow |
| Ref forwarding | Enables composition with primitives | Adds typing complexity |

## 20. Form Architecture

### Purpose

Design forms around clear ownership of values, validation, submission, and side effects.

### Rules

- Keep simple forms local with controlled inputs.
- Use a form library for complex forms with validation, touched state, arrays, async validation, or multi-step flows.
- Keep validation schemas in `src/features/<feature>/lib/`.
- Keep feature-specific form components in `src/features/<feature>/ui/forms/`.
- Keep reusable inputs in `src/shared/components/`.
- Do not put feature validation rules inside shared input primitives.
- Keep server mutations in feature model hooks or API modules, not inside reusable fields.

### Example Placement

```txt
src/features/auth/
  api/
    auth.api.ts
  lib/
    auth.validation.ts
    auth.types.ts
  model/
    hooks/
      use-login.ts
  ui/
    forms/
      login-form.tsx
```

### Pattern Choices

| Need | Pattern |
| --- | --- |
| Simple field state | Controlled components |
| Form library field integration | Controller component |
| Reusable field behavior | Custom hook |
| Feature-specific validation variants | Strategy pattern |
| Shared design-system inputs | Controlled/uncontrolled hybrid APIs |

## 21. State Architecture

### Boundaries

| State Type | Owner | Examples | Guidance |
| --- | --- | --- | --- |
| Local UI state | Component | open/closed, active item | Keep close to usage |
| Server state | Query/cache layer | users, invoices, products | Do not duplicate in global stores |
| Form state | Form library or form component | field values, errors, touched | Use form libraries for complex forms |
| URL state | Router/search params | filters, pagination, selected tab | Use for shareable/navigation state |
| Global client state | App-level store/provider | theme, auth shell state, draft UI prefs | Use only for truly global client state |
| Derived state | Computed from existing state | filtered list, totals | Compute directly or memoize when needed |

### Rules

- Do not duplicate server state in global stores.
- Keep state as close as possible to where it is used.
- Use URL state for shareable filters, pagination, tabs, and search params.
- Use form state libraries for complex forms.
- Use global stores only for truly global client state.
- Avoid derived state unless memoized or computed directly.
- Avoid effects that only synchronize derived values.

## 22. Context Architecture

### When To Use Context

- A subtree needs a shared dependency or state value.
- Prop drilling crosses many layers that do not care about the value.
- A provider scopes behavior for a feature or compound component.
- A service should be injected for testability or environment differences.

### When Not To Use Context

- Passing props through one or two levels is clear enough.
- The value changes frequently and many consumers will rerender.
- The context hides a dependency that should be explicit.
- The state is server state and belongs in a query cache.

### Guidance

- Split context by read/write concerns.
- Memoize provider values.
- Keep providers feature-scoped when possible.
- Compose providers in `src/processes/providers/` for app-wide wiring.
- Use context for compound components when the state is local to the widget.
- Use context as dependency injection for services, not as a dumping ground.
- Prefer context composition for app-wide providers instead of deeply nesting provider setup inside route files.

## 23. Design-System Architecture

### Layers

- Primitive components: `Button`, `Input`, `Text`, `Stack`.
- Headless logic: hooks or headless components for behavior.
- Styled components: project visual defaults.
- Component contracts: stable public props and behavior.
- Variant APIs: small, typed variants for visual states.
- Slot APIs: named escape points for stable internal regions.
- Controlled/uncontrolled APIs: support simple and advanced usage.
- Accessibility requirements: keyboard, roles, labels, focus, ARIA.
- Escape hatches: `className`, `style` when appropriate, slots, render props, polymorphism.

### Rules

- Avoid boolean prop explosion. Prefer typed variants.
- Keep feature-specific rules out of reusable primitives.
- Test public behavior instead of internals.
- Keep default behavior accessible.
- Prefer explicit props for common needs and escape hatches for uncommon needs.
- Do not make every component polymorphic.
- Do not expose every internal element as a slot.

### Placement

Reusable shared components live in `src/shared/components/`. Keep feature-specific UI inside `src/features/<feature>/ui/` until reuse appears.

## 24. Feature Architecture

Use Feature-Sliced inspired structure:

```txt
src/features/accounts/
  api/
    accounts.api.ts
  config/
    accounts.constants.ts
  lib/
    accounts.types.ts
    accounts.validation.ts
    accounts.mappers.ts
  model/
    hooks/
      use-accounts.ts
    context/
    stores/
  ui/
    pages/
      accounts-page.tsx
    forms/
    blocks/
      accounts-table.tsx
    elements/
      account-row.tsx
```

### Rules

- Keep feature-specific code inside the feature.
- Move code to `shared/` only after real cross-feature reuse.
- Keep route files in `src/app/` thin and compositional.
- Put API clients in `src/shared/services/` and feature endpoint modules in `src/features/<feature>/api/`.
- Put feature types, schemas, mappers, and validation in `src/features/<feature>/lib/`.
- Avoid circular dependencies.
- Avoid a shared folder dumping ground.
- Avoid direct imports from unrelated features.

## 25. React Anti-patterns

Watch for:

- Huge components.
- God hooks.
- Overusing context.
- Duplicating server state in stores.
- Premature abstraction.
- Deep prop drilling.
- Boolean prop explosion.
- Too many providers.
- Business logic inside reusable UI primitives.
- Reusable components that know feature-specific rules.
- Mixing data fetching, rendering, validation, and side effects in one component.
- Using effects for derived state.
- Incorrect memoization.
- Ref abuse.
- Imperative APIs without need.
- HOCs stacked so deeply that data flow is unclear.
- Pattern use for aesthetic preference instead of a concrete problem.

For each anti-pattern, recommend the smallest refactor that improves ownership and clarity.

## 26. Pattern Selection Guide

| Need | Recommended Pattern |
| --- | --- |
| Flexible visual structure | Compound Components / Slots |
| Expose internal state to consumer | Render Props / Children as Function |
| Parent-owned state | Controlled Component |
| Reusable behavior | Custom Hook |
| Styling freedom | Headless Component |
| Consumer override of state changes | State Reducer |
| Safely merge internal and external props | Prop Getters |
| Hide external service | Adapter / Facade |
| Swap behavior by condition | Strategy Pattern |
| Inject environment-specific behavior | Dependency Injection / Provider |
| Wrap legacy or integration boundaries | Higher-Order Component |
| Trigger focus, scroll, or native APIs | Ref forwarding / Imperative handle |
| Complex form state and validation | Form library + Controller Components |
| Feature isolation | Feature-based architecture |
| Design-system primitives | Compound + Controlled + Headless + Slots |
| Simple one-off UI | Plain component, no advanced pattern |

Use this table as a starting point, not as a rule engine. The final recommendation must reflect the actual constraints.

## 27. Code Examples

### Compound Tabs

Place in `src/shared/components/tabs.tsx` if reused across features.

```tsx
import {
  createContext,
  type KeyboardEvent,
  type ReactElement,
  type ReactNode,
  useContext,
  useId,
  useMemo,
  useState,
} from "react";

type TabsContextValue = {
  baseId: string;
  value: string;
  setValue: (value: string) => void;
};

const TabsContext = createContext<TabsContextValue | null>(null);

type TabsProps = {
  value?: string;
  defaultValue: string;
  onValueChange?: (value: string) => void;
  children: ReactNode;
};

type TabsListProps = {
  children: ReactNode;
};

type TabsTriggerProps = {
  value: string;
  children: ReactNode;
};

type TabsContentProps = {
  value: string;
  children: ReactNode;
};

const useTabsContext = () => {
  const context = useContext(TabsContext);

  if (!context) {
    throw new Error("Tabs components must be used inside Tabs.");
  }

  return context;
};

const TabsList = ({ children }: TabsListProps) => {
  return <div role="tablist">{children}</div>;
};

const TabsTrigger = ({ value, children }: TabsTriggerProps) => {
  const { baseId, value: selectedValue, setValue } = useTabsContext();
  const isSelected = selectedValue === value;

  const clickHandler = () => {
    setValue(value);
  };

  const keyDownHandler = (event: KeyboardEvent<HTMLButtonElement>) => {
    if (event.key === "Enter" || event.key === " ") {
      event.preventDefault();
      setValue(value);
    }
  };

  return (
    <button
      id={`${baseId}-trigger-${value}`}
      type="button"
      role="tab"
      aria-selected={isSelected}
      aria-controls={`${baseId}-content-${value}`}
      tabIndex={isSelected ? 0 : -1}
      onClick={clickHandler}
      onKeyDown={keyDownHandler}
    >
      {children}
    </button>
  );
};

const TabsContent = ({ value, children }: TabsContentProps) => {
  const { baseId, value: selectedValue } = useTabsContext();

  if (selectedValue !== value) {
    return null;
  }

  return (
    <div
      id={`${baseId}-content-${value}`}
      role="tabpanel"
      aria-labelledby={`${baseId}-trigger-${value}`}
    >
      {children}
    </div>
  );
};

type TabsComponent = ((props: TabsProps) => ReactElement) & {
  List: typeof TabsList;
  Trigger: typeof TabsTrigger;
  Content: typeof TabsContent;
};

const Tabs = (({ value, defaultValue, onValueChange, children }: TabsProps) => {
  const baseId = useId();
  const [internalValue, setInternalValue] = useState(defaultValue);
  const selectedValue = value ?? internalValue;

  const setValue = (nextValue: string) => {
    if (value === undefined) {
      setInternalValue(nextValue);
    }

    onValueChange?.(nextValue);
  };

  const contextValue = useMemo<TabsContextValue>(() => {
    return { baseId, value: selectedValue, setValue };
  }, [baseId, selectedValue]);

  return <TabsContext.Provider value={contextValue}>{children}</TabsContext.Provider>;
}) as TabsComponent;

Tabs.List = TabsList;
Tabs.Trigger = TabsTrigger;
Tabs.Content = TabsContent;

export default Tabs;
```

### Controlled Input

Place in `src/shared/components/text-input.tsx` if reused.

```tsx
import type { ChangeEvent, InputHTMLAttributes } from "react";

export type TextInputProps = Omit<InputHTMLAttributes<HTMLInputElement>, "value" | "onChange"> & {
  value: string;
  onValueChange: (value: string) => void;
};

const TextInput = ({ value, onValueChange, ...props }: TextInputProps) => {
  const changeHandler = (event: ChangeEvent<HTMLInputElement>) => {
    onValueChange(event.target.value);
  };

  return <input {...props} value={value} onChange={changeHandler} />;
};

export default TextInput;
```

### Controlled / Uncontrolled Accordion

Place in `src/shared/components/accordion.tsx`.

```tsx
import { type ReactNode, useState } from "react";

export type AccordionProps = {
  value?: string;
  defaultValue?: string;
  onValueChange?: (value: string) => void;
  children: (options: {
    value: string | undefined;
    setValue: (value: string) => void;
  }) => ReactNode;
};

const Accordion = ({ value, defaultValue, onValueChange, children }: AccordionProps) => {
  const [internalValue, setInternalValue] = useState<string | undefined>(defaultValue);
  const selectedValue = value ?? internalValue;

  const setValue = (nextValue: string) => {
    if (value === undefined) {
      setInternalValue(nextValue);
    }

    onValueChange?.(nextValue);
  };

  return <>{children({ value: selectedValue, setValue })}</>;
};

export default Accordion;
```

### Headless Modal

Place behavior in `src/shared/hooks/use-modal.ts` and styled UI in `src/shared/components/modal.tsx`.

```tsx
import { type ButtonHTMLAttributes, type HTMLAttributes, useState } from "react";

type UseModalResult = {
  isOpen: boolean;
  open: () => void;
  close: () => void;
  getTriggerProps: (
    props?: ButtonHTMLAttributes<HTMLButtonElement>,
  ) => ButtonHTMLAttributes<HTMLButtonElement>;
  getDialogProps: (props?: HTMLAttributes<HTMLDivElement>) => HTMLAttributes<HTMLDivElement>;
};

export const useModal = (): UseModalResult => {
  const [isOpen, setIsOpen] = useState(false);

  const open = () => {
    setIsOpen(true);
  };

  const close = () => {
    setIsOpen(false);
  };

  const getTriggerProps: UseModalResult["getTriggerProps"] = (props = {}) => {
    const userClickHandler = props.onClick;
    const clickHandler: NonNullable<ButtonHTMLAttributes<HTMLButtonElement>["onClick"]> = (
      event,
    ) => {
      userClickHandler?.(event);
      open();
    };

    return {
      ...props,
      "aria-haspopup": "dialog",
      "aria-expanded": isOpen,
      onClick: clickHandler,
    };
  };

  const getDialogProps: UseModalResult["getDialogProps"] = (props = {}) => {
    return {
      ...props,
      role: "dialog",
      "aria-modal": true,
    };
  };

  return { isOpen, open, close, getTriggerProps, getDialogProps };
};
```

### Slot-Based Card

Place in `src/shared/components/card.tsx` if generic.

```tsx
import type { ReactNode } from "react";

type CardSlots = {
  header?: ReactNode;
  media?: ReactNode;
  footer?: ReactNode;
};

export type CardProps = {
  title: string;
  slots?: CardSlots;
  children: ReactNode;
};

const Card = ({ title, slots, children }: CardProps) => {
  return (
    <article>
      {slots?.media}
      <header>{slots?.header ?? <h2>{title}</h2>}</header>
      <div>{children}</div>
      {slots?.footer ? <footer>{slots.footer}</footer> : null}
    </article>
  );
};

export default Card;
```

### Polymorphic Button

Place in `src/shared/components/button.tsx`. Keep the supported element set small.

```tsx
import {
  forwardRef,
  type AnchorHTMLAttributes,
  type ButtonHTMLAttributes,
  type ReactNode,
  type Ref,
} from "react";

type ButtonVariant = "primary" | "secondary" | "ghost";

type BaseButtonProps = {
  variant?: ButtonVariant;
  children: ReactNode;
};

type NativeButtonProps = BaseButtonProps &
  ButtonHTMLAttributes<HTMLButtonElement> & {
    as?: "button";
  };

type AnchorButtonProps = BaseButtonProps &
  AnchorHTMLAttributes<HTMLAnchorElement> & {
    as: "a";
    href: string;
  };

export type ButtonProps = NativeButtonProps | AnchorButtonProps;

const Button = forwardRef<HTMLButtonElement | HTMLAnchorElement, ButtonProps>(
  ({ as = "button", variant = "primary", children, ...props }, ref) => {
    const className = `button button-${variant}`;

    if (as === "a") {
      return (
        <a {...props} ref={ref as Ref<HTMLAnchorElement>} className={className}>
          {children}
        </a>
      );
    }

    return (
      <button
        {...(props as ButtonHTMLAttributes<HTMLButtonElement>)}
        ref={ref as Ref<HTMLButtonElement>}
        className={className}
      >
        {children}
      </button>
    );
  },
);

Button.displayName = "Button";

export default Button;
```

### Render-Prop DataLoader

Prefer a query hook for new app code. Use this shape when a component boundary is useful.

```tsx
import type { ReactNode } from "react";

type DataLoaderState<TData> = {
  data: TData | null;
  loading: boolean;
  error: Error | null;
};

export type DataLoaderProps<TData> = {
  state: DataLoaderState<TData>;
  children: (state: DataLoaderState<TData>) => ReactNode;
};

const DataLoader = <TData,>({ state, children }: DataLoaderProps<TData>) => {
  return <>{children(state)}</>;
};

export default DataLoader;
```

### Custom Hook For Reusable Behavior

Place feature-specific hooks in `src/features/search/model/hooks/use-search-params-state.ts`.

```tsx
import { useSearchParams } from "react-router-dom";

type UseSearchParamsStateResult = {
  query: string;
  setQuery: (query: string) => void;
};

export const useSearchParamsState = (): UseSearchParamsStateResult => {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get("query") ?? "";

  const setQuery = (nextQuery: string) => {
    setSearchParams((currentParams) => {
      const nextParams = new URLSearchParams(currentParams);

      if (nextQuery) {
        nextParams.set("query", nextQuery);
      } else {
        nextParams.delete("query");
      }

      return nextParams;
    });
  };

  return { query, setQuery };
};
```

### Adapter For Analytics Service

Place shared service contracts in `src/shared/services/analytics/analytics.types.ts`.

```ts
export type AnalyticsEvent = {
  name: string;
  properties?: Record<string, string | number | boolean>;
};

export type AnalyticsService = {
  track: (event: AnalyticsEvent) => void;
  identify: (userId: string) => void;
};
```

Place an implementation in `src/shared/services/analytics/browser-analytics.service.ts`.

```ts
import type { AnalyticsEvent, AnalyticsService } from "@/shared/services/analytics/analytics.types";

type VendorAnalytics = {
  track: (name: string, properties?: Record<string, string | number | boolean>) => void;
  identify: (userId: string) => void;
};

export const createBrowserAnalyticsService = (
  vendorAnalytics: VendorAnalytics,
): AnalyticsService => {
  const track = (event: AnalyticsEvent) => {
    vendorAnalytics.track(event.name, event.properties);
  };

  const identify = (userId: string) => {
    vendorAnalytics.identify(userId);
  };

  return { track, identify };
};
```

### Strategy For Validation

Place feature validation in `src/features/auth/lib/auth.validation.ts`.

```ts
export type LoginValues = {
  email: string;
  password: string;
};

export type ValidationResult = {
  isValid: boolean;
  errors: Partial<Record<keyof LoginValues, string>>;
};

export type LoginValidationStrategy = {
  validate: (values: LoginValues) => ValidationResult;
};

export const passwordLoginValidationStrategy: LoginValidationStrategy = {
  validate: (values) => {
    const errors: ValidationResult["errors"] = {};

    if (!values.email.includes("@")) {
      errors.email = "Enter a valid email.";
    }

    if (values.password.length < 8) {
      errors.password = "Use at least 8 characters.";
    }

    return { isValid: Object.keys(errors).length === 0, errors };
  },
};
```

### Feature Folder Structure

```txt
src/app/accounts/page.tsx
src/features/accounts/
  api/
    accounts.api.ts
  lib/
    accounts.mappers.ts
    accounts.types.ts
  model/
    hooks/
      use-accounts.ts
  ui/
    pages/
      accounts-page.tsx
    blocks/
      accounts-table.tsx
    elements/
      account-row.tsx
src/shared/
  components/
    button.tsx
    text-input.tsx
  services/
    http-client.ts
```

Example route:

```tsx
import AccountsPage from "@/features/accounts/ui/pages/accounts-page";

const AccountsRoute = () => {
  return <AccountsPage />;
};

export default AccountsRoute;
```

## 28. Testing Guidance

### Tools

- Use React Testing Library for component behavior.
- Use hook tests only for reusable hooks with meaningful state transitions.
- Use accessibility assertions for labels, roles, names, focus, keyboard behavior, and ARIA state.
- Use integration tests when data, routing, forms, and UI behavior interact.
- Run tests with Bun when documenting commands, such as `bun run test`.

### What To Test

- Public behavior, not implementation details.
- Controlled and uncontrolled usage.
- Keyboard behavior for widgets like tabs, menus, accordions, dialogs, and selects.
- Focus management and escape behavior for modals and dropdowns.
- ARIA roles, labels, and selected/expanded state.
- Contract tests for reusable design-system components.
- Error, loading, empty, and success states.
- Adapter contracts with mocked vendors.
- Strategy selection and behavior.

### What To Avoid

- Testing context internals directly.
- Testing private state names.
- Snapshot-only coverage for interactive components.
- Mocking so deeply that the test no longer checks integration.
- Testing generated class names instead of user-visible behavior.

### Review Checklist

- Is the state owned by the smallest responsible component or layer?
- Is server state kept in the query/cache layer instead of copied into a store?
- Is form state owned by the form or form library?
- Is URL state used for shareable filters, pagination, tabs, and search?
- Does the reusable component avoid feature-specific rules?
- Does the API have good defaults and escape hatches?
- Is the public API smaller than the internal implementation?
- Are advanced patterns justified by real reuse, API, accessibility, or scalability needs?
- Are examples and file placement aligned with the project conventions?
