# Taste Skill — Premium Frontend Design Reference

Source: https://github.com/Leonxlnx/taste-skill

This is the taste-skill by Leonxlnx, integrated as a design reference for building premium, non-generic frontends. When building pages, follow these rules alongside the SEO and Astro-specific guidelines.

---

## 1. ACTIVE BASELINE CONFIGURATION

```
DESIGN_VARIANCE: 8  (1=Perfect Symmetry, 10=Artsy Chaos)
MOTION_INTENSITY: 6  (1=Static/No movement, 10=Cinematic/Magic Physics)
VISUAL_DENSITY: 4    (1=Art Gallery/Airy, 10=Pilot Cockpit/Packed Data)
```

Adapt these values dynamically based on what the user explicitly requests. Use these as global variables driving design decisions in all sections below.

## 2. DEFAULT ARCHITECTURE & CONVENTIONS

**DEPENDENCY VERIFICATION [MANDATORY]:** Before importing ANY 3rd party library, check `package.json`. If missing, output the install command first.

**Styling Policy:** Use Tailwind CSS (v3/v4) for 90% of styling.
- TAILWIND VERSION LOCK: Check `package.json` first. Do not use v4 syntax in v3 projects.
- ANTI-EMOJI POLICY: NEVER use emojis in code, markup, text content, or alt text. Use high-quality icons (Phosphor, Radix) or clean SVG primitives.

**Responsiveness & Spacing:**
- Standardize breakpoints (sm, md, lg, xl)
- Contain page layouts using `max-w-[1400px] mx-auto` or `max-w-7xl`
- **Viewport Stability:** NEVER use `h-screen` for full-height sections. ALWAYS use `min-h-[100dvh]`
- **Grid over Flex-Math:** NEVER use complex flexbox percentage math. ALWAYS use CSS Grid (`grid grid-cols-1 md:grid-cols-3 gap-6`)

**Icons:** Use `@phosphor-icons/react` or `@radix-ui/react-icons`. Standardize `strokeWidth` globally.

## 3. DESIGN ENGINEERING DIRECTIVES (Bias Correction)

### Rule 1: Deterministic Typography
- Display/Headlines: `text-4xl md:text-6xl tracking-tighter leading-none`
- ANTI-SLOP: Discourage Inter for premium vibes. Use **Geist, Outfit, Cabinet Grotesk, or Satoshi**
- Serif fonts are BANNED for Dashboard/Software UIs
- Body: `text-base text-gray-600 leading-relaxed max-w-[65ch]`

### Rule 2: Color Calibration
- Max 1 Accent Color. Saturation < 80%
- THE LILA BAN: "AI Purple/Blue" aesthetic is BANNED. No purple button glows, no neon gradients
- Use neutral bases (Zinc/Slate) with high-contrast singular accents (Emerald, Electric Blue, Deep Rose)
- Stick to ONE palette for the entire output

### Rule 3: Layout Diversification
- ANTI-CENTER BIAS: Centered Hero sections are BANNED when LAYOUT_VARIANCE > 4
- Use "Split Screen" (50/50), "Left Aligned content/Right Aligned asset", or "Asymmetric White-space"

### Rule 4: Materiality, Shadows, and Anti-Card Overuse
- For VISUAL_DENSITY > 7, generic card containers are BANNED. Use `border-t`, `divide-y`, or negative space
- Use cards ONLY when elevation communicates hierarchy
- When a shadow is used, tint it to the background hue

### Rule 5: Interactive UI States
Must implement full interaction cycles:
- **Loading:** Skeletal loaders matching layout sizes (not generic spinners)
- **Empty States:** Beautifully composed empty states
- **Error States:** Clear inline error reporting
- **Tactile Feedback:** On `:active`, use `-translate-y-[1px]` or `scale-[0.98]`

### Rule 6: Data & Form Patterns
- Label MUST sit above input. Helper text optional. Error text below input. Use `gap-2` for input blocks.

## 4. CREATIVE PROACTIVITY

- **Liquid Glass Refraction:** Go beyond `backdrop-blur`. Add `border-white/10` and `shadow-[inset_0_1px_0_rgba(255,255,255,0.1)]`
- **Magnetic Micro-physics (MOTION > 5):** Buttons pull toward cursor. Use `useMotionValue`/`useTransform`, NOT useState
- **Perpetual Micro-Interactions (MOTION > 5):** Embed continuous animations (Pulse, Typewriter, Float, Shimmer)
- **Spring Physics:** `type: "spring", stiffness: 100, damping: 20` for all interactive elements
- **Staggered Orchestration:** Use `staggerChildren` or `animation-delay: calc(var(--index) * 100ms)`

## 5. PERFORMANCE GUARDRAILS

