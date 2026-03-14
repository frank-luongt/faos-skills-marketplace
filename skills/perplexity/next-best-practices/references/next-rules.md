# Next.js Best Practices — Complete Reference

Compiled from Vercel's next-best-practices skill (MIT License).
Source: https://github.com/vercel-labs/next-skills

---

## File Conventions

Next.js App Router uses file-based routing with special file conventions.

### Project Structure

Reference: https://nextjs.org/docs/app/getting-started/project-structure

```
app/
├── layout.tsx          # Root layout (required)
├── page.tsx            # Home page (/)
├── loading.tsx         # Loading UI
├── error.tsx           # Error UI
├── not-found.tsx       # 404 UI
├── global-error.tsx    # Global error UI
├── route.ts            # API endpoint
├── template.tsx        # Re-rendered layout
├── default.tsx         # Parallel route fallback
├── blog/
│   ├── page.tsx        # /blog
│   └── [slug]/
│       └── page.tsx    # /blog/:slug
└── (group)/            # Route group (no URL impact)
    └── page.tsx
```

### Special Files

| File | Purpose |
|------|---------|
| `page.tsx` | UI for a route segment |
| `layout.tsx` | Shared UI for segment and children |
| `loading.tsx` | Loading UI (Suspense boundary) |
| `error.tsx` | Error UI (Error boundary) |
| `not-found.tsx` | 404 UI |
| `route.ts` | API endpoint |
| `template.tsx` | Like layout but re-renders on navigation |
| `default.tsx` | Fallback for parallel routes |

### Route Segments

```
app/
├── blog/               # Static segment: /blog
├── [slug]/             # Dynamic segment: /:slug
├── [...slug]/          # Catch-all: /a/b/c
├── [[...slug]]/        # Optional catch-all: / or /a/b/c
└── (marketing)/        # Route group (ignored in URL)
```

### Parallel Routes

```
app/
├── @analytics/
│   └── page.tsx
├── @sidebar/
│   └── page.tsx
└── layout.tsx          # Receives { analytics, sidebar } as props
```

### Intercepting Routes

```
app/
├── feed/
│   └── page.tsx
├── @modal/
│   └── (.)photo/[id]/  # Intercepts /photo/[id] from /feed
│       └── page.tsx
└── photo/[id]/
    └── page.tsx
```

Conventions:
- `(.)` - same level
- `(..)` - one level up
- `(..)(..)` - two levels up
- `(...)` - from root

### Private Folders

```
app/
├── _components/        # Private folder (not a route)
│   └── Button.tsx
└── page.tsx
```

Prefix with `_` to exclude from routing.

### Middleware / Proxy

#### Next.js 14-15: `middleware.ts`

```ts
// middleware.ts (root of project)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Auth, redirects, rewrites, etc.
  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

#### Next.js 16+: `proxy.ts`

Renamed for clarity - same capabilities, different names:

```ts
// proxy.ts (root of project)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function proxy(request: NextRequest) {
  // Same logic as middleware
  return NextResponse.next();
}

export const proxyConfig = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

| Version | File | Export | Config |
|---------|------|--------|--------|
| v14-15 | `middleware.ts` | `middleware()` | `config` |
| v16+ | `proxy.ts` | `proxy()` | `proxyConfig` |

**Migration**: Run `npx @next/codemod@latest upgrade` to auto-rename.

### File Conventions Reference

Reference: https://nextjs.org/docs/app/api-reference/file-conventions

---

## RSC Boundaries

Detect and prevent invalid patterns when crossing Server/Client component boundaries.

### Detection Rules

#### 1. Async Client Components Are Invalid

Client components **cannot** be async functions. Only Server Components can be async.

**Detect:** File has `'use client'` AND component is `async function` or returns `Promise`

```tsx
// Bad: async client component
'use client'
export default async function UserProfile() {
  const user = await getUser() // Cannot await in client component
  return <div>{user.name}</div>
}

// Good: Remove async, fetch data in parent server component
// page.tsx (server component - no 'use client')
export default async function Page() {
  const user = await getUser()
  return <UserProfile user={user} />
}

// UserProfile.tsx (client component)
'use client'
export function UserProfile({ user }: { user: User }) {
  return <div>{user.name}</div>
}
```

```tsx
// Bad: async arrow function client component
'use client'
const Dashboard = async () => {
  const data = await fetchDashboard()
  return <div>{data}</div>
}

// Good: Fetch in server component, pass data down
```

#### 2. Non-Serializable Props to Client Components

Props passed from Server -> Client must be JSON-serializable.

**Detect:** Server component passes these to a client component:
- Functions (except Server Actions with `'use server'`)
- `Date` objects
- `Map`, `Set`, `WeakMap`, `WeakSet`
- Class instances
- `Symbol` (unless globally registered)
- Circular references

```tsx
// Bad: Function prop
// page.tsx (server)
export default function Page() {
  const handleClick = () => console.log('clicked')
  return <ClientButton onClick={handleClick} />
}

// Good: Define function inside client component
// ClientButton.tsx
'use client'
export function ClientButton() {
  const handleClick = () => console.log('clicked')
  return <button onClick={handleClick}>Click</button>
}
```

```tsx
// Bad: Date object (silently becomes string, then crashes)
// page.tsx (server)
export default async function Page() {
  const post = await getPost()
  return <PostCard createdAt={post.createdAt} /> // Date object
}

// PostCard.tsx (client) - will crash on .getFullYear()
'use client'
export function PostCard({ createdAt }: { createdAt: Date }) {
  return <span>{createdAt.getFullYear()}</span> // Runtime error!
}

// Good: Serialize to string on server
// page.tsx (server)
export default async function Page() {
  const post = await getPost()
  return <PostCard createdAt={post.createdAt.toISOString()} />
}

// PostCard.tsx (client)
'use client'
export function PostCard({ createdAt }: { createdAt: string }) {
  const date = new Date(createdAt)
  return <span>{date.getFullYear()}</span>
}
```

```tsx
// Bad: Class instance
const user = new UserModel(data)
<ClientProfile user={user} /> // Methods will be stripped

// Good: Pass plain object
const user = await getUser()
<ClientProfile user={{ id: user.id, name: user.name }} />
```

```tsx
// Bad: Map/Set
<ClientComponent items={new Map([['a', 1]])} />

// Good: Convert to array/object
<ClientComponent items={Object.fromEntries(map)} />
<ClientComponent items={Array.from(set)} />
```

#### 3. Server Actions Are the Exception

Functions marked with `'use server'` CAN be passed to client components.

```tsx
// Valid: Server Action can be passed
// actions.ts
'use server'
export async function submitForm(formData: FormData) {
  // server-side logic
}

// page.tsx (server)
import { submitForm } from './actions'
export default function Page() {
  return <ClientForm onSubmit={submitForm} /> // OK!
}

// ClientForm.tsx (client)
'use client'
export function ClientForm({ onSubmit }: { onSubmit: (data: FormData) => Promise<void> }) {
  return <form action={onSubmit}>...</form>
}
```

### Quick Reference

| Pattern | Valid? | Fix |
|---------|--------|-----|
| `'use client'` + `async function` | No | Fetch in server parent, pass data |
| Pass `() => {}` to client | No | Define in client or use server action |
| Pass `new Date()` to client | No | Use `.toISOString()` |
| Pass `new Map()` to client | No | Convert to object/array |
| Pass class instance to client | No | Pass plain object |
| Pass server action to client | Yes | - |
| Pass `string/number/boolean` | Yes | - |
| Pass plain object/array | Yes | - |

---

## Async Patterns

In Next.js 15+, `params`, `searchParams`, `cookies()`, and `headers()` are asynchronous.

### Async Params and SearchParams

Always type them as `Promise<...>` and await them.

#### Pages and Layouts

```tsx
type Props = { params: Promise<{ slug: string }> }

export default async function Page({ params }: Props) {
  const { slug } = await params
}
```

#### Route Handlers

```tsx
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
}
```

#### SearchParams

```tsx
type Props = {
  params: Promise<{ slug: string }>
  searchParams: Promise<{ query?: string }>
}

export default async function Page({ params, searchParams }: Props) {
  const { slug } = await params
  const { query } = await searchParams
}
```

#### Synchronous Components

Use `React.use()` for non-async components:

```tsx
import { use } from 'react'

type Props = { params: Promise<{ slug: string }> }

export default function Page({ params }: Props) {
  const { slug } = use(params)
}
```

#### generateMetadata

```tsx
type Props = { params: Promise<{ slug: string }> }

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params
  return { title: slug }
}
```

### Async Cookies and Headers

```tsx
import { cookies, headers } from 'next/headers'

export default async function Page() {
  const cookieStore = await cookies()
  const headersList = await headers()

  const theme = cookieStore.get('theme')
  const userAgent = headersList.get('user-agent')
}
```

### Migration Codemod

