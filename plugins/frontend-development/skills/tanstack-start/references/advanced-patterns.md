# TanStack Start: Advanced Patterns

Advanced patterns for authentication, middleware, ISR, route context, server components, and static generation.

## Pattern 1: Full Authentication with Sessions

### Login/Logout Server Functions

```tsx
// server/auth.ts
import { createServerFn } from '@tanstack/react-start'
import { redirect } from '@tanstack/react-router'
import { useAppSession } from '../utils/session'

export const loginFn = createServerFn({ method: 'POST' })
  .inputValidator((data: { email: string; password: string }) => data)
  .handler(async ({ data }) => {
    const user = await authenticateUser(data.email, data.password)

    if (!user) {
      return { error: 'Invalid credentials' }
    }

    const session = await useAppSession()
    await session.update({
      userId: user.id,
      email: user.email,
    })

    throw redirect({ to: '/dashboard' })
  })

export const logoutFn = createServerFn({ method: 'POST' })
  .handler(async () => {
    const session = await useAppSession()
    await session.clear()
    throw redirect({ to: '/' })
  })

export const getCurrentUser = createServerFn({ method: 'GET' })
  .handler(async () => {
    const session = await useAppSession()
    const userId = session.get('userId')

    if (!userId) return null
    return await getUserById(userId)
  })
```

### Protect Routes with `beforeLoad`

```tsx
// src/routes/_authenticated.tsx
import { createFileRoute, Outlet, redirect } from '@tanstack/react-router'
import { getCurrentUser } from '../server/auth'

export const Route = createFileRoute('/_authenticated')({
  beforeLoad: async ({ location }) => {
    const user = await getCurrentUser()

    if (!user) {
      // Redirect to login preserving the original URL
      throw redirect({
        to: '/login',
        search: { redirect: location.href },
      })
    }

    return { user }  // Typed context for child routes
  },
  component: AuthenticatedLayout,
})

function AuthenticatedLayout() {
  return <Outlet />
}

// Usage in child route:
// src/routes/_authenticated/dashboard.tsx
export const Route = createFileRoute('/_authenticated/dashboard')({
  component: DashboardPage,
})

function DashboardPage() {
  const { user } = Route.useRouteContext()  // Typed from beforeLoad
  return <h1>{user.email}'s Dashboard</h1>
}
```

### Auth.js / NextAuth in TanStack Start

```tsx
// server/authjs.ts
import { createServerFn } from '@tanstack/react-start'
import { getRequest } from '@tanstack/react-start/server'
import { getSession } from 'start-authjs'
import { authConfig } from '../utils/auth'

export const fetchSession = createServerFn({ method: 'GET' })
  .handler(async () => {
    const request = getRequest()
    return await getSession(request, authConfig)
  })

// In __root.tsx
export const Route = createRootRouteWithContext<{ session: AuthSession | null }>()({
  beforeLoad: async () => {
    const session = await fetchSession()
    return { session }
  },
  component: RootComponent,
})
```

## Pattern 2: Middleware in Server Functions

```tsx
// middleware/auth.ts
import { createMiddleware } from '@tanstack/react-start'
import { auth } from './my-auth'

export const authMiddleware = createMiddleware()
  .server(async ({ next, request, data }) => {
    const session = await auth.getSession({ headers: request.headers })

    if (!session) {
      throw new Error('Unauthorized')
    }

    // Pass data to next middleware or handler
    return await next({
      context: { session },
    })
  })
```

```tsx
// Usage in server function
import { createServerFn } from '@tanstack/react-start'
import { zodValidator } from '@tanstack/zod-adapter'
import { z } from 'zod'
import { authMiddleware } from '../middleware/auth'

export const updateProfile = createServerFn({ method: 'POST' })
  .inputValidator(zodValidator(z.object({
    name: z.string().min(1),
  })))
  .middleware([authMiddleware])
  .handler(async ({ data, context }) => {
    // context.session is typed
    const { session } = context
    return await db.users.update({
      where: { id: session.userId },
      data,
    })
  })
```

