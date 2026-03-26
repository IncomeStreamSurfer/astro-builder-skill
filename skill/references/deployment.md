# Deployment Reference

## Table of Contents
1. [Vercel Setup](#vercel)
2. [Netlify Setup](#netlify)
3. [Environment Variables](#environment-variables)
4. [Pre-Deploy Checklist](#pre-deploy-checklist)

---

## Vercel

### Install Adapter
```bash
npx astro add vercel
```

### Configure astro.config.mjs
```javascript
import vercel from '@astrojs/vercel';

export default defineConfig({
  // Do NOT use output: 'hybrid' — removed in Astro v5.
  // Default 'static' works. For server routes, add: export const prerender = false;
  adapter: vercel({
    webAnalytics: { enabled: true }, // Free Vercel analytics
  }),
  site: 'https://yourdomain.com',
});
```

### vercel.json (optional, for headers/redirects)
```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "SAMEORIGIN" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" }
      ]
    },
    {
      "source": "/fonts/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }
      ]
    }
  ]
}
```

### Deploy
```bash
npx vercel
```

Or connect your Git repo in the Vercel dashboard for automatic deployments.

---

## Netlify

### Install Adapter
```bash
npx astro add netlify
```

### Configure astro.config.mjs
```javascript
import netlify from '@astrojs/netlify';

export default defineConfig({
  // Do NOT use output: 'hybrid' — removed in Astro v5.
  adapter: netlify(),
  site: 'https://yourdomain.com',
});
```

### netlify.toml
```toml
[build]
  command = "npm run build"
  publish = "dist"

[[headers]]
  for = "/*"
  [headers.values]
    X-Content-Type-Options = "nosniff"
    X-Frame-Options = "SAMEORIGIN"
    Referrer-Policy = "strict-origin-when-cross-origin"

[[headers]]
  for = "/fonts/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[redirects]]
  from = "https://www.yourdomain.com/*"
  to = "https://yourdomain.com/:splat"
  status = 301
  force = true
```

### Deploy
```bash
npx netlify deploy --prod
```

Or connect your Git repo in the Netlify dashboard.

---

## Environment Variables

Set these in your deployment platform's dashboard:

### Required
- `TURSO_DATABASE_URL` — Your Turso database URL (if using database)
- `TURSO_AUTH_TOKEN` — Your Turso auth token (if using database)
- `ADMIN_SECRET` — Admin dashboard password (if using database)

### Build Variables
- `SITE_URL` — Your production URL (for sitemap generation)

---

## Pre-Deploy Checklist

Run through this before every deployment:

### Build
- [ ] `npm run build` completes without errors
- [ ] `npm run preview` works locally

### SEO
- [ ] `site` field in `astro.config.mjs` matches production domain
- [ ] Every page has unique title and description
- [ ] All images have alt text
- [ ] Sitemap generates correctly (check `dist/sitemap-index.xml`)
- [ ] robots.txt is correct (check `dist/robots.txt`)
- [ ] JSON-LD structured data on key pages
- [ ] Canonical URLs are correct
- [ ] OG images exist and are 1200x630px

### Performance
- [ ] Images use Astro `<Image>` component
- [ ] Interactive components use `client:visible` where appropriate
- [ ] No unnecessary `client:load` directives
- [ ] Fonts preloaded if using custom fonts

### Security
- [ ] `.env` is in `.gitignore`
- [ ] No secrets in client-side code (no `PUBLIC_` prefix on sensitive vars)
- [ ] Admin routes are protected
- [ ] HTTPS enforced

### Functionality
- [ ] Contact form submits correctly
- [ ] Content collections render correctly (blog, services, testimonials)
- [ ] Admin dashboard loads and displays data (if applicable)
- [ ] All internal links work
- [ ] Mobile navigation works
- [ ] 404 page exists and is styled