```bash
npx @next/codemod@latest next-async-request-api .
```

---

## Bundling

Fix common bundling issues with third-party packages.

### Server-Incompatible Packages

Some packages use browser APIs (`window`, `document`, `localStorage`) and fail in Server Components.

#### Error Signs

```
ReferenceError: window is not defined
ReferenceError: document is not defined
ReferenceError: localStorage is not defined
Module not found: Can't resolve 'fs'
```

#### Solution 1: Mark as Client-Only

If the package is only needed on client:

```tsx
// Bad: Fails - package uses window
import SomeChart from 'some-chart-library'

export default function Page() {
  return <SomeChart />
}

// Good: Use dynamic import with ssr: false
import dynamic from 'next/dynamic'

const SomeChart = dynamic(() => import('some-chart-library'), {
  ssr: false,
})

export default function Page() {
  return <SomeChart />
}
```

#### Solution 2: Externalize from Server Bundle

For packages that should run on server but have bundling issues:

```js
// next.config.js
module.exports = {
  serverExternalPackages: ['problematic-package'],
}
```

Use this for:
- Packages with native bindings (sharp, bcrypt)
- Packages that don't bundle well (some ORMs)
- Packages with circular dependencies

#### Solution 3: Client Component Wrapper

Wrap the entire usage in a client component:

```tsx
// components/ChartWrapper.tsx
'use client'

import { Chart } from 'chart-library'

export function ChartWrapper(props) {
  return <Chart {...props} />
}

// app/page.tsx (server component)
import { ChartWrapper } from '@/components/ChartWrapper'

export default function Page() {
  return <ChartWrapper data={data} />
}
```

### CSS Imports

Import CSS files instead of using `<link>` tags. Next.js handles bundling and optimization.

```tsx
// Bad: Manual link tag
<link rel="stylesheet" href="/styles.css" />

// Good: Import CSS
import './styles.css'

// Good: CSS Modules
import styles from './Button.module.css'
```

### Polyfills

Next.js includes common polyfills automatically. Don't load redundant ones from polyfill.io or similar CDNs.

Already included: `Array.from`, `Object.assign`, `Promise`, `fetch`, `Map`, `Set`, `Symbol`, `URLSearchParams`, and 50+ others.

```tsx
// Bad: Redundant polyfills
<script src="https://polyfill.io/v3/polyfill.min.js?features=fetch,Promise,Array.from" />

// Good: Next.js includes these automatically
```

### ESM/CommonJS Issues

#### Error Signs

```
SyntaxError: Cannot use import statement outside a module
Error: require() of ES Module
Module not found: ESM packages need to be imported
```

#### Solution: Transpile Package

```js
// next.config.js
module.exports = {
  transpilePackages: ['some-esm-package', 'another-package'],
}
```

### Common Problematic Packages

| Package | Issue | Solution |
|---------|-------|----------|
| `sharp` | Native bindings | `serverExternalPackages: ['sharp']` |
| `bcrypt` | Native bindings | `serverExternalPackages: ['bcrypt']` or use `bcryptjs` |
| `canvas` | Native bindings | `serverExternalPackages: ['canvas']` |
| `recharts` | Uses window | `dynamic(() => import('recharts'), { ssr: false })` |
| `react-quill` | Uses document | `dynamic(() => import('react-quill'), { ssr: false })` |
| `mapbox-gl` | Uses window | `dynamic(() => import('mapbox-gl'), { ssr: false })` |
| `monaco-editor` | Uses window | `dynamic(() => import('@monaco-editor/react'), { ssr: false })` |
| `lottie-web` | Uses document | `dynamic(() => import('lottie-react'), { ssr: false })` |

### Bundle Analysis

Analyze bundle size with the built-in analyzer (Next.js 16.1+):

```bash
next experimental-analyze
```

This opens an interactive UI to:
- Filter by route, environment (client/server), and type
- Inspect module sizes and import chains
- View treemap visualization

Save output for comparison:

```bash
next experimental-analyze --output
# Output saved to .next/diagnostics/analyze
```

Reference: https://nextjs.org/docs/app/guides/package-bundling

### Migrating from Webpack to Turbopack

Turbopack is the default bundler in Next.js 15+. If you have custom webpack config, migrate to Turbopack-compatible alternatives:

```js
// next.config.js
module.exports = {
  // Good: Works with Turbopack
  serverExternalPackages: ['package'],
  transpilePackages: ['package'],

  // Bad: Webpack-only - migrate away from this
  webpack: (config) => {
    // custom webpack config
  },
}
```

Reference: https://nextjs.org/docs/app/building-your-application/upgrading/from-webpack-to-turbopack

---

## Data Patterns

Choose the right data fetching pattern for each use case.

### Decision Tree

```
Need to fetch data?
├── From a Server Component?
│   └── Use: Fetch directly (no API needed)
│
├── From a Client Component?
│   ├── Is it a mutation (POST/PUT/DELETE)?
│   │   └── Use: Server Action
│   └── Is it a read (GET)?
│       └── Use: Route Handler OR pass from Server Component
│
├── Need external API access (webhooks, third parties)?
│   └── Use: Route Handler
│
└── Need REST API for mobile app / external clients?
    └── Use: Route Handler
```

### Pattern 1: Server Components (Preferred for Reads)

Fetch data directly in Server Components - no API layer needed.

```tsx
// app/users/page.tsx
async function UsersPage() {
  // Direct database access - no API round-trip
  const users = await db.user.findMany();

  // Or fetch from external API
  const posts = await fetch('https://api.example.com/posts').then(r => r.json());

  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

**Benefits**:
- No API to maintain
- No client-server waterfall
- Secrets stay on server
- Direct database access

### Pattern 2: Server Actions (Preferred for Mutations)

Server Actions are the recommended way to handle mutations.

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;

  await db.post.create({ data: { title } });

  revalidatePath('/posts');
}

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });

  revalidateTag('posts');
}
```

```tsx
// app/posts/new/page.tsx
import { createPost } from '@/app/actions';

export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

**Benefits**:
- End-to-end type safety
- Progressive enhancement (works without JS)
- Automatic request handling
- Integrated with React transitions

**Constraints**:
- POST only (no GET caching semantics)
- Internal use only (no external access)
- Cannot return non-serializable data

### Pattern 3: Route Handlers (APIs)

Use Route Handlers when you need a REST API.

```tsx
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';

// GET is cacheable
export async function GET(request: NextRequest) {
  const posts = await db.post.findMany();
  return NextResponse.json(posts);
}

// POST for mutations
export async function POST(request: NextRequest) {
  const body = await request.json();
  const post = await db.post.create({ data: body });
  return NextResponse.json(post, { status: 201 });
}
```

**When to use**:
- External API access (mobile apps, third parties)
- Webhooks from external services
- GET endpoints that need HTTP caching
- OpenAPI/Swagger documentation needed

**When NOT to use**:
- Internal data fetching (use Server Components)
- Mutations from your UI (use Server Actions)

### Avoiding Data Waterfalls

#### Problem: Sequential Fetches

```tsx
// Bad: Sequential waterfalls
async function Dashboard() {
  const user = await getUser();        // Wait...
  const posts = await getPosts();      // Then wait...
  const comments = await getComments(); // Then wait...

  return <div>...</div>;
}
```

#### Solution 1: Parallel Fetching with Promise.all

```tsx
// Good: Parallel fetching
async function Dashboard() {
  const [user, posts, comments] = await Promise.all([
    getUser(),
    getPosts(),
    getComments(),
  ]);

  return <div>...</div>;
}
```

#### Solution 2: Streaming with Suspense

```tsx
// Good: Show content progressively
import { Suspense } from 'react';

async function Dashboard() {
  return (
    <div>
      <Suspense fallback={<UserSkeleton />}>
        <UserSection />
      </Suspense>
      <Suspense fallback={<PostsSkeleton />}>
        <PostsSection />
      </Suspense>
    </div>
  );
}

async function UserSection() {
  const user = await getUser(); // Fetches independently
  return <div>{user.name}</div>;
}

async function PostsSection() {
  const posts = await getPosts(); // Fetches independently
  return <PostList posts={posts} />;
}
```

#### Solution 3: Preload Pattern

```tsx
// lib/data.ts
import { cache } from 'react';

export const getUser = cache(async (id: string) => {
  return db.user.findUnique({ where: { id } });
});

export const preloadUser = (id: string) => {
  void getUser(id); // Fire and forget
};
```

```tsx
// app/user/[id]/page.tsx
import { getUser, preloadUser } from '@/lib/data';

export default async function UserPage({ params }) {
  const { id } = await params;

  // Start fetching early
  preloadUser(id);

  // Do other work...

  // Data likely ready by now
  const user = await getUser(id);
  return <div>{user.name}</div>;
}
```

### Client Component Data Fetching

When Client Components need data:

#### Option 1: Pass from Server Component (Preferred)

```tsx
// Server Component
async function Page() {
  const data = await fetchData();
  return <ClientComponent initialData={data} />;
}

