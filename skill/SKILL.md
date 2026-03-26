---
name: astro-builder
description: "Build complete, SEO-optimized websites using Astro with shadcn/ui, Keystatic CMS, and optional Turso database. Use this skill whenever the user wants to create a website, build a site that ranks on Google, start an Astro project, make a landing page, build a business website, create a blog, or any web project where SEO matters. Also trigger when the user mentions Astro, Keystatic, or wants a site built with modern web tech. Even if they just say 'build me a website' or 'I need a site for my business' — this skill handles it."
---

# Astro SEO Website Builder

You build complete, deployment-ready websites in Astro that are engineered to rank on Google. Every site you create ships with proper meta tags, structured data, semantic HTML, optimized images, sitemaps, and real researched content — not placeholder text.

## Your Tech Stack

- **Astro** — Static-first framework, zero JS by default, perfect for SEO
- **React + shadcn/ui** — Interactive components via Astro islands
- **Keystatic** — File-based CMS with admin UI at `/keystatic`
- **Turso + Drizzle** — Edge SQLite database (optional, for forms/data)
- **Tailwind CSS** — Styling via shadcn's design system

## Workflow Overview

Every project follows this sequence. Don't skip steps — each one matters for the final result.

### Phase 1: Discovery & Research
1. Ask the user about their niche, business, and goals
2. Ask if they need a database (for contact forms, leads, bookings, etc.)
3. Research the niche — what's ranking, what keywords matter, what content gaps exist
4. Propose a site structure with page list, content strategy, and keyword targets
5. Get user approval before building

### Phase 2: Project Setup
1. Scaffold the Astro project
2. Install and configure all integrations
3. Set up the base layout, SEO components, and design system

### Phase 3: Build Every Page
1. Create each page with real, researched content
2. Full SEO on every page — meta tags, OG tags, structured data, semantic HTML
3. Wire up interactive components (forms, navigation, etc.)

### Phase 4: CMS & Database
1. Configure Keystatic with content collections matching the site structure
2. If database requested: set up Turso, create schemas, build API routes
3. Build admin dashboard for database data if applicable

### Phase 5: Deployment Prep
1. Generate sitemap, robots.txt, and manifest
2. Configure deployment (Vercel/Netlify)
3. Final SEO audit of every page

---

## Phase 1: Discovery & Research

### Ask the User

Before writing any code, you need to understand the project. Ask these questions (use AskUserQuestion if available):

1. **What's your niche/business?** — "I'm a plumber in Dublin" or "I sell handmade candles online"
2. **What's the site's primary goal?** — Lead generation, e-commerce, portfolio, blog, SaaS landing page
3. **How many pages do you want?** — Small (5-10 pages) or content-heavy (15-30+ pages with blog)
4. **Do you need a database?** — For storing form submissions, bookings, user data, etc. If yes, you'll set up Turso.
5. **Do you have a domain?** — Needed for sitemap/canonical URLs. Use a placeholder if not yet purchased.
6. **Any design preferences?** — Colors, style, existing branding

### Research the Niche

Once you know the niche, do real research. Use web search to find:

- **What's currently ranking** for the target keywords — look at the top 5-10 results
- **Content gaps** — what questions aren't being answered well
- **Long-tail keywords** — specific phrases with less competition
- **Local SEO signals** if it's a local business (Google Business categories, local schema)
- **Competitor site structures** — what pages they have, what they're missing

### Propose the Site Structure

Based on research, create a concrete proposal. Example for a plumber:

```
Proposed Pages:
1. Home — "Plumber Dublin | 24/7 Emergency Plumbing Services"
2. Services — Overview of all services
3. Services/Emergency-Plumbing — Targeted landing page
4. Services/Bathroom-Renovations — Targeted landing page
5. Services/Boiler-Repair — Targeted landing page
6. About — Trust signals, team, qualifications
7. Blog — Ongoing content for SEO
8. Blog/[slug] — Individual posts targeting long-tail keywords
9. Contact — Form + map + phone
10. Areas We Serve — Local SEO pages
11. Testimonials — Social proof

Target Keywords: emergency plumber dublin, plumber near me dublin, boiler repair dublin...
Content Strategy: Weekly blog posts targeting "how to fix [common issue]" queries
```

Get the user's sign-off before proceeding.

---

## Phase 2: Project Setup

Read `references/astro-setup.md` for the exact commands and configuration. The setup sequence is:

### 1. Create the Project
```bash
npm create astro@latest [project-name] -- --template minimal --install --no-git
cd [project-name]
```

### 2. Install Core Integrations
```bash
npx astro add react tailwind sitemap
npm install @keystatic/core @keystatic/astro
npm install @astrojs/check typescript
```

### 3. Install shadcn/ui
Read `references/shadcn-setup.md` for the full setup. Key steps:
- Configure tsconfig.json path aliases
- Run `npx shadcn@latest init`
- Add components as needed: `npx shadcn@latest add button card form input textarea`

### 4. If Database Requested
Read `references/turso-setup.md` for full Turso configuration.
```bash
npm install @libsql/client drizzle-orm
```

### 5. Project Structure
After setup, your project should look like:
```
src/
├── components/
│   ├── ui/          # shadcn components
│   ├── SEOHead.astro
│   ├── Header.astro
│   ├── Footer.astro
│   ├── Navigation.tsx    # React island
│   └── ContactForm.tsx   # React island
├── content/
│   ├── config.ts
│   └── blog/
├── layouts/
│   └── BaseLayout.astro
├── lib/
│   ├── turso.ts     # If database
│   └── utils.ts
├── pages/
│   ├── index.astro
│   ├── about.astro
│   ├── contact.astro
│   ├── blog/
│   │   ├── index.astro
│   │   └── [...slug].astro
│   ├── api/         # API routes
│   └── keystatic/
│       └── [...params].ts
├── styles/
│   └── globals.css
└── middleware.ts
keystatic.config.ts      # At project root
astro.config.mjs
```

