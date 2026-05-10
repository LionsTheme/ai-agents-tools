---
name: astro
description: Astro web framework for content-driven sites with islands architecture, zero JS by default, content collections, and server endpoints. Use when building blogs, marketing sites, docs, e-commerce, or multi-framework sites with React/Vue/Svelte/Solid components.
---

# Astro

Astro is an all-in-one web framework for building fast, content-focused websites. Its islands architecture delivers zero JavaScript by default, hydrating only interactive components. Supports React, Vue, Svelte, Solid, Preact, and more through integrations.

## When to Use This Skill

- Building content-driven websites (blogs, docs, marketing, e-commerce)
- Creating static sites with optional SSR for dynamic routes
- Integrating components from multiple frameworks (React, Vue, Svelte, Solid)
- Implementing islands architecture for optimal performance
- Using Content Collections for type-safe content management
- Setting up View Transitions for SPA-like navigation
- Configuring server endpoints or Astro Actions for forms
- Deploying to Vercel, Netlify, Cloudflare, Node.js, or static hosts
- Migrating from Gatsby, Next.js Pages Router, or static site generators

## Core Concepts

### 1. Islands Architecture

| Concept              | Description                                              |
| -------------------- | -------------------------------------------------------- |
| **Astro Components** | `.astro` files. Render to static HTML with zero JS.      |
| **Islands**          | Interactive UI components hydrated on the client.        |
| **Client Directives**| Control when/how JS loads: `client:load`, `client:visible`, `client:idle`, `client:media`, `client:only` |
| **Server Islands**   | Deferred server-rendered components via `server:defer`.  |
| **Hydration**        | Process of making static HTML interactive by loading JS. |

### 2. Rendering Modes

| Mode         | Config                 | Description                                |
| ------------ | ---------------------- | ------------------------------------------ |
| **Static**   | `output: 'static'`     | Default. All pages prerendered at build.   |
| **Server**   | `output: 'server'`     | Pages rendered on request (SSR).           |
| **Hybrid**   | `output: 'hybrid'`     | Static by default, SSR with `export const prerender = false`. |

### 3. Astro vs Other Frameworks

| Feature              | Astro                       | Next.js                  | Gatsby              |
| -------------------- | --------------------------- | ------------------------ | ------------------- |
| **Default JS**       | Zero                        | Full hydration           | Full hydration      |
| **Architecture**     | Islands (multi-framework)   | Server Components (RSC)  | React-only          |
| **Content**          | Content Collections + Zod   | Manual MDX setup         | GraphQL layer       |
| **Build speed**      | Fast (Vite)                 | Moderate (Turbopack)     | Slow (Webpack)      |
| **Static output**    | Default                     | Optional                 | Default             |
| **SSR**              | Optional (server/hybrid)    | Yes                      | No (SSG only)       |
| **Framework agnostic**| React, Vue, Svelte, Solid  | React only               | React only          |

## Project Structure

```
my-site/
├── src/
│   ├── pages/                    # Routes — file-based routing
│   │   ├── index.astro           # Home page (/)
│   │   ├── about.astro           # /about
│   │   ├── blog/
│   │   │   ├── index.astro       # /blog
│   │   │   └── [slug].astro      # /blog/:slug (dynamic)
│   │   └── api/
│   │       └── search.json.ts    # /api/search.json endpoint
│   ├── layouts/                  # Reusable page layouts
│   │   └── BaseLayout.astro
│   ├── components/               # Astro and framework components
│   │   ├── Header.astro
│   │   └── ReactCounter.jsx
│   ├── content/                  # Content Collections
│   │   ├── blog/
│   │   │   ├── post-1.md
│   │   │   └── post-2.md
│   │   └── config.ts             # Collection schemas
│   ├── actions/                  # Astro Actions (server functions)
│   │   └── index.ts
│   ├── middleware.ts             # Global middleware
│   └── styles/
│       └── global.css
├── public/                       # Static assets (no processing)
│   └── favicon.svg
├── astro.config.mjs              # Astro configuration
├── package.json
└── tsconfig.json
```

## Quick Start

```bash
# Create a new project
npm create astro@latest
# or with a template
npm create astro@latest -- --template blog

cd my-site
npm i
npm run dev
```

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';

