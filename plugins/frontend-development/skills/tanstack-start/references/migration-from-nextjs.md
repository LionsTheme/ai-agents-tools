# Migration from Next.js to TanStack Start

Step-by-step guide to migrate Next.js applications (App Router and Pages Router) to TanStack Start.

## Equivalent Reference

| Next.js                                      | TanStack Start                               |
| -------------------------------------------- | -------------------------------------------- |
| `"use server"` Server Action                 | `createServerFn().handler()`                 |
| `"use client"` Client Component              | Regular component (everything is client-first)|
| `async function Page()` (Server Component)   | `createFileRoute` + `loader`                 |
| `generateStaticParams()`                     | `loader` without dynamic logic + build compile|
| `cookies()` / `headers()`                    | `getRequest()`, `getRequestHeader()`          |
| `revalidatePath()` / `revalidateTag()`       | `router.invalidate()`                        |
| `redirect()`                                 | `throw redirect()`                            |
| `notFound()`                                 | `throw new Error()` + `notFoundComponent`    |
| `layout.tsx`                                 | `__root.tsx` or `_layout.tsx`                |
| `page.tsx`                                   | `index.tsx`                                  |
| `loading.tsx`                                | `<Suspense>` + `defer()` in loader           |
| `error.tsx`                                  | `errorComponent` on route                    |
| `not-found.tsx`                              | `notFoundComponent` on route                 |
| `route.ts` (API Route)                       | `createServerFn({ method: 'GET' })`          |
| `metadata` / `generateMetadata()`            | `head: () => ({ meta: [...], links: [...] })`|

## Step 1: Create New Project

```bash
pnpx @tanstack/cli@latest create my-app
cd my-app
pnpm i
```

## Step 2: Migrate Server Actions → Server Functions

### Next.js (before)
```tsx
// app/actions/users.ts
'use server'

import { z } from 'zod'

const UserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
})

export async function createUser(formData: FormData) {
  const data = UserSchema.parse({
    name: formData.get('name'),
    email: formData.get('email'),
  })

  await db.user.create({ data })
  revalidatePath('/users')
}
```

### TanStack Start (after)
```tsx
// server/users.ts
import { createServerFn } from '@tanstack/react-start'
import { zodValidator } from '@tanstack/zod-adapter'
import { z } from 'zod'

const UserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
})

export const createUser = createServerFn({ method: 'POST' })
  .inputValidator(zodValidator(UserSchema))
  .handler(async ({ data }) => {
    // data is validated and typed
    await db.user.create({ data })
    return { success: true }
  })

// In the component:
// import { useRouter } from '@tanstack/react-router'
// const router = useRouter()
// await createUser({ data: { name, email } })
// router.invalidate()  // Refreshes loaders
```

## Step 3: Migrate Server Components → Loaders

### Next.js (before) — App Router
```tsx
// app/posts/page.tsx
export default async function PostsPage() {
  const posts = await db.post.findMany()
  return <PostList posts={posts} />
}
```

### TanStack Start (after)
```tsx
// src/routes/posts/index.tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/posts/')({
  loader: async () => {
    const posts = await db.post.findMany()
    return posts
  },
  component: PostsPage,
})

function PostsPage() {
  const posts = Route.useLoaderData()
  return <PostList posts={posts} />
}
```

## Step 4: Migrate Route Handlers → Server Functions

### Next.js (before)
```tsx
// app/api/users/route.ts
export async function GET(request: NextRequest) {
  const users = await db.user.findMany()
  return NextResponse.json(users)
}

export async function POST(request: NextRequest) {
  const body = await request.json()
  const user = await db.user.create({ data: body })
  return NextResponse.json(user, { status: 201 })
}
```

### TanStack Start (after)
```tsx
// server/users.ts
import { createServerFn } from '@tanstack/react-start'

export const getUsers = createServerFn({ method: 'GET' })
  .handler(async () => {
    return await db.user.findMany()
  })

export const createUser = createServerFn({ method: 'POST' })
  .inputValidator((data: CreateUserInput) => data)
  .handler(async ({ data }) => {
    return await db.user.create({ data })
  })

// Call from anywhere (client or server)
// const users = await getUsers()
// const newUser = await createUser({ data: { name, email } })
```

## Step 5: Migrate Layouts

### Next.js (before)
```
app/
├── layout.tsx      → <html><body>{children}</body></html>
├── page.tsx
├── dashboard/
│   ├── layout.tsx  → Nested layout
│   └── page.tsx
```

### TanStack Start (after)
```
src/routes/
├── __root.tsx            → <html><head/><body><Outlet/></body></html>
├── index.tsx
├── _dashboard.tsx        → Nested layout (_ prefix)
└── _dashboard/
    └── index.tsx         → /dashboard
```