// Client Component
'use client';
function ClientComponent({ initialData }) {
  const [data, setData] = useState(initialData);
  // ...
}
```

#### Option 2: Fetch on Mount (When Necessary)

```tsx
'use client';
import { useEffect, useState } from 'react';

function ClientComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/data')
      .then(r => r.json())
      .then(setData);
  }, []);

  if (!data) return <Loading />;
  return <div>{data.value}</div>;
}
```

#### Option 3: Server Action for Reads (Works But Not Ideal)

Server Actions can be called from Client Components for reads, but this is not their intended purpose:

```tsx
'use client';
import { getData } from './actions';
import { useEffect, useState } from 'react';

function ClientComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    getData().then(setData);
  }, []);

  return <div>{data?.value}</div>;
}
```

**Note**: Server Actions always use POST, so no HTTP caching. Prefer Route Handlers for cacheable reads.

### Quick Reference

| Pattern | Use Case | HTTP Method | Caching |
|---------|----------|-------------|---------|
| Server Component fetch | Internal reads | Any | Full Next.js caching |
| Server Action | Mutations, form submissions | POST only | No |
| Route Handler | External APIs, webhooks | Any | GET can be cached |
| Client fetch to API | Client-side reads | Any | HTTP cache headers |

---

## Debug Tricks

Tricks to speed up debugging Next.js applications.

### MCP Endpoint (Dev Server)

Next.js exposes a `/_next/mcp` endpoint in development for AI-assisted debugging via MCP (Model Context Protocol).

- **Next.js 16+**: Enabled by default, use `next-devtools-mcp`
- **Next.js < 16**: Requires `experimental.mcpServer: true` in next.config.js

Reference: https://nextjs.org/docs/app/guides/mcp

**Important**: Find the actual port of the running Next.js dev server (check terminal output or `package.json` scripts). Don't assume port 3000.

#### Request Format

The endpoint uses JSON-RPC 2.0 over HTTP POST:

```bash
curl -X POST http://localhost:<port>/_next/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": "1",
    "method": "tools/call",
    "params": {
      "name": "<tool-name>",
      "arguments": {}
    }
  }'
```

#### Available Tools

##### `get_errors`
Get current errors from dev server (build errors, runtime errors with source-mapped stacks):
```json
{ "name": "get_errors", "arguments": {} }
```

##### `get_routes`
Discover all routes by scanning filesystem:
```json
{ "name": "get_routes", "arguments": {} }
// Optional: { "name": "get_routes", "arguments": { "routerType": "app" } }
```
Returns: `{ "appRouter": ["/", "/api/users/[id]", ...], "pagesRouter": [...] }`

##### `get_project_metadata`
Get project path and dev server URL:
```json
{ "name": "get_project_metadata", "arguments": {} }
```
Returns: `{ "projectPath": "/path/to/project", "devServerUrl": "http://localhost:3000" }`

##### `get_page_metadata`
Get runtime metadata about current page render (requires active browser session):
```json
{ "name": "get_page_metadata", "arguments": {} }
```
Returns segment trie data showing layouts, boundaries, and page components.

##### `get_logs`
Get path to Next.js development log file:
```json
{ "name": "get_logs", "arguments": {} }
```
Returns path to `<distDir>/logs/next-development.log`

##### `get_server_action_by_id`
Locate a Server Action by ID:
```json
{ "name": "get_server_action_by_id", "arguments": { "actionId": "<action-id>" } }
```

#### Example: Get Errors

```bash
curl -X POST http://localhost:<port>/_next/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":"1","method":"tools/call","params":{"name":"get_errors","arguments":{}}}'
```

### Rebuild Specific Routes (Next.js 16+)

Use `--debug-build-paths` to rebuild only specific routes instead of the entire app:

```bash
# Rebuild a specific route
next build --debug-build-paths "/dashboard"

# Rebuild routes matching a glob
next build --debug-build-paths "/api/*"

# Dynamic routes
next build --debug-build-paths "/blog/[slug]"
```

Use this to:
- Quickly verify a build fix without full rebuild
- Debug static generation issues for specific pages
- Iterate faster on build errors

---

## Directives

### React Directives

These are React directives, not Next.js specific.

#### `'use client'`

Marks a component as a Client Component. Required for:
- React hooks (`useState`, `useEffect`, etc.)
- Event handlers (`onClick`, `onChange`)
- Browser APIs (`window`, `localStorage`)

```tsx
'use client'

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

Reference: https://react.dev/reference/rsc/use-client

#### `'use server'`

Marks a function as a Server Action. Can be passed to Client Components.

```tsx
'use server'

export async function submitForm(formData: FormData) {
  // Runs on server
}
```

Or inline within a Server Component:

```tsx
export default function Page() {
  async function submit() {
    'use server'
    // Runs on server
  }
  return <form action={submit}>...</form>
}
```

Reference: https://react.dev/reference/rsc/use-server

### Next.js Directive

#### `'use cache'`

Marks a function or component for caching. Part of Next.js Cache Components.

```tsx
'use cache'

export async function getCachedData() {
  return await fetchData()
}
```

Requires `cacheComponents: true` in `next.config.ts`.

For detailed usage including cache profiles, `cacheLife()`, `cacheTag()`, and `updateTag()`, see the `next-cache-components` skill.

Reference: https://nextjs.org/docs/app/api-reference/directives/use-cache

---

## Error Handling

Handle errors gracefully in Next.js applications.

Reference: https://nextjs.org/docs/app/getting-started/error-handling

### Error Boundaries

#### `error.tsx`

Catches errors in a route segment and its children:

```tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

**Important:** `error.tsx` must be a Client Component.

#### `global-error.tsx`

Catches errors in root layout:

```tsx
'use client'

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  )
}
```

**Important:** Must include `<html>` and `<body>` tags.

### Server Actions: Navigation API Gotcha

**Do NOT wrap navigation APIs in try-catch.** They throw special errors that Next.js handles internally.

Reference: https://nextjs.org/docs/app/api-reference/functions/redirect#behavior

```tsx
'use server'

import { redirect } from 'next/navigation'
import { notFound } from 'next/navigation'

// Bad: try-catch catches the navigation "error"
async function createPost(formData: FormData) {
  try {
    const post = await db.post.create({ ... })
    redirect(`/posts/${post.id}`)  // This throws!
  } catch (error) {
    // redirect() throw is caught here - navigation fails!
    return { error: 'Failed to create post' }
  }
}

// Good: Call navigation APIs outside try-catch
async function createPost(formData: FormData) {
  let post
  try {
    post = await db.post.create({ ... })
  } catch (error) {
    return { error: 'Failed to create post' }
  }
  redirect(`/posts/${post.id}`)  // Outside try-catch
}

// Good: Re-throw navigation errors
async function createPost(formData: FormData) {
  try {
    const post = await db.post.create({ ... })
    redirect(`/posts/${post.id}`)
  } catch (error) {
    if (error instanceof Error && error.message === 'NEXT_REDIRECT') {
      throw error  // Re-throw navigation errors
    }
    return { error: 'Failed to create post' }
  }
}
```

Same applies to:
- `redirect()` - 307 temporary redirect
- `permanentRedirect()` - 308 permanent redirect
- `notFound()` - 404 not found
- `forbidden()` - 403 forbidden
- `unauthorized()` - 401 unauthorized

Use `unstable_rethrow()` to re-throw these errors in catch blocks:

```tsx
import { unstable_rethrow } from 'next/navigation'

async function action() {
  try {
    // ...
    redirect('/success')
  } catch (error) {
    unstable_rethrow(error) // Re-throws Next.js internal errors
    return { error: 'Something went wrong' }
  }
}
```

### Redirects

```tsx
import { redirect, permanentRedirect } from 'next/navigation'

// 307 Temporary - use for most cases
redirect('/new-path')

// 308 Permanent - use for URL migrations (cached by browsers)
permanentRedirect('/new-url')
```

### Auth Errors

Trigger auth-related error pages:

```tsx
import { forbidden, unauthorized } from 'next/navigation'

async function Page() {
  const session = await getSession()

  if (!session) {
    unauthorized() // Renders unauthorized.tsx (401)
  }

  if (!session.hasAccess) {
    forbidden() // Renders forbidden.tsx (403)
  }

  return <Dashboard />
}
```

Create corresponding error pages:

```tsx
// app/forbidden.tsx
export default function Forbidden() {
  return <div>You don't have access to this resource</div>
}

