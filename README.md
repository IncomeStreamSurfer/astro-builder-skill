# astro-builder-skill

SEO-optimized Astro website builder skill for **Claude Code**, **Codex CLI**, and **Antigravity**.

Includes the [taste-skill](https://github.com/Leonxlnx/taste-skill) design system for premium, non-generic frontend output.

## Install

```bash
# Claude Code
npx astro-builder-skill claude-code

# Codex CLI
npx astro-builder-skill codex

# Antigravity (workspace-local, creates .agents/skills/ in current dir)
npx astro-builder-skill antigravity

# Antigravity (global, all workspaces)
npx astro-builder-skill antigravity-global

# All targets
npx astro-builder-skill all
```

Or install from GitHub directly:

```bash
npx github:IncomeStreamSurfer/astro-builder-skill claude-code
```

## What it does

Builds complete, deployment-ready websites in Astro that rank on Google. The skill walks the agent through:

1. **Niche research** — analyzes competitors, finds keyword gaps, proposes site structure
2. **Full project setup** — Astro + React + Tailwind CSS + premium fonts (Geist, Satoshi)
3. **Every page built with real content** — SEO meta tags, structured data, Open Graph, internal linking
4. **Premium design** — taste-skill rules prevent generic AI output (no purple gradients, no Inter, no card grids)
5. **Content collections** — Type-safe Markdown content for blog posts, services, testimonials
6. **Optional Turso database** — for contact forms, leads, waitlists with admin dashboard
7. **Deployment config** — Vercel/Netlify, sitemap, robots.txt, security headers

## Where it installs

| Target | Directory |
|---|---|
| `claude-code` | `~/.claude/skills/astro-builder/` |
| `codex` | `~/.codex/skills/astro-builder/` |
| `antigravity` | `.agents/skills/astro-builder/` (workspace-local) |
| `antigravity-global` | `~/.gemini/antigravity/skills/astro-builder/` |

## Files included

- `SKILL.md` — Main skill instructions
- `references/astro-setup.md` — Project scaffolding and config
- `references/seo-checklist.md` — SEO component code and per-page checklist
- `references/tailwind-components.md` — Tailwind component patterns (nav, forms, FAQ, admin)
- `references/content-collections.md` — Astro content collection schemas and queries
- `references/taste-skill.md` — Premium frontend design rules (layout, typography, color, motion)
- `references/turso-setup.md` — Database, API routes, and admin dashboard
- `references/deployment.md` — Vercel/Netlify deployment guides
