# Astro: Advanced Patterns

Advanced patterns for View Transitions, Server Islands, Astro Actions, Content Collection references, Middleware, RSS/XML generation, and custom error pages.

## Pattern 1: View Transitions in Depth

### Custom Animations with CSS

```css
/* src/styles/transitions.css */
@view-transition {
  navigation: auto;
}

/* Custom animation for root */
:root {
  view-transition-name: root;
}

/* Fade transition */
::view-transition-old(root) {
  animation: fade-out 0.3s ease-out;
}
::view-transition-new(root) {
  animation: fade-in 0.3s ease-in;
}

/* Slide transition */
::view-transition-old(slide) {
  animation: slide-out-left 0.3s ease-out;
}
::view-transition-new(slide) {
  animation: slide-in-right 0.3s ease-in;
}

@keyframes fade-out {
  from { opacity: 1; }
  to { opacity: 0; }
}
@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}
@keyframes slide-out-left {
  from { transform: translateX(0); }
  to { transform: translateX(-100%); }
}
@keyframes slide-in-right {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}
```

### Transition Events and Fallback

```astro
---
import { ClientRouter } from 'astro:transitions';
---

<ClientRouter />

<script>
  // Listen for transition events
  document.addEventListener('astro:before-swap', (e) => {
    // Runs before the new page replaces the old one
    console.log('Navigating to:', e.to);
  });

  document.addEventListener('astro:after-swap', () => {
    // Runs after transition completes
    console.log('Navigation complete');
  });

  // Detect browser support
  if (!document.startViewTransition) {
    // Fallback: use CSS animations or just swap
    document.documentElement.classList.add('no-view-transitions');
  }
</script>
```

### Named Elements for Morph Animations

```html
<!-- Page A: blog/index.astro -->
<a href="/blog/post-1" class="card" transition:name="post-1">
  <img src="/thumb-1.jpg" transition:name="post-1-image" />
  <h2 transition:name="post-1-title">Post Title</h2>
</a>

<!-- Page B: blog/post-1.astro -->
<article>
  <img src="/hero-1.jpg" transition:name="post-1-image" class="w-full" />
  <h1 transition:name="post-1-title" class="text-4xl">Post Title</h1>
</article>
```

### Programmatic View Transitions

```tsx
// Trigger transitions from framework components
function navigateWithTransition(url: string) {
  if (document.startViewTransition) {
    document.startViewTransition(() => {
      window.location.href = url;
    });
  } else {
    window.location.href = url;
  }
}
```

## Pattern 2: Server Islands — Advanced Patterns

### With Props from Static Context

```astro
---
// src/pages/product/[id].astro
import { getEntry } from 'astro:content';
import ProductReviews from '../../components/ProductReviews.astro';
import RelatedProducts from '../../components/RelatedProducts.astro';

export async function getStaticPaths() {
  const products = await getCollection('products');
  return products.map((p) => ({ params: { id: p.id }, props: { product: p } }));
}

const { product } = Astro.props;
---

<main>
  <!-- Static: instant render -->
  <h1>{product.data.name}</h1>
  <p>{product.data.description}</p>

  <!-- Server Island: deferred, parallel loading -->
  <ProductReviews
    server:defer
    productId={product.id}
    avgRating={product.data.avgRating}
  >
    <div slot="fallback" class="animate-pulse">
      <div class="h-4 bg-gray-200 rounded w-3/4 mb-2"></div>
      <div class="h-4 bg-gray-200 rounded w-1/2"></div>
    </div>
  </ProductReviews>

  <RelatedProducts server:defer category={product.data.category}>
    <p slot="fallback">Loading related products...</p>
  </RelatedProducts>
</main>
```

### Server Island Component Pattern

```astro
---
// src/components/ProductReviews.astro
interface Props {
  productId: string;
  avgRating: number;
}

const { productId, avgRating } = Astro.props;

// This fetch happens on-demand, after the page loads
const response = await fetch(
  `https://api.example.com/products/${productId}/reviews`
);
const reviews = await response.json();
---

<div class="reviews mt-8">
  <div class="flex items-center gap-2 mb-4">
    <span class="text-2xl font-bold">{avgRating}</span>
    <span class="text-yellow-400">{'★'.repeat(Math.round(avgRating))}</span>
    <span class="text-gray-500">({reviews.length} reviews)</span>
  </div>

  {reviews.map((review) => (
    <div class="border-b py-4">
      <p class="font-semibold">{review.author}</p>
      <p>{review.text}</p>
    </div>
  ))}
</div>
```

## Pattern 3: Astro Actions — Advanced Validation

### Complex Form with File Upload

```typescript
// src/actions/index.ts
import { defineAction, ActionError } from 'astro:actions';
import { z } from 'astro/zod';

const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB

export const server = {
  submitContact: defineAction({
    accept: 'form',
    input: z.object({
      name: z.string().min(2, 'Name too short'),
      email: z.string().email('Invalid email'),
      message: z.string().min(10, 'Message too short'),
      attachment: z
        .instanceof(File)
        .optional()
        .refine((file) => !file || file.size <= MAX_FILE_SIZE, 'File too large')
        .refine(
          (file) => !file || ['image/png', 'image/jpeg', 'application/pdf'].includes(file.type),
          'Invalid file type',
        ),
    }),
    handler: async (input) => {
      try {
        await saveContactMessage(input);

        if (input.attachment) {
          await uploadAttachment(input.attachment);
        }

        return { success: true };
      } catch (error) {
        throw new ActionError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to submit contact form',
        });
      }
    },
  }),
};
```

```astro
---
import { actions } from 'astro:actions';
---

