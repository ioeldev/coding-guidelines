---
name: coding-guidelines
description: Apply personal coding conventions when writing, reviewing, or refactoring code, including descriptive naming, flat control flow, app/features/shared organization, strict TypeScript, React Router data mode, and React patterns.
---

# Coding Guidelines

Apply these rules when writing, reviewing, or refactoring code. Keep changes focused on the task and preserve existing behavior unless the task requires changing it.

## Core Principle

Write code that another developer can understand by reading it from top to bottom.

Prioritize descriptive naming, flat control flow, strict types, and clear responsibility boundaries. Prefer straightforward code over clever brevity or unnecessary abstractions.

## Naming

- Use self-descriptive names that communicate domain meaning and intent.
- Prefer `subscription`, `paymentRequest`, and `eligibleProperties` over `s`, `data`, and `items`.
- Use concrete actions such as `cancelSubscription`, `calculateMonthlyRent`, and `findExpiredSessions`.
- Avoid vague verbs such as `handle`, `process`, `manage`, and `advance` when a specific action can be named.
- Use descriptive callback parameters, including inside `map`, `filter`, and `reduce`.
- Name booleans as predicates: `isExpired`, `hasPermission`, `canWithdraw`.
- Include units when relevant: `timeoutMs`, `amountInCents`.
- Avoid abbreviations unless they are established domain terminology.
- Rename unclear code instead of adding comments that explain its names.
- Use comments to explain non-obvious decisions, constraints, or tradeoffs.

## Control Flow

- Handle invalid input, missing data, loading states, and errors first.
- Use early returns or throws to keep the main path flat.
- Keep the successful path after guards, at the lowest indentation level.
- Avoid nested `if/else` blocks.
- Do not add `else` after a branch that returns or throws.
- Omit braces for single-line `if` guards.
- Use braces for multiline bodies.
- Avoid nested ternaries and complex conditions that require mental decoding.

```ts
function calculateDiscount(order: Order): number {
  if (order.isCancelled) return 0;

  if (order.totalInCents < MINIMUM_DISCOUNT_TOTAL) return 0;

  return order.totalInCents * DISCOUNT_RATE;
}
```

## Spacing and Logical Steps

Use blank lines to make the execution flow visible.

- Separate validation, retrieval, transformation, persistence, side effects, and return values.
- Add a blank line after a guard or a tightly related group of guards.
- Keep statements belonging to the same logical step together.
- Avoid dense blocks that combine an entire workflow without spacing.
- Do not insert blank lines mechanically between every statement.

```ts
const subscription = await this.subscriptions.findById(subscriptionId);

if (!subscription) throw new SubscriptionNotFoundError(subscriptionId);

this.ensureSubscriptionCanBeCancelled(subscription);

const cancelledSubscription = await this.subscriptions.cancel(subscriptionId);

return cancelledSubscription;
```

## Functions and Abstractions

- Give each function one clear responsibility.
- Aim for functions under approximately 25 lines.
- Treat function length as a readability target, not a hard limit.
- Do not fragment a coherent operation merely to reduce its line count.
- Extract meaningful responsibilities, substantial logic, or actual reuse.
- Inline trivial expressions when a helper would make the reader jump around.
- Avoid forwarding-only functions and layers that add no meaningful behavior.
- Do not introduce generic frameworks for hypothetical future requirements.
- Use object parameters for functions with three or more arguments.

## File Organization

Keep the purpose of each file visible near the top.

- Order files as imports, essential types/constants, main implementation, then supporting helpers.
- Do not place a collection of small utility functions before the main implementation.
- Put file-local helpers below the exported component, class, or function.
- Keep service-specific helpers inside the service as private methods.
- Extract shared helpers into specifically named modules only when actual reuse justifies it.
- Avoid catch-all files such as `utils.ts`, `helpers.ts`, and `common.ts`.
- Split files by responsibility, not merely by length.
- Keep one React component per file.

## Frontend Architecture

Use `app/`, `features/`, and `shared/` for React applications. Organize features by business area, such as projects, tasks, auth, or billing.

