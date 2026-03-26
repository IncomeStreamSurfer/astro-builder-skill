# Keystatic CMS Setup Reference

## Table of Contents
1. [Installation](#installation)
2. [Configuration](#configuration)
3. [Content Collections](#content-collections)
4. [API Route](#api-route)
5. [Accessing Content in Pages](#accessing-content)
6. [Storage Modes](#storage-modes)

---

## Installation

```bash
npm install @keystatic/core @keystatic/astro
```

Add to `astro.config.mjs`:
```javascript
import keystatic from '@keystatic/astro';

export default defineConfig({
  integrations: [
    // ... other integrations
    keystatic(),
  ],
  output: 'hybrid', // Required for Keystatic admin UI
});
```

---

## Configuration

Create `keystatic.config.ts` at the PROJECT ROOT (not in src/):

```typescript
import { config, collection, singleton, fields } from '@keystatic/core';

export default config({
  storage: {
    kind: 'local', // Files stored in your project
  },
  ui: {
    brand: {
      name: 'Your Site Name',
    },
  },
  collections: {
    posts: collection({
      label: 'Blog Posts',
      slugField: 'title',
      path: 'src/content/blog/*',
      format: { contentField: 'content' },
      schema: {
        title: fields.slug({ name: { label: 'Title' } }),
        description: fields.text({
          label: 'Description',
          description: 'SEO meta description (under 160 chars)',
          validation: { length: { max: 160 } },
        }),
        pubDate: fields.date({ label: 'Publish Date', defaultValue: { kind: 'today' } }),
        updatedDate: fields.date({ label: 'Updated Date' }),
        image: fields.image({
          label: 'Featured Image',
          directory: 'public/images/blog',
          publicPath: '/images/blog/',
        }),
        imageAlt: fields.text({ label: 'Image Alt Text' }),
        tags: fields.array(fields.text({ label: 'Tag' }), {
          label: 'Tags',
          itemLabel: (props) => props.value,
        }),
        author: fields.text({ label: 'Author', defaultValue: 'Site Author' }),
        draft: fields.checkbox({ label: 'Draft', defaultValue: false }),
        content: fields.markdoc({
          label: 'Content',
          options: {
            image: {
              directory: 'public/images/blog',
              publicPath: '/images/blog/',
            },
          },
        }),
      },
    }),

    services: collection({
      label: 'Services',
      slugField: 'title',
      path: 'src/content/services/*',
      format: { contentField: 'content' },
      schema: {
        title: fields.slug({ name: { label: 'Service Name' } }),
        description: fields.text({
          label: 'Description',
          validation: { length: { max: 160 } },
        }),
        icon: fields.text({ label: 'Icon Name (Lucide icon)' }),
        order: fields.integer({ label: 'Display Order', defaultValue: 0 }),
        featured: fields.checkbox({ label: 'Featured on Home Page', defaultValue: false }),
        content: fields.markdoc({ label: 'Service Details' }),
      },
    }),

    testimonials: collection({
      label: 'Testimonials',
      slugField: 'name',
      path: 'src/content/testimonials/*',
      schema: {
        name: fields.slug({ name: { label: 'Client Name' } }),
        role: fields.text({ label: 'Client Role/Company' }),
        quote: fields.text({ label: 'Testimonial Quote', multiline: true }),
        rating: fields.integer({
          label: 'Star Rating',
          defaultValue: 5,
          validation: { min: 1, max: 5 },
        }),
        image: fields.image({
          label: 'Client Photo',
          directory: 'public/images/testimonials',
          publicPath: '/images/testimonials/',
        }),
      },
    }),
  },

  singletons: {
    siteSettings: singleton({
      label: 'Site Settings',
      path: 'src/content/settings/site',
      schema: {
        siteName: fields.text({ label: 'Site Name' }),
        siteDescription: fields.text({ label: 'Site Description' }),
        phone: fields.text({ label: 'Phone Number' }),
        email: fields.text({ label: 'Email Address' }),
        address: fields.text({ label: 'Business Address', multiline: true }),
        socialLinks: fields.object({
          facebook: fields.url({ label: 'Facebook URL' }),
          twitter: fields.url({ label: 'Twitter/X URL' }),
          instagram: fields.url({ label: 'Instagram URL' }),
          linkedin: fields.url({ label: 'LinkedIn URL' }),
        }),
      },
    }),
  },
});
```

---

## API Route

Create `src/pages/api/keystatic/[...params].ts`:

```typescript
import { makeHandler } from '@keystatic/astro/api';

export const prerender = false;

export const ALL = makeHandler();
```

This enables the Keystatic admin UI at `/keystatic`.

---

## Accessing Content in Pages

### Blog listing page

```astro
---
import { getCollection } from 'astro:content';
import BaseLayout from '../layouts/BaseLayout.astro';

const posts = await getCollection('blog', ({ data }) => !data.draft);
const sortedPosts = posts.sort(
  (a, b) => new Date(b.data.pubDate).getTime() - new Date(a.data.pubDate).getTime()
);
---

<BaseLayout title="Blog | Your Site" description="Latest articles and insights">
  <section class="max-w-4xl mx-auto px-4 py-12">
    <h1 class="text-4xl font-bold mb-8">Blog</h1>
    <div class="grid gap-8">
      {sortedPosts.map((post) => (
        <article class="border-b pb-8">
          <a href={`/blog/${post.slug}`} class="group">
            <h2 class="text-2xl font-semibold group-hover:text-primary transition-colors">
              {post.data.title}
            </h2>
            <p class="text-muted-foreground mt-2">{post.data.description}</p>
            <time class="text-sm text-muted-foreground mt-1 block">
              {new Date(post.data.pubDate).toLocaleDateString('en-IE', {
                year: 'numeric', month: 'long', day: 'numeric'
              })}
            </time>
          </a>
        </article>
      ))}
    </div>
  </section>
</BaseLayout>
```

### Singleton data (site settings)

Keystatic singletons are accessed via file reads since they're stored as files. For Astro content collections, define a matching collection or read the JSON/YAML file directly.

---

## Storage Modes

### Local (development)
```typescript
storage: { kind: 'local' }
```
Content is stored as files in your project. Good for development and static sites.

### GitHub (production with collaboration)
```typescript
storage: {
  kind: 'github',
  repo: 'owner/repo',
}
```
Content changes create GitHub commits/PRs. Requires GitHub OAuth setup.

### Cloud (Keystatic Cloud)
```typescript
storage: {
  kind: 'cloud',
  project: 'your-team/your-project',
}
```
Managed storage by Keystatic. Easiest for non-technical users.

---

## Important Notes

- Keystatic admin runs at `/keystatic` — exclude this from your sitemap
- Keystatic requires `output: 'hybrid'` or `output: 'server'` in astro.config.mjs
- Content is file-based (Markdown/YAML/JSON) — it does NOT connect to databases
- For database data (form submissions, etc.), build a separate admin dashboard
- Images uploaded via Keystatic go to the configured `directory` path
- The markdoc content field supports rich text, images, and custom components
