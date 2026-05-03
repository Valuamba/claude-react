# Frontend Architecture Guideline

## React + TypeScript + React Router + Vite

This document describes a clean architectural approach for building React applications using TypeScript, React Router, Vite, and a Feature-Sliced style structure.

The goal is to make the project scalable, readable, and easy to maintain.

---

# 1. Main Architectural Idea

The architecture should answer the question:

> "Where does this code belong from a product and business perspective?"

Not only:

> "Is this a component, hook, type, or utility?"

A good base approach for medium and large React projects is:

```txt
Feature-Sliced Design + page-local structure
```

This means the project has clear top-level layers:

```txt
app
pages
widgets
features
entities
shared
```

Inside each page, you can still keep your preferred structure:

```txt
Page/
  hooks/
    usePresenter.ts
  components/
    ComponentName/
      hooks/
      ComponentName.tsx
      ComponentName.module.css
  PageName.tsx
  PageName.module.css
```

But I would slightly adjust it to fit better with scalable architecture.

---

# 2. Recommended Project Structure

```txt
src/
  app/
    entrypoint/
      main.tsx
    providers/
      AppProvider.tsx
      QueryProvider.tsx
      RouterProvider.tsx
    router/
      routes.tsx
    styles/
      globals.css
    config/
      env.ts

  pages/
    user-profile/
      ui/
        UserProfilePage.tsx
        UserProfilePage.module.css

        components/
          ProfileHeader/
            ProfileHeader.tsx
            ProfileHeader.module.css
            hooks/
              useProfileHeader.ts

          ProfileDetails/
            ProfileDetails.tsx
            ProfileDetails.module.css

      model/
        useUserProfilePresenter.ts

      index.ts

  widgets/
    app-header/
      ui/
        AppHeader.tsx
        AppHeader.module.css
      index.ts

  features/
    auth/
      login/
        ui/
          LoginForm.tsx
          LoginForm.module.css
        model/
          login.schema.ts
          useLoginForm.ts
        api/
          login.api.ts
          login.dto.ts
        index.ts

    user/
      update-profile/
        ui/
          UpdateProfileForm.tsx
        model/
          updateProfile.schema.ts
          useUpdateProfile.ts
        api/
          updateProfile.api.ts
          updateProfile.dto.ts
        index.ts

  entities/
    user/
      api/
        user.api.ts
        user.dto.ts
        user.mapper.ts
        user.schema.ts
      model/
        user.types.ts
        user.queries.ts
        user.keys.ts
      ui/
        UserAvatar/
          UserAvatar.tsx
          UserAvatar.module.css
      index.ts

  shared/
    api/
      httpClient.ts
      ApiError.ts
      createRequest.ts

    ui/
      Button/
        Button.tsx
        Button.module.css
        index.ts
      Input/
      Modal/
      Spinner/
      Skeleton/

    lib/
      date/
        formatDate.ts
      string/
        capitalize.ts
      react/
        composeProviders.tsx

    config/
      routes.ts
      query.ts

    types/
      Brand.ts
```

---

# 3. Improved Version of Your Page Structure

Your structure:

```txt
page/
Page/
  hooks/
    usePresenter.ts
  components/
    ComponentName/
      hooks/
      ComponentName.tsx
      ComponentName.module.css
  PageName.tsx
  PageName.module.css
```

is fine for page-local code.

But I would recommend this version:

```txt
pages/
  page-name/
    ui/
      PageName.tsx
      PageName.module.css
      components/
        ComponentName/
          ComponentName.tsx
          ComponentName.module.css
          hooks/
            useComponentName.ts
    model/
      usePageNamePresenter.ts
    index.ts
```

Why this is better:

```txt
ui      -> rendering, JSX, visual components
model   -> page state, presenter, form logic, query composition
index.ts -> public API of the page
```

This separates visual code from logic and makes the structure easier to scale.

---

# 4. What Belongs Where

## `app`

The `app` layer contains application-level setup.

```txt
app/
  router/
  providers/
  styles/
  config/
  entrypoint/
```

