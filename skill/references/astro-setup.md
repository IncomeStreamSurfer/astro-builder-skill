# Astro Project Setup Reference

## Table of Contents
1. [Create Project](#create-project)
2. [Install Integrations](#install-integrations)
3. [Configure astro.config.mjs](#configure-astro-config)
4. [Base Layout](#base-layout)
5. [Global Styles](#global-styles)
6. [Content Collections](#content-collections)
7. [Middleware](#middleware)

---

## Create Project

```bash
npm create astro@latest [project-name] -- --template minimal --install --no-git
cd [project-name]
```

## Install Integrations

```bash
# Core
npx astro add react tailwind sitemap

# Typography plugin
npm install @tailwindcss/typography

# Premium fonts (per taste-skill)
npm install @fontsource/geist-sans @fontsource/geist-mono

# TypeScript
npm install @astrojs/check typescript

# Optional: Vercel adapter (for hybrid/SSR)
npx astro add vercel

# Optional: Netlify adapter
npx astro add netlify
```

## Configure astro.config.mjs

```javascript
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import tailwind from '@astrojs/tailwind';
import sitemap from '@astrojs/sitemap';
import vercel from '@astrojs/vercel'; // or netlify

export default defineConfig({
  site: 'https://yourdomain.com', // REQUIRED for sitemap & canonical URLs
  // NOTE: Do NOT use output: 'hybrid' — it was removed in Astro v5.
  // Use output: 'static' (the default). For server-rendered routes, use
  // export const prerender = false; in individual pages/API routes.
  adapter: vercel(), // or netlify() — only needed if you have server-rendered routes
  integrations: [
    react(),
    tailwind({ applyBaseStyles: false }),
    sitemap({
      filter: (page) => !page.includes('/admin'),
    }),
  ],
  trailingSlash: 'never',
  build: {
    format: 'directory',
  },
  image: {
    domains: [], // Add external image domains here
  },
});
```

## tsconfig.json

```json
{
  "extends": "astro/tsconfigs/strict",
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "react",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## Base Layout

Create `src/layouts/BaseLayout.astro`:

```astro
---
import SEOHead from '../components/SEOHead.astro';
import Header from '../components/Header.astro';
import Footer from '../components/Footer.astro';
import '../styles/globals.css';

interface Props {
  title: string;
  description: string;
  ogImage?: string;
  ogType?: string;
  canonicalUrl?: string;
  author?: string;
  publishDate?: string;
  modifiedDate?: string;
  tags?: string[];
  structuredData?: object;
}

const props = Astro.props;
---

<!doctype html>
<html lang="en">
  <head>
    <SEOHead {...props} />
  </head>
  <body class="min-h-screen flex flex-col bg-background text-foreground">
    <Header />
    <main class="flex-1">
      <slot />
    </main>
    <Footer />
  </body>
</html>
```

## Global Styles

Create `src/styles/globals.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

## Content Collections

**IMPORTANT:** Config file goes at `src/content.config.ts` (NOT `src/content/config.ts`). Every collection needs a `loader`. See `references/content-collections.md` for full schemas.

Create `src/content.config.ts`:

```typescript
import { z, defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';

const blogCollection = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.date(),
    updatedDate: z.date().optional(),
    image: z.string().optional(),
    imageAlt: z.string().optional(),
    tags: z.array(z.string()).default([]),
    author: z.string().default('Site Author'),
    draft: z.boolean().default(false),
    canonicalUrl: z.string().optional(),
  }),
});

export const collections = {
  blog: blogCollection,
};
```

Example blog post at `src/content/blog/first-post.md`:
```markdown
---
title: "Your SEO-Optimized Title Here"
description: "A compelling meta description under 160 characters"
pubDate: 2026-03-26
image: "/images/blog/first-post.jpg"
imageAlt: "Descriptive alt text"
tags: ["keyword1", "keyword2"]
author: "Author Name"
---

# Heading That Matches Search Intent

Your content here...
```

## Dynamic Blog Routes

Create `src/pages/blog/[...slug].astro`:

```astro
---
import { getCollection } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog', ({ data }) => !data.draft);
  return posts.map((entry) => ({
    params: { slug: entry.slug },
    props: { entry },
  }));
}

const { entry } = Astro.props;
const { Content } = await entry.render();

const structuredData = {
  '@context': 'https://schema.org',
  '@type': 'BlogPosting',
  headline: entry.data.title,
  description: entry.data.description,
  image: entry.data.image,
  datePublished: entry.data.pubDate.toISOString(),
  dateModified: (entry.data.updatedDate || entry.data.pubDate).toISOString(),
  author: { '@type': 'Person', name: entry.data.author },
};
---

<BaseLayout
  title={entry.data.title}
  description={entry.data.description}
  ogImage={entry.data.image}
  ogType="article"
  author={entry.data.author}
  publishDate={entry.data.pubDate.toISOString()}
  tags={entry.data.tags}
  structuredData={structuredData}
>
  <article class="max-w-3xl mx-auto px-4 py-12 prose prose-lg">
    <Content />
  </article>
</BaseLayout>
```

## Middleware

Create `src/middleware.ts`:

```typescript
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware(async (context, next) => {
  // Add security headers
  const response = await next();

  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'SAMEORIGIN');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');

  return response;
});
```
