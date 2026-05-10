---
name: tanstack-start
description: TanStack Start (RC) full-stack React with server functions, SSR, streaming, typed routing, and multi-platform deployment. Use when building full-stack React apps, migrating from Next.js, needing type-safe server functions, SSR with Vite, or deploying to Vercel/Cloudflare/Node.js.
---

# TanStack Start

TanStack Start is a full-stack framework for React (also supports Solid) powered by TanStack Router and Vite. It provides full-document SSR, streaming, server functions, bundling, and deployment to any hosting provider or runtime.

## When to Use This Skill

- Starting a full-stack project with React and Vite
- Migrating from Next.js App Router or Pages Router
- Implementing server functions with extreme type-safety (RPC)
- Building APIs with typed validation (Zod integrated)
- Configuring SSR, streaming, ISR, or Static Generation
- Deploying to Vercel, Cloudflare Workers, Netlify, Node.js, Bun, or Docker
- Integrating TanStack Query for data fetching
- Implementing authentication with sessions and cookies

## Core Concepts

### 1. Architecture

| Concept                | Description                                                      |
| ---------------------- | ---------------------------------------------------------------- |
| **File-based Routing** | Routes defined by files in `src/routes/`                         |
| **Server Functions**   | Typed RPC: functions that run on server but are called from client |
| **Loaders**            | Data preloaded on server before rendering route                  |
| **SSR**                | Server-Side Rendering: HTML rendered on server                   |
| **Streaming**          | Progressive HTML delivery with `<Suspense>` + `defer`            |
| **Static Generation**  | Prerendered at build time with `staticServerFn`                  |
| **ISR**                | Incremental Static Regeneration via `Cache-Control` headers      |
| **Client-first**       | Router works on client immediately; loader data is hydrated      |

### 2. Next.js vs TanStack Start Comparison

| Feature                   | Next.js App Router                | TanStack Start                    |
| ------------------------- | --------------------------------- | --------------------------------- |
| **Server functions**      | `"use server"` + Server Actions   | `createServerFn()` typed (RPC)    |
| **Type safety**           | Partial (manual types)            | Complete (data, params, context)  |
| **Router**                | `next/navigation`                 | TanStack Router (typed, devtools) |
| **Data fetching**         | `async` Server Components + fetch | `loader` + `useLoaderData()`      |
| **Caching**               | Heuristic cache                   | SWR with `staleTime`/`gcTime`     |
| **Bundler**               | Turbopack / Webpack               | Vite                              |
| **Deployment**            | Vercel-first                      | Multi-platform (Vercel, CF, Node, Netlify, Bun) |
| **Typed routes**          | No                                | Path params + search params typed with validation |

### 3. Project Structure

```
my-app/
├── src/
│   ├── routes/
│   │   ├── __root.tsx              # Root layout (required)
│   │   ├── index.tsx               # Home page (/)
│   │   ├── about.tsx               # /about route
│   │   ├── posts/
│   │   │   ├── index.tsx           # /posts
│   │   │   └── $postId.tsx         # /posts/:postId (dynamic)
│   │   ├── _authenticated/         # Layout group (_ prefix)
│   │   │   ├── dashboard.tsx       # /dashboard (protected)
│   │   │   └── settings.tsx        # /settings
│   │   └── api/
│   │       └── users.ts            # API route (optional)
│   ├── router.tsx                  # Router configuration
│   ├── routeTree.gen.ts            # Generated route tree (auto)
│   └── styles.css                  # Global styles
├── public/                         # Static assets
├── vite.config.ts                  # Vite + TanStack Start config
├── package.json
└── tsconfig.json
```

## Quick Start

### Create a Project

```bash
# Using the official CLI
pnpx @tanstack/cli@latest create my-app
cd my-app
pnpm i
pnpm dev
```

```json
// package.json
{
  "type": "module",
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "start": "node .output/server/index.mjs"
  }
}
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import { tanstackStart } from '@tanstack/react-start/plugin/vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [
    tanstackStart(),  // Main TanStack Start plugin
    react(),
  ],
})
```

## Essential Patterns

### Pattern 1: Root Layout (`__root.tsx`)

Every app needs a root layout. Defines metadata, styles, navigation, and HTML structure:

```tsx
// src/routes/__root.tsx
import {
  HeadContent,
  Link,
  Outlet,
  Scripts,
  createRootRoute,
} from '@tanstack/react-router'
import { TanStackRouterDevtools } from '@tanstack/react-router-devtools'
import appCss from '../styles/app.css?url'

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
      <head>
        <HeadContent />
      </head>
      <body>
        <nav className="flex gap-4 p-4 bg-gray-100">
          <Link to="/" activeProps={{ className: 'font-bold' }}>
            Home
          </Link>
          <Link to="/posts" activeProps={{ className: 'font-bold' }}>
            Posts
          </Link>
        </nav>
        <main className="p-4">
          <Outlet />
        </main>
        <TanStackRouterDevtools position="bottom-right" />
        <Scripts />
      </body>
    </html>
  )
}
```

### Pattern 2: Create a Page (Simple Route)

```tsx
// src/routes/index.tsx — Home page at /
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/')({
  component: HomePage,
})

function HomePage() {
  return (
    <div>
      <h1 className="text-3xl font-bold">Welcome to TanStack Start</h1>
      <p>Full-stack framework with Vite and React.</p>
    </div>
  )
}
```

### Pattern 3: Route with Loader (Server Data Fetching)

Loaders run on the server during SSR and on the client during subsequent navigations (SWR):

```tsx
// src/routes/posts/index.tsx — Post list
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/posts/')({
  loader: async () => {
    const res = await fetch('https://api.example.com/posts')
    if (!res.ok) throw new Error('Error fetching posts')
    return res.json()
  },
  component: PostsPage,
})

function PostsPage() {
  const posts = Route.useLoaderData()

  return (
    <div>
      <h1 className="text-2xl font-bold mb-4">Posts</h1>
      <ul className="space-y-2">
        {posts.map((post: Post) => (
          <li key={post.id}>
            <Link to="/posts/$postId" params={{ postId: post.id }}>
              {post.title}
            </Link>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

### Pattern 4: Dynamic Route with Typed Parameters

```tsx
// src/routes/posts/$postId.tsx — Post detail
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/posts/$postId')({
  loader: async ({ params }) => {
    const res = await fetch(`https://api.example.com/posts/${params.postId}`)
    if (!res.ok) throw new Error('Post not found')
    return res.json()
  },
  component: PostDetailPage,
})

function PostDetailPage() {
  const { postId } = Route.useParams()   // Typed: { postId: string }
  const post = Route.useLoaderData()

  return (
    <div>
      <Link to="/posts" className="text-blue-500">&larr; Back</Link>
      <h1 className="text-3xl font-bold mt-2">{post.title}</h1>
      <p className="text-gray-600 mt-4">{post.body}</p>
    </div>
  )
}
```

### Pattern 5: Server Functions — Typed CRUD

Server functions are the heart of TanStack Start: RPC functions that run on the server but are called from the client with full type-safety:

```tsx
// src/routes/posts/$postId.tsx (continued)
import { createServerFn } from '@tanstack/react-start'
import { useRouter } from '@tanstack/react-router'

// Server function with Zod validation
const deletePost = createServerFn({ method: 'POST' })
  .inputValidator((data: { postId: string }) => data)  // Zod recommended for complex schemas
  .handler(async ({ data }) => {
    // This code NEVER reaches the client
    await db.posts.delete({ where: { id: data.postId } })
    return { success: true }
  })

const updatePost = createServerFn({ method: 'POST' })
  .inputValidator((data: { postId: string; title: string; body: string }) => data)
  .handler(async ({ data }) => {
    return await db.posts.update({
      where: { id: data.postId },
      data: { title: data.title, body: data.body },
    })
  })

function PostDetailPage() {
  const router = useRouter()
  const post = Route.useLoaderData()

  const handleDelete = async () => {
    try {
      await deletePost({ data: { postId: post.id } })
      router.invalidate()  // Re-runs loaders
      router.navigate({ to: '/posts' })
    } catch (error) {
      console.error('Delete failed:', error)
    }
  }

  return (
    <div>
      {/* ... content ... */}
      <button
        onClick={handleDelete}
        className="bg-red-500 text-white px-4 py-2 rounded"
      >
        Delete Post
      </button>
    </div>
  )
}
```

### Pattern 6: Create Data with Forms

```tsx
// src/routes/posts/create.tsx
import { createFileRoute, useRouter } from '@tanstack/react-router'
import { createServerFn } from '@tanstack/react-start'
import { useState } from 'react'

const createPost = createServerFn({ method: 'POST' })
  .inputValidator((data: { title: string; body: string }) => data)
  .handler(async ({ data }) => {
    // Additional validation or business logic here
    if (!data.title.trim()) throw new Error('Title is required')
    return await db.posts.create({ data })
  })

