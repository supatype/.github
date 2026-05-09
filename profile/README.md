<p align="center">
  <img src="https://raw.githubusercontent.com/supatype/.github/refs/heads/main/profile/supatype-icon.svg" width="80" alt="Supatype" />
</p>

<h1 align="center">Supatype</h1>

<p align="center"><strong>Define your types. We generate your backend platform.</strong></p>

Supatype is a schema-first platform for frontend engineers. Write TypeScript data models and get a complete, production-ready backend: Postgres database, REST API, row-level security, auth, admin panel, realtime subscriptions, storage, and edge functions. All generated. All type-safe.

```ts
// schema/index.ts
import type {
  LoggedIn,
  Model,
  OwnerFrom,
  Public,
  RelatedTo,
  RichText,
  Role,
  SupatypeAuthUser,
  Timestamp,
  UUID,
} from '@supatype/types'

export type Post = Model<
  {
    id: UUID
    title: string
    body: RichText
    published: boolean
    authUser: RelatedTo<SupatypeAuthUser>
    created_at: Timestamp
    updated_at: Timestamp
  },
  {
    access: {
      read: Public
      create: LoggedIn
      update: OwnerFrom<'authUser'>
      delete: Role<'admin'>
    }
  }
>
```

```bash
npx supatype push   # applies migrations, generates types, updates RLS
```

That's it. Your Postgres schema, PostgREST API, generated TypeScript types, and admin panel are live.

---

## How it works

```
TypeScript model types (Model<…>)
        │
        ▼
  ┌─────────────────┐
  │  Schema Engine  │  (Rust — closed source)
  │  Parser → Diff  │
  │  → SQL + Types  │
  └─────────────────┘
        │
        ├── SQL migrations + rollbacks
        ├── TypeScript types + client SDK
        ├── Row-level security policies
        ├── Admin panel configuration
        ├── PostgREST + gateway / routing config (where applicable)
        └── Realtime trigger functions
```

The schema engine is the core of Supatype — a Rust binary that introspects your Postgres database, computes a safe diff against your schema, and generates everything downstream. It ships as a compiled binary; your schema is always the source of truth.

---

## Features

**Schema-driven everything**
Export `Model<…>` types and storage `Bucket<…>` types from a single module. The database, API, generated types, policies, and admin UI are all derived from them and stay in sync.

**Type-safe client SDK**
`@supatype/client` wraps PostgREST with full TypeScript inference. Query, mutate, subscribe — all typed end-to-end from your schema.

```ts
const { data } = await client.from('post').select('*').eq('published', true)
// data is Post[] — typed from your schema
```

**Row-level security from access rules**
Declarative access control in your schema compiles to Postgres RLS policies. Security lives at the database, not in middleware.

```ts
// second type argument on Model<Fields, { access: … }>
access: {
  read: Public,
  create: LoggedIn, // enforced in Postgres, not just your API
  update: OwnerFrom<'authUser'>,
  delete: Role<'editor'>,
}
```

**Auto-generated admin panel**
`@supatype/studio` provides a full CMS-style admin interface — content management, data explorer, SQL runner, auth management, storage browser, migration history, logs, and API docs — all generated from your schema. No configuration required.

**Realtime subscriptions**
WebSocket-based live queries, RLS-aware, with React hooks:

```ts
const posts = useQuery('post', { filter: { published: true } })
// updates live as data changes
```

**Auth built in**
JWT-based auth is served by **`supatype-server`** on the same port as the rest of the API (`/auth/v1/…`). `client.auth.signUp()`, `client.auth.signInWithPassword()`, OAuth flows, and the `useAuth()` React hook — wired from day one.

**Storage**
S3-compatible object storage with image transforms, signed URLs, and schema-defined buckets. Image and file fields in your schema automatically provision storage buckets with matching RLS.

**Edge functions**
Deno-based edge functions via `supatype functions deploy`. Runs alongside your other services, accessible from the same API gateway.

**AI-powered schema generation** *(Phase 15)*