- Keep Vite's `index.html` and `vite.config.ts` at the project root, and `main.tsx` and global styles under `src/`.
- Use `app/` for application composition: router, providers, application layout, navigation shell, and application-wide error handling.
- Use `features/<business-area>/` for that area's pages, route definitions, API functions, types, components, and hooks.
- Keep page-specific components and hooks beside their owning page. Promote them to feature scope when multiple pages need them.
- Use `shared/` for code used across business areas that has no feature-specific ownership, such as UI primitives and an HTTP client.
- Keep business-specific code in its owning feature even when another feature uses it. Use explicit imports and avoid circular dependencies; compose collaborating features in `app/` when necessary.
- Do not collect business-specific code in global `components/` or `hooks/` directories.
- Folder ownership follows business concepts; nested URLs do not require matching folder nesting.
- Start related behavior inside its owning feature. Split a separate feature when it has substantial independent behavior or reuse.
- Create folders and files only when needed. Do not scaffold empty layers or require every page to have a loader, action, hook, or components folder.
- Prefer descriptive kebab-case filenames with `.page.tsx`, `.loader.ts`, `.action.ts`, `.routes.ts`, `.api.ts`, and `.types.ts` suffixes where applicable.

Example for a project management application:

```text
public/
src/
  main.tsx
  index.css
  app/
    router.ts
    app-layout.tsx
    app-error-boundary.tsx
    providers.tsx
  features/
    auth/
      auth.routes.ts
      auth.api.ts
      pages/
        login/
          login.page.tsx
          login.action.ts
    projects/
      projects.routes.ts
      projects.api.ts
      projects.types.ts
      pages/
        project-list/
          project-list.page.tsx
          project-list.loader.ts
          components/
            create-project-dialog.tsx
        project-details/
          project-details.page.tsx
          project-details.loader.ts
          components/
            project-header.tsx
            project-members.tsx
      components/
        project-card.tsx
    tasks/
      tasks.routes.ts
      tasks.api.ts
      tasks.types.ts
      pages/
        task-details/
          task-details.page.tsx
          task-details.loader.ts
          task-details.action.ts
          components/
            task-comments.tsx
      components/
        task-status-badge.tsx
  shared/
    components/
      ui/
        button.tsx
        dialog.tsx
    hooks/
      use-media-query.ts
    lib/
      http-client.ts
index.html
vite.config.ts
tsconfig.json
package.json
bun.lock
```

## React Router Data Mode

Default to React Router data mode for new React + Vite applications. Respect an existing application's router and mode unless migration is requested.

- Create one router outside the React tree with `createBrowserRouter` in `app/router.ts`.
- Render it through `RouterProvider` from `react-router/dom` in `main.tsx`. Import router APIs and types from `react-router`, using APIs compatible with the installed version.
- Export feature route objects from `<feature>.routes.ts`, typed with `satisfies RouteObject[]`, and compose them in `app/router.ts`.
- Keep paths, index routes, and nesting declared in route configuration. Render nested pages through `<Outlet />` in layouts.
- Use route `loader` functions for data required by a route and `useLoaderData` to consume it. Avoid fetching the same data again in a component effect.
- Use route `action` functions for mutations associated with routes. Use `<Form>` or `useSubmit` for submissions with navigation, and `useFetcher` / `<fetcher.Form>` for loads or submissions without navigation.
- Use `useNavigation` for navigation pending states and `fetcher.state` for fetcher pending states. Return expected validation errors as action data and display them near the relevant fields.
- Let the router revalidate loader data after successful actions. Customize revalidation only for a concrete requirement.
- Keep HTTP requests in feature API modules; loaders and actions orchestrate route parameters, validation, API calls, and redirects.
- Validate route parameters and form input at the boundary. Pass `request.signal` to fetch calls when appropriate so cancelled navigation can cancel requests.
- Use route error boundaries with `useRouteError` and `isRouteErrorResponse` where appropriate. Provide an application-level fallback and more local boundaries when useful.
- Use `<Link>` and `<NavLink>` for internal navigation, and redirects from loaders/actions when the outcome determines the destination.
- Use `route.lazy` to split page implementations when useful. Map descriptive page, loader, and action exports to the router's `Component`, `loader`, and `action` properties; keep matching properties such as `path` outside the lazy implementation.
- When loading independent page, loader, or action modules lazily, import them in parallel. A lightweight eager loader can start data fetching while the page bundle loads.
- Prefer separate page, loader, and action files when they clarify responsibilities; small route implementations may colocate them.
- Data mode in a Vite SPA runs loaders and actions in the browser. Keep secrets and privileged operations on a backend; client routing does not enforce authorization.
- Keep Vite's React setup. Data mode does not require the React Router framework Vite plugin, filesystem routing, generated framework route types, or framework-only `clientLoader` / `clientAction` conventions.