export const Route = createFileRoute('/posts/create')({
  component: CreatePostPage,
})

function CreatePostPage() {
  const router = useRouter()
  const [title, setTitle] = useState('')
  const [body, setBody] = useState('')
  const [isSubmitting, setIsSubmitting] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setIsSubmitting(true)
    setError(null)

    try {
      await createPost({ data: { title, body } })
      setTitle('')
      setBody('')
      router.invalidate()
      router.navigate({ to: '/posts' })
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Error creating post')
    } finally {
      setIsSubmitting(false)
    }
  }

  return (
    <div>
      <h1 className="text-2xl font-bold mb-4">Create Post</h1>
      {error && (
        <div className="bg-red-100 text-red-700 p-2 rounded mb-4">{error}</div>
      )}
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label htmlFor="title" className="block text-sm font-medium">Title</label>
          <input
            id="title"
            type="text"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            className="w-full border rounded p-2"
            required
          />
        </div>
        <div>
          <label htmlFor="body" className="block text-sm font-medium">Body</label>
          <textarea
            id="body"
            value={body}
            onChange={(e) => setBody(e.target.value)}
            className="w-full border rounded p-2"
            rows={4}
            required
          />
        </div>
        <button
          type="submit"
          disabled={isSubmitting}
          className="bg-blue-500 text-white px-4 py-2 rounded disabled:opacity-50"
        >
          {isSubmitting ? 'Creating...' : 'Create Post'}
        </button>
      </form>
    </div>
  )
}
```

### Pattern 7: Server Functions with Zod Validation

For robust validation, use Zod with `zodValidator`:

```tsx
import { createServerFn } from '@tanstack/react-start'
import { zodValidator } from '@tanstack/zod-adapter'
import { z } from 'zod'

const UserSchema = z.object({
  name: z.string().min(1, 'Name required'),
  email: z.string().email('Invalid email'),
  age: z.number().min(0).max(150),
})

export const createUser = createServerFn({ method: 'POST' })
  .inputValidator(zodValidator(UserSchema))
  .handler(async ({ data }) => {
    // data is fully typed and validated
    return `Created: ${data.name} (${data.email})`
  })

// Client-side call: type-safe
// createUser({ data: { name: 'John', email: 'john@example.com', age: 30 } })
```

### Pattern 8: Nested Layouts (Route Groups)

Group routes with shared layouts using the `_` prefix:

```tsx
// src/routes/_authenticated.tsx — Protected layout
import { createFileRoute, Link, Outlet, redirect } from '@tanstack/react-router'

export const Route = createFileRoute('/_authenticated')({
  beforeLoad: async ({ location }) => {
    const user = await getCurrentUser()

    if (!user) {
      throw redirect({
        to: '/login',
        search: { redirect: location.href },
      })
    }

    return { user }  // Available in child routes
  },
  component: AuthenticatedLayout,
})

function AuthenticatedLayout() {
  const { user } = Route.useRouteContext()

  return (
    <div className="flex">
      <aside className="w-64 bg-gray-800 text-white min-h-screen p-4">
        <p className="font-bold mb-4">{user.name}</p>
        <nav className="space-y-2">
          <Link to="/dashboard" className="block">Dashboard</Link>
          <Link to="/settings" className="block">Settings</Link>
          <Link to="/profile" className="block">Profile</Link>
        </nav>
      </aside>
      <main className="flex-1 p-6">
        <Outlet />
      </main>
    </div>
  )
}

// src/routes/_authenticated/dashboard.tsx
export const Route = createFileRoute('/_authenticated/dashboard')({
  component: DashboardPage,
})

function DashboardPage() {
  const { user } = Route.useRouteContext()  // Typed from beforeLoad
  return <h1>Welcome, {user.name}</h1>
}
```

### Pattern 9: Streaming with `defer` + `<Suspense>`

For pages with slow data, use streaming to avoid blocking the initial render:

```tsx
// src/routes/product/$id.tsx
import { createFileRoute } from '@tanstack/react-router'
import { defer } from '@tanstack/react-start'
import { Suspense } from 'react'

export const Route = createFileRoute('/product/$id')({
  loader: async ({ params }) => {
    // Fast load (blocking)
    const product = await getProduct(params.id)

    return {
      product,                                                // Immediate
      reviews: defer(getReviews(params.id)),                 // Stream
      recommendations: defer(getRecommendations(params.id)), // Stream
    }
  },
  component: ProductPage,
})

