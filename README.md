# astro-builder-skill

SEO-optimized Astro website builder skill for **Claude Code**, **Codex CLI**, and **Antigravity**.

## Install

```bash
# Claude Code
npx astro-builder-skill claude-code

# Codex CLI
npx astro-builder-skill codex

# Antigravity
npx astro-builder-skill antigravity

# All three
npx astro-builder-skill all
```

## What it does

Builds complete, deployment-ready websites in Astro that rank on Google. The skill walks the agent through:

1. **Niche research** — analyzes competitors, finds keyword gaps, proposes site structure
2. **Full project setup** — Astro + React + shadcn/ui + Tailwind + Keystatic CMS
3. **Every page built with real content** — SEO meta tags, structured data, Open Graph, internal linking
4. **Optional Turso database** — for contact forms, leads, waitlists with admin dashboard
5. **Deployment config** — Vercel/Netlify, sitemap, robots.txt, security headers

## Where it installs

| Target | Directory |
|---|---|
| `claude-code` | `~/.claude/skills/astro-builder/` |
| `codex` | `~/.codex/skills/astro-builder/` |
| `antigravity` | `~/.gemini/skills/astro-builder/` |

## Files included

- `SKILL.md` — Main skill instructions
- `references/astro-setup.md` — Project scaffolding and config
- `references/seo-checklist.md` — SEO component code and per-page checklist
- `references/shadcn-setup.md` — shadcn/ui installation and component patterns
- `references/turso-setup.md` — Database, API routes, and admin dashboard
- `references/keystatic-setup.md` — CMS configuration
- `references/deployment.md` — Vercel/Netlify deployment guides