// app/unauthorized.tsx
export default function Unauthorized() {
  return <div>Please log in to continue</div>
}
```

### Not Found

#### `not-found.tsx`

Custom 404 page for a route segment:

```tsx
export default function NotFound() {
  return (
    <div>
      <h2>Not Found</h2>
      <p>Could not find the requested resource</p>
    </div>
  )
}
```

#### Triggering Not Found

```tsx
import { notFound } from 'next/navigation'

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  const post = await getPost(id)

  if (!post) {
    notFound()  // Renders closest not-found.tsx
  }

  return <div>{post.title}</div>
}
```

### Error Hierarchy

Errors bubble up to the nearest error boundary:

```
app/
├── error.tsx           # Catches errors from all children
├── blog/
│   ├── error.tsx       # Catches errors in /blog/*
│   └── [slug]/
│       ├── error.tsx   # Catches errors in /blog/[slug]
│       └── page.tsx
└── layout.tsx          # Errors here go to global-error.tsx
```

---

## Font Optimization

Use `next/font` for automatic font optimization with zero layout shift.

### Google Fonts

```tsx
// app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({ subsets: ['latin'] })

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  )
}
```

### Multiple Fonts

```tsx
import { Inter, Roboto_Mono } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
})

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  variable: '--font-roboto-mono',
})

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body>{children}</body>
    </html>
  )
}
```

Use in CSS:
```css
body {
  font-family: var(--font-inter);
}

code {
  font-family: var(--font-roboto-mono);
}
```

### Font Weights and Styles

```tsx
// Single weight
const inter = Inter({
  subsets: ['latin'],
  weight: '400',
})

// Multiple weights
const inter = Inter({
  subsets: ['latin'],
  weight: ['400', '500', '700'],
})

// Variable font (recommended) - includes all weights
const inter = Inter({
  subsets: ['latin'],
  // No weight needed - variable fonts support all weights
})

// With italic
const inter = Inter({
  subsets: ['latin'],
  style: ['normal', 'italic'],
})
```

### Local Fonts

```tsx
import localFont from 'next/font/local'

const myFont = localFont({
  src: './fonts/MyFont.woff2',
})

// Multiple files for different weights
const myFont = localFont({
  src: [
    {
      path: './fonts/MyFont-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: './fonts/MyFont-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
  ],
})

// Variable font
const myFont = localFont({
  src: './fonts/MyFont-Variable.woff2',
  variable: '--font-my-font',
})
```

### Tailwind CSS Integration

```tsx
// app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
})

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.variable}>
      <body>{children}</body>
    </html>
  )
}
```

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-inter)'],
      },
    },
  },
}
```

### Preloading Subsets

Only load needed character subsets:

```tsx
// Latin only (most common)
const inter = Inter({ subsets: ['latin'] })

// Multiple subsets
const inter = Inter({ subsets: ['latin', 'latin-ext', 'cyrillic'] })
```

### Display Strategy

Control font loading behavior:

```tsx
const inter = Inter({
  subsets: ['latin'],
  display: 'swap', // Default - shows fallback, swaps when loaded
})

// Options:
// 'auto' - browser decides
// 'block' - short block period, then swap
// 'swap' - immediate fallback, swap when ready (recommended)
// 'fallback' - short block, short swap, then fallback
// 'optional' - short block, no swap (use if font is optional)
```

### Don't Use Manual Font Links

Always use `next/font` instead of `<link>` tags for Google Fonts.

```tsx
// Bad: Manual link tag (blocks rendering, no optimization)
<link href="https://fonts.googleapis.com/css2?family=Inter" rel="stylesheet" />

// Bad: Missing display and preconnect
<link href="https://fonts.googleapis.com/css2?family=Inter" rel="stylesheet" />

// Good: Use next/font (self-hosted, zero layout shift)
import { Inter } from 'next/font/google'

const inter = Inter({ subsets: ['latin'] })
```

### Common Mistakes

```tsx
// Bad: Importing font in every component
// components/Button.tsx
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'] }) // Creates new instance each time!

// Good: Import once in layout, use CSS variable
// app/layout.tsx
const inter = Inter({ subsets: ['latin'], variable: '--font-inter' })

// Bad: Using @import in CSS (blocks rendering)
/* globals.css */
@import url('https://fonts.googleapis.com/css2?family=Inter');

// Good: Use next/font (self-hosted, no network request)
import { Inter } from 'next/font/google'

// Bad: Loading all weights when only using a few
const inter = Inter({ subsets: ['latin'] }) // Loads all weights

// Good: Specify only needed weights (for non-variable fonts)
const inter = Inter({ subsets: ['latin'], weight: ['400', '700'] })

// Bad: Missing subset - loads all characters
const inter = Inter({})

// Good: Always specify subset
const inter = Inter({ subsets: ['latin'] })
```

### Font in Specific Components

```tsx
// For component-specific fonts, export from a shared file
// lib/fonts.ts
import { Inter, Playfair_Display } from 'next/font/google'

export const inter = Inter({ subsets: ['latin'], variable: '--font-inter' })
export const playfair = Playfair_Display({ subsets: ['latin'], variable: '--font-playfair' })

// components/Heading.tsx
import { playfair } from '@/lib/fonts'

export function Heading({ children }) {
  return <h1 className={playfair.className}>{children}</h1>
}
```

---

## Functions

Next.js function APIs.

Reference: https://nextjs.org/docs/app/api-reference/functions

### Navigation Hooks (Client)