Examples:

```tsx
// app/providers/AppProvider.tsx

import { QueryClientProvider } from '@tanstack/react-query';
import { RouterProvider } from 'react-router';
import { queryClient } from '@/shared/config/query';
import { router } from '@/app/router/routes';

export function AppProvider() {
  return (
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router} />
    </QueryClientProvider>
  );
}
```

The `app` layer is responsible for:

```txt
- app initialization
- global providers
- routing
- global styles
- environment configuration
```

---

## `pages`

A page is a screen.

Examples:

```txt
pages/
  home/
  user-profile/
  orders/
  settings/
```

A page may contain:

```txt
- local components
- page-level presenter
- page-level styles
- page-specific logic
```

Example:

```txt
pages/
  orders/
    ui/
      OrdersPage.tsx
      components/
        OrdersTable/
        OrdersFilters/
    model/
      useOrdersPresenter.ts
```

Pages can import from:

```txt
widgets
features
entities
shared
```

But lower layers should not import pages.

---

## `widgets`

A widget is a large reusable UI block.

Examples:

```txt
widgets/
  app-header/
  sidebar/
  user-menu/
  dashboard-layout/
  order-summary/
```

A widget may combine:

```txt
- shared UI primitives
- entities
- features
```

Example:

```txt
widgets/app-header
```

This can contain navigation, current user info, logout button, and menu logic.

It is not a primitive because it has product meaning.

---

## `features`

A feature is a user action or business scenario.

Examples:

```txt
features/
  auth/login
  auth/logout
  user/update-profile
  cart/add-product
  order/cancel-order
  product/search-products
```

A good rule:

> If you can name it as a user action, it is probably a feature.

Examples:

```txt
login
logout
updateProfile
addToCart
cancelOrder
deleteComment
```

A feature often contains:

```txt
ui/
model/
api/
```

Example:

```txt
features/user/update-profile/
  ui/
    UpdateProfileForm.tsx
  model/
    updateProfile.schema.ts
    useUpdateProfile.ts
  api/
    updateProfile.api.ts
    updateProfile.dto.ts
```

---

## `entities`

An entity is a business object.

Examples:

```txt
entities/
  user
  product
  order
  comment
  invoice
```

Entity code describes the object itself.

Example:

```txt
entities/user/
  api/
    user.api.ts
    user.dto.ts
    user.mapper.ts
    user.schema.ts
  model/
    user.types.ts
    user.queries.ts
    user.keys.ts
  ui/
    UserAvatar/
      UserAvatar.tsx
      UserAvatar.module.css
  index.ts
```

Good examples of entity UI:

```txt
UserAvatar
UserName
ProductCard
OrderStatusBadge
CommentItem
```

---

## `shared`

The `shared` layer contains reusable code without business meaning.

Examples:

```txt
shared/
  ui/
  api/
  lib/
  config/
  types/
```

Your "primitives" belong here:

```txt
shared/ui/
  Button
  Input
  Select
  Modal
  Checkbox
  Spinner
  Skeleton
```

Important rule:

```txt
shared must not know about User, Order, Product, Auth, or any business entity.
```

A `Button` should not import `useUser`.

A `Modal` should not know anything about orders or payments.

---

# 5. Component Placement Rules

## Page-local component

Use this when the component is used only inside one page.

```txt
pages/orders/ui/components/OrdersTable/
```

## Shared primitive

Use this when the component has no business logic.

```txt
shared/ui/Button
shared/ui/Modal
shared/ui/Input
```

## Entity UI component

Use this when the component displays a business entity.

```txt
entities/user/ui/UserAvatar
entities/product/ui/ProductCard
entities/order/ui/OrderStatusBadge
```

## Feature component

Use this when the component performs a user action.

```txt
features/order/cancel-order
features/user/update-profile
features/cart/add-product
```

## Widget

Use this when the component is a large composed block.

```txt
widgets/app-header
widgets/sidebar
widgets/order-summary
```

---

# 6. `usePresenter` Pattern

The `usePresenter` pattern is good if it does not become a "god hook".