---

## Phase 3: Build Every Page — The SEO Engine

This is where the skill earns its keep. Every single page must be fully SEO-optimized. Read `references/seo-checklist.md` for the complete checklist you apply to every page.

### The SEOHead Component

Create `src/components/SEOHead.astro` — this goes in the `<head>` of every page. It handles:
- Title tag (unique per page, includes primary keyword, under 60 chars)
- Meta description (unique, includes CTA, under 160 chars)
- Canonical URL
- Open Graph tags (title, description, image, type, url)
- Twitter Card tags
- Article metadata (author, publish date, tags) for blog posts
- JSON-LD structured data appropriate to the page type

Read `references/seo-checklist.md` for the exact component code and usage patterns.

### Content Writing Rules

When writing page content, follow these principles:

- **Write for humans first, search engines second.** Google rewards content that genuinely helps people.
- **Every page targets a specific keyword cluster.** The primary keyword appears in the title, H1, first paragraph, meta description, and URL slug. Secondary keywords appear naturally throughout.
- **Use proper heading hierarchy.** One H1 per page. H2s for major sections. H3s for subsections. Never skip levels.
- **Internal linking.** Every page links to at least 2-3 other pages on the site. Use descriptive anchor text, not "click here".
- **Image alt text.** Every image gets descriptive alt text that includes relevant keywords where natural.
- **Content length.** Service pages: 800-1500 words. Blog posts: 1500-3000 words. Home page: 500-1000 words.
- **Schema markup.** Every page gets appropriate JSON-LD. See the checklist for which schema types to use where.

### shadcn/ui Component Patterns

When building interactive elements, remember the Astro islands architecture:

- **Static content** (headings, text, images) — Use `.astro` components, zero JS
- **Interactive elements** (forms, mobile nav, tabs, accordions) — Use React `.tsx` wrappers with `client:load` or `client:visible`
- **Context sharing** — Multiple interactive elements that share state must be wrapped in a single React component

Common shadcn components you'll use:
- `Button`, `Card`, `Input`, `Textarea` — Contact forms
- `Sheet` / `Dialog` — Mobile navigation
- `Accordion` — FAQ sections (great for SEO with FAQ schema)
- `Tabs` — Service comparisons
- `Table` — Pricing, feature comparisons

---

## Phase 4: CMS & Database

### Keystatic Configuration

Read `references/keystatic-setup.md` for the full config. Keystatic manages your file-based content (blog posts, service pages, testimonials). Configure collections that match your site structure.

The admin UI lives at `/keystatic` — the user can add/edit content through a visual editor without touching code.

### Turso Database (If Requested)

Read `references/turso-setup.md` for the complete setup.

Turso handles dynamic data — form submissions, leads, bookings, newsletter signups. The typical setup:

1. **Create the database client** in `src/lib/turso.ts`
2. **Define schemas** with Drizzle ORM
3. **Create API routes** in `src/pages/api/` for form handling
4. **Build an admin dashboard** to view submitted data

### Admin Dashboard

If the user wants a database, build a simple admin dashboard at `/admin` that shows submitted data. This uses:
- Middleware for basic authentication (check for admin session)
- shadcn Table, Card, and Badge components for the data display
- API routes to fetch data from Turso
- The admin can also access Keystatic at `/keystatic` for content management

The admin dashboard is a React island that fetches data from your API routes and displays it in shadcn Tables. Read `references/turso-setup.md` for the admin dashboard patterns.

---

## Phase 5: Deployment Prep

### Sitemap & Robots.txt

The `@astrojs/sitemap` integration auto-generates your sitemap. Make sure `site` is set in `astro.config.mjs`.

Create a dynamic `robots.txt` at `src/pages/robots.txt.ts`:
```typescript
import type { APIRoute } from 'astro';

const getRobotsTxt = (siteURL: string) => `User-agent: *
Allow: /
Disallow: /admin
Disallow: /keystatic
Sitemap: ${siteURL}sitemap-index.xml`;

export const GET: APIRoute = ({ site }) => {
  return new Response(getRobotsTxt(site?.toString() || ''));
};
```

### Deployment Configuration

Read `references/deployment.md` for Vercel and Netlify configs.

For sites with API routes or Turso, you need SSR or hybrid rendering:
```javascript
// astro.config.mjs
import vercel from '@astrojs/vercel';

export default defineConfig({
  output: 'hybrid', // Static by default, SSR where needed
  adapter: vercel(),
});
```

### Final SEO Audit

Before declaring the site done, audit every page against `references/seo-checklist.md`. Check:
- Every page has unique title, description, canonical URL
- All images have alt text
- JSON-LD validates (test with Google's Rich Results Test)
- Internal links work
- Mobile responsive
- Core Web Vitals pass (Lighthouse score)
- Sitemap includes all pages
- robots.txt is correct

---

## Important Notes

- **Always use `output: 'hybrid'`** when the site has API routes, forms, or Keystatic admin. Pure static won't work for these.
- **shadcn components need `client:load` or `client:visible`** — they won't be interactive without hydration directives.
- **Keystatic is file-based, not database-based.** It manages content (blog posts, page text). Turso manages dynamic data (form submissions, user data). They serve different purposes.
- **Research before building.** The niche research phase directly determines the quality of your SEO. Don't rush it.
- **Real content wins.** Google can detect thin/AI-generated content. Write substantive, helpful content that actually answers the searcher's question. Use the research to understand what people are looking for, then deliver it.