function ProductPage() {
  const { product, reviews, recommendations } = Route.useLoaderData()

  return (
    <div>
      {/* Immediate render */}
      <ProductHeader product={product} />

      {/* Stream: shown when ready */}
      <Suspense fallback={<Skeleton />}>
        <ReviewsList reviewsPromise={reviews} />
      </Suspense>

      <Suspense fallback={<Skeleton />}>
        <RecommendationsCarousel recommendationsPromise={recommendations} />
      </Suspense>
    </div>
  )
}

// Component consuming streamed data
function ReviewsList({ reviewsPromise }: { reviewsPromise: Promise<Review[]> }) {
  const reviews = React.use(reviewsPromise)  // React 19: use() for promises
  return <ul>{reviews.map(r => <li key={r.id}>{r.text}</li>)}</ul>
}
```

### Pattern 10: TanStack Query Integration

Prefetching in loader + consumption in component:

```tsx
// src/routes/posts/$postId.tsx
import { createFileRoute } from '@tanstack/react-router'
import { useQuery, queryOptions } from '@tanstack/react-query'

const postQueryOptions = (postId: string) =>
  queryOptions({
    queryKey: ['post', postId],
    queryFn: () => fetchPost(postId),
    staleTime: 5 * 60 * 1000,  // 5 minutes client-side
  })

export const Route = createFileRoute('/posts/$postId')({
  loader: async ({ context, params }) => {
    // Prefetch during SSR — reused on client without refetch
    await context.queryClient.ensureQueryData(postQueryOptions(params.postId))
  },
  component: PostPage,
})

function PostPage() {
  const { postId } = Route.useParams()
  const { data: post } = useQuery(postQueryOptions(postId))

  return <PostDetail post={post} />
}
```

**QueryClient setup in the app:**

```tsx
// src/router.tsx
import { createRouter } from '@tanstack/react-router'
import { QueryClient } from '@tanstack/react-query'
import { routeTree } from './routeTree.gen'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000,
      gcTime: 10 * 60_000,
    },
  },
})

export const router = createRouter({
  routeTree,
  context: { queryClient },
  defaultPreloading: 'intent',
})
```

## File Conventions

| File                 | Purpose                                              |
| -------------------- | ---------------------------------------------------- |
| `__root.tsx`         | Root layout (required). HTML, head, scripts.         |
| `index.tsx`          | Directory index route                                |
| `$param.tsx`         | Dynamic segment (e.g., `$postId` → `:postId`)        |
| `_layout.tsx`        | Layout group (`_` prefix): groups routes with layout  |
| `_layout.tsx`        | No own route, only wraps children                    |
| `notFoundComponent`  | Route property for custom 404                        |
| `errorComponent`     | Route property for error boundary                    |

## Navigation API

```tsx
import { Link, useRouter, useNavigate } from '@tanstack/react-router'

// Link with typed params
<Link to="/posts/$postId" params={{ postId: '123' }}>
  View Post
</Link>

// Link with typed search params
<Link to="/search" search={{ q: 'tanstack', page: 1 }}>
  Search
</Link>

// Programmatic navigation
const router = useRouter()
router.navigate({ to: '/posts/$postId', params: { postId: '123' } })
router.navigate({ to: '/search', search: { q: 'react' } })

// Invalidate and reload data
router.invalidate()
```

## Error Handling

```tsx
// Error boundary at route level
export const Route = createFileRoute('/posts/')({
  loader: async () => {
    const res = await fetch('/api/posts')
    if (!res.ok) throw new Error('Failed to fetch posts')
    return res.json()
  },
  errorComponent: ({ error, reset }) => (
    <div className="bg-red-50 border border-red-200 rounded p-4">
      <h3 className="text-red-800 font-bold">Error</h3>
      <p className="text-red-600">{error.message}</p>
      <button
        onClick={reset}
        className="mt-2 bg-red-500 text-white px-4 py-2 rounded"
      >
        Retry
      </button>
    </div>
  ),
  component: PostsPage,
})

// Custom 404
export const Route = createFileRoute('/posts/$postId')({
  loader: async ({ params }) => {
    const post = await getPost(params.postId)
    if (!post) throw new Error('Post not found')  // Shows errorComponent
    return post
  },
  notFoundComponent: () => (
    <div className="text-center py-20">
      <h1 className="text-4xl font-bold">404</h1>
      <p className="text-gray-500">Post not found</p>
    </div>
  ),
  component: PostPage,
})
```

## Advanced Server Functions

```tsx
import { createServerFn } from '@tanstack/react-start'
import {
  getRequest,
  getRequestHeader,
  setResponseHeaders,
  setResponseStatus,
} from '@tanstack/react-start/server'