export default defineConfig({
  output: 'static',  // 'static' | 'server' | 'hybrid'
  integrations: [react()],
});
```

## Essential Patterns

### Pattern 1: Create a Page (`.astro` Component)

```astro
---
// src/pages/index.astro
// The frontmatter (---) runs at build time / server-side
import BaseLayout from '../layouts/BaseLayout.astro';

const pageTitle = 'Welcome to Astro';
const features = ['Zero JS by default', 'Multi-framework', 'Content Collections'];
---

<BaseLayout title={pageTitle}>
  <h1 class="text-4xl font-bold">{pageTitle}</h1>
  <p class="text-lg text-gray-600 mt-2">
    Build fast, content-driven websites.
  </p>

  <ul class="mt-4 space-y-2">
    {features.map((feature) => (
      <li class="flex items-center gap-2">
        <span class="text-green-500">✓</span>
        {feature}
      </li>
    ))}
  </ul>
</BaseLayout>
```

### Pattern 2: Layout Component

Layouts are Astro components that wrap pages with shared structure:

```astro
---
// src/layouts/BaseLayout.astro
interface Props {
  title: string;
  description?: string;
}

const { title, description = 'My Astro site' } = Astro.props;
---

<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="description" content={description} />
    <title>{title}</title>
  </head>
  <body class="min-h-screen bg-white dark:bg-gray-950">
    <header class="border-b p-4">
      <nav class="max-w-4xl mx-auto flex gap-4">
        <a href="/">Home</a>
        <a href="/blog">Blog</a>
        <a href="/about">About</a>
      </nav>
    </header>

    <main class="max-w-4xl mx-auto p-4">
      <slot />  <!-- Page content injected here -->
    </main>

    <footer class="border-t p-4 text-center text-sm text-gray-500">
      &copy; {new Date().getFullYear()} My Site
    </footer>
  </body>
</html>
```

### Pattern 3: Content Collections — Type-Safe Content

**Define schemas:**

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const blog = defineCollection({
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    author: z.string().default('Anonymous'),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false),
  }),
});

export const collections = { blog };
```

**Query and display content:**

```astro
---
// src/pages/blog/index.astro
import { getCollection } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';

const posts = await getCollection('blog', ({ data }) => {
  return !data.draft;  // Filter out drafts
});

// Sort by date
posts.sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
---

<BaseLayout title="Blog">
  <h1 class="text-3xl font-bold mb-6">Blog</h1>

  <div class="grid gap-6">
    {posts.map((post) => (
      <article class="border rounded-lg p-4 hover:shadow-md transition">
        <a href={`/blog/${post.slug}`}>
          <h2 class="text-xl font-semibold">{post.data.title}</h2>
          <p class="text-gray-600 mt-1">{post.data.description}</p>
          <time class="text-sm text-gray-400 mt-2 block">
            {post.data.pubDate.toLocaleDateString()}
          </time>
        </a>
      </article>
    ))}
  </div>
</BaseLayout>
```

### Pattern 4: Dynamic Routes (`[slug].astro`)

```astro
---
// src/pages/blog/[slug].astro
import { getCollection, getEntry } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await post.render();
---

<BaseLayout title={post.data.title} description={post.data.description}>
  <article class="prose lg:prose-xl mx-auto">
    <h1>{post.data.title}</h1>
    <div class="flex gap-2 text-sm text-gray-500 mb-6">
      <time>{post.data.pubDate.toLocaleDateString()}</time>
      <span>·</span>
      <span>{post.data.author}</span>
    </div>

    <Content />
  </article>
</BaseLayout>
```

### Pattern 5: Framework Components (React Islands)

```bash
npx astro add react
```

```astro
---
// src/pages/index.astro
import BaseLayout from '../layouts/BaseLayout.astro';
import LikeButton from '../components/LikeButton';
import NewsletterForm from '../components/NewsletterForm.jsx';
---

<BaseLayout title="Home">
  <!-- Astro component: zero JS -->
  <HeroBanner />

  <!-- React island: hydrates on page load -->
  <LikeButton client:load />

  <!-- React island: hydrates only when visible -->
  <NewsletterForm client:visible />
</BaseLayout>
```

