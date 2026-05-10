# TanStack Start: Deployment Guide

Configuration and deployment guides for the different environments supported by TanStack Start.

## Node.js / Docker Deployment

The simplest deployment: a Node.js server running the compiled output.

### Configuration

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

```bash
# Production build
pnpm build

# Start server
pnpm start
# Server running on http://localhost:3000
```

### Dockerfile

```dockerfile
# Dockerfile
FROM node:20-alpine AS base
WORKDIR /app

FROM base AS deps
COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm install --frozen-lockfile

FROM base AS build
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN pnpm build

FROM base AS runner
ENV NODE_ENV=production
COPY --from=build /app/.output ./.output

EXPOSE 3000
CMD ["node", ".output/server/index.mjs"]
```

## Cloudflare Workers Deployment

TanStack Start natively supports Cloudflare Workers.

### Configuration

```bash
pnpm add -D wrangler
```

```json
// package.json
{
  "scripts": {
    "dev": "vite dev",
    "build": "vite build && tsc --noEmit",
    "preview": "vite preview",
    "deploy": "npm run build && wrangler deploy",
    "cf-typegen": "wrangler types"
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
    tanstackStart({
      // Target Cloudflare Workers
      target: 'cloudflare-pages',
    }),
    react(),
  ],
})
```

```toml
# wrangler.toml
name = "my-app"
compatibility_date = "2024-12-30"
main = ".output/server/index.mjs"

[site]
bucket = ".output/public"
```

```bash
# Deploy to Cloudflare
pnpm deploy
```

### Environment Variables on Cloudflare

```toml
# wrangler.toml
[vars]
DATABASE_URL = "your-db-url"

# Secrets (do not commit)
# wrangler secret put SECRET_API_KEY
```

```typescript
// In server functions
const dbUrl = process.env.DATABASE_URL
```

## Vercel Deployment

TanStack Start works on Vercel through the `@tanstack/react-start` preset.

### Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import { tanstackStart } from '@tanstack/react-start/plugin/vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [
    tanstackStart({
      target: 'vercel',
    }),
    react(),
  ],
})
```

```json
// package.json
{
  "scripts": {
    "build": "vite build",
    "dev": "vite dev"
  }
}
```

```json
// vercel.json
{
  "buildCommand": "vite build",
  "outputDirectory": ".output",
  "framework": "vite"
}
```

### Vercel Project Settings

1. Connect the repository on Vercel
2. Framework: **Vite**
3. Build Command: `vite build`
4. Output Directory: `.output`
5. Install Command: `pnpm install`

## Netlify Deployment

Requires the official `@netlify/vite-plugin-tanstack-start` plugin.

```bash
pnpm add -D @netlify/vite-plugin-tanstack-start
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import { tanstackStart } from '@tanstack/react-start/plugin/vite'
import netlify from '@netlify/vite-plugin-tanstack-start'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [
    tanstackStart(),
    netlify(),
    react(),
  ],
})
```

```toml
# netlify.toml
[build]
  command = "vite build"
  publish = ".output/public"

[functions]
  directory = ".output/functions"
  node_bundler = "esbuild"
```

## Bun Deployment

```json
// package.json
{
  "scripts": {
    "start": "bun .output/server/index.mjs"
  }
}
```

```bash
pnpm build
pnpm start
```

## Railway / Nitro Deployment

TanStack Start uses Nitro as the server layer, enabling deployment to multiple platforms:

```bash
# Railway, Render, Fly.io — work with the Node.js preset
pnpm build
# The .output/server/index.mjs file is the entrypoint
```

## Environment Variables by Environment

| Environment   | Server-side                          | Client-side                            |
| ------------- | ------------------------------------ | -------------------------------------- |
| **Node.js**   | `process.env.VAR`                   | `import.meta.env.VITE_*`              |
| **Cloudflare**| `process.env.VAR` (wrangler.toml)  | `import.meta.env.VITE_*`              |
| **Vercel**    | `process.env.VAR` (dashboard)       | `NEXT_PUBLIC_*` or `VITE_*`           |
| **Netlify**   | `process.env.VAR` (dashboard)       | `import.meta.env.VITE_*`              |

## Post-Deploy Verification

```bash
# 1. Check health endpoint
curl -I https://your-app.com/

# 2. Verify SSR (must return HTML)
curl https://your-app.com/ | grep "<html"

# 3. Check cache headers
curl -I https://your-app.com/posts | grep -i cache

# 4. Test server functions
curl -X POST https://your-app.com/api/health
```
