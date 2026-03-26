# shadcn/ui + Astro Setup Reference

## Table of Contents
1. [Installation](#installation)
2. [Configuration](#configuration)
3. [Adding Components](#adding-components)
4. [Usage Patterns in Astro](#usage-patterns)
5. [Common Component Recipes](#common-recipes)

---

## Installation

### Prerequisites
React and Tailwind must already be installed:
```bash
npx astro add react tailwind
```

### Step 1: Configure tsconfig.json
Path aliases MUST be set before running shadcn init:

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

### Step 2: Initialize shadcn
```bash
npx shadcn@latest init
```

When prompted, choose:
- Style: Default (or New York)
- Base color: Slate (or your preference)
- CSS variables: Yes

This creates `components.json` and updates your CSS.

### Step 3: Add Components
```bash
npx shadcn@latest add button card input textarea form
npx shadcn@latest add sheet dialog accordion tabs table badge
npx shadcn@latest add navigation-menu dropdown-menu
```

Components are installed to `src/components/ui/`.

---

## Configuration

### Tailwind Config
Ensure your `tailwind.config.mjs` covers Astro files:

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: ["class"],
  content: ["./src/**/*.{astro,html,js,jsx,md,mdx,ts,tsx}"],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: { "2xl": "1400px" },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
};
```

### utils.ts
Create `src/lib/utils.ts`:
```typescript
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Install dependencies:
```bash
npm install clsx tailwind-merge tailwindcss-animate
```

---

## Usage Patterns

### The Golden Rule: Islands Architecture

Astro renders everything as static HTML by default. shadcn components that need interactivity (clicks, state, hover effects) require hydration directives.

**Static OK (no directive needed):**
- Using shadcn styles/classes in `.astro` files
- Purely visual components with no JS behavior

**Needs hydration directive:**
- Anything with onClick, onChange, useState
- Dialogs, Sheets, Dropdowns, Accordions
- Forms with validation
- Any component using React hooks

### Pattern: Wrapper Components

The most important pattern. Since React context doesn't cross island boundaries, group related interactive components into a single React wrapper:

```tsx
// src/components/ContactForm.tsx
import { useState } from 'react';
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

export default function ContactForm() {
  const [status, setStatus] = useState<'idle' | 'sending' | 'sent' | 'error'>('idle');

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setStatus('sending');
    const formData = new FormData(e.currentTarget);

    try {
      const res = await fetch('/api/contact', { method: 'POST', body: formData });
      setStatus(res.ok ? 'sent' : 'error');
    } catch {
      setStatus('error');
    }
  };

  if (status === 'sent') {
    return (
      <Card>
        <CardContent className="pt-6">
          <p className="text-green-600 font-medium">Thank you! We'll be in touch soon.</p>
        </CardContent>
      </Card>
    );
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>Get in Touch</CardTitle>
      </CardHeader>
      <CardContent>
        <form onSubmit={handleSubmit} className="space-y-4">
          <Input name="name" placeholder="Your Name" required />
          <Input name="email" type="email" placeholder="Your Email" required />
          <Input name="phone" type="tel" placeholder="Your Phone (optional)" />
          <Textarea name="message" placeholder="How can we help?" required rows={5} />
          <Button type="submit" disabled={status === 'sending'} className="w-full">
            {status === 'sending' ? 'Sending...' : 'Send Message'}
          </Button>
          {status === 'error' && (
            <p className="text-red-600 text-sm">Something went wrong. Please try again.</p>
          )}
        </form>
      </CardContent>
    </Card>
  );
}
```

Use in Astro:
```astro
---
import ContactForm from '../components/ContactForm';
---

<ContactForm client:visible />
```

---

## Common Recipes

### Mobile Navigation with Sheet

```tsx
// src/components/MobileNav.tsx
import { useState } from 'react';
import { Menu } from 'lucide-react';
import { Button } from "@/components/ui/button";
import { Sheet, SheetContent, SheetTrigger } from "@/components/ui/sheet";

const navLinks = [
  { href: '/', label: 'Home' },
  { href: '/services', label: 'Services' },
  { href: '/about', label: 'About' },
  { href: '/blog', label: 'Blog' },
  { href: '/contact', label: 'Contact' },
];

export default function MobileNav() {
  const [open, setOpen] = useState(false);

  return (
    <Sheet open={open} onOpenChange={setOpen}>
      <SheetTrigger asChild>
        <Button variant="ghost" size="icon" className="md:hidden">
          <Menu className="h-6 w-6" />
          <span className="sr-only">Toggle menu</span>
        </Button>
      </SheetTrigger>
      <SheetContent side="right">
        <nav className="flex flex-col gap-4 mt-8">
          {navLinks.map((link) => (
            <a
              key={link.href}
              href={link.href}
              onClick={() => setOpen(false)}
              className="text-lg font-medium hover:text-primary transition-colors"
            >
              {link.label}
            </a>
          ))}
        </nav>
      </SheetContent>
    </Sheet>
  );
}
```

### FAQ Accordion (great for SEO)

```tsx
// src/components/FAQ.tsx
import {
  Accordion,
  AccordionContent,
  AccordionItem,
  AccordionTrigger,
} from "@/components/ui/accordion";

interface FAQItem {
  question: string;
  answer: string;
}

interface Props {
  items: FAQItem[];
}

export default function FAQ({ items }: Props) {
  return (
    <Accordion type="single" collapsible className="w-full">
      {items.map((item, index) => (
        <AccordionItem key={index} value={`item-${index}`}>
          <AccordionTrigger className="text-left">
            {item.question}
          </AccordionTrigger>
          <AccordionContent>
            {item.answer}
          </AccordionContent>
        </AccordionItem>
      ))}
    </Accordion>
  );
}
```

Use with FAQ schema:
```astro
---
import FAQ from '../components/FAQ';
import BaseLayout from '../layouts/BaseLayout.astro';

const faqItems = [
  { question: "How much does it cost?", answer: "Our prices start from..." },
  { question: "How long does it take?", answer: "Typically 2-3 days..." },
];

const faqSchema = {
  "@context": "https://schema.org",
  "@type": "FAQPage",
  mainEntity: faqItems.map(item => ({
    "@type": "Question",
    name: item.question,
    acceptedAnswer: { "@type": "Answer", text: item.answer }
  }))
};
---

<BaseLayout title="FAQ" description="..." structuredData={faqSchema}>
  <section class="max-w-3xl mx-auto px-4 py-12">
    <h1>Frequently Asked Questions</h1>
    <FAQ items={faqItems} client:visible />
  </section>
</BaseLayout>
```

### Admin Data Table

```tsx
// src/components/admin/DataTable.tsx
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import { Badge } from "@/components/ui/badge";
import { useEffect, useState } from "react";

interface Submission {
  id: number;
  name: string;
  email: string;
  message: string;
  created_at: string;
  status: string;
}

export default function DataTable() {
  const [data, setData] = useState<Submission[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/admin/submissions')
      .then(res => res.json())
      .then(setData)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <p>Loading...</p>;

  return (
    <Table>
      <TableHeader>
        <TableRow>
          <TableHead>Name</TableHead>
          <TableHead>Email</TableHead>
          <TableHead>Message</TableHead>
          <TableHead>Date</TableHead>
          <TableHead>Status</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        {data.map((row) => (
          <TableRow key={row.id}>
            <TableCell className="font-medium">{row.name}</TableCell>
            <TableCell>{row.email}</TableCell>
            <TableCell className="max-w-xs truncate">{row.message}</TableCell>
            <TableCell>{new Date(row.created_at).toLocaleDateString()}</TableCell>
            <TableCell>
              <Badge variant={row.status === 'new' ? 'default' : 'secondary'}>
                {row.status}
              </Badge>
            </TableCell>
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
}
```
