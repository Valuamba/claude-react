---
name: react-architecture
description: >
  Frontend architecture guideline for React + TypeScript + Vite projects.
  Enforces Feature-Sliced Design (FSD) layers, usePresenter pattern,
  DTO/mapper/domain model separation, TanStack Query for server state,
  and correct dependency direction between layers.
  Use when creating, scaffolding, reviewing, or refactoring React application code.
---

You are a frontend architecture assistant. Follow the guideline in `reference.md` strictly when generating or reviewing React + TypeScript code.

## When to activate

Claude SHOULD invoke this skill automatically when the user:

- Creates a new React page, feature, entity, widget, or shared component
- Scaffolds a new React + TypeScript + Vite project
- Asks where to place a component, hook, API call, or type
- Asks about project structure or architecture decisions
- Creates or refactors API integration code (DTOs, mappers, queries)
- Works with TanStack Query, React Hook Form, Zustand, or Zod in an FSD context
- Asks about dependency direction between layers
- Requests a code review of React application structure

## Core rules (always enforce)

1. **FSD layers**: `app > pages > widgets > features > entities > shared`. Never import upward.
2. **Page structure**: `ui/` for JSX, `model/` for presenter/logic, `index.ts` as public API.
3. **usePresenter pattern**: page-level hook returns `{ state, actions }`. No JSX, no direct DTO access, no raw fetch.
4. **DTO -> Mapper -> Domain Model**: UI never touches DTOs. Mappers live in the entity/feature `api/` folder.
5. **Zod validation**: validate external data at runtime with Zod schemas before mapping.
6. **Server state**: TanStack Query only. Never store server data in `useState`, Zustand, or Redux.
7. **Client state**: local UI state stays in `useState`/`useReducer` close to the component. Global client state uses Zustand sparingly.
8. **URL state**: search, filters, pagination, sorting belong in React Router search params.
9. **Form state**: React Hook Form + Zod + `@hookform/resolvers`.
10. **Public API**: each slice exports only its public interface via `index.ts`. No deep imports into `api/`, `model/`, or internal components.
11. **shared layer**: must not know about any business entity (User, Order, Product, etc.).
12. **HTTP client**: lives in `shared/api/`. Must not contain business logic or toasts.
13. **Error handling**: API layer throws `ApiError`. UI layer decides how to display errors.

## How to apply

When generating code:
- Read `reference.md` for the full guideline, code examples, and recommended stack.
- Place files in the correct FSD layer and follow the naming conventions from the reference.
- Use the exact patterns (query keys, mappers, schemas, presenters) shown in the reference.

When reviewing code:
- Check dependency direction violations (lower layers importing higher layers).
- Flag DTOs leaking into UI components.
- Flag server state stored in useState/Zustand instead of TanStack Query.
- Flag missing Zod validation on API responses.
- Flag presenters that contain JSX or direct fetch logic.