A presenter should:

```txt
- read route params
- read search params
- call query hooks
- call mutation hooks
- prepare view model for UI
- store page-local UI state
- return state and actions for the component
```

A presenter should not:

```txt
- contain JSX
- directly work with DTOs
- contain low-level fetch logic
- know implementation details of httpClient
- store server data in useState
```

Example:

```tsx
// pages/user-profile/model/useUserProfilePresenter.ts

import { useParams, useNavigate } from 'react-router';
import { useUser } from '@/entities/user';
import { useUpdateProfile } from '@/features/user/update-profile';

export function useUserProfilePresenter() {
  const { userId } = useParams<{ userId: string }>();
  const navigate = useNavigate();

  const userQuery = useUser(userId!);
  const updateProfile = useUpdateProfile();

  const isInitialLoading = userQuery.isPending;
  const isSaving = updateProfile.isPending;

  const handleBack = () => {
    navigate(-1);
  };

  return {
    state: {
      user: userQuery.data,
      isInitialLoading,
      isSaving,
      error: userQuery.error,
    },
    actions: {
      handleBack,
      updateProfile: updateProfile.mutate,
    },
  };
}
```

Page component:

```tsx
// pages/user-profile/ui/UserProfilePage.tsx

import { useUserProfilePresenter } from '../model/useUserProfilePresenter';
import { UserAvatar } from '@/entities/user';
import { UpdateProfileForm } from '@/features/user/update-profile';

export function UserProfilePage() {
  const { state, actions } = useUserProfilePresenter();

  if (state.isInitialLoading) return <div>Loading...</div>;
  if (state.error) return <div>Error</div>;
  if (!state.user) return null;

  return (
    <main>
      <button onClick={actions.handleBack}>Back</button>

      <UserAvatar user={state.user} />

      <UpdateProfileForm
        user={state.user}
        isSaving={state.isSaving}
        onSubmit={actions.updateProfile}
      />
    </main>
  );
}
```

---

# 7. API Architecture

## Main Rule

The API layer should return domain models, not DTOs.

DTO is a backend contract.

UI should not depend on backend fields like:

```txt
first_name
created_at
avatar_url
```

Bad:

```tsx
<p>{userDto.first_name}</p>
```

Good:

```tsx
<p>{user.fullName}</p>
```

---

# 8. DTO, Domain Model, Mapper

## DTO

DTO repeats the backend response structure.

```ts
// entities/user/api/user.dto.ts

export type UserDto = {
  id: string;
  first_name: string;
  last_name: string;
  avatar_url: string | null;
  created_at: string;
};
```

## Domain Type

Domain type is convenient for frontend code.

```ts
// entities/user/model/user.types.ts

export type User = {
  id: string;
  fullName: string;
  avatarUrl: string | null;
  createdAt: Date;
};
```

## Mapper

Mapper converts DTO into domain model.

```ts
// entities/user/api/user.mapper.ts

import type { UserDto } from './user.dto';
import type { User } from '../model/user.types';

export function mapUserDtoToUser(dto: UserDto): User {
  return {
    id: dto.id,
    fullName: `${dto.first_name} ${dto.last_name}`,
    avatarUrl: dto.avatar_url,
    createdAt: new Date(dto.created_at),
  };
}
```

This gives you control over backend changes.

If backend renames `first_name` to `firstName`, only the mapper should change.

---

# 9. Runtime Validation with Zod

TypeScript checks types during development, but it does not guarantee that the backend actually returned the expected data.

For external data, it is useful to validate responses at runtime.

Use Zod for this.

```ts
// entities/user/api/user.schema.ts

import { z } from 'zod';

export const userDtoSchema = z.object({
  id: z.string(),
  first_name: z.string(),
  last_name: z.string(),
  avatar_url: z.string().nullable(),
  created_at: z.string(),
});

export type UserDto = z.infer<typeof userDtoSchema>;
```

Then validate API response:

```ts
// entities/user/api/user.api.ts

import { httpClient } from '@/shared/api/httpClient';
import { userDtoSchema } from './user.schema';
import { mapUserDtoToUser } from './user.mapper';

export async function getUser(userId: string) {
  const response = await httpClient.get(`/users/${userId}`);

  const dto = userDtoSchema.parse(response);

  return mapUserDtoToUser(dto);
}
```

---

# 10. HTTP Client

Low-level HTTP logic should live in:

```txt
shared/api
```

Example:

```ts
// shared/api/httpClient.ts

import { ApiError } from './ApiError';

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;

type RequestOptions = RequestInit & {
  auth?: boolean;
};

async function request<T>(
  path: string,
  options: RequestOptions = {},
): Promise<T> {
  const response = await fetch(`${API_BASE_URL}${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
  });

  if (!response.ok) {
    throw new ApiError({
      status: response.status,
      message: response.statusText,
    });
  }

  return response.json() as Promise<T>;
}

export const httpClient = {
  get: <T>(path: string, options?: RequestOptions) =>
    request<T>(path, { ...options, method: 'GET' }),

  post: <T>(path: string, body?: unknown, options?: RequestOptions) =>
    request<T>(path, {
      ...options,
      method: 'POST',
      body: JSON.stringify(body),
    }),

  patch: <T>(path: string, body?: unknown, options?: RequestOptions) =>
    request<T>(path, {
      ...options,
      method: 'PATCH',
      body: JSON.stringify(body),
    }),

  delete: <T>(path: string, options?: RequestOptions) =>
    request<T>(path, { ...options, method: 'DELETE' }),
};
```

Important rule:

```txt
shared/api/httpClient.ts should not know about User, Order, Product, or any business entity.
```

---

# 11. Server State

For backend data, use:

```txt
TanStack Query
```

Do not manually store backend data in:

```txt
useState
Zustand
Redux
```

unless there is a very specific reason.

Server state examples:

```txt
- current user
- products
- orders
- comments
- invoices
- permissions
```

Example query keys:

```ts
// entities/user/model/user.keys.ts

export const userKeys = {
  all: ['users'] as const,
  byId: (userId: string) => [...userKeys.all, userId] as const,
};
```

Query hook:

```ts
// entities/user/model/user.queries.ts

import { useQuery } from '@tanstack/react-query';
import { getUser } from '../api/user.api';
import { userKeys } from './user.keys';

export function useUser(userId: string) {
  return useQuery({
    queryKey: userKeys.byId(userId),
    queryFn: () => getUser(userId),
    enabled: Boolean(userId),
  });
}
```

Mutation:

```ts
// features/user/update-profile/model/useUpdateProfile.ts

import { useMutation, useQueryClient } from '@tanstack/react-query';
import { updateProfile } from '../api/updateProfile.api';
import { userKeys } from '@/entities/user';

export function useUpdateProfile() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateProfile,
    onSuccess: async (_, variables) => {
      await queryClient.invalidateQueries({
        queryKey: userKeys.byId(variables.userId),
      });
    },
  });
}
```

---

# 12. Client State

State should be divided by type.

## 1. Local UI State

Examples:

```txt
- dropdown opened or closed
- active tab
- local modal state
- expanded section
- hover state
```

Use:

```txt
useState
useReducer
custom hooks
```

Keep it close to the component.

---

## 2. Server State

Examples:

```txt
- user
- orders
- products
- permissions
- comments
```

Use:

```txt
TanStack Query
```

Do not duplicate this state in Zustand or Redux.

---

## 3. URL State

Examples:

```txt
- search value
- filters
- pagination
- sorting
- selected tab
```

Use:

```txt
React Router search params
```

Good rule:

> If the user should be able to copy the URL and keep the same screen state, store it in the URL.

---

## 4. Global Client State

Examples:

```txt
- theme
- sidebar collapsed
- local UI preferences
- client-side feature flags
- draft state
```

Use:

```txt
Zustand
```

Do not use global state for everything.

---

## 5. Form State

Use:

```txt
React Hook Form
Zod
@hookform/resolvers
```

Example:

```txt
features/user/update-profile/model/updateProfile.schema.ts
features/user/update-profile/model/useUpdateProfileForm.ts
```

---

# 13. Important State Rule

Do not store state that can be calculated.

Bad:

```tsx
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');
```

Good:

```tsx
const fullName = `${firstName} ${lastName}`;
```

State should be minimal.

---

# 14. Routing

Routing should live in:

```txt
app/router/routes.tsx
```

Example:

```tsx
// app/router/routes.tsx