### Middleware with Validation

```tsx
// middleware/workspace.ts
import { createMiddleware } from '@tanstack/react-start'
import { zodValidator } from '@tanstack/zod-adapter'
import { z } from 'zod'

const workspaceSchema = z.object({
  workspaceId: z.string(),
})

export const workspaceMiddleware = createMiddleware({ type: 'function' })
  .inputValidator(zodValidator(workspaceSchema))
  .server(async ({ next, data }) => {
    const workspace = await db.workspaces.findUnique({
      where: { id: data.workspaceId },
    })

    if (!workspace) {
      throw new Error('Workspace not found')
    }

    return next({ context: { workspace } })
  })
```

## Pattern 3: ISR (Incremental Static Regeneration)

```tsx
// src/routes/blog/$slug.tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/blog/$slug')({
  loader: async ({ params }) => {
    const post = await fetchPost(params.slug)
    return post
  },
  headers: ({ loaderData }) => ({
    // CDN Cache: 1 hour, stale for 7 days (revalidates in background)
    'Cache-Control': loaderData
      ? 'public, max-age=3600, stale-while-revalidate=604800'
      : 'no-store',
    // CDN-specific headers
    'CDN-Cache-Control': 'max-age=3600, stale-while-revalidate=604800',
  }),
  staleTime: 5 * 60_000,  // 5 minutes client-side before revalidating
  gcTime: 30 * 60_000,    // 30 minutes in memory
  component: BlogPostPage,
})

function BlogPostPage() {
  const post = Route.useLoaderData()
  return <article>{post.content}</article>
}
```

## Pattern 4: Route Context (Shared Data Across Routes)

```tsx
// src/router.tsx
import { createRouter } from '@tanstack/react-router'
import { QueryClient } from '@tanstack/react-query'
import { routeTree } from './routeTree.gen'

// Define the context type
interface RouterContext {
  queryClient: QueryClient
  theme: 'light' | 'dark'
}

const queryClient = new QueryClient()

export const router = createRouter({
  routeTree,
  context: {
    queryClient,
    theme: 'light',
  },
})

declare module '@tanstack/react-router' {
  interface Register {
    router: typeof router
  }
}
```

```tsx
// Access context in loaders
export const Route = createFileRoute('/posts/')({
  loader: async ({ context }) => {
    const { queryClient, theme } = context
    // Use queryClient for prefetching, theme for conditional logic
    await queryClient.ensureQueryData(postsQueryOptions)
    return { theme }
  },
  component: PostsPage,
})
```

## Pattern 5: Server Components (RSC-like)

TanStack Start supports "Server Components" via `getServerComponent`:

```tsx
// server/components.ts
import { createServerFn } from '@tanstack/react-start'

const getServerComponent = createServerFn({ method: 'GET' })
  .handler(async () => {
    // Renders JSX on server, returns serializable to client
    return {
      src: <HeavyServerComponent />,  // Executed on server
    }
  })

// On the client
import { useSuspenseQuery } from '@tanstack/react-query'

function MyPage() {
  const { data } = useSuspenseQuery({
    queryKey: ['server-component'],
    queryFn: () => getServerComponent(),
    structuralSharing: false,  // Required for RSC
  })

  return <CompositeComponent src={data.src} />
}
```

## Pattern 6: Static Generation (Build Time)

```tsx
// server/static.ts
import { createServerFn } from '@tanstack/react-start'
import { staticFunctionMiddleware } from '@tanstack/start-static-server-functions'

// This function only runs at build time
export const getStaticContent = createServerFn({ method: 'GET' })
  .middleware([staticFunctionMiddleware])  // Always last
  .handler(async () => {
    // Runs ONCE at build, result cached statically
    return {
      title: 'Static Content',
      data: await db.content.findMany(),
      buildTime: new Date().toISOString(),
    }
  })
```

## Pattern 7: Cookie and Header Handling