```tsx
// __root.tsx — Equivalent to app/layout.tsx
export const Route = createRootRoute({
  head: () => ({
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { title: 'My App' },
    ],
    links: [
      { rel: 'stylesheet', href: appCss },
    ],
  }),
  component: RootComponent,
})

function RootComponent() {
  return (
    <html>
      <head><HeadContent /></head>
      <body>
        <Outlet />
        <Scripts />
      </body>
    </html>
  )
}
```

## Step 6: Migrate `generateMetadata` → `head`

### Next.js (before)
```tsx
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug)
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: { images: [post.image] },
  }
}
```

### TanStack Start (after)
```tsx
export const Route = createFileRoute('/posts/$slug')({
  loader: async ({ params }) => await getPost(params.slug),
  head: ({ loaderData }) => ({
    meta: [
      { title: loaderData?.title },
      { name: 'description', content: loaderData?.excerpt },
      { property: 'og:image', content: loaderData?.image },
    ],
  }),
  component: PostPage,
})
```

## Step 7: Migrate `redirect()` and `notFound()`

### Next.js (before)
```tsx
import { redirect, notFound } from 'next/navigation'

export default async function Page({ params }) {
  const post = await getPost(params.id)
  if (!post) notFound()
  if (!session) redirect('/login')
}
```

### TanStack Start (after)
```tsx
import { redirect } from '@tanstack/react-router'

export const Route = createFileRoute('/posts/$postId')({
  loader: async ({ params }) => {
    const post = await getPost(params.postId)
    if (!post) throw new Error('Not found')  // Shows notFoundComponent
    return post
  },
  beforeLoad: async ({ location }) => {
    if (!session) {
      throw redirect({ to: '/login', search: { redirect: location.href } })
    }
  },
  component: PostPage,
})
```

## Step 8: Migrate `generateStaticParams`

### Next.js (before)
```tsx
export async function generateStaticParams() {
  const posts = await db.post.findMany({ select: { slug: true } })
  return posts.map((p) => ({ slug: p.slug }))
}
```

### TanStack Start (after)
TanStack Start doesn't have explicit `generateStaticParams`. Use Static Generation with `staticFunctionMiddleware` or simply define normal routes with loaders (default behavior is SSR + SWR):

```tsx
// With static generation:
import { staticFunctionMiddleware } from '@tanstack/start-static-server-functions'

const getStaticPost = createServerFn({ method: 'GET' })
  .middleware([staticFunctionMiddleware])
  .handler(async () => {
    return await db.post.findMany({ select: { slug: true } })
  })
```

## Step 9: Migrate Caching

### Next.js
```tsx
// Time-based revalidation
fetch(url, { next: { revalidate: 3600 } })

// Tag-based revalidation
revalidateTag('posts')
revalidatePath('/posts')
```

### TanStack Start
```tsx
// SWR Cache in router
export const Route = createFileRoute('/posts/')({
  loader: async () => fetchPosts(),
  staleTime: 30_000,   // Revalidates every 30s (SWR)
  gcTime: 5 * 60_000,  // Garbage collection: 5min
})

// Cache headers for CDN/ISR
export const Route = createFileRoute('/blog/$slug')({
  loader: async ({ params }) => fetchPost(params.slug),
  headers: () => ({
    'Cache-Control': 'public, max-age=3600, stale-while-revalidate=604800',
  }),
})

// Manual invalidation after mutation
const router = useRouter()
router.invalidate()  // Re-runs active loaders
```

## Migration Checklist

- [ ] Create project with `@tanstack/cli create`
- [ ] Migrate Server Actions to `createServerFn().handler()`
- [ ] Migrate Server Components to `createFileRoute` + `loader`
- [ ] Migrate Route Handlers to `createServerFn({ method })`
- [ ] Set up `__root.tsx` with `<html>`, `<head>`, `<body>`
- [ ] Migrate nested layouts to `_layout.tsx` (`_` prefix)
- [ ] Migrate `generateMetadata` to `head` on route
- [ ] Migrate `redirect()` to `throw redirect()`
- [ ] Migrate `notFound()` to `notFoundComponent` or `errorComponent`
- [ ] Replace `revalidatePath/Tag` with `router.invalidate()`
- [ ] Migrate `cookies()`/`headers()` to `getRequest()`/`getRequestHeader()`
- [ ] Update environment variables: `NEXT_PUBLIC_*` → `VITE_*`
- [ ] Configure `vite.config.ts` with `tanstackStart()` plugin
- [ ] Update `package.json`: scripts with Vite
- [ ] Configure deployment target in `vite.config.ts`
- [ ] Test SSR with `curl` (must return HTML)
- [ ] Test server functions with fetch/curl
- [ ] Verify typed routes (no typos in paths)