import { createBrowserRouter } from 'react-router';
import { UserProfilePage } from '@/pages/user-profile';

export const router = createBrowserRouter([
  {
    path: '/',
    lazy: async () => {
      const { HomePage } = await import('@/pages/home');
      return { Component: HomePage };
    },
  },
  {
    path: '/users/:userId',
    Component: UserProfilePage,
  },
]);
```

Practical rule:

```txt
- Route params and search params are read in the page presenter.
- API loading is usually handled by TanStack Query.
- React Router loaders are useful for critical route-level data, preloading, guards, and framework-style routing.
```

---

# 15. Forms

A form usually belongs to a feature if it represents a business action.

Example:

```txt
features/user/update-profile/
  ui/
    UpdateProfileForm.tsx
  model/
    updateProfile.schema.ts
    useUpdateProfileForm.ts
  api/
    updateProfile.api.ts
    updateProfile.dto.ts
```

Schema:

```ts
// features/user/update-profile/model/updateProfile.schema.ts

import { z } from 'zod';

export const updateProfileSchema = z.object({
  firstName: z.string().min(1),
  lastName: z.string().min(1),
});

export type UpdateProfileFormValues = z.infer<typeof updateProfileSchema>;
```

Form hook:

```tsx
// features/user/update-profile/model/useUpdateProfileForm.ts

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import {
  updateProfileSchema,
  type UpdateProfileFormValues,
} from './updateProfile.schema';

export function useUpdateProfileForm(defaultValues: UpdateProfileFormValues) {
  return useForm<UpdateProfileFormValues>({
    defaultValues,
    resolver: zodResolver(updateProfileSchema),
  });
}
```

---

# 16. Public API with `index.ts`

Each slice should expose only what is allowed to be used from outside.

Example:

```ts
// entities/user/index.ts

export type { User } from './model/user.types';
export { useUser } from './model/user.queries';
export { UserAvatar } from './ui/UserAvatar/UserAvatar';
export { userKeys } from './model/user.keys';
```

Bad:

```ts
import { mapUserDtoToUser } from '@/entities/user/api/user.mapper';
```

Good:

```ts
import { useUser, UserAvatar } from '@/entities/user';
```

DTOs, mappers, and private helpers are usually not exported.

---

# 17. Error Handling

Create a single API error class.

```ts
// shared/api/ApiError.ts

type ApiErrorParams = {
  status: number;
  message: string;
  code?: string;
};

export class ApiError extends Error {
  status: number;
  code?: string;

  constructor({ status, message, code }: ApiErrorParams) {
    super(message);

    this.name = 'ApiError';
    this.status = status;
    this.code = code;
  }
}
```

Rules:

```txt
- httpClient normalizes errors
- API functions do not show toasts
- query/mutation hooks handle cache and invalidation
- page or feature decides how to show the error to the user
```

API layer should not do this:

```ts
toast.error('Something went wrong');
```

This belongs to UI, page, or feature logic.

---

# 18. Dependency Rules

Good dependency direction:

```txt
app
  imports pages, widgets, features, entities, shared

pages
  imports widgets, features, entities, shared

widgets
  imports features, entities, shared

features
  imports entities, shared

entities
  imports shared

shared
  imports nothing from app/pages/widgets/features/entities
```

Bad:

```ts
// entities/user/model/user.ts

import { routes } from '@/app/router/routes';
```

Bad:

```ts
// shared/ui/Button/Button.tsx

import { useUser } from '@/entities/user';
```

Good:

```ts
// features/user/update-profile