// Read cookies and request headers
const protectedFn = createServerFn({ method: 'GET' }).handler(async () => {
  const request = getRequest()
  const authHeader = getRequestHeader('Authorization')

  // Validate authentication
  if (!authHeader) throw new Error('Unauthorized')

  // Set response headers (caching, cookies, etc.)
  setResponseHeaders(new Headers({
    'Cache-Control': 'public, max-age=3600',
    'CDN-Cache-Control': 'max-age=3600, stale-while-revalidate=600',
  }))

  setResponseStatus(200)
  return { data: 'protected' }
})

// Server-only function (cannot be called from client)
import { createServerOnlyFn } from '@tanstack/react-start'

const getEnvVar = createServerOnlyFn(() => process.env.DATABASE_URL)
```

## Caching and Revalidation

```tsx
// SWR Cache: automatic stale-while-revalidate
export const Route = createFileRoute('/posts/')({
  loader: async () => fetchPosts(),
  staleTime: 30_000,   // 30s: use cache, revalidate in background
  gcTime: 5 * 60_000,  // 5min: keep in memory after leaving route
})

// ISR via headers
export const Route = createFileRoute('/blog/$slug')({
  loader: async ({ params }) => fetchPost(params.slug),
  headers: () => ({
    'Cache-Control': 'public, max-age=3600, stale-while-revalidate=604800',
  }),
  staleTime: 5 * 60_000,
})

// Static Generation (build time)
import { createServerFn } from '@tanstack/react-start'
import { staticFunctionMiddleware } from '@tanstack/start-static-server-functions'

const getStaticData = createServerFn({ method: 'GET' })
  .middleware([staticFunctionMiddleware])
  .handler(async () => {
    return 'Generated at build time'
  })
```

## Environment Variables

```tsx
// Only available in server functions (never exposed to client)
const dbUrl = process.env.DATABASE_URL       // Direct access
const apiKey = process.env.SECRET_API_KEY     // Safe on server

// VITE_ prefixed variables available on client
const publicVar = import.meta.env.VITE_PUBLIC_API_URL
```

## Best Practices

### Do's

- **Use `createServerFn`** for all server logic — never call APIs directly from the client
- **Validate inputs** with `inputValidator` + Zod — prevents malformed data
- **Use `beforeLoad`** to protect routes — redirects before rendering
- **Use `loader` + `useLoaderData()`** — data preloaded on server, reused on client
- **Use `router.invalidate()`** after mutations — refreshes loaders
- **Use `defer` + `<Suspense>`** for slow data — doesn't block first render
- **Use `staleTime` and `gcTime`** — control caching per route
- **Use TanStack Router Devtools** — debugging routes, loaders, and cache
- **Use `Link` with typed `params` and `search`** — prevents broken routes

### Don'ts

- **Don't call APIs directly in components** — use server functions or loaders
- **Don't expose secrets** — `process.env` without `VITE_` only works in server functions
- **Don't use `useEffect` for data fetching** — use loaders with SWR cache
- **Don't forget `errorComponent`** — without it, loader errors crash the app
- **Don't over-nest layouts** — each layout adds to the component tree
- **Don't use `fetch` on client without cache** — loaders already have built-in SWR
- **Don't put business logic in components** — extract it to server functions
- **Don't forget `router.invalidate()` after mutations** — or data desyncs
- **Don't forget `__root.tsx`** — it's required and defines the HTML shell

## Failure Handling

- **Loader error:** If a loader throws, the route's `errorComponent` is shown. If it fails 3 times, it escalates to the layout's error boundary.
- **Server function error:** Server functions throw exceptions that the client catches. Always handle with `try/catch`.
- **Validation failure:** If `inputValidator` fails, the error is thrown before executing the handler. The client receives the validation error.

## References

For detailed guides on specific topics, see the following files:

- **[Deployment Guide](references/deployment-guide.md)** — Deployment to Vercel, Cloudflare Workers, Node.js, Docker, Netlify, Bun, and Railway
- **[Advanced Patterns](references/advanced-patterns.md)** — Authentication, middleware, ISR, route context, server components, and static generation
- **[Migration from Next.js](references/migration-from-nextjs.md)** — Step-by-step guide to migrate from Next.js App Router or Pages Router