<form method="POST" action={actions.submitContact} enctype="multipart/form-data">
  <label>
    Name:
    <input type="text" name="name" required minlength="2" />
  </label>
  <label>
    Email:
    <input type="email" name="email" required />
  </label>
  <label>
    Message:
    <textarea name="message" required minlength="10"></textarea>
  </label>
  <label>
    Attachment:
    <input type="file" name="attachment" accept="image/png,image/jpeg,application/pdf" />
  </label>
  <button type="submit">Send</button>

  <!-- Display action errors -->
  {Astro.props.error && (
    <p class="text-red-500">{Astro.props.error}</p>
  )}
</form>
```

### Calling Actions from Framework Components

```tsx
// src/components/ContactForm.tsx
import { actions } from 'astro:actions';
import { useState } from 'react';

export default function ContactForm() {
  const [error, setError] = useState<string | null>(null);
  const [success, setSuccess] = useState(false);

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setError(null);

    const formData = new FormData(e.currentTarget);

    const { data, error: actionError } = await actions.submitContact(formData);

    if (actionError) {
      setError(actionError.message);
      return;
    }

    setSuccess(true);
  }

  if (success) return <p class="text-green-500">Message sent! ✅</p>;

  return (
    <form onSubmit={handleSubmit}>
      {/* ... form fields ... */}
      {error && <p class="text-red-500">{error}</p>}
      <button type="submit">Send</button>
    </form>
  );
}
```

## Pattern 4: Content Collection References

Link entries across collections:

```typescript
// src/content/config.ts
import { defineCollection, reference, z } from 'astro:content';

const blog = defineCollection({
  schema: z.object({
    title: z.string(),
    author: reference('authors'),          // Single reference
    relatedPosts: z.array(reference('blog')).optional(), // Multiple references
  }),
});

const authors = defineCollection({
  schema: z.object({
    name: z.string(),
    avatar: z.string().url(),
    bio: z.string(),
  }),
});

export const collections = { blog, authors };
```

```astro
---
// Resolve references
import { getEntry, getEntries } from 'astro:content';

const post = await getEntry('blog', 'post-1');

// Single reference
const author = await getEntry(post.data.author);
// author.data.name, author.data.avatar, etc.

// Array of references
const relatedPosts = await getEntries(post.data.relatedPosts || []);
---
```

## Pattern 5: Middleware — Auth + Locale

```typescript
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware(async (context, next) => {
  // Locale detection
  const acceptLang = context.request.headers.get('accept-language');
  const cookieLocale = context.cookies.get('locale')?.value;
  const locale = cookieLocale || acceptLang?.split(',')[0]?.split('-')[0] || 'en';

  context.locals.locale = locale;

  // CSRF protection for mutations
  if (context.request.method === 'POST') {
    const origin = context.request.headers.get('origin');
    const host = context.url.host;

    if (origin && !origin.endsWith(host)) {
      return new Response('Forbidden', { status: 403 });
    }
  }

  // Rate limiting (simple example)
  const clientIp = context.request.headers.get('x-forwarded-for') || 'unknown';
  // ... check rate limit against clientIp

  const response = await next();

  // Add security headers
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');

  return response;
});
```

## Pattern 6: RSS/XML Feed Generation

```typescript
// src/pages/rss.xml.ts
import rss from '@astrojs/rss';
import { getCollection } from 'astro:content';
import type { APIRoute } from 'astro';

export const GET: APIRoute = async (context) => {
  const blog = await getCollection('blog', ({ data }) => !data.draft);

  return rss({
    title: 'My Blog',
    description: 'A blog about web development',
    site: context.site!,
    items: blog.map((post) => ({
      title: post.data.title,
      description: post.data.description,
      pubDate: post.data.pubDate,
      link: `/blog/${post.slug}`,
      customData: `<author>${post.data.author}</author>`,
    })),
    customData: `<language>en</language>`,
  });
};
```

## Pattern 7: Custom Error Pages

```astro
---
// src/pages/404.astro
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout title="Page Not Found">
  <div class="text-center py-20">
    <h1 class="text-6xl font-bold text-gray-300">404</h1>
    <p class="text-xl mt-4">Page not found</p>
    <a href="/" class="text-blue-500 hover:underline mt-4 inline-block">
      Go back home
    </a>
  </div>
</BaseLayout>
```

```astro
---
// src/pages/500.astro
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout title="Server Error">
  <div class="text-center py-20">
    <h1 class="text-6xl font-bold text-red-300">500</h1>
    <p class="text-xl mt-4">Something went wrong on our end.</p>
    <p class="text-gray-500 mt-2">Please try again later.</p>
  </div>
</BaseLayout>
```

## Pattern 8: Hybrid Rendering (Static + SSR)

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'hybrid',
  adapter: node({ mode: 'standalone' }),
});
```

```astro
---
// src/pages/products.astro — Static (default in hybrid)
import { getCollection } from 'astro:content';

const products = await getCollection('products');
// This page is prerendered at build time
---

<h1>Products</h1>
<!-- ... -->
```

```astro
---
// src/pages/dashboard.astro — SSR (opt-in)
export const prerender = false;

// This page is rendered on each request
const user = Astro.locals.user;
---

<h1>Dashboard for {user?.name}</h1>
```

## Pattern 9: Image Optimization with `getImage`

```astro
---
import { getImage } from 'astro:assets';
import myImage from '../assets/hero.jpg';

const optimized = await getImage({
  src: myImage,
  width: 800,
  height: 400,
  format: 'webp',
});
---

<img src={optimized.src} alt="Hero" width={optimized.width} height={optimized.height} />
```