| Hook | Purpose | Reference |
|------|---------|-----------|
| `useRouter` | Programmatic navigation (`push`, `replace`, `back`, `refresh`) | [Docs](https://nextjs.org/docs/app/api-reference/functions/use-router) |
| `usePathname` | Get current pathname | [Docs](https://nextjs.org/docs/app/api-reference/functions/use-pathname) |
| `useSearchParams` | Read URL search parameters | [Docs](https://nextjs.org/docs/app/api-reference/functions/use-search-params) |
| `useParams` | Access dynamic route parameters | [Docs](https://nextjs.org/docs/app/api-reference/functions/use-params) |
| `useSelectedLayoutSegment` | Active child segment (one level) | [Docs](https://nextjs.org/docs/app/api-reference/functions/use-selected-layout-segment) |
| `useSelectedLayoutSegments` | All active segments below layout | [Docs](https://nextjs.org/docs/app/api-reference/functions/use-selected-layout-segments) |
| `useLinkStatus` | Check link prefetch status | [Docs](https://nextjs.org/docs/app/api-reference/functions/use-link-status) |
| `useReportWebVitals` | Report Core Web Vitals metrics | [Docs](https://nextjs.org/docs/app/api-reference/functions/use-report-web-vitals) |

### Server Functions

| Function | Purpose | Reference |
|----------|---------|-----------|
| `cookies` | Read/write cookies | [Docs](https://nextjs.org/docs/app/api-reference/functions/cookies) |
| `headers` | Read request headers | [Docs](https://nextjs.org/docs/app/api-reference/functions/headers) |
| `draftMode` | Enable preview of unpublished CMS content | [Docs](https://nextjs.org/docs/app/api-reference/functions/draft-mode) |
| `after` | Run code after response finishes streaming | [Docs](https://nextjs.org/docs/app/api-reference/functions/after) |
| `connection` | Wait for connection before dynamic rendering | [Docs](https://nextjs.org/docs/app/api-reference/functions/connection) |
| `userAgent` | Parse User-Agent header | [Docs](https://nextjs.org/docs/app/api-reference/functions/userAgent) |

### Generate Functions

| Function | Purpose | Reference |
|----------|---------|-----------|
| `generateStaticParams` | Pre-render dynamic routes at build time | [Docs](https://nextjs.org/docs/app/api-reference/functions/generate-static-params) |
| `generateMetadata` | Dynamic metadata | [Docs](https://nextjs.org/docs/app/api-reference/functions/generate-metadata) |
| `generateViewport` | Dynamic viewport config | [Docs](https://nextjs.org/docs/app/api-reference/functions/generate-viewport) |
| `generateSitemaps` | Multiple sitemaps for large sites | [Docs](https://nextjs.org/docs/app/api-reference/functions/generate-sitemaps) |
| `generateImageMetadata` | Multiple OG images per route | [Docs](https://nextjs.org/docs/app/api-reference/functions/generate-image-metadata) |

### Request/Response

| Function | Purpose | Reference |
|----------|---------|-----------|
| `NextRequest` | Extended Request with helpers | [Docs](https://nextjs.org/docs/app/api-reference/functions/next-request) |
| `NextResponse` | Extended Response with helpers | [Docs](https://nextjs.org/docs/app/api-reference/functions/next-response) |
| `ImageResponse` | Generate OG images | [Docs](https://nextjs.org/docs/app/api-reference/functions/image-response) |

### Common Examples

#### Navigation

Use `next/link` for internal navigation instead of `<a>` tags.

```tsx
// Bad: Plain anchor tag
<a href="/about">About</a>

// Good: Next.js Link
import Link from 'next/link'

<Link href="/about">About</Link>
```

Active link styling:

```tsx
'use client'

import Link from 'next/link'
import { usePathname } from 'next/navigation'

export function NavLink({ href, children }) {
  const pathname = usePathname()

  return (
    <Link href={href} className={pathname === href ? 'active' : ''}>
      {children}
    </Link>
  )
}
```

#### Static Generation

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map((post) => ({ slug: post.slug }))
}
```

#### After Response

```tsx
import { after } from 'next/server'

export async function POST(request: Request) {
  const data = await processRequest(request)

  after(async () => {
    await logAnalytics(data)
  })

  return Response.json({ success: true })
}
```

---

## Hydration Errors

Diagnose and fix React hydration mismatch errors.

### Error Signs

- "Hydration failed because the initial UI does not match"
- "Text content does not match server-rendered HTML"

### Debugging

In development, click the hydration error to see the server/client diff.

### Common Causes and Fixes

#### Browser-only APIs

```tsx
// Bad: Causes mismatch - window doesn't exist on server
<div>{window.innerWidth}</div>

// Good: Use client component with mounted check
'use client'
import { useState, useEffect } from 'react'

export function ClientOnly({ children }: { children: React.ReactNode }) {
  const [mounted, setMounted] = useState(false)
  useEffect(() => setMounted(true), [])
  return mounted ? children : null
}
```

#### Date/Time Rendering

Server and client may be in different timezones:

```tsx
// Bad: Causes mismatch
<span>{new Date().toLocaleString()}</span>

// Good: Render on client only
'use client'
const [time, setTime] = useState<string>()
useEffect(() => setTime(new Date().toLocaleString()), [])
```

#### Random Values or IDs

```tsx
// Bad: Random values differ between server and client
<div id={Math.random().toString()}>

// Good: Use useId hook
import { useId } from 'react'

function Input() {
  const id = useId()
  return <input id={id} />
}
```

#### Invalid HTML Nesting

```tsx
// Bad: Invalid - div inside p
<p><div>Content</div></p>

// Bad: Invalid - p inside p
<p><p>Nested</p></p>

// Good: Valid nesting
<div><p>Content</p></div>
```

#### Third-party Scripts

Scripts that modify DOM during hydration.

```tsx
// Good: Use next/script with afterInteractive
import Script from 'next/script'

export default function Page() {
  return (
    <Script
      src="https://example.com/script.js"
      strategy="afterInteractive"
    />
  )
}
```

---

## Image Optimization

Use `next/image` for automatic image optimization.

### Always Use next/image

```tsx
// Bad: Avoid native img
<img src="/hero.png" alt="Hero" />

// Good: Use next/image
import Image from 'next/image'
<Image src="/hero.png" alt="Hero" width={800} height={400} />
```

### Required Props

Images need explicit dimensions to prevent layout shift:

```tsx
// Local images - dimensions inferred automatically
import heroImage from './hero.png'
<Image src={heroImage} alt="Hero" />

// Remote images - must specify width/height
<Image src="https://example.com/image.jpg" alt="Hero" width={800} height={400} />

// Or use fill for parent-relative sizing
<div style={{ position: 'relative', width: '100%', height: 400 }}>
  <Image src="/hero.png" alt="Hero" fill style={{ objectFit: 'cover' }} />
</div>
```

### Remote Images Configuration

Remote domains must be configured in `next.config.js`:

```js
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
        pathname: '/images/**',
      },
      {
        protocol: 'https',
        hostname: '*.cdn.com', // Wildcard subdomain
      },
    ],
  },
}
```

### Responsive Images

Use `sizes` to tell the browser which size to download:

```tsx
// Full-width hero
<Image
  src="/hero.png"
  alt="Hero"
  fill
  sizes="100vw"
/>

// Responsive grid (3 columns on desktop, 1 on mobile)
<Image
  src="/card.png"
  alt="Card"
  fill
  sizes="(max-width: 768px) 100vw, 33vw"
/>

// Fixed sidebar image
<Image
  src="/avatar.png"
  alt="Avatar"
  width={200}
  height={200}
  sizes="200px"
/>
```

### Blur Placeholder

Prevent layout shift with placeholders:

```tsx
// Local images - automatic blur hash
import heroImage from './hero.png'
<Image src={heroImage} alt="Hero" placeholder="blur" />

// Remote images - provide blurDataURL
<Image
  src="https://example.com/image.jpg"
  alt="Hero"
  width={800}
  height={400}
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..."
/>

// Or use color placeholder
<Image
  src="https://example.com/image.jpg"
  alt="Hero"
  width={800}
  height={400}
  placeholder="empty"
  style={{ backgroundColor: '#e0e0e0' }}
/>
```

### Priority Loading

Use `priority` for above-the-fold images (LCP):

```tsx
// Hero image - loads immediately
<Image src="/hero.png" alt="Hero" fill priority />

// Below-fold images - lazy loaded by default (no priority needed)
<Image src="/card.png" alt="Card" width={400} height={300} />
```

### Common Mistakes

```tsx
// Bad: Missing sizes with fill - downloads largest image
<Image src="/hero.png" alt="Hero" fill />

// Good: Add sizes for proper responsive behavior
<Image src="/hero.png" alt="Hero" fill sizes="100vw" />

// Bad: Using width/height for aspect ratio only
<Image src="/hero.png" alt="Hero" width={16} height={9} />

// Good: Use actual display dimensions or fill with sizes
<Image src="/hero.png" alt="Hero" fill sizes="100vw" style={{ objectFit: 'cover' }} />

// Bad: Remote image without config
<Image src="https://untrusted.com/image.jpg" alt="Image" width={400} height={300} />
// Error: Invalid src prop, hostname not configured

// Good: Add hostname to next.config.js remotePatterns
```

### Static Export

When using `output: 'export'`, use `unoptimized` or custom loader:

```tsx
// Option 1: Disable optimization
<Image src="/hero.png" alt="Hero" width={800} height={400} unoptimized />

// Option 2: Global config
// next.config.js
module.exports = {
  output: 'export',
  images: { unoptimized: true },
}

// Option 3: Custom loader (Cloudinary, Imgix, etc.)
const cloudinaryLoader = ({ src, width, quality }) => {
  return `https://res.cloudinary.com/demo/image/upload/w_${width},q_${quality || 75}/${src}`
}

<Image loader={cloudinaryLoader} src="sample.jpg" alt="Sample" width={800} height={400} />
```

---

## Metadata

Add SEO metadata to Next.js pages using the Metadata API.

### Important: Server Components Only

The `metadata` object and `generateMetadata` function are **only supported in Server Components**. They cannot be used in Client Components.

If the target page has `'use client'`:
1. Remove `'use client'` if possible, move client logic to child components
2. Or extract metadata to a parent Server Component layout
3. Or split the file: Server Component with metadata imports Client Components

### Static Metadata

```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Page Title',
  description: 'Page description for search engines',
}
```

### Dynamic Metadata

```tsx
import type { Metadata } from 'next'

type Props = { params: Promise<{ slug: string }> }

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params
  const post = await getPost(slug)
  return { title: post.title, description: post.description }
}
```

### Avoid Duplicate Fetches

Use React `cache()` when the same data is needed for both metadata and page:

```tsx
import { cache } from 'react'

export const getPost = cache(async (slug: string) => {
  return await db.posts.findFirst({ where: { slug } })
})
```

### Viewport

Separate from metadata for streaming support:

```tsx
import type { Viewport } from 'next'

export const viewport: Viewport = {
  width: 'device-width',
  initialScale: 1,
  themeColor: '#000000',
}

// Or dynamic
export function generateViewport({ params }): Viewport {
  return { themeColor: getThemeColor(params) }
}
```

### Title Templates

In root layout for consistent naming:

```tsx
export const metadata: Metadata = {
  title: { default: 'Site Name', template: '%s | Site Name' },
}
```

### Metadata File Conventions

Reference: https://nextjs.org/docs/app/getting-started/project-structure#metadata-file-conventions

Place these files in `app/` directory (or route segments):

| File | Purpose |
|------|---------|
| `favicon.ico` | Favicon |
| `icon.png` / `icon.svg` | App icon |
| `apple-icon.png` | Apple app icon |
| `opengraph-image.png` | OG image |
| `twitter-image.png` | Twitter card image |
| `sitemap.ts` / `sitemap.xml` | Sitemap (use `generateSitemaps` for multiple) |
| `robots.ts` / `robots.txt` | Robots directives |
| `manifest.ts` / `manifest.json` | Web app manifest |

### SEO Best Practice: Static Files Are Often Enough

For most sites, **static metadata files provide excellent SEO coverage**:

```
app/
├── favicon.ico
├── opengraph-image.png     # Works for both OG and Twitter
├── sitemap.ts
├── robots.ts
└── layout.tsx              # With title/description metadata
```

**Tips:**
- A single `opengraph-image.png` covers both Open Graph and Twitter (Twitter falls back to OG)
- Static `title` and `description` in layout metadata is sufficient for most pages
- Only use dynamic `generateMetadata` when content varies per page

### OG Image Generation

Generate dynamic Open Graph images using `next/og`.

#### Important Rules

1. **Use `next/og`** - not `@vercel/og` (it's built into Next.js)
2. **No searchParams** - OG images can't access search params, use route params instead
3. **Avoid Edge runtime** - Use default Node.js runtime

```tsx
// Good
import { ImageResponse } from 'next/og'

// Bad
// import { ImageResponse } from '@vercel/og'
// export const runtime = 'edge'
```

#### Basic OG Image

```tsx
// app/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const alt = 'Site Name'
export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

export default function Image() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 128,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        Hello World
      </div>
    ),
    { ...size }
  )
}
```

#### Dynamic OG Image

```tsx
// app/blog/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const alt = 'Blog Post'
export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

type Props = { params: Promise<{ slug: string }> }

export default async function Image({ params }: Props) {
  const { slug } = await params
  const post = await getPost(slug)

  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 48,
          background: 'linear-gradient(to bottom, #1a1a1a, #333)',
          color: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          flexDirection: 'column',
          alignItems: 'center',
          justifyContent: 'center',
          padding: 48,
        }}
      >
        <div style={{ fontSize: 64, fontWeight: 'bold' }}>{post.title}</div>
        <div style={{ marginTop: 24, opacity: 0.8 }}>{post.description}</div>
      </div>
    ),
    { ...size }
  )
}
```

#### Custom Fonts

```tsx
import { ImageResponse } from 'next/og'
import { join } from 'path'
import { readFile } from 'fs/promises'

export default async function Image() {
  const fontPath = join(process.cwd(), 'assets/fonts/Inter-Bold.ttf')
  const fontData = await readFile(fontPath)

  return new ImageResponse(
    (
      <div style={{ fontFamily: 'Inter', fontSize: 64 }}>
        Custom Font Text
      </div>
    ),
    {
      width: 1200,
      height: 630,
      fonts: [{ name: 'Inter', data: fontData, style: 'normal' }],
    }
  )
}
```

#### File Naming

- `opengraph-image.tsx` - Open Graph (Facebook, LinkedIn)
- `twitter-image.tsx` - Twitter/X cards (optional, falls back to OG)

#### Styling Notes

ImageResponse uses Flexbox layout:
- Use `display: 'flex'`
- No CSS Grid support
- Styles must be inline objects

#### Multiple OG Images

Use `generateImageMetadata` for multiple images per route:

```tsx
// app/blog/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export async function generateImageMetadata({ params }) {
  const images = await getPostImages(params.slug)
  return images.map((img, idx) => ({
    id: idx,
    alt: img.alt,
    size: { width: 1200, height: 630 },
    contentType: 'image/png',
  }))
}

export default async function Image({ params, id }) {
  const images = await getPostImages(params.slug)
  const image = images[id]
  return new ImageResponse(/* ... */)
}
```

#### Multiple Sitemaps

Use `generateSitemaps` for large sites:

```tsx
// app/sitemap.ts
import type { MetadataRoute } from 'next'

export async function generateSitemaps() {
  // Return array of sitemap IDs
  return [{ id: 0 }, { id: 1 }, { id: 2 }]
}

export default async function sitemap({
  id,
}: {
  id: number
}): Promise<MetadataRoute.Sitemap> {
  const start = id * 50000
  const end = start + 50000
  const products = await getProducts(start, end)

  return products.map((product) => ({
    url: `https://example.com/product/${product.id}`,
    lastModified: product.updatedAt,
  }))
}
```

Generates `/sitemap/0.xml`, `/sitemap/1.xml`, etc.

---

## Parallel Routes

Parallel routes render multiple pages in the same layout. Intercepting routes show a different UI when navigating from within your app vs direct URL access. Together they enable modal patterns.

### File Structure

```
app/
├── @modal/                    # Parallel route slot
│   ├── default.tsx            # Required! Returns null
│   ├── (.)photos/             # Intercepts /photos/*
│   │   └── [id]/
│   │       └── page.tsx       # Modal content
│   └── [...]catchall/         # Optional: catch unmatched
│       └── page.tsx
├── photos/
│   └── [id]/
│       └── page.tsx           # Full page (direct access)
├── layout.tsx                 # Renders both children and @modal
└── page.tsx
```

### Step 1: Root Layout with Slot

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html>
      <body>
        {children}
        {modal}
      </body>
    </html>
  );
}
```

### Step 2: Default File (Critical!)

**Every parallel route slot MUST have a `default.tsx`** to prevent 404s on hard navigation.

```tsx
// app/@modal/default.tsx
export default function Default() {
  return null;
}
```

Without this file, refreshing any page will 404 because Next.js can't determine what to render in the `@modal` slot.

### Step 3: Intercepting Route (Modal)

The `(.)` prefix intercepts routes at the same level.

```tsx
// app/@modal/(.)photos/[id]/page.tsx
import { Modal } from '@/components/modal';

export default async function PhotoModal({
  params
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params;
  const photo = await getPhoto(id);

  return (
    <Modal>
      <img src={photo.url} alt={photo.title} />
    </Modal>
  );
}
```

### Step 4: Full Page (Direct Access)

```tsx
// app/photos/[id]/page.tsx
export default async function PhotoPage({
  params
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params;
  const photo = await getPhoto(id);

  return (
    <div className="full-page">
      <img src={photo.url} alt={photo.title} />
      <h1>{photo.title}</h1>
    </div>
  );
}
```

### Step 5: Modal Component with Correct Closing

**Critical: Use `router.back()` to close modals, NOT `router.push()` or `<Link>`.**

```tsx
// components/modal.tsx
'use client';

import { useRouter } from 'next/navigation';
import { useCallback, useEffect, useRef } from 'react';

export function Modal({ children }: { children: React.ReactNode }) {
  const router = useRouter();
  const overlayRef = useRef<HTMLDivElement>(null);

  // Close on escape key
  useEffect(() => {
    function onKeyDown(e: KeyboardEvent) {
      if (e.key === 'Escape') {
        router.back(); // Correct
      }
    }
    document.addEventListener('keydown', onKeyDown);
    return () => document.removeEventListener('keydown', onKeyDown);
  }, [router]);

  // Close on overlay click
  const handleOverlayClick = useCallback((e: React.MouseEvent) => {
    if (e.target === overlayRef.current) {
      router.back(); // Correct
    }
  }, [router]);

  return (
    <div
      ref={overlayRef}
      onClick={handleOverlayClick}
      className="fixed inset-0 bg-black/50 flex items-center justify-center z-50"
    >
      <div className="bg-white rounded-lg p-6 max-w-2xl w-full mx-4">
        <button
          onClick={() => router.back()} // Correct!
          className="absolute top-4 right-4"
        >
          Close
        </button>
        {children}
      </div>
    </div>
  );
}
```

#### Why NOT `router.push('/')` or `<Link href="/">`?

Using `push` or `Link` to "close" a modal:
1. Adds a new history entry (back button shows modal again)
2. Doesn't properly clear the intercepted route
3. Can cause the modal to flash or persist unexpectedly

`router.back()` correctly:
1. Removes the intercepted route from history
2. Returns to the previous page
3. Properly unmounts the modal

### Route Matcher Reference

Matchers match **route segments**, not filesystem paths:

| Matcher | Matches | Example |
|---------|---------|---------|
| `(.)` | Same level | `@modal/(.)photos` intercepts `/photos` |
| `(..)` | One level up | `@modal/(..)settings` from `/dashboard/@modal` intercepts `/settings` |
| `(..)(..)` | Two levels up | Rarely used |
| `(...)` | From root | `@modal/(...)photos` intercepts `/photos` from anywhere |

**Common mistake**: Thinking `(..)` means "parent folder" - it means "parent route segment".

### Handling Hard Navigation

When users directly visit `/photos/123` (bookmark, refresh, shared link):
- The intercepting route is bypassed
- The full `photos/[id]/page.tsx` renders
- Modal doesn't appear (expected behavior)

If you want the modal to appear on direct access too, you need additional logic:

```tsx
// app/photos/[id]/page.tsx
import { Modal } from '@/components/modal';

export default async function PhotoPage({ params }) {
  const { id } = await params;
  const photo = await getPhoto(id);

  // Option: Render as modal on direct access too
  return (
    <Modal>
      <img src={photo.url} alt={photo.title} />
    </Modal>
  );
}
```

### Common Gotchas

#### 1. Missing `default.tsx` -> 404 on Refresh

Every `@slot` folder needs a `default.tsx` that returns `null` (or appropriate content).

#### 2. Modal Persists After Navigation

You're using `router.push()` instead of `router.back()`.

#### 3. Nested Parallel Routes Need Defaults Too

If you have `@modal` inside a route group, each level needs its own `default.tsx`:

```
app/
├── (marketing)/
│   ├── @modal/
│   │   └── default.tsx     # Needed!
│   └── layout.tsx
└── layout.tsx
```

#### 4. Intercepted Route Shows Wrong Content

Check your matcher:
- `(.)photos` intercepts `/photos` from the same route level
- If your `@modal` is in `app/dashboard/@modal`, use `(.)photos` to intercept `/dashboard/photos`, not `/photos`

#### 5. TypeScript Errors with `params`

In Next.js 15+, `params` is a Promise:

```tsx
// Correct
export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
}
```

### Complete Example: Photo Gallery Modal

```
app/
├── @modal/
│   ├── default.tsx
│   └── (.)photos/
│       └── [id]/
│           └── page.tsx
├── photos/
│   ├── page.tsx           # Gallery grid
│   └── [id]/
│       └── page.tsx       # Full photo page
├── layout.tsx
└── page.tsx
```

Links in the gallery:

```tsx
// app/photos/page.tsx
import Link from 'next/link';

export default async function Gallery() {
  const photos = await getPhotos();

  return (
    <div className="grid grid-cols-3 gap-4">
      {photos.map(photo => (
        <Link key={photo.id} href={`/photos/${photo.id}`}>
          <img src={photo.thumbnail} alt={photo.title} />
        </Link>
      ))}
    </div>
  );
}
```

Clicking a photo -> Modal opens (intercepted)
Direct URL -> Full page renders
Refresh while modal open -> Full page renders

---

## Route Handlers

Create API endpoints with `route.ts` files.

### Basic Usage

```tsx
// app/api/users/route.ts
export async function GET() {
  const users = await getUsers()
  return Response.json(users)
}

export async function POST(request: Request) {
  const body = await request.json()
  const user = await createUser(body)
  return Response.json(user, { status: 201 })
}
```

### Supported Methods

`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`

### GET Handler Conflicts with page.tsx

**A `route.ts` and `page.tsx` cannot coexist in the same folder.**

```
app/
├── api/
│   └── users/
│       └── route.ts    # /api/users
└── users/
    ├── page.tsx        # /users (page)
    └── route.ts        # Warning: Conflicts with page.tsx!
```

If you need both a page and an API at the same path, use different paths:

```
app/
├── users/
│   └── page.tsx        # /users (page)
└── api/
    └── users/
        └── route.ts    # /api/users (API)
```

### Environment Behavior

Route handlers run in a **Server Component-like environment**:

- Yes: Can use `async/await`
- Yes: Can access `cookies()`, `headers()`
- Yes: Can use Node.js APIs
- No: Cannot use React hooks
- No: Cannot use React DOM APIs
- No: Cannot use browser APIs

```tsx
// Bad: This won't work - no React DOM in route handlers
import { renderToString } from 'react-dom/server'

export async function GET() {
  const html = renderToString(<Component />)  // Error!
  return new Response(html)
}
```

### Dynamic Route Handlers

```tsx
// app/api/users/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  const user = await getUser(id)

  if (!user) {
    return Response.json({ error: 'Not found' }, { status: 404 })
  }

  return Response.json(user)
}
```

### Request Helpers

```tsx
export async function GET(request: Request) {
  // URL and search params
  const { searchParams } = new URL(request.url)
  const query = searchParams.get('q')

  // Headers
  const authHeader = request.headers.get('authorization')

  // Cookies (Next.js helper)
  const cookieStore = await cookies()
  const token = cookieStore.get('token')

  return Response.json({ query, token })
}
```

### Response Helpers

```tsx
// JSON response
return Response.json({ data })

// With status
return Response.json({ error: 'Not found' }, { status: 404 })

// With headers
return Response.json(data, {
  headers: {
    'Cache-Control': 'max-age=3600',
  },
})

// Redirect
return Response.redirect(new URL('/login', request.url))

// Stream
return new Response(stream, {
  headers: { 'Content-Type': 'text/event-stream' },
})
```

### When to Use Route Handlers vs Server Actions

| Use Case | Route Handlers | Server Actions |
|----------|----------------|----------------|
| Form submissions | No | Yes |
| Data mutations from UI | No | Yes |
| Third-party webhooks | Yes | No |
| External API consumption | Yes | No |
| Public REST API | Yes | No |
| File uploads | Both work | Both work |

**Prefer Server Actions** for mutations triggered from your UI.
**Use Route Handlers** for external integrations and public APIs.

---

## Runtime Selection

### Use Node.js Runtime by Default

Use the default Node.js runtime for new routes and pages. Only use Edge runtime if the project already uses it or there's a specific requirement.

```tsx
// Good: Default - no runtime config needed (uses Node.js)
export default function Page() { ... }

// Caution: Only if already used in project or specifically required
export const runtime = 'edge'
```

### When to Use Each

#### Node.js Runtime (Default)

- Full Node.js API support
- File system access (`fs`)
- Full `crypto` support
- Database connections
- Most npm packages work

#### Edge Runtime

- Only for specific edge-location latency requirements
- Limited API (no `fs`, limited `crypto`)
- Smaller cold start
- Geographic distribution needs

### Detection

**Before adding `runtime = 'edge'`**, check:
1. Does the project already use Edge runtime?
2. Is there a specific latency requirement?
3. Are all dependencies Edge-compatible?

If unsure, use Node.js runtime.

---

## Scripts

Loading third-party scripts in Next.js.

### Use next/script

Always use `next/script` instead of native `<script>` tags for better performance.

```tsx
// Bad: Native script tag
<script src="https://example.com/script.js"></script>

// Good: Next.js Script component
import Script from 'next/script'

<Script src="https://example.com/script.js" />
```

### Inline Scripts Need ID

Inline scripts require an `id` attribute for Next.js to track them.

```tsx
// Bad: Missing id
<Script dangerouslySetInnerHTML={{ __html: 'console.log("hi")' }} />

// Good: Has id
<Script id="my-script" dangerouslySetInnerHTML={{ __html: 'console.log("hi")' }} />

// Good: Inline with id
<Script id="show-banner">
  {`document.getElementById('banner').classList.remove('hidden')`}
</Script>
```

### Don't Put Script in Head

`next/script` should not be placed inside `next/head`. It handles its own positioning.

```tsx
// Bad: Script inside Head
import Head from 'next/head'
import Script from 'next/script'

<Head>
  <Script src="/analytics.js" />
</Head>

// Good: Script outside Head
<Head>
  <title>Page</title>
</Head>
<Script src="/analytics.js" />
```

### Loading Strategies

```tsx
// afterInteractive (default) - Load after page is interactive
<Script src="/analytics.js" strategy="afterInteractive" />

// lazyOnload - Load during idle time
<Script src="/widget.js" strategy="lazyOnload" />

// beforeInteractive - Load before page is interactive (use sparingly)
// Only works in app/layout.tsx or pages/_document.js
<Script src="/critical.js" strategy="beforeInteractive" />

// worker - Load in web worker (experimental)
<Script src="/heavy.js" strategy="worker" />
```

### Google Analytics

Use `@next/third-parties` instead of inline GA scripts.

```tsx
// Bad: Inline GA script
<Script src="https://www.googletagmanager.com/gtag/js?id=G-XXXXX" />
<Script id="ga-init">
  {`window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', 'G-XXXXX');`}
</Script>

// Good: Next.js component
import { GoogleAnalytics } from '@next/third-parties/google'

export default function Layout({ children }) {
  return (
    <html>
      <body>{children}</body>
      <GoogleAnalytics gaId="G-XXXXX" />
    </html>
  )
}
```

### Google Tag Manager

```tsx
import { GoogleTagManager } from '@next/third-parties/google'

export default function Layout({ children }) {
  return (
    <html>
      <GoogleTagManager gtmId="GTM-XXXXX" />
      <body>{children}</body>
    </html>
  )
}
```

### Other Third-Party Scripts

```tsx
// YouTube embed
import { YouTubeEmbed } from '@next/third-parties/google'

<YouTubeEmbed videoid="dQw4w9WgXcQ" />

// Google Maps
import { GoogleMapsEmbed } from '@next/third-parties/google'

<GoogleMapsEmbed
  apiKey="YOUR_API_KEY"
  mode="place"
  q="Brooklyn+Bridge,New+York,NY"
/>
```

### Quick Reference

| Pattern | Issue | Fix |
|---------|-------|-----|
| `<script src="...">` | No optimization | Use `next/script` |
| `<Script>` without id | Can't track inline scripts | Add `id` attribute |
| `<Script>` inside `<Head>` | Wrong placement | Move outside Head |
| Inline GA/GTM scripts | No optimization | Use `@next/third-parties` |
| `strategy="beforeInteractive"` outside layout | Won't work | Only use in root layout |

---

## Self-Hosting

Deploy Next.js outside of Vercel with confidence.

### Quick Start: Standalone Output

For Docker or any containerized deployment, use standalone output:

```js
// next.config.js
module.exports = {
  output: 'standalone',
};
```

This creates a minimal `standalone` folder with only production dependencies:

```
.next/
├── standalone/
│   ├── server.js          # Entry point
│   ├── node_modules/      # Only production deps
│   └── .next/             # Build output
└── static/                # Must be copied separately
```

### Docker Deployment

#### Dockerfile

```dockerfile
FROM node:20-alpine AS base

# Install dependencies
FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci

# Build
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production

# Create non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy standalone output
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public

USER nextjs

EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]
```

#### Docker Compose

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### PM2 Deployment

For traditional server deployments:

```js
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'nextjs',
    script: '.next/standalone/server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
  }],
};
```

```bash
npm run build
pm2 start ecosystem.config.js
```

### ISR and Cache Handlers

#### The Problem

ISR (Incremental Static Regeneration) uses filesystem caching by default. This **breaks with multiple instances**:

- Instance A regenerates page -> saves to its local disk
- Instance B serves stale page -> doesn't see Instance A's cache
- Load balancer sends users to random instances -> inconsistent content

#### Solution: Custom Cache Handler

Next.js 14+ supports custom cache handlers for shared storage:

```js
// next.config.js
module.exports = {
  cacheHandler: require.resolve('./cache-handler.js'),
  cacheMaxMemorySize: 0, // Disable in-memory cache
};
```

##### Redis Cache Handler Example

```js
// cache-handler.js
const Redis = require('ioredis');

const redis = new Redis(process.env.REDIS_URL);
const CACHE_PREFIX = 'nextjs:';

module.exports = class CacheHandler {
  constructor(options) {
    this.options = options;
  }

  async get(key) {
    const data = await redis.get(CACHE_PREFIX + key);
    if (!data) return null;

    const parsed = JSON.parse(data);
    return {
      value: parsed.value,
      lastModified: parsed.lastModified,
    };
  }

  async set(key, data, ctx) {
    const cacheData = {
      value: data,
      lastModified: Date.now(),
    };

    // Set TTL based on revalidate option
    if (ctx?.revalidate) {
      await redis.setex(
        CACHE_PREFIX + key,
        ctx.revalidate,
        JSON.stringify(cacheData)
      );
    } else {
      await redis.set(CACHE_PREFIX + key, JSON.stringify(cacheData));
    }
  }

  async revalidateTag(tags) {
    // Implement tag-based invalidation
    // This requires tracking which keys have which tags
  }
};
```

##### S3 Cache Handler Example

```js
// cache-handler.js
const { S3Client, GetObjectCommand, PutObjectCommand } = require('@aws-sdk/client-s3');

const s3 = new S3Client({ region: process.env.AWS_REGION });
const BUCKET = process.env.CACHE_BUCKET;

module.exports = class CacheHandler {
  async get(key) {
    try {
      const response = await s3.send(new GetObjectCommand({
        Bucket: BUCKET,
        Key: `cache/${key}`,
      }));
      const body = await response.Body.transformToString();
      return JSON.parse(body);
    } catch (err) {
      if (err.name === 'NoSuchKey') return null;
      throw err;
    }
  }

  async set(key, data, ctx) {
    await s3.send(new PutObjectCommand({
      Bucket: BUCKET,
      Key: `cache/${key}`,
      Body: JSON.stringify({
        value: data,
        lastModified: Date.now(),
      }),
      ContentType: 'application/json',
    }));
  }
};
```

### What Works vs What Needs Setup

| Feature | Single Instance | Multi-Instance | Notes |
|---------|----------------|----------------|-------|
| SSR | Yes | Yes | No special setup |
| SSG | Yes | Yes | Built at deploy time |
| ISR | Yes | Needs cache handler | Filesystem cache breaks |
| Image Optimization | Yes | Yes | CPU-intensive, consider CDN |
| Middleware | Yes | Yes | Runs on Node.js |
| Edge Runtime | Limited | Limited | Some features Node-only |
| `revalidatePath/Tag` | Yes | Needs cache handler | Must share cache |
| `next/font` | Yes | Yes | Fonts bundled at build |
| Draft Mode | Yes | Yes | Cookie-based |

### Image Optimization

Next.js Image Optimization works out of the box but is CPU-intensive.

#### Option 1: Built-in (Simple)

Works automatically, but consider:
- Set `deviceSizes` and `imageSizes` in config to limit variants
- Use `minimumCacheTTL` to reduce regeneration

```js
// next.config.js
module.exports = {
  images: {
    minimumCacheTTL: 60 * 60 * 24, // 24 hours
    deviceSizes: [640, 750, 1080, 1920], // Limit sizes
  },
};
```

#### Option 2: External Loader (Recommended for Scale)

Offload to Cloudinary, Imgix, or similar:

```js
// next.config.js
module.exports = {
  images: {
    loader: 'custom',
    loaderFile: './lib/image-loader.js',
  },
};
```

```js
// lib/image-loader.js
export default function cloudinaryLoader({ src, width, quality }) {
  const params = ['f_auto', 'c_limit', `w_${width}`, `q_${quality || 'auto'}`];
  return `https://res.cloudinary.com/demo/image/upload/${params.join(',')}${src}`;
}
```

### Environment Variables

#### Build-time vs Runtime

```js
// Available at build time only (baked into bundle)
NEXT_PUBLIC_API_URL=https://api.example.com

// Available at runtime (server-side only)
DATABASE_URL=postgresql://...
API_SECRET=...
```

#### Runtime Configuration

For truly dynamic config, don't use `NEXT_PUBLIC_*`. Instead:

```tsx
// app/api/config/route.ts
export async function GET() {
  return Response.json({
    apiUrl: process.env.API_URL,
    features: process.env.FEATURES?.split(','),
  });
}
```

### OpenNext: Serverless Without Vercel

[OpenNext](https://open-next.js.org/) adapts Next.js for AWS Lambda, Cloudflare Workers, etc.

```bash
npx create-sst@latest
# or
npx @opennextjs/aws build
```

Supports:
- AWS Lambda + CloudFront
- Cloudflare Workers
- Netlify Functions
- Deno Deploy

### Health Check Endpoint

Always include a health check for load balancers:

```tsx
// app/api/health/route.ts
export async function GET() {
  try {
    // Optional: check database connection
    // await db.$queryRaw`SELECT 1`;

    return Response.json({ status: 'healthy' }, { status: 200 });
  } catch (error) {
    return Response.json({ status: 'unhealthy' }, { status: 503 });
  }
}
```

### Pre-Deployment Checklist

1. **Build locally first**: `npm run build` - catch errors before deploy
2. **Test standalone output**: `node .next/standalone/server.js`
3. **Set `output: 'standalone'`** for Docker
4. **Configure cache handler** for multi-instance ISR
5. **Set `HOSTNAME="0.0.0.0"`** for containers
6. **Copy `public/` and `.next/static/`** - not included in standalone
7. **Add health check endpoint**
8. **Test ISR revalidation** after deployment
9. **Monitor memory usage** - Node.js defaults may need tuning

### Testing Cache Handler

**Critical**: Test your cache handler on every Next.js upgrade:

```bash
# Start multiple instances
PORT=3001 node .next/standalone/server.js &
PORT=3002 node .next/standalone/server.js &

# Trigger ISR revalidation
curl http://localhost:3001/api/revalidate?path=/posts

# Verify both instances see the update
curl http://localhost:3001/posts
curl http://localhost:3002/posts
# Should return identical content
```

---

## Suspense Boundaries

Client hooks that cause CSR bailout without Suspense boundaries.

### useSearchParams

Always requires Suspense boundary in static routes. Without it, the entire page becomes client-side rendered.

```tsx
// Bad: Entire page becomes CSR
'use client'

import { useSearchParams } from 'next/navigation'

export default function SearchBar() {
  const searchParams = useSearchParams()
  return <div>Query: {searchParams.get('q')}</div>
}
```

```tsx
// Good: Wrap in Suspense
import { Suspense } from 'react'
import SearchBar from './search-bar'

export default function Page() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <SearchBar />
    </Suspense>
  )
}
```

### usePathname

Requires Suspense boundary when route has dynamic parameters.

```tsx
// In dynamic route [slug]
// Bad: No Suspense
'use client'
import { usePathname } from 'next/navigation'

export function Breadcrumb() {
  const pathname = usePathname()
  return <nav>{pathname}</nav>
}
```

```tsx
// Good: Wrap in Suspense
<Suspense fallback={<BreadcrumbSkeleton />}>
  <Breadcrumb />
</Suspense>
```

If you use `generateStaticParams`, Suspense is optional.

### Quick Reference

| Hook | Suspense Required |
|------|-------------------|
| `useSearchParams()` | Yes |
| `usePathname()` | Yes (dynamic routes) |
| `useParams()` | No |
| `useRouter()` | No |