References: [Data mode setup](https://reactrouter.com/start/data/installation), [route objects](https://reactrouter.com/start/data/route-object), and [actions](https://reactrouter.com/start/data/actions).

## Runtime and Package Manager

- Default to Bun for new TypeScript and JavaScript projects, scripts, and tooling.
- Use `bun install` to install dependencies, `bun add` / `bun add -d` to add dependencies, and `bun remove` to remove them.
- Use `bun run <script>` for package scripts, `bun <file.ts>` to execute TypeScript, and `bunx <package>` for package executables.
- Use `bun test` for new projects unless the project requires another test runner.
- Commit the Bun lockfile generated by the installed Bun version.
- Respect an existing project's runtime, package manager, lockfile, and deployment constraints. Do not migrate them to Bun unless requested.
- Keep strict type checking as a separate check; executing TypeScript with Bun does not replace it.

## TypeScript

- Use strict types.
- Avoid `any`, unjustified type assertions, and non-null assertions.
- Accept untrusted input as `unknown` and validate it at the boundary.
- Model mutually exclusive states as discriminated unions using `kind`.
- Avoid combinations of booleans that permit impossible states.
- Exhaustively handle closed unions and finish with an `assertNever()` check.
- Do not add exhaustive checks to ordinary conditionals.
- Define explicit parameter and return types for public service methods and exported APIs.
- Let TypeScript infer obvious local values.
- Reuse an existing `assertNever()` helper instead of duplicating it.

```ts
type PaymentState =
  | { kind: 'pending' }
  | { kind: 'paid'; receiptId: string }
  | { kind: 'failed'; reason: string };

function getPaymentLabel(state: PaymentState): string {
  switch (state.kind) {
    case 'pending':
      return 'Awaiting payment';

    case 'paid':
      return `Receipt ${state.receiptId}`;

    case 'failed':
      return state.reason;
  }

  return assertNever(state);
}

function assertNever(value: never): never {
  throw new Error(`Unexpected variant: ${JSON.stringify(value)}`);
}
```

## Expressions

- Default to `const`. Use `let` only when reassignment is necessary.
- Use `Boolean(value)` instead of `!!value`.
- Write ternaries with positive conditions: `isEnabled ? enabledValue : disabledValue`.
- Avoid inverted ternaries such as `!isEnabled ? disabledValue : enabledValue`.
- Use named intermediate values for complex expressions.
- Break dense transformation chains into meaningful steps when that improves readability.
- Avoid repeated calculations or conversions when a descriptive variable makes the intent clearer.

## React

- Keep one component per file.
- Handle loading, error, and empty states with early returns.
- Call hooks unconditionally and before conditional returns.
- For data-mode routes, use loaders and actions for route data and mutations. Keep component-owned queries, fetchers, and local interactions in the leaf component that owns the behavior; avoid duplicating loader requests.
- Avoid drilling server data and mutation callbacks through intermediate components.
- Keep reusable presentation components driven by props.
- Use explicit ternaries for conditional JSX: `count > 0 ? <Results /> : null`.
- Never use `&&` for conditional JSX rendering.
- Prefer inline callbacks: `onClick={() => cancelSubscription(subscriptionId)}`.
- Do not hoist simple handlers solely to give them a name.
- Do not add `useCallback` by default.
- Use `useCallback` only when passing to a `memo()` child and stable identity provides a concrete benefit.
- Extract substantial interaction logic into a meaningful hook or service.
- Keep JSX focused on rendering; avoid embedding business workflows inside event callbacks.

## Promises and Errors

- Prefer `.then().catch()` for simple asynchronous calls with straightforward success and failure paths.
- Return, await, or explicitly manage promises so rejections are handled.
- Use `async/await` for sequential workflows when it makes the steps clearer.
- Use `try/catch` when recovery or contextual error translation is needed.
- Use `finally` or equivalent cleanup when work must run after both success and failure.
- Do not swallow errors or convert failures into apparent success.
- Keep loading and cleanup behavior correct on every path.
- Preserve useful error context without exposing sensitive information.
