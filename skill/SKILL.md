---
name: astro-builder
description: "Build complete, SEO-optimized websites using Astro with Tailwind CSS, content collections, and optional Turso database. Use this skill whenever the user wants to create a website, build a site that ranks on Google, start an Astro project, make a landing page, build a business website, create a blog, or any web project where SEO matters. Also trigger when the user mentions Astro or wants a site built with modern web tech. Even if they just say 'build me a website' or 'I need a site for my business' — this skill handles it."
---

# Astro SEO Website Builder

You build complete, deployment-ready websites in Astro that are engineered to rank on Google. Every site ships with proper meta tags, structured data, semantic HTML, optimized images, sitemaps, and real researched content — not placeholder text.

Every frontend you produce must be premium and non-generic. Read `references/taste-skill.md` before writing any UI code — it contains the design rules that prevent generic "AI slop" output.

## Tech Stack

- **Astro** — Static-first framework, zero JS by default, perfect for SEO
- **React** — Interactive components via Astro islands (forms, nav, animations)
- **Tailwind CSS** — All styling. No shadcn defaults — customize everything per the taste-skill rules
- **Astro Content Collections** — Type-safe Markdown/MDX content for blog posts, services, testimonials
- **Turso + Drizzle** — Edge SQLite database (optional, for forms/dynamic data)

## Workflow Overview

Every project follows this sequence. Don't skip steps.

### Phase 1: Discovery & Research
1. Ask the user about their niche, business, and goals
2. Ask if they need a database (for contact forms, leads, bookings, etc.)
3. Research the niche — what's ranking, what keywords matter, what content gaps exist
4. Propose a site structure with page list, content strategy, and keyword targets
5. Get user approval before building

### Phase 2: Project Setup
1. Scaffold the Astro project
2. Install and configure integrations (React, Tailwind, sitemap)
3. Set up the base layout, SEO components, and design system

### Phase 3: Build Every Page
1. Read `references/taste-skill.md` for design rules
2. If user wants generated images, read `references/image-generation.md` and generate images for each page
3. Create each page with real, researched content
4. Full SEO on every page — meta tags, OG tags, structured data, semantic HTML
5. Wire up interactive components (forms, navigation, etc.)

### Phase 4: Content Collections & Database
1. Configure Astro content collections for blog posts, services, etc.
2. If database requested: set up Turso, create schemas, build API routes
3. Build admin dashboard for database data if applicable

### Phase 5: Deployment Prep
1. Generate sitemap, robots.txt
2. Configure deployment (Vercel/Netlify)
3. Final SEO audit of every page

---

## Phase 1: Discovery & Research

### Ask the User

Before writing any code, understand the project:

1. **What's your niche/business?** — "I'm a plumber in Dublin" or "I sell handmade candles online"
2. **What's the site's primary goal?** — Lead generation, e-commerce, portfolio, blog, SaaS landing page
3. **How many pages do you want?** — Small (5-10 pages) or content-heavy (15-30+ pages with blog)
4. **Do you need a database?** — For storing form submissions, bookings, user data, etc. If yes, set up Turso.
5. **Do you want generated images?** — "I can generate custom stock images for your site using Gemini. If you'd like this, add your `GEMINI_API_KEY` to the `.env` file."
6. **Do you have a domain?** — Needed for sitemap/canonical URLs. Use a placeholder if not yet purchased.
6. **Any design preferences?** — Colors, style, existing branding

### Research the Niche

Use web search to find:

- **What's currently ranking** for the target keywords — top 5-10 results
- **Content gaps** — what questions aren't being answered well
- **Long-tail keywords** — specific phrases with less competition
- **Local SEO signals** if it's a local business
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

Read `references/astro-setup.md` for exact commands and configuration.

### 1. Create the Project
```bash
npm create astro@latest [project-name] -- --template minimal --install --no-git
cd [project-name]
```

### 2. Install Integrations
```bash
npx astro add react tailwind sitemap
npm install @astrojs/check typescript
npm install @fontsource/geist-sans @fontsource/geist-mono  # Premium fonts per taste-skill
```

### 3. If Database Requested
Read `references/turso-setup.md` for full Turso configuration.
```bash
npm install @libsql/client drizzle-orm
```

### 4. Project Structure
```
src/
├── components/
│   ├── SEOHead.astro
│   ├── Header.astro
│   ├── Footer.astro
│   ├── Navigation.tsx    # React island
│   └── ContactForm.tsx   # React island
├── content.config.ts     # Content collection schemas (NOT in content/)
├── content/
│   ├── blog/
│   ├── services/
│   └── testimonials/
├── layouts/
│   └── BaseLayout.astro
├── lib/
│   ├── turso.ts          # If database
│   └── utils.ts
├── pages/
│   ├── index.astro
│   ├── about.astro
│   ├── contact.astro
│   ├── blog/
│   │   ├── index.astro
│   │   └── [...slug].astro
│   └── api/              # API routes (if database)
├── styles/
│   └── globals.css
└── middleware.ts
astro.config.mjs
```

---

## Phase 3: Build Every Page — Design + SEO

Before writing any page, read both:
- `references/seo-checklist.md` — SEO rules for every page
- `references/taste-skill.md` — Design rules to prevent generic output

### The SEOHead Component

Create `src/components/SEOHead.astro` — goes in `<head>` of every page. Handles:
- Title tag (unique per page, primary keyword, under 60 chars)
- Meta description (unique, CTA, under 160 chars)
- Canonical URL
- Open Graph tags
- Twitter Card tags
- JSON-LD structured data

See `references/seo-checklist.md` for the exact component code.

### Content Writing Rules

