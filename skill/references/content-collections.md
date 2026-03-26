# Astro Content Collections Reference

## Table of Contents
1. [Setup](#setup)
2. [Defining Schemas](#defining-schemas)
3. [Creating Content](#creating-content)
4. [Querying Content](#querying-content)
5. [Dynamic Routes](#dynamic-routes)

---

## Setup

Content collections live in `src/content/`. Each subfolder is a collection.

**IMPORTANT (Astro v5+):** The config file MUST be at `src/content.config.ts` (NOT `src/content/config.ts`). The old `src/content/config.ts` location is legacy and will throw `LegacyContentConfigError`. Every collection MUST have a `loader` defined.

```
src/
├── content.config.ts    # Collection schemas (NOT in content/ folder!)
├── content/
│   ├── blog/
│   │   ├── first-post.md
│   │   └── second-post.md
│   ├── services/
│   │   ├── emergency-plumbing.md
│   │   └── boiler-repair.md
│   └── testimonials/
│       ├── john-murphy.json
│       └── sarah-walsh.json
```

## Defining Schemas

Create `src/content.config.ts` (in the `src/` root, NOT inside `src/content/`):

```typescript
import { z, defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';

const blogCollection = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    description: z.string().max(160),
    pubDate: z.date(),
    updatedDate: z.date().optional(),
    image: z.string().optional(),
    imageAlt: z.string().optional(),
    tags: z.array(z.string()).default([]),
    author: z.string().default('Site Author'),
    draft: z.boolean().default(false),
  }),
});

const servicesCollection = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/services' }),
  schema: z.object({
    title: z.string(),
    description: z.string().max(160),
    icon: z.string().optional(),
    order: z.number().default(0),
    featured: z.boolean().default(false),
  }),
});

const testimonialsCollection = defineCollection({
  loader: glob({ pattern: '**/*.json', base: './src/content/testimonials' }),
  schema: z.object({
    name: z.string(),
    role: z.string(),
    quote: z.string(),
    rating: z.number().min(1).max(5).default(5),
    image: z.string().optional(),
  }),
});

export const collections = {
  blog: blogCollection,
  services: servicesCollection,
  testimonials: testimonialsCollection,
};
```

## Creating Content

### Blog post (`src/content/blog/first-post.md`):
```markdown
---
title: "Your SEO-Optimized Title Here"
description: "Compelling meta description under 160 characters with CTA"
pubDate: 2026-03-26
image: "/images/blog/first-post.jpg"
imageAlt: "Descriptive alt text with natural keywords"
tags: ["keyword1", "keyword2"]
author: "Author Name"
---

# Heading That Matches Search Intent

Your content here. Remember: 1500-3000 words for blog posts.
Link to [related service pages](/services/emergency-plumbing) naturally.
```

### Service page (`src/content/services/emergency-plumbing.md`):
```markdown
---
title: "Emergency Plumbing"
description: "24/7 emergency plumber in Dublin. Fast response, fair prices."
icon: "Wrench"
order: 1
featured: true
---

Content about the service. 800-1500 words. Include pricing info,
process explanation, and FAQ section.
```

### Testimonial (`src/content/testimonials/john-murphy.json`):
```json
{
  "name": "Cillian Murray",
  "role": "Homeowner, Rathmines",
  "quote": "Called at 11pm on a Sunday and they were here in 30 minutes. Fixed the burst pipe and cleaned up. Couldn't ask for more.",
  "rating": 5
}
```

Note: Use realistic, creative names — not generic "John Doe" (see taste-skill rules).

## Querying Content

### Get all posts (sorted by date):
```typescript
import { getCollection } from 'astro:content';

const posts = await getCollection('blog', ({ data }) => !data.draft);
const sorted = posts.sort(
  (a, b) => new Date(b.data.pubDate).getTime() - new Date(a.data.pubDate).getTime()
);
```

### Get a single entry:
```typescript
import { getEntry } from 'astro:content';

const post = await getEntry('blog', 'first-post');
const { Content } = await post.render();
```

### Get featured services:
```typescript
const services = await getCollection('services');
const featured = services
  .filter(s => s.data.featured)
  .sort((a, b) => a.data.order - b.data.order);
```

### Get testimonials:
```typescript
const testimonials = await getCollection('testimonials');
```

## Dynamic Routes

### Blog post pages (`src/pages/blog/[...slug].astro`):

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
  structuredData={structuredData}
>
  <article class="max-w-3xl mx-auto px-4 py-12 prose prose-lg prose-zinc">
    <Content />
  </article>
</BaseLayout>
```

### Service pages (`src/pages/services/[...slug].astro`):

```astro
---
import { getCollection } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';

export async function getStaticPaths() {
  const services = await getCollection('services');
  return services.map((entry) => ({
    params: { slug: entry.slug },
    props: { entry },
  }));
}

const { entry } = Astro.props;
const { Content } = await entry.render();

const structuredData = {
  '@context': 'https://schema.org',
  '@type': 'Service',
  name: entry.data.title,
  description: entry.data.description,
};
---

<BaseLayout
  title={`${entry.data.title} | Your Business`}
  description={entry.data.description}
  structuredData={structuredData}
>
  <article class="max-w-4xl mx-auto px-4 py-12">
    <Content />
  </article>
</BaseLayout>
```