```tsx
import { createServerFn } from '@tanstack/react-start'
import {
  getRequest,
  getRequestHeader,
  getCookie,
  setCookie,
  setResponseHeaders,
  setResponseStatus,
} from '@tanstack/react-start/server'

export const cookieHandler = createServerFn({ method: 'POST' })
  .handler(async () => {
    const request = getRequest()
    const userAgent = getRequestHeader('User-Agent')
    const themeCookie = getCookie('theme')

    // Set cookie
    setCookie('lastVisit', new Date().toISOString(), {
      path: '/',
      maxAge: 60 * 60 * 24 * 30,  // 30 days
      httpOnly: true,
      secure: true,
      sameSite: 'lax',
    })

    // Set response headers
    setResponseHeaders(new Headers({
      'Cache-Control': 'no-store',
      'X-Custom-Header': 'value',
    }))

    setResponseStatus(200)

    return {
      userAgent,
      theme: themeCookie || 'system',
    }
  })
```

## Pattern 8: Locale and Timezone via Middleware

```tsx
// src/start.ts
import { createStart, createMiddleware } from '@tanstack/react-start'
import {
  getRequestHeader,
  getCookie,
  setCookie,
} from '@tanstack/react-start/server'

const localeTzMiddleware = createMiddleware().server(async ({ next }) => {
  const header = getRequestHeader('Accept-Language')
  const headerLocale = header?.split(',')[0] || 'en-US'
  const cookieLocale = getCookie('locale')
  const cookieTz = getCookie('tz')

  const locale = cookieLocale || headerLocale
  const timeZone = cookieTz || 'UTC'

  setCookie('locale', locale, {
    path: '/',
    maxAge: 60 * 60 * 24 * 365,
  })

  return next({ context: { locale, timeZone } })
})

export const startInstance = createStart(() => ({
  requestMiddleware: [localeTzMiddleware],
}))
```

## Pattern 9: Optimistic Updates with TanStack Query

```tsx
// hooks/useUpdatePost.ts
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { updatePost } from '../server/posts'

export function useUpdatePost(postId: string) {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: updatePost,
    onMutate: async (newData) => {
      // Cancel pending refetches
      await queryClient.cancelQueries({ queryKey: ['post', postId] })

      // Snapshot previous value for rollback
      const previous = queryClient.getQueryData(['post', postId])

      // Optimistic update
      queryClient.setQueryData(['post', postId], (old: Post) => ({
        ...old,
        ...newData.data,
      }))

      return { previous }
    },
    onError: (err, newData, context) => {
      // Rollback on error
      queryClient.setQueryData(['post', postId], context?.previous)
      toast.error('Update failed')
    },
    onSettled: () => {
      // Refetch to get actual server data
      queryClient.invalidateQueries({ queryKey: ['post', postId] })
    },
  })
}
```

## Pattern 10: Route Loader with Smart Prefetching

```tsx
// src/routes/posts/index.tsx
import { createFileRoute } from '@tanstack/react-router'
import { queryOptions } from '@tanstack/react-query'

const postsQueryOptions = queryOptions({
  queryKey: ['posts'],
  queryFn: () => fetchPosts(),
  staleTime: 60_000,
})

export const Route = createFileRoute('/posts/')({
  loader: ({ context }) => context.queryClient.ensureQueryData(postsQueryOptions),
  component: PostsPage,
})

function PostsPage() {
  const { data: posts } = useQuery(postsQueryOptions)

  return (
    <div>
      {posts.map((post) => (
        <Link
          key={post.id}
          to="/posts/$postId"
          params={{ postId: post.id }}
          // Prefetch on hover
          preload="intent"
        >
          {post.title}
        </Link>
      ))}
    </div>
  )
}

// Global prefetching configuration in router.tsx
export const router = createRouter({
  routeTree,
  context: { queryClient },
  defaultPreloading: 'intent',  // Prefetch on hover
  defaultPreloadingDelay: 100,  // 100ms delay before prefetch
})
```