```tsx
// src/components/LikeButton.tsx
import { useState } from 'react';

export default function LikeButton() {
  const [likes, setLikes] = useState(0);

  return (
    <button
      onClick={() => setLikes(likes + 1)}
      className="bg-pink-500 text-white px-4 py-2 rounded hover:bg-pink-600 transition"
    >
      ❤️ {likes} Likes
    </button>
  );
}
```

### Pattern 6: Client Directives Reference

| Directive          | Behavior                                                 |
| ------------------ | -------------------------------------------------------- |
| `client:load`      | Hydrate immediately on page load.                        |
| `client:visible`   | Hydrate when element enters viewport.                    |
| `client:idle`      | Hydrate when browser is idle (uses `requestIdleCallback`).|
| `client:media`     | Hydrate when CSS media query matches.                    |
| `client:only`      | Skip SSR; render and hydrate only on client. Requires framework name. |

```astro
<!-- Hydrate on load (high priority) -->
<CookieBanner client:load />

<!-- Lazy hydrate (low priority, when visible) -->
<TestimonialsCarousel client:visible />

<!-- Hydrate when idle -->
<AnalyticsTracker client:idle />

<!-- Only on mobile -->
<MobileMenu client:media="(max-width: 768px)" />

<!-- Client-only, no SSR -->
<ReactChart client:only="react" />
```

### Pattern 7: Astro Actions — Form Handling

```typescript
// src/actions/index.ts
import { defineAction } from 'astro:actions';
import { z } from 'astro/zod';

export const server = {
  subscribe: defineAction({
    accept: 'form',
    input: z.object({
      email: z.string().email(),
    }),
    handler: async ({ email }) => {
      // Store email, call external API, etc.
      await saveSubscriber(email);
      return { success: true, message: `Subscribed: ${email}` };
    },
  }),

  getGreeting: defineAction({
    input: z.object({
      name: z.string().min(1),
    }),
    handler: async ({ name }) => {
      return `Hello, ${name}!`;
    },
  }),
};
```

```astro
---
// src/pages/index.astro
import { actions } from 'astro:actions';
---

<!-- Zero-JS form submission -->
<form method="POST" action={actions.subscribe}>
  <label>
    Email:
    <input type="email" name="email" required />
  </label>
  <button type="submit">Subscribe</button>
</form>

<!-- Or call from script -->
<script>
  import { actions } from 'astro:actions';

  const result = await actions.getGreeting({ name: 'World' });
  console.log(result); // "Hello, World!"
</script>
```

### Pattern 8: API Endpoints

```typescript
// src/pages/api/search.json.ts
import type { APIRoute } from 'astro';

export const GET: APIRoute = async ({ request }) => {
  const url = new URL(request.url);
  const query = url.searchParams.get('q');

  if (!query) {
    return new Response(JSON.stringify({ error: 'Missing query' }), {
      status: 400,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  const results = await searchContent(query);

  return new Response(JSON.stringify({ results }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' },
  });
};

export const POST: APIRoute = async ({ request }) => {
  const body = await request.json();
  // Process POST data
  return new Response(JSON.stringify({ received: true }), {
    status: 201,
    headers: { 'Content-Type': 'application/json' },
  });
};
```

### Pattern 9: SSR with On-Demand Rendering

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'server',
  adapter: node({ mode: 'standalone' }),
});
```

```astro
---
// src/pages/dashboard.astro
// In 'server' or 'hybrid' mode, data is fetched on each request
const response = await fetch('https://api.example.com/stats', {
  headers: { Authorization: `Bearer ${Astro.locals.apiToken}` },
});
const stats = await response.json();
---

<h1>Dashboard</h1>
<p>Active users: {stats.activeUsers}</p>
<p>Total revenue: ${stats.revenue}</p>
```

### Pattern 10: View Transitions (SPA-like Navigation)

```astro
---
// src/layouts/BaseLayout.astro
import { ClientRouter } from 'astro:transitions';
---

<!doctype html>
<html lang="en" transition:name="root" transition:animate="fade">
  <head>
    <meta charset="utf-8" />
    <!-- ... other head elements ... -->
    <ClientRouter />
  </head>
  <body>
    <header transition:name="header" transition:animate="slide">
      <nav><!-- ... --></nav>
    </header>
    <main transition:animate="fade">
      <slot />
    </main>
  </body>
