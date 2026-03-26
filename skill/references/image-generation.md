# Image Generation with Gemini

Generate custom stock images, hero images, and placeholder visuals for the website using Google's Gemini image generation API.

## Prerequisites

The user must add their Gemini API key to `.env`:

```
GEMINI_API_KEY=your_key_here
```

During Phase 1 (Discovery), ask the user:
> "Do you want to generate custom images for the site? If so, add your `GEMINI_API_KEY` to the `.env` file."

If no key is provided, skip image generation and use placeholder images with proper alt text instead.

## Setup

### Install Dependencies

No extra packages needed — uses `curl` via a shell script or `fetch` in an Astro API route.

### Add to `.env`

```
GEMINI_API_KEY=your_gemini_api_key
```

### Add to `astro.config.mjs`

Ensure the env variable is accessible:
```typescript
// .env is auto-loaded by Astro via Vite
// Access in server code: import.meta.env.GEMINI_API_KEY
```

## Image Generation Script

Create `scripts/generate-image.sh` in the project root:

```bash
#!/bin/bash
set -e

# Usage: ./scripts/generate-image.sh "prompt" "output-filename" "aspect-ratio"
# Example: ./scripts/generate-image.sh "Modern plumbing workshop, clean professional photography" "hero-plumber" "16:9"

PROMPT="$1"
OUTPUT_NAME="${2:-generated-image}"
ASPECT_RATIO="${3:-16:9}"

if [ -z "$GEMINI_API_KEY" ]; then
  source .env 2>/dev/null || true
  GEMINI_API_KEY="${GEMINI_API_KEY}"
fi

if [ -z "$GEMINI_API_KEY" ]; then
  echo "Error: GEMINI_API_KEY not set. Add it to .env"
  exit 1
fi

MODEL_ID="gemini-2.0-flash-preview-image-generation"
API_URL="https://generativelanguage.googleapis.com/v1beta/models/${MODEL_ID}:generateContent?key=${GEMINI_API_KEY}"

RESPONSE=$(curl -s -X POST \
  -H "Content-Type: application/json" \
  "$API_URL" \
  -d "{
    \"contents\": [{
      \"role\": \"user\",
      \"parts\": [{
        \"text\": \"${PROMPT}. High quality stock photography style, professional, clean, modern. No text overlays, no watermarks.\"
      }]
    }],
    \"generationConfig\": {
      \"responseModalities\": [\"IMAGE\", \"TEXT\"],
      \"imageConfig\": {
        \"aspectRatio\": \"${ASPECT_RATIO}\",
        \"imageSize\": \"1K\"
      }
    }
  }")

# Extract base64 image data from response
IMAGE_DATA=$(echo "$RESPONSE" | python3 -c "
import sys, json
data = json.load(sys.stdin)
parts = data.get('candidates', [{}])[0].get('content', {}).get('parts', [])
for part in parts:
    if 'inlineData' in part:
        print(part['inlineData']['data'])
        break
" 2>/dev/null)

if [ -z "$IMAGE_DATA" ]; then
  echo "Error: No image data in response"
  echo "$RESPONSE" | python3 -c "import sys,json; print(json.dumps(json.load(sys.stdin),indent=2))" 2>/dev/null || echo "$RESPONSE"
  exit 1
fi

# Save image
mkdir -p public/images/generated
echo "$IMAGE_DATA" | base64 -d > "public/images/generated/${OUTPUT_NAME}.png"
echo "✓ Saved: public/images/generated/${OUTPUT_NAME}.png"
```

Make it executable:
```bash
chmod +x scripts/generate-image.sh
```

## TypeScript API Route (Alternative)

For generating images at build time or on-demand, create `src/lib/generate-image.ts`:

```typescript
interface GenerateImageOptions {
  prompt: string;
  aspectRatio?: "1:1" | "16:9" | "9:16" | "4:3" | "3:4";
  outputPath?: string;
}

export async function generateImage({
  prompt,
  aspectRatio = "16:9",
}: GenerateImageOptions): Promise<Buffer | null> {
  const apiKey = import.meta.env.GEMINI_API_KEY;

  if (!apiKey) {
    console.warn("GEMINI_API_KEY not set — skipping image generation");
    return null;
  }

  const MODEL_ID = "gemini-2.0-flash-preview-image-generation";
  const url = `https://generativelanguage.googleapis.com/v1beta/models/${MODEL_ID}:generateContent?key=${apiKey}`;

  const response = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      contents: [
        {
          role: "user",
          parts: [
            {
              text: `${prompt}. High quality stock photography style, professional, clean, modern. No text overlays, no watermarks.`,
            },
          ],
        },
      ],
      generationConfig: {
        responseModalities: ["IMAGE", "TEXT"],
        imageConfig: {
          aspectRatio,
          imageSize: "1K",
        },
      },
    }),
  });

  const data = await response.json();
  const parts = data?.candidates?.[0]?.content?.parts || [];

  for (const part of parts) {
    if (part.inlineData) {
      return Buffer.from(part.inlineData.data, "base64");
    }
  }

  console.error("No image in response:", JSON.stringify(data, null, 2));
  return null;
}
```

## Image Generation Strategy

### When to Generate

Generate images during **Phase 3** (Build Every Page) after the site structure is approved:

1. **Hero images** — one per major page (home, about, services)
2. **Service images** — one per service page
3. **Blog post featured images** — one per blog post
4. **Team/about photos** — use abstract or workspace imagery (never AI faces)

### Prompt Guidelines (aligned with taste-skill)

Write prompts that match the site's niche and the taste-skill design rules:

**DO:**
- Describe the scene, lighting, and mood specifically
- Reference the business niche ("dental clinic reception area", "artisan coffee beans close-up")
- Request specific photography styles ("editorial", "product photography", "architectural")
- Match the site's color palette ("warm neutral tones", "cool minimal palette")

**DON'T:**
- Generate faces or portraits (uncanny valley risk)
- Use generic stock-photo language ("happy diverse team in office")
- Request text or logos in images (AI text is unreliable)
- Generate images that look obviously AI (oversaturated, too perfect)

### Example Prompts by Niche

```bash
# Plumber website
./scripts/generate-image.sh "Professional plumbing tools laid out on a clean workbench, warm workshop lighting, shallow depth of field" "hero-plumber" "16:9"
./scripts/generate-image.sh "Modern bathroom renovation, white tiles, brass fixtures, editorial interior photography" "service-bathroom" "16:9"
./scripts/generate-image.sh "Close-up of copper pipe fittings, industrial aesthetic, moody lighting" "service-pipes" "4:3"

# Bakery website
./scripts/generate-image.sh "Artisan sourdough bread on a wooden cutting board, natural window light, food photography" "hero-bakery" "16:9"
./scripts/generate-image.sh "Pastry chef hands shaping croissant dough, flour-dusted marble counter, warm tones" "about-baking" "16:9"

# SaaS landing page
./scripts/generate-image.sh "Minimal desk setup with laptop showing abstract dashboard UI, clean modern office, soft lighting" "hero-saas" "16:9"
./scripts/generate-image.sh "Abstract geometric shapes, gradient mesh, tech-inspired, cool blue-gray tones" "feature-abstract" "1:1"
```

### Aspect Ratios

| Use Case | Ratio | Notes |
|---|---|---|
| Hero/banner | `16:9` | Full-width sections |
| Blog featured | `16:9` | Open Graph / social sharing |
| Service card | `4:3` | Inline content images |
| Square thumbnail | `1:1` | Grid layouts, testimonials |
| Mobile hero | `9:16` | Portrait orientation |

## Using Generated Images in Astro

Reference generated images in components:

```astro
---
// In any .astro page
const heroImage = "/images/generated/hero-plumber.png";
---

<img
  src={heroImage}
  alt="Professional plumbing tools arranged on a clean workbench"
  width="1024"
  height="576"
  loading="eager"
  class="w-full h-auto object-cover rounded-lg"
/>
```

For optimized images, use Astro's built-in `<Image>` component:

```astro
---
import { Image } from 'astro:assets';
import heroImage from '../../public/images/generated/hero-plumber.png';
---

<Image
  src={heroImage}
  alt="Professional plumbing tools arranged on a clean workbench"
  width={1024}
  height={576}
  format="webp"
  quality={80}
  loading="eager"
/>
```

## Fallback: No API Key

If the user doesn't provide a Gemini API key, use placeholder images:

```astro
---
// Placeholder with proper dimensions and alt text
const hasGeneratedImages = import.meta.env.GEMINI_API_KEY;
---

{hasGeneratedImages ? (
  <img src="/images/generated/hero.png" alt="Description" />
) : (
  <div class="bg-zinc-100 dark:bg-zinc-800 rounded-lg flex items-center justify-center aspect-video">
    <span class="text-zinc-400 text-sm">Image placeholder — add GEMINI_API_KEY to .env to generate</span>
  </div>
)}
```
