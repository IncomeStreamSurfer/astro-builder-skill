# Tailwind Component Patterns for Astro

## Table of Contents
1. [Design System Setup](#design-system-setup)
2. [Navigation](#navigation)
3. [Contact Form](#contact-form)
4. [FAQ Accordion](#faq-accordion)
5. [Testimonials](#testimonials)
6. [Admin Data Table](#admin-data-table)

---

## Design System Setup

### Global Styles (`src/styles/globals.css`)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 98%;
    --foreground: 240 10% 3.9%;
    --muted: 240 4.8% 95.9%;
    --muted-foreground: 240 3.8% 46.1%;
    --accent: 160 84% 39%;        /* Customize per project */
    --accent-foreground: 0 0% 98%;
    --border: 240 5.9% 90%;
    --radius: 0.75rem;
  }
}

@layer base {
  body {
    @apply bg-zinc-50 text-zinc-900 antialiased;
    font-family: 'Geist', 'Satoshi', system-ui, sans-serif;
  }

  h1, h2, h3, h4 {
    @apply tracking-tight;
  }
}
```

### Tailwind Config (`tailwind.config.mjs`)

```javascript
export default {
  content: ["./src/**/*.{astro,html,js,jsx,md,mdx,ts,tsx}"],
  theme: {
    extend: {
      fontFamily: {
        sans: ['Geist', 'Satoshi', 'system-ui', 'sans-serif'],
        mono: ['Geist Mono', 'JetBrains Mono', 'monospace'],
      },
      colors: {
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
      },
      maxWidth: {
        content: '65ch',
      },
    },
  },
  plugins: [require('@tailwindcss/typography')],
};
```

Install typography plugin:
```bash
npm install @tailwindcss/typography
```

---

## Navigation

### Header (`src/components/Header.astro`)

```astro
---
import MobileNav from './MobileNav';

const navLinks = [
  { href: '/', label: 'Home' },
  { href: '/services', label: 'Services' },
  { href: '/about', label: 'About' },
  { href: '/blog', label: 'Blog' },
  { href: '/contact', label: 'Contact' },
];
---

<header class="sticky top-0 z-40 w-full border-b border-zinc-200/60 bg-zinc-50/80 backdrop-blur-xl">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex h-16 items-center justify-between">
      <a href="/" class="text-xl font-semibold tracking-tight">
        Brand
      </a>

      <!-- Desktop nav -->
      <nav class="hidden md:flex items-center gap-8">
        {navLinks.map((link) => (
          <a
            href={link.href}
            class="text-sm text-zinc-600 hover:text-zinc-900 transition-colors"
          >
            {link.label}
          </a>
        ))}
        <a
          href="/contact"
          class="inline-flex items-center justify-center px-4 py-2 text-sm font-medium text-white bg-zinc-900 rounded-lg hover:bg-zinc-800 active:scale-[0.98] transition-all"
        >
          Get in Touch
        </a>
      </nav>

      <!-- Mobile nav (React island) -->
      <MobileNav client:load navLinks={navLinks} />
    </div>
  </div>
</header>
```

### Mobile Nav (`src/components/MobileNav.tsx`)

```tsx
import { useState } from 'react';

interface NavLink {
  href: string;
  label: string;
}

export default function MobileNav({ navLinks }: { navLinks: NavLink[] }) {
  const [open, setOpen] = useState(false);

  return (
    <div className="md:hidden">
      <button
        onClick={() => setOpen(!open)}
        className="p-2 text-zinc-600 hover:text-zinc-900"
        aria-label="Toggle menu"
      >
        <svg className="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={1.5}>
          {open ? (
            <path strokeLinecap="round" strokeLinejoin="round" d="M6 18L18 6M6 6l12 12" />
          ) : (
            <path strokeLinecap="round" strokeLinejoin="round" d="M3.75 9h16.5m-16.5 6.75h16.5" />
          )}
        </svg>
      </button>

      {open && (
        <div className="absolute top-16 left-0 right-0 bg-white border-b border-zinc-200 shadow-lg">
          <nav className="flex flex-col p-4 gap-1">
            {navLinks.map((link) => (
              <a
                key={link.href}
                href={link.href}
                onClick={() => setOpen(false)}
                className="px-4 py-3 text-base text-zinc-700 hover:bg-zinc-50 rounded-lg transition-colors"
              >
                {link.label}
              </a>
            ))}
          </nav>
        </div>
      )}
    </div>
  );
}
```

---

## Contact Form

```tsx
// src/components/ContactForm.tsx
import { useState } from 'react';

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
      <div className="rounded-2xl border border-emerald-200 bg-emerald-50 p-8">
        <p className="text-emerald-800 font-medium">Thank you! We'll be in touch soon.</p>
      </div>
    );
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-5">
      <div className="space-y-2">
        <label htmlFor="name" className="block text-sm font-medium text-zinc-700">Name</label>
        <input
          id="name" name="name" type="text" required
          className="w-full px-4 py-3 rounded-xl border border-zinc-200 bg-white text-zinc-900 placeholder:text-zinc-400 focus:outline-none focus:ring-2 focus:ring-zinc-900/10 focus:border-zinc-400 transition-all"
          placeholder="Cillian Murray"
        />
      </div>

      <div className="space-y-2">
        <label htmlFor="email" className="block text-sm font-medium text-zinc-700">Email</label>
        <input
          id="email" name="email" type="email" required
          className="w-full px-4 py-3 rounded-xl border border-zinc-200 bg-white text-zinc-900 placeholder:text-zinc-400 focus:outline-none focus:ring-2 focus:ring-zinc-900/10 focus:border-zinc-400 transition-all"
          placeholder="cillian@example.ie"
        />
      </div>

      <div className="space-y-2">
        <label htmlFor="phone" className="block text-sm font-medium text-zinc-700">Phone <span className="text-zinc-400">(optional)</span></label>
        <input
          id="phone" name="phone" type="tel"
          className="w-full px-4 py-3 rounded-xl border border-zinc-200 bg-white text-zinc-900 placeholder:text-zinc-400 focus:outline-none focus:ring-2 focus:ring-zinc-900/10 focus:border-zinc-400 transition-all"
          placeholder="+353 1 234 5678"
        />
      </div>

      <div className="space-y-2">
        <label htmlFor="message" className="block text-sm font-medium text-zinc-700">How can we help?</label>
        <textarea
          id="message" name="message" required rows={5}
          className="w-full px-4 py-3 rounded-xl border border-zinc-200 bg-white text-zinc-900 placeholder:text-zinc-400 focus:outline-none focus:ring-2 focus:ring-zinc-900/10 focus:border-zinc-400 transition-all resize-none"
          placeholder="Tell us about your project..."
        />
      </div>

      <button
        type="submit"
        disabled={status === 'sending'}
        className="w-full px-6 py-3 text-sm font-medium text-white bg-zinc-900 rounded-xl hover:bg-zinc-800 active:scale-[0.98] disabled:opacity-50 disabled:cursor-not-allowed transition-all"
      >
        {status === 'sending' ? 'Sending...' : 'Send Message'}
      </button>

      {status === 'error' && (
        <p className="text-sm text-red-600">Something went wrong. Please try again.</p>
      )}
    </form>
  );
}
```

---

## FAQ Accordion

```tsx
// src/components/FAQ.tsx
import { useState } from 'react';

interface FAQItem {
  question: string;
  answer: string;
}

export default function FAQ({ items }: { items: FAQItem[] }) {
  const [openIndex, setOpenIndex] = useState<number | null>(null);

  return (
    <div className="divide-y divide-zinc-200">
      {items.map((item, i) => (
        <div key={i}>
          <button
            onClick={() => setOpenIndex(openIndex === i ? null : i)}
            className="w-full flex items-center justify-between py-5 text-left text-zinc-900 hover:text-zinc-600 transition-colors"
          >
            <span className="text-base font-medium pr-4">{item.question}</span>
            <svg
              className={`w-5 h-5 shrink-0 transition-transform ${openIndex === i ? 'rotate-180' : ''}`}
              fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={1.5}
            >
              <path strokeLinecap="round" strokeLinejoin="round" d="M19.5 8.25l-7.5 7.5-7.5-7.5" />
            </svg>
          </button>
          {openIndex === i && (
            <div className="pb-5 text-zinc-600 text-base leading-relaxed max-w-content">
              {item.answer}
            </div>
          )}
        </div>
      ))}
    </div>
  );
}
```

Use with FAQ schema for SEO — see `seo-checklist.md` for the JSON-LD template.

---

## Testimonials

Static Astro component — no JS needed:

```astro
---
// src/components/TestimonialGrid.astro
import { getCollection } from 'astro:content';

const testimonials = await getCollection('testimonials');
---

<div class="grid grid-cols-1 md:grid-cols-2 gap-6">
  {testimonials.map((t) => (
    <blockquote class="p-6 rounded-2xl border border-zinc-200 bg-white">
      <div class="flex gap-1 mb-3">
        {Array.from({ length: t.data.rating }).map((_, i) => (
          <svg key={i} class="w-4 h-4 text-amber-400" fill="currentColor" viewBox="0 0 20 20">
            <path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z" />
          </svg>
        ))}
      </div>
      <p class="text-zinc-700 leading-relaxed mb-4">"{t.data.quote}"</p>
      <footer class="text-sm">
        <span class="font-medium text-zinc-900">{t.data.name}</span>
        <span class="text-zinc-500"> — {t.data.role}</span>
      </footer>
    </blockquote>
  ))}
</div>
```

---

## Admin Data Table

```tsx
// src/components/admin/AdminDashboard.tsx
import { useEffect, useState } from 'react';

interface Contact {
  id: number;
  name: string;
  email: string;
  phone: string | null;
  message: string;
  status: string;
  created_at: string;
}

export default function AdminDashboard() {
  const [contacts, setContacts] = useState<Contact[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/admin/submissions')
      .then(res => res.json())
      .then(data => { setContacts(Array.isArray(data) ? data : []); setLoading(false); })
      .catch(() => setLoading(false));
  }, []);

  if (loading) {
    return (
      <div className="space-y-3">
        {[1,2,3].map(i => (
          <div key={i} className="h-16 rounded-xl bg-zinc-100 animate-pulse" />
        ))}
      </div>
    );
  }

  const newCount = contacts.filter(c => c.status === 'new').length;

  return (
    <div className="space-y-8">
      {/* Stats */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div className="p-6 rounded-2xl border border-zinc-200 bg-white">
          <p className="text-sm text-zinc-500 mb-1">Total</p>
          <p className="text-3xl font-semibold tracking-tight">{contacts.length}</p>
        </div>
        <div className="p-6 rounded-2xl border border-zinc-200 bg-white">
          <p className="text-sm text-zinc-500 mb-1">New</p>
          <p className="text-3xl font-semibold tracking-tight text-emerald-600">{newCount}</p>
        </div>
        <div className="p-6 rounded-2xl border border-zinc-200 bg-white">
          <p className="text-sm text-zinc-500 mb-1">Responded</p>
          <p className="text-3xl font-semibold tracking-tight">{contacts.length - newCount}</p>
        </div>
      </div>

      {/* Table */}
      <div className="rounded-2xl border border-zinc-200 bg-white overflow-hidden">
        <table className="w-full text-sm">
          <thead>
            <tr className="border-b border-zinc-100">
              <th className="text-left px-6 py-3 font-medium text-zinc-500">Name</th>
              <th className="text-left px-6 py-3 font-medium text-zinc-500">Email</th>
              <th className="text-left px-6 py-3 font-medium text-zinc-500 hidden lg:table-cell">Message</th>
              <th className="text-left px-6 py-3 font-medium text-zinc-500">Date</th>
              <th className="text-left px-6 py-3 font-medium text-zinc-500">Status</th>
            </tr>
          </thead>
          <tbody className="divide-y divide-zinc-50">
            {contacts.map((c) => (
              <tr key={c.id} className="hover:bg-zinc-50/50 transition-colors">
                <td className="px-6 py-4 font-medium text-zinc-900">{c.name}</td>
                <td className="px-6 py-4 text-zinc-600">{c.email}</td>
                <td className="px-6 py-4 text-zinc-600 max-w-xs truncate hidden lg:table-cell">{c.message}</td>
                <td className="px-6 py-4 text-zinc-500 font-mono text-xs">{new Date(c.created_at).toLocaleDateString()}</td>
                <td className="px-6 py-4">
                  <span className={`inline-flex px-2.5 py-0.5 rounded-full text-xs font-medium ${
                    c.status === 'new'
                      ? 'bg-emerald-50 text-emerald-700'
                      : 'bg-zinc-100 text-zinc-600'
                  }`}>
                    {c.status}
                  </span>
                </td>
              </tr>
            ))}
            {contacts.length === 0 && (
              <tr>
                <td colSpan={5} className="px-6 py-12 text-center text-zinc-400">
                  No submissions yet
                </td>
              </tr>
            )}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```