import { User } from '@/entities/user';
import { Button } from '@/shared/ui/Button';
```

---

# 19. Basic Stack

## Core

```txt
React
TypeScript
Vite
React Router
CSS Modules
```

## Data / API

```txt
@tanstack/react-query
zod
fetch or axios
orval / openapi-typescript
msw
```

Use:

```txt
TanStack Query -> server state
Zod -> validation
MSW -> API mocks for tests and development
Orval/OpenAPI TypeScript -> generated API types from OpenAPI
```

## State

```txt
@tanstack/react-query -> server state
zustand -> global client state
react-hook-form -> form state
```

Redux Toolkit can be used, but I would not add Redux by default unless the project already has a strong reason for it.

## UI

```txt
CSS Modules
clsx
Radix UI
Storybook
```

Use Radix UI if you need accessible headless primitives.

Use Storybook if you have many reusable UI components or a design system.

## Testing

```txt
vitest
@testing-library/react
@testing-library/user-event
@testing-library/jest-dom
msw
playwright
```

Use:

```txt
Vitest -> unit tests
React Testing Library -> component tests
MSW -> mocked API
Playwright -> end-to-end tests
```

## Tooling

```txt
eslint
typescript-eslint
prettier
eslint-plugin-react-hooks
eslint-plugin-import or eslint-plugin-boundaries
husky
lint-staged
```

---

# 20. Basic Install

```bash
npm create vite@latest my-app -- --template react-ts

npm i react-router @tanstack/react-query zod react-hook-form @hookform/resolvers clsx zustand

npm i -D vitest jsdom @testing-library/react @testing-library/user-event @testing-library/jest-dom msw eslint @eslint/js typescript-eslint prettier
```

For OpenAPI:

```bash
npm i -D orval
```

or:

```bash
npm i -D openapi-typescript
```

---

# 21. When to Create Page, Feature, Entity, Widget, Shared

## Keep inside page when:

```txt
- it is used only on one page
- it has no independent business meaning
- it is small
- reuse is not needed yet
```

## Move to `shared/ui` when:

```txt
- it has no business logic
- it can be reused in any project
- it is a primitive component
```

Examples:

```txt
Button
Input
Modal
Select
Checkbox
Spinner
Skeleton
```

## Move to `entity` when:

```txt
- it is related to a business object
- it is reused in multiple places
- it displays User, Product, Order, etc.
```

Examples:

```txt
UserAvatar
ProductCard
OrderStatusBadge
CommentItem
```

## Move to `feature` when:

```txt
- it is a user action
- it has form, mutation, or validation
- it represents a reusable business scenario
```

Examples:

```txt
LoginForm
UpdateProfileForm
AddToCartButton
CancelOrderButton
```

## Move to `widget` when:

```txt
- it is a large UI block
- it combines features/entities/shared
- it is used on several pages
```

Examples:

```txt
AppHeader
Sidebar
OrderSummary
DashboardLayout
```

---

# 22. Checklist for Every New Page

Before creating a new page:

```txt
1. Create a slice inside pages/page-name.
2. Put the page component into ui.
3. Put local components into ui/components.
4. Put presenter into model/usePageNamePresenter.ts.
5. Do not call API directly from the page.
6. Use entities/features hooks for API data.
7. Do not pass DTOs into UI.
8. Store search/filter/pagination in the URL.
9. Store server data in TanStack Query.
10. Store form state in React Hook Form.
11. Move reusable code only when reuse is real.
```

---

# 23. Final Recommended Stack

```txt
React
TypeScript
Vite
React Router
Feature-Sliced Design
TanStack Query
Zustand
React Hook Form
Zod
CSS Modules
clsx
Vitest
React Testing Library
MSW
Playwright
ESLint
Prettier
```

---

# 24. Final Principle

```txt
UI should not know about backend DTOs.

shared should not know about business logic.

entity should not know about feature.

feature should not know about page.

page composes everything together.

presenter orchestrates the page, but should not become a business layer.

server state belongs to TanStack Query.

client UI state belongs close to the component.

global client state should be rare and intentional.
```
