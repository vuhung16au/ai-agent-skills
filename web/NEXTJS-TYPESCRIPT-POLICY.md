# NEXTJS-TYPESCRIPT-POLICY.md

## Purpose

Define standards for building Next.js applications with TypeScript. This policy covers App Router conventions, component boundaries, server/client separation, type safety, accessibility, and Vercel deployment readiness.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- Next.js 13+ applications using the App Router.
- React Server Components (RSC) and Client Components.
- API Routes and Route Handlers.
- TypeScript configuration and type definitions.
- Deployment configurations targeting Vercel.

---

## Core Principles

- **App Router first.** Use the `app/` directory and React Server Components by default.
- **Server/client boundary is explicit.** Default to server; add `"use client"` only when necessary.
- **TypeScript strictness is non-negotiable.** Use `strict: true` in `tsconfig.json`.
- **Components are small and focused.** One responsibility per component.
- **Accessibility is built in, not added later.**

---

## Required Behavior

### Project Structure
- Use the `app/` directory (App Router). Do not mix `pages/` and `app/` in new projects.
- Colocate components, hooks, and utilities with the feature they serve unless shared across multiple features.
- Place shared UI components in `components/`, shared utilities in `lib/`, and types in `types/`.
- Use route groups `(groupName)` for layout segmentation without affecting URL structure.

### TypeScript
- Enable `strict: true` in `tsconfig.json`.
- Never use `any` — use `unknown` when the type is genuinely unknown and narrow it explicitly.
- Define explicit return types for all functions that return JSX or complex objects.
- Use `interface` for object shapes that may be extended; use `type` for unions, intersections, and aliases.
- Use `zod` or equivalent for runtime validation of external data (API responses, form inputs).

### Server and Client Components
- All components are Server Components by default — do not add `"use client"` unless the component uses:
  - Browser APIs (`window`, `document`, `navigator`).
  - React hooks (`useState`, `useEffect`, `useContext`, etc.).
  - Event handlers.
- Push `"use client"` boundaries as far down the tree as possible.
- Never import a Client Component directly in a Server Component without checking whether the dependency chain forces unnecessary client-side rendering.
- Fetch data in Server Components using `async`/`await` directly; do not use `useEffect` for data fetching.

```tsx
// Server Component (no "use client" needed)
export default async function UserProfile({ params }: { params: { id: string } }) {
  const user = await fetchUser(params.id); // direct async call
  return <ProfileCard user={user} />;
}
```

### Data Fetching
- Use `fetch()` with Next.js cache options (`{ cache: "force-cache" }`, `{ next: { revalidate: N } }`, or `{ cache: "no-store" }`).
- Validate and type-check all fetched data at the boundary.
- Handle loading and error states with `loading.tsx` and `error.tsx` files in the relevant route segment.

### Route Handlers (API)
- Define route handlers in `app/api/.../route.ts`.
- Export named HTTP method functions (`GET`, `POST`, `PUT`, `DELETE`).
- Return `NextResponse.json()` with appropriate status codes.
- Validate request bodies before processing.

```ts
export async function POST(request: Request) {
  const body = await request.json();
  const parsed = schema.safeParse(body);
  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error }, { status: 400 });
  }
  // ...
}
```

### Component Design
- Use function components only. No class components.
- Destructure props with explicit TypeScript types at the parameter level.
- Use `React.FC` sparingly; prefer explicit return types.
- Keep components under 150 lines; extract sub-components when they grow larger.

### Styling
- Use Tailwind CSS or CSS Modules — not inline `style` objects for anything beyond dynamic values.
- Never use global CSS selectors that could leak across components.
- Apply responsive classes using Tailwind's breakpoint prefixes (`sm:`, `md:`, `lg:`).

### Accessibility
- All interactive elements must be keyboard-navigable.
- Images must have descriptive `alt` text or `alt=""` for decorative images.
- Use semantic HTML elements (`<nav>`, `<main>`, `<section>`, `<button>`) instead of generic `<div>` wrappers.
- Ensure sufficient color contrast (WCAG AA minimum).
- Add `aria-label` or `aria-describedby` where the visual context is insufficient for screen readers.

### Performance
- Use `next/image` for all images to enable automatic optimization.
- Use `next/link` for all internal navigation.
- Use `next/font` for web font loading.
- Lazy-load heavy Client Components with `React.lazy` and `Suspense`.

### Vercel Deployment
- Do not rely on filesystem writes in production — use edge-compatible storage.
- Use environment variables prefixed with `NEXT_PUBLIC_` only for client-safe values.
- Set `output: "standalone"` in `next.config.js` only when deploying outside Vercel.
- Use Vercel-native features (`revalidateTag`, `revalidatePath`) for cache invalidation.

---

## Things to Avoid

- Using `pages/` directory for new features in an App Router project.
- Adding `"use client"` to Server Components unnecessarily.
- Fetching data in `useEffect` when a Server Component can fetch it directly.
- Using `any` type anywhere in TypeScript code.
- Unvalidated external data flowing into components or database queries.
- Inline styles for layout and design decisions.
- Missing error and loading boundaries in route segments.

---

## Output Expectations

Generated Next.js/TypeScript code must:
- Use App Router conventions (`app/` directory, `layout.tsx`, `page.tsx`).
- Default to Server Components; add `"use client"` only when justified with a comment.
- Have all props and function returns explicitly typed.
- Include `zod` validation for external data.
- Include `loading.tsx` and `error.tsx` stubs for route segments with async data.
- Be deployable to Vercel without modification.

---

## Optional Checklist

- [ ] `strict: true` in `tsconfig.json`.
- [ ] No `any` types.
- [ ] `"use client"` used only where necessary.
- [ ] Data fetching in Server Components, not `useEffect`.
- [ ] External data validated with `zod`.
- [ ] All images use `next/image`.
- [ ] All internal links use `next/link`.
- [ ] Semantic HTML elements used.
- [ ] `alt` text on all images.
- [ ] Route handler validates request body before processing.