```bash
npx supatype ai init "a multi-tenant SaaS with teams, projects, and comments"
```

Generates a complete, production-ready schema with models, relations, access rules, and composites.

**MCP server** *(Phase 15.5)*
Expose your project as an MCP-compliant server. Claude, Cursor, and other AI agents get type-safe, permission-aware tools for CRUD and custom actions — zero additional configuration.

**Schema-aware branching** *(Phase 22)*

```bash
npx supatype branch create feature/add-tags
```

Forks your database, applies schema changes in isolation, reports data-aware migration impact. On cloud, every PR gets a live preview environment.

---

## Repositories


| Repo                                             | Description                                                            | Visibility |
| ------------------------------------------------ | ---------------------------------------------------------------------- | ---------- |
| [supatype](https://github.com/supatype/supatype) | TypeScript monorepo — `@supatype/types` (`Model<…>`), client, React hooks, CLI, studio | Public     |
| `supatype-schema-engine`                         | Rust schema engine — parser, differ, SQL/type/RLS generator            | Private    |
| `supatype-server`                                | Go **unified gateway** — PostgREST, auth (`/auth/v1/`), storage, realtime, and related routes on one process | Public     |
| `supatype-postgres`                              | Hardened Postgres distribution with pg_guard and extensions            | Public     |


---

## Getting started

**Local development (CLI)**

Local dev is driven by the **`supatype` CLI** (`pnpm supatype` in the monorepo, or install **`@supatype/cli`** globally). You do **not** need to run a root-level `docker-compose` stack by hand: `supatype dev` starts Postgres (Docker **or** native per `supatype.config.ts`), pulls the engine/server binaries, applies your schema, and runs **`supatype-server`** as the unified gateway (REST, auth, storage, realtime).

```bash
npm install -g @supatype/cli
supatype init my-app
cd my-app
# add a package.json with @supatype/cli + @supatype/types, then install deps
supatype dev        # Postgres + supatype-server (+ optional Studio when configured)
```

**Install the client**

```bash
npm install @supatype/types @supatype/client @supatype/react
```

```ts
import { createClient } from '@supatype/client'
import type { Database } from './types/supatype'  // generated

export const client = createClient<Database>({
  url: process.env.NEXT_PUBLIC_SUPATYPE_URL!,
  anonKey: process.env.NEXT_PUBLIC_SUPATYPE_ANON_KEY!,
})
```

**Cloud**

Coming soon.

---

## Pricing


| Plan           | Price   | For                              |
| -------------- | ------- | -------------------------------- |
| **Starter**    | TBC  | Hobby and prototype projects     |
| **Pro**        | TBC  | Production apps and small teams  |
| **Team**       | TBC | Growing teams, multiple projects |
| **Enterprise** | Custom  | SOC 2, HIPAA, dedicated support  |


Self-hosting is always free.

---

## Self-hosting

Supatype runs entirely on-premises. No cloud dependency, no licence keys for the open-source packages.

```bash
supatype self-host install   # installs systemd services
supatype self-host serve     # starts Postgres + server
```

The schema engine binary is distributed separately and requires a licence for production self-hosted deployments at scale. Community (development) use is free.

---

## Philosophy

**Schema is source of truth.** The database is derived, never the master copy. You define types; infrastructure follows.

**Use battle-tested components.** Supatype orchestrates PostgREST, storage (S3-compatible), Deno for edge functions, and related pieces behind **`supatype-server`** — it doesn't reinvent SQL or your database. The value is in the schema engine and the integration layer.

**Escape hatches everywhere.** Raw SQL, custom migrations, manual PostgREST config, direct Postgres access — the platform never fights you.

**Type safety end-to-end.** Schema → migrations → API → client SDK → React hooks. No runtime surprises.

---

## Status

Supatype is in active development. Core schema engine, client SDK, studio, realtime, storage, auth, and CLI are production-ready. Cloud platform is in early access.

[Discord](https://discord.gg/supatype)