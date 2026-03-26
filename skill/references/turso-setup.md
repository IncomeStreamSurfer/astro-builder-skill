# Turso Database Setup Reference

## Table of Contents
1. [Installation](#installation)
2. [Database Client](#database-client)
3. [Schema with Drizzle ORM](#schema-with-drizzle)
4. [API Routes for Forms](#api-routes)
5. [Admin Dashboard](#admin-dashboard)
6. [Environment Variables](#environment-variables)

---

## Installation

```bash
npm install @libsql/client drizzle-orm
npm install -D drizzle-kit
```

### Turso CLI Setup (user needs to do this)
```bash
# Install Turso CLI
curl -sSfL https://get.tur.so/install.sh | bash

# Login
turso auth login

# Create a database
turso db create [database-name]

# Get the URL
turso db show [database-name] --url

# Create an auth token
turso db tokens create [database-name]
```

---

## Environment Variables

Create `.env`:
```
TURSO_DATABASE_URL=libsql://[database-name]-[username].turso.io
TURSO_AUTH_TOKEN=[your-auth-token]
```

IMPORTANT: Do NOT prefix with `PUBLIC_` — these must stay server-side only.

Add to `.gitignore`:
```
.env
.env.local
```

---

## Database Client

Create `src/lib/turso.ts`:

```typescript
import { createClient } from "@libsql/client/web";

export const turso = createClient({
  url: import.meta.env.TURSO_DATABASE_URL,
  authToken: import.meta.env.TURSO_AUTH_TOKEN,
});
```

### With Drizzle ORM (recommended)

Create `src/lib/db.ts`:

```typescript
import { drizzle } from 'drizzle-orm/libsql';
import { createClient } from '@libsql/client/web';
import * as schema from './schema';

const client = createClient({
  url: import.meta.env.TURSO_DATABASE_URL,
  authToken: import.meta.env.TURSO_AUTH_TOKEN,
});

export const db = drizzle(client, { schema });
```

---

## Schema with Drizzle

Create `src/lib/schema.ts`:

```typescript
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core';
import { sql } from 'drizzle-orm';

// Contact form submissions
export const contacts = sqliteTable('contacts', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  name: text('name').notNull(),
  email: text('email').notNull(),
  phone: text('phone'),
  message: text('message').notNull(),
  status: text('status').default('new').notNull(),
  createdAt: text('created_at').default(sql`CURRENT_TIMESTAMP`).notNull(),
});

// Newsletter subscribers
export const subscribers = sqliteTable('subscribers', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  email: text('email').notNull().unique(),
  subscribedAt: text('subscribed_at').default(sql`CURRENT_TIMESTAMP`).notNull(),
});

// Generic form submissions (for custom forms)
export const formSubmissions = sqliteTable('form_submissions', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  formName: text('form_name').notNull(),
  data: text('data').notNull(), // JSON stringified form data
  createdAt: text('created_at').default(sql`CURRENT_TIMESTAMP`).notNull(),
});
```

### Drizzle Config

Create `drizzle.config.ts` at project root:

```typescript
import type { Config } from 'drizzle-kit';

export default {
  schema: './src/lib/schema.ts',
  out: './drizzle',
  dialect: 'turso',
  dbCredentials: {
    url: process.env.TURSO_DATABASE_URL!,
    authToken: process.env.TURSO_AUTH_TOKEN!,
  },
} satisfies Config;
```

### Push Schema to Database
```bash
npx drizzle-kit push
```

---

## API Routes

### Contact Form Submission

Create `src/pages/api/contact.ts`:

```typescript
import type { APIRoute } from 'astro';
import { db } from '@/lib/db';
import { contacts } from '@/lib/schema';

export const prerender = false;

export const POST: APIRoute = async ({ request }) => {
  try {
    const formData = await request.formData();
    const name = formData.get('name') as string;
    const email = formData.get('email') as string;
    const phone = formData.get('phone') as string | null;
    const message = formData.get('message') as string;

    // Validation
    if (!name || !email || !message) {
      return new Response(
        JSON.stringify({ error: 'Name, email, and message are required' }),
        { status: 400, headers: { 'Content-Type': 'application/json' } }
      );
    }

    // Email format validation
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      return new Response(
        JSON.stringify({ error: 'Invalid email address' }),
        { status: 400, headers: { 'Content-Type': 'application/json' } }
      );
    }

    // Insert into database
    await db.insert(contacts).values({
      name,
      email,
      phone: phone || null,
      message,
    });

    return new Response(
      JSON.stringify({ success: true, message: 'Thank you for your message!' }),
      { status: 200, headers: { 'Content-Type': 'application/json' } }
    );
  } catch (error) {
    console.error('Contact form error:', error);
    return new Response(
      JSON.stringify({ error: 'An error occurred. Please try again.' }),
      { status: 500, headers: { 'Content-Type': 'application/json' } }
    );
  }
};
```

### Newsletter Subscription

Create `src/pages/api/subscribe.ts`:

```typescript
import type { APIRoute } from 'astro';
import { db } from '@/lib/db';
import { subscribers } from '@/lib/schema';

export const prerender = false;

export const POST: APIRoute = async ({ request }) => {
  try {
    const formData = await request.formData();
    const email = formData.get('email') as string;

    if (!email) {
      return new Response(
        JSON.stringify({ error: 'Email is required' }),
        { status: 400, headers: { 'Content-Type': 'application/json' } }
      );
    }

    await db.insert(subscribers).values({ email }).onConflictDoNothing();

    return new Response(
      JSON.stringify({ success: true }),
      { status: 200, headers: { 'Content-Type': 'application/json' } }
    );
  } catch (error) {
    return new Response(
      JSON.stringify({ error: 'Failed to subscribe' }),
      { status: 500, headers: { 'Content-Type': 'application/json' } }
    );
  }
};
```

### Admin API: Get Submissions

Create `src/pages/api/admin/submissions.ts`:

```typescript
import type { APIRoute } from 'astro';
import { db } from '@/lib/db';
import { contacts } from '@/lib/schema';
import { desc } from 'drizzle-orm';

export const prerender = false;

export const GET: APIRoute = async ({ cookies }) => {
  // Simple auth check — replace with your auth system
  const adminToken = cookies.get('admin_token')?.value;
  if (adminToken !== import.meta.env.ADMIN_SECRET) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), { status: 401 });
  }

  try {
    const submissions = await db
      .select()
      .from(contacts)
      .orderBy(desc(contacts.createdAt));

    return new Response(JSON.stringify(submissions), {
      status: 200,
      headers: { 'Content-Type': 'application/json' },
    });
  } catch (error) {
    return new Response(JSON.stringify({ error: 'Failed to fetch' }), { status: 500 });
  }
};
```

---

## Admin Dashboard

### Authentication Middleware

Add admin route protection to `src/middleware.ts`:

```typescript
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware(async (context, next) => {
  // Protect admin routes
  if (context.url.pathname.startsWith('/admin')) {
    const adminToken = context.cookies.get('admin_token')?.value;
    if (adminToken !== import.meta.env.ADMIN_SECRET) {
      // Redirect to login
      return context.redirect('/admin/login');
    }
  }

  const response = await next();

  // Security headers
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'SAMEORIGIN');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');

  return response;
});
```

### Admin Login Page

Create `src/pages/admin/login.astro`:

```astro
---
export const prerender = false;

import BaseLayout from '../../layouts/BaseLayout.astro';

if (Astro.request.method === 'POST') {
  const formData = await Astro.request.formData();
  const password = formData.get('password');

  if (password === import.meta.env.ADMIN_SECRET) {
    Astro.cookies.set('admin_token', import.meta.env.ADMIN_SECRET, {
      path: '/',
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
      maxAge: 60 * 60 * 24, // 24 hours
    });
    return Astro.redirect('/admin');
  }
}
---

<BaseLayout title="Admin Login" description="Admin login" noindex={true}>
  <div class="flex items-center justify-center min-h-[60vh]">
    <form method="POST" class="w-full max-w-sm space-y-4 p-6">
      <h1 class="text-2xl font-bold">Admin Login</h1>
      <input
        type="password"
        name="password"
        placeholder="Enter admin password"
        class="w-full px-4 py-2 border rounded"
        required
      />
      <button type="submit" class="w-full px-4 py-2 bg-primary text-primary-foreground rounded">
        Login
      </button>
    </form>
  </div>
</BaseLayout>
```

### Admin Dashboard Page

Create `src/pages/admin/index.astro`:

```astro
---
export const prerender = false;

import BaseLayout from '../../layouts/BaseLayout.astro';
import AdminDashboard from '../../components/admin/AdminDashboard';
---

<BaseLayout title="Admin Dashboard" description="Site administration" noindex={true}>
  <div class="max-w-7xl mx-auto px-4 py-8">
    <div class="flex items-center justify-between mb-8">
      <h1 class="text-3xl font-bold">Dashboard</h1>
      <div class="flex gap-4">
        <a href="/keystatic" class="px-4 py-2 bg-secondary text-secondary-foreground rounded hover:opacity-90">
          Content Manager
        </a>
        <a href="/admin/logout" class="px-4 py-2 text-muted-foreground hover:text-foreground">
          Logout
        </a>
      </div>
    </div>
    <AdminDashboard client:load />
  </div>
</BaseLayout>
```

### Admin Dashboard Component

Create `src/components/admin/AdminDashboard.tsx`:

```tsx
import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import {
  Table, TableBody, TableCell, TableHead,
  TableHeader, TableRow,
} from "@/components/ui/table";
import { Badge } from "@/components/ui/badge";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";

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
      .then(data => {
        setContacts(Array.isArray(data) ? data : []);
        setLoading(false);
      })
      .catch(() => setLoading(false));
  }, []);

  const newCount = contacts.filter(c => c.status === 'new').length;
  const totalCount = contacts.length;

  if (loading) {
    return <p className="text-muted-foreground">Loading dashboard...</p>;
  }

  return (
    <div className="space-y-8">
      {/* Stats Cards */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium text-muted-foreground">
              Total Submissions
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p className="text-3xl font-bold">{totalCount}</p>
          </CardContent>
        </Card>
        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium text-muted-foreground">
              New (Unread)
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p className="text-3xl font-bold text-primary">{newCount}</p>
          </CardContent>
        </Card>
        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium text-muted-foreground">
              Responded
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p className="text-3xl font-bold">{totalCount - newCount}</p>
          </CardContent>
        </Card>
      </div>

      {/* Data Table */}
      <Card>
        <CardHeader>
          <CardTitle>Contact Submissions</CardTitle>
        </CardHeader>
        <CardContent>
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Name</TableHead>
                <TableHead>Email</TableHead>
                <TableHead>Phone</TableHead>
                <TableHead className="max-w-xs">Message</TableHead>
                <TableHead>Date</TableHead>
                <TableHead>Status</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {contacts.map((contact) => (
                <TableRow key={contact.id}>
                  <TableCell className="font-medium">{contact.name}</TableCell>
                  <TableCell>{contact.email}</TableCell>
                  <TableCell>{contact.phone || '—'}</TableCell>
                  <TableCell className="max-w-xs truncate">{contact.message}</TableCell>
                  <TableCell>{new Date(contact.created_at).toLocaleDateString()}</TableCell>
                  <TableCell>
                    <Badge variant={contact.status === 'new' ? 'default' : 'secondary'}>
                      {contact.status}
                    </Badge>
                  </TableCell>
                </TableRow>
              ))}
              {contacts.length === 0 && (
                <TableRow>
                  <TableCell colSpan={6} className="text-center text-muted-foreground py-8">
                    No submissions yet
                  </TableCell>
                </TableRow>
              )}
            </TableBody>
          </Table>
        </CardContent>
      </Card>
    </div>
  );
}
```

---

## Environment Variables for Admin

Add to `.env`:
```
TURSO_DATABASE_URL=libsql://...
TURSO_AUTH_TOKEN=...
ADMIN_SECRET=your-secure-password-here
```

Add `ADMIN_SECRET` to your deployment environment variables as well.
