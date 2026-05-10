# Astro: Deployment Guide

Configuration and deployment guides for the different environments supported by Astro.

## Static Hosting (Default)

When `output: 'static'` (default), Astro generates a fully static site.

```bash
npm run build
# Output: dist/
```

Deploy `dist/` to any static host:

| Platform        | Setup                                                              |
| --------------- | ------------------------------------------------------------------ |
| **GitHub Pages**| Deploy `dist/` via Actions or branch                               |
| **Cloudflare Pages** | Set build command: `npm run build`, output: `dist`           |
| **Netlify**     | Set publish directory: `dist`                                      |
| **Vercel**      | Auto-detects Astro; output: `dist`                                 |
| **Render**      | Publish directory: `dist`                                          |
| **Surge**       | `surge dist/`                                                     |
| **Firebase**    | `firebase deploy` (public dir: `dist`)                            |

## Vercel Deployment

```bash
npx astro add vercel
```

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import vercel from '@astrojs/vercel';

export default defineConfig({
  output: 'server',  // or 'hybrid'
  adapter: vercel(),
});
```

### Vercel Configuration Options

```javascript
import { defineConfig } from 'astro/config';
import vercel from '@astrojs/vercel';

export default defineConfig({
  output: 'server',
  adapter: vercel({
    // Edge Functions (lighter, faster, limited APIs)
    runtime: 'edge',

    // Serverless Functions (full Node.js APIs)
    // runtime: 'serverless', // Default

    // Analytics
    analytics: true,

    // Image optimization
    imageService: true,

    // ISR: Incremental Static Regeneration (seconds)
    isr: 60,

    // Web analytics
    webAnalytics: { enabled: true },
  }),
});
```

### Environment Variables on Vercel

Set in Vercel dashboard → Settings → Environment Variables.

```env
# Access in Astro
API_KEY=secret
PUBLIC_SITE_URL=https://my-site.vercel.app
```

## Netlify Deployment

```bash
npx astro add netlify
```

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import netlify from '@astrojs/netlify';

export default defineConfig({
  output: 'server',
  adapter: netlify({
    // Edge Functions
    edgeMiddleware: true,
  }),
});
```

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "20"
```

## Cloudflare Deployment

```bash
npx astro add cloudflare
```

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  output: 'server',
  adapter: cloudflare({
    // 'advanced' for Workers, 'directory' for Pages
    mode: 'advanced',

    // Runtime mode
    runtime: {
      mode: 'local',     // Dev mode
      // mode: 'remote', // Production mode
    },
  }),
});
```

```toml
# wrangler.toml
name = "my-astro-site"
compatibility_date = "2024-12-30"

[site]
bucket = "./dist"
```

## Node.js / Docker Deployment

```bash
npx astro add node
```

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'server',
  adapter: node({
    mode: 'standalone',  // 'standalone' | 'middleware'
  }),
  server: {
    host: true,          // Listen on all network interfaces
    port: 4321,
  },
});
```

```json
// package.json
{
  "scripts": {
    "build": "astro build",
    "start": "node ./dist/server/entry.mjs"
  }
}
```

### Dockerfile

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/package.json ./package.json

EXPOSE 4321
CMD ["node", "dist/server/entry.mjs"]
```

### Middleware Mode (Express, Connect)

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'server',
  adapter: node({ mode: 'middleware' }),
});
```

```javascript
// server.js — Custom server
import express from 'express';
import { handler as astroHandler } from './dist/server/entry.mjs';

const app = express();

// Custom middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// Astro handles everything else
app.use(astroHandler);

app.listen(4321, () => {
  console.log('Server running on http://localhost:4321');
});
```

## Output Modes Summary

| `output`    | Adapter Required | Pages Rendered        | Use Case                        |
| ----------- | ---------------- | --------------------- | ------------------------------- |
| `static`    | No               | Build time            | Blogs, marketing, docs          |
| `server`    | Yes (Node, etc.) | Request time (all)    | Dashboards, authenticated apps  |
| `hybrid`    | Yes              | Build + Request time  | Mostly static + few SSR pages   |

## Post-Deploy Verification

```bash
# 1. Check that HTML is served
curl -I https://your-site.com/

# 2. Verify SSR (if applicable) — must return fresh data
curl https://your-site.com/dashboard | grep "Active users"

# 3. Verify content collections are working
curl https://your-site.com/blog/

# 4. Check for zero-JS output (static pages)
curl https://your-site.com/ | grep -c "<script"

# 5. Test API endpoints
curl https://your-site.com/api/search.json?q=test

# 6. Verify actions
curl -X POST https://your-site.com/_actions/subscribe \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com"}'
```

## Environment Variables by Deploy Target

| Target           | Server-side               | Client-side (PUBLIC_)          |
| ---------------- | ------------------------- | ------------------------------ |
| **Static**       | N/A (build time only)    | `import.meta.env.PUBLIC_*`    |
| **Node.js**      | `import.meta.env.VAR`    | `import.meta.env.PUBLIC_*`    |
| **Vercel**       | `import.meta.env.VAR`    | `import.meta.env.PUBLIC_*`    |
| **Netlify**      | `import.meta.env.VAR`    | `import.meta.env.PUBLIC_*`    |
| **Cloudflare**   | `import.meta.env.VAR`    | `import.meta.env.PUBLIC_*`    |