</html>
```

```astro
<!-- Transition directives on individual elements -->
<aside transition:name="hero">
  <img src="/hero.jpg" alt="Hero" />
</aside>

<!-- Persist element state across navigations -->
<video controls autoplay transition:persist>
  <source src="/bg-video.mp4" type="video/mp4" />
</video>

<!-- Animate specific element -->
<title transition:animate="fade">About My Site</title>
```

## Server Islands (Deferred Server Rendering)

```astro
---
// src/pages/product/[id].astro
import Avatar from '../../components/Avatar.astro';
import Reviews from '../../components/Reviews.astro';
import GenericAvatar from '../../components/GenericAvatar.astro';
---

<!-- Instant static content -->
<h1>{product.name}</h1>
<p class="price">${product.price}</p>

<!-- Server Island: rendered on-demand, parallel to page -->
<Avatar server:defer>
  <!-- Fallback shown while loading -->
  <GenericAvatar slot="fallback" />
</Avatar>

<!-- Another deferred component — loads independently -->
<Reviews server:defer productId={product.id}>
  <p slot="fallback">Loading reviews...</p>
</Reviews>
```

## Middleware

```typescript
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware(async (context, next) => {
  // Auth check
  const isAuthenticated = context.cookies.has('session');

  // Add data accessible via Astro.locals
  context.locals.user = isAuthenticated
    ? await getUserFromSession(context.cookies.get('session')!.value)
    : null;

  // Redirect if needed
  if (context.url.pathname.startsWith('/admin') && !isAuthenticated) {
    return context.redirect('/login');
  }

  return next();
});
```

## Environment Variables

```astro
---
// Available in frontmatter (build time or server-side)
const apiKey = import.meta.env.API_KEY;

// Public variables (prefixed with PUBLIC_)
const publicUrl = import.meta.env.PUBLIC_SITE_URL;
```

```env
# .env
API_KEY=secret-key
PUBLIC_SITE_URL=https://example.com
```

## Best Practices

### Do's

- **Use `.astro` components by default** — they render to static HTML with zero JS overhead
- **Use Content Collections** — type-safe schemas with Zod for all markdown/MDX content
- **Use `client:visible` over `client:load`** — lazy hydrate for better performance
- **Use the Islands pattern** — isolate interactive parts as framework components
- **Use `getStaticPaths()`** for dynamic routes in static mode
- **Use View Transitions** — SPA-like navigation without a full JS framework
- **Use server islands (`server:defer`)** for slow/expensive server components
- **Use Astro Actions** for form handling — zero JS fallback with progressive enhancement
- **Keep integrations minimal** — add only the adapters and frameworks you actually use

### Don'ts

- **Don't make entire pages interactive** — only hydrate the islands that need JS
- **Don't use React as default** — use `.astro` components for static content
- **Don't ignore `getStaticPaths()` in static mode** — required for dynamic routes
- **Don't expose secrets to client** — use `import.meta.env` without `PUBLIC_` prefix
- **Don't forget alt text on images** — Astro's `<Image />` requires it
- **Don't put client directives (`client:*`) on `.astro` components** — they don't hydrate
- **Don't overuse `client:only`** — loses SSR benefits
- **Don't forget to set `prerender = false`** for SSR pages in hybrid mode

## Failure Handling

- **Content Collection error:** Missing required frontmatter fields fail at build time with clear Zod error messages.
- **Dynamic route without `getStaticPaths()` in static mode:** Build fails. Must provide static paths.
- **Server endpoint error:** Returns 500 with error message. Use `try/catch` and return appropriate status codes.
- **Client directive on `.astro` component:** Warning emitted. Astro components cannot hydrate.

## References

For detailed guides on specific topics, see the following files:

- **[Advanced Patterns](references/advanced-patterns.md)** — View Transitions in depth, Server Islands patterns, Actions with complex validation, Content Collection references, Middleware patterns, RSS/XML generation
- **[Deployment Guide](references/deployment-guide.md)** — Deployment to Vercel, Netlify, Cloudflare, Node.js, Docker, and static hosting
- **[Framework Integrations](references/framework-integrations.md)** — React, Vue, Svelte, Solid, Preact, Alpine.js, Tailwind, MDX setup and patterns