- Apply grain/noise filters to `fixed inset-0 z-50 pointer-events-none` pseudo-elements only
- Never animate `top`, `left`, `width`, or `height`. Animate via `transform` and `opacity` only
- Z-Index: Use strictly for systemic layers (Navbars, Modals, Overlays)

## 6. TECHNICAL REFERENCE (Dial Definitions)

**DESIGN_VARIANCE:**
- 1-3: Symmetrical grids, equal paddings
- 4-7: Overlapping margins, varied aspect ratios, left-aligned headers
- 8-10: Masonry layouts, CSS Grid fractional units, massive empty zones
- MOBILE OVERRIDE: Levels 4-10 MUST fall back to single-column on < 768px

**MOTION_INTENSITY:**
- 1-3: No auto animations. CSS `:hover`/`:active` only
- 4-7: `transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1)`, animation-delay cascades
- 8-10: Complex scroll-triggered reveals, Framer Motion hooks. NEVER use `addEventListener('scroll')`

**VISUAL_DENSITY:**
- 1-3: Lots of white space, huge section gaps — expensive and clean
- 4-7: Normal spacing for standard web apps
- 8-10: Tiny paddings, 1px separators, packed data. Use `font-mono` for all numbers

## 7. AI TELLS (Forbidden Patterns)

**Visual & CSS:**
- NO Neon/Outer Glows — use inner borders or tinted shadows
- NO Pure Black (#000000) — use Off-Black, Zinc-950, Charcoal
- NO Oversaturated Accents
- NO Excessive Gradient Text on large headers
- NO Custom Mouse Cursors

**Typography:**
- NO Inter Font — use Geist, Outfit, Cabinet Grotesk, Satoshi
- NO Oversized H1s — control hierarchy with weight and color
- Serif ONLY for creative/editorial, NEVER on Dashboards

**Layout & Spacing:**
- Align perfectly. Avoid floating elements with awkward gaps
- NO 3-Column Card Layouts — use 2-column Zig-Zag, asymmetric grid, or horizontal scroll

**Content & Data:**
- NO Generic Names ("John Doe") — use creative realistic names
- NO Generic Avatars — use photo placeholders or specific styling
- NO Fake Numbers (99.99%, 50%) — use organic data (47.2%, +1 (312) 847-1928)
- NO Startup Slop Names ("Acme", "Nexus") — invent premium contextual names
- NO Filler Words ("Elevate", "Seamless", "Unleash") — use concrete verbs
- NO Broken Unsplash Links — use `https://picsum.photos/seed/{random}/800/600`

## 8. THE CREATIVE ARSENAL

Pull from this library of advanced concepts:

**Navigation:** Mac OS Dock Magnification, Magnetic Buttons, Gooey Menus, Dynamic Islands, Radial Menus, Mega Menu Reveals

**Layout:** Bento Grids, Masonry, Chroma Grids, Split Screen Scroll, Curtain Reveals

**Cards:** Parallax Tilt, Spotlight Border, Glassmorphism Panels, Holographic Foil, Tinder Swipe Stacks, Morphing Modals

**Scroll-Animations:** Sticky Scroll Stacks, Horizontal Scroll Hijack, Zoom Parallax, Scroll Progress Paths, Liquid Swipe Transitions

**Typography:** Kinetic Marquees, Text Mask Reveals, Text Scramble Effects, Circular Text Paths, Gradient Stroke Animation

**Micro-Interactions:** Particle Explosion Buttons, Skeleton Shimmer, Directional Hover Buttons, Ripple Click Effects, Mesh Gradient Backgrounds

## 9. ASTRO-SPECIFIC ADAPTATION

Since we're in Astro (not pure React/Next.js), adapt the taste-skill rules:

- **Static by default:** Most page content is `.astro` components with zero JS. The taste-skill's typography, color, layout, and spacing rules apply directly via Tailwind classes.
- **Islands for interactivity:** When motion/interaction is needed (magnetic buttons, scroll animations, form states), create isolated React `.tsx` components with `client:visible` or `client:load`.
- **No RSC concerns:** Astro doesn't use React Server Components. All `.astro` files are server-rendered. React islands are always client-side.
- **Font loading:** Load premium fonts (Geist, Satoshi, etc.) via `@fontsource` packages or Google Fonts in the `<head>`.

## 10. FINAL PRE-FLIGHT CHECK

- Is mobile layout collapse (`w-full`, `px-4`, `max-w-7xl mx-auto`) guaranteed?
- Do full-height sections use `min-h-[100dvh]` not `h-screen`?
- Are empty, loading, and error states provided for interactive components?
- Are cards omitted in favor of spacing where possible?
- Is the design non-generic? No purple gradients, no Inter, no centered 3-card layouts?