- **Write for humans first, search engines second.**
- **Every page targets a specific keyword cluster.** Primary keyword in title, H1, first paragraph, meta description, URL slug.
- **Proper heading hierarchy.** One H1. H2s for sections. H3s for subsections. Never skip levels.
- **Internal linking.** Every page links to 2-3+ other pages. Descriptive anchor text.
- **Image alt text.** Every image gets descriptive alt text with natural keywords.
- **Content length.** Service pages: 800-1500 words. Blog posts: 1500-3000 words. Home: 500-1000 words.
- **Schema markup.** Every page gets appropriate JSON-LD.

### Design Rules (from taste-skill)

These are non-negotiable for every page you build:

- **Typography:** Use Geist, Satoshi, Outfit, or Cabinet Grotesk. NEVER Inter. Headlines: `text-4xl md:text-6xl tracking-tighter leading-none`.
- **Colors:** Max 1 accent color, saturation < 80%. No purple/blue AI gradients. Neutral bases (Zinc/Slate).
- **Layout:** No centered hero when DESIGN_VARIANCE > 4. Use split-screen, asymmetric, or left-aligned layouts.
- **Cards:** Only when elevation communicates hierarchy. No generic 3-column card layouts.
- **Spacing:** `min-h-[100dvh]` not `h-screen`. Grid not flexbox math. `max-w-7xl mx-auto`.
- **States:** Every interactive component needs loading, empty, and error states.
- **No AI slop:** No emojis, no "John Doe", no "Acme Corp", no filler words like "seamless" or "unleash".

### Astro Islands for Interactivity

- **Static content** (headings, text, images) — `.astro` components, zero JS
- **Interactive elements** (forms, mobile nav, scroll animations) — React `.tsx` with `client:visible` or `client:load`
- **Animations:** For motion (MOTION_INTENSITY > 5), use Framer Motion in isolated React islands. Spring physics: `type: "spring", stiffness: 100, damping: 20`.

Read `references/tailwind-components.md` for component patterns (nav, forms, FAQ, etc.)

---

## Phase 4: Content Collections & Database

### Astro Content Collections

Read `references/content-collections.md` for the full setup. Content collections replace a CMS — content lives as Markdown files in `src/content/` with type-safe schemas.

**IMPORTANT:** Define schemas in `src/content.config.ts` (NOT `src/content/config.ts` — the old path throws `LegacyContentConfigError` in Astro v5+). Every collection must have a `loader` (use `glob()` from `astro/loaders`).

Define schemas in `src/content.config.ts`:
- **Blog posts:** title, description, pubDate, image, tags, author, draft
- **Services:** title, description, icon, order, featured
- **Testimonials:** name, role, quote, rating

Query collections in pages with `getCollection()` and `getEntry()`.

### Turso Database (If Requested)

Read `references/turso-setup.md` for complete setup. Turso handles dynamic data — form submissions, leads, bookings, newsletter signups.

1. Create the database client in `src/lib/turso.ts`
2. Define schemas with Drizzle ORM
3. Create API routes in `src/pages/api/` for form handling
4. Build an admin dashboard to view submitted data

### Admin Dashboard (If Database)

Build at `/admin` using React islands with Tailwind-styled tables and cards. Uses:
- Middleware for basic auth
- API routes to fetch data from Turso
- Clean, tasteful data presentation (per taste-skill rules — no generic card grids)

---

## Phase 5: Deployment Prep

### Sitemap & Robots.txt

`@astrojs/sitemap` auto-generates the sitemap. Set `site` in `astro.config.mjs`.

Dynamic `robots.txt` at `src/pages/robots.txt.ts`:
```typescript
import type { APIRoute } from 'astro';

const getRobotsTxt = (siteURL: string) => `User-agent: *
Allow: /
Disallow: /admin
Sitemap: ${siteURL}sitemap-index.xml`;

export const GET: APIRoute = ({ site }) => {
  return new Response(getRobotsTxt(site?.toString() || ''));
};
```

### Deployment Configuration

Read `references/deployment.md` for Vercel/Netlify configs.

For sites with API routes or Turso, add an adapter but keep the default static output. Mark individual server routes with `export const prerender = false;`:
```javascript
import vercel from '@astrojs/vercel';

export default defineConfig({
  // Do NOT use output: 'hybrid' — removed in Astro v5.
  // Default is 'static'. Server-rendered routes opt out individually.
  adapter: vercel(),
});
```

In each API route or server-rendered page:
```typescript
// src/pages/api/contact.ts
export const prerender = false; // This route runs on the server

export const POST: APIRoute = async ({ request }) => {
  // ...
};
```

### Final SEO Audit

Audit every page against `references/seo-checklist.md`:
- Unique title, description, canonical URL per page
- All images have alt text
- JSON-LD validates
- Internal links work
- Mobile responsive (test taste-skill mobile override rules)
- Sitemap includes all pages
- robots.txt correct

---

## Important Notes

- **Do NOT use `output: 'hybrid'`** — it was removed in Astro v5. Use the default `'static'` output. For server-rendered routes (API endpoints, form handlers), add `export const prerender = false;` in each file.
- **React islands need hydration directives** — `client:load` or `client:visible`.
- **Content collections are file-based.** Blog posts and services live as `.md` files in `src/content/`. Turso is for dynamic data (form submissions) only.
- **Research before building.** The niche research phase determines SEO quality.
- **Real content wins.** Write substantive, helpful content. No lorem ipsum.
- **Premium design always.** Follow the taste-skill rules. Every page should look like a $10k agency build, not generic AI output.
- **Image generation is optional.** If the user provides a `GEMINI_API_KEY`, generate custom images per `references/image-generation.md`. If not, use styled placeholder divs with proper alt text.
