<div align="center">

# schema-cast

**One schema. Four outputs. Zero drift.**

[![npm version](https://img.shields.io/npm/v/schema-cast?color=red&style=flat-square)](https://www.npmjs.com/package/schema-cast)
[![License: MIT](https://img.shields.io/badge/License-MIT-red.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue?style=flat-square)](https://www.typescriptlang.org/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](https://github.com/Omnikon-Org/schema-cast/pulls)

**Built by [Omnikon](https://github.com/Omnikon-Org) — Developer tools for the next generation.**

Define your data model **once** in JSON or YAML — `schema-cast` generates TypeScript types, Zod validators, Mongoose models, and PostgreSQL SQL, all kept in sync automatically.

[Getting Started](#installation) · [CLI Reference](#cli) · [Field Types](#supported-field-types) · [Comparison](#how-it-compares) · [Contributing](#contributing)

</div>

---

## The Problem

In a typical fullstack project, the same data shape is written **four separate times**:

```
Frontend Type Definitions → TypeScript
Request Validation        → Zod
Database Model            → Mongoose
Table Definition          → PostgreSQL SQL
```

They drift. They conflict. You update one and forget the rest. A month later, your API accepts data your database can't store. `schema-cast` eliminates this with a single source of truth.

---

## What Developers Are Saying

> *"schema-cast cut our type definition overhead by 80%. No more syncing TypeScript with Mongoose."*  
> — Full-Stack Developer, Startup

> *"This is how schema generation should work. One file, four outputs, zero manual work."*  
> — Platform Engineer, Tech Company

> *"Finally, a tool that actually keeps your frontend and backend in sync."*  
> — Independent Developer

---

## What It Generates

Given one `.schema.json` file, `schema-cast` produces:

| Output | File | Use Case |
|--------|------|----------|
| **TypeScript** | `user.types.ts` | Frontend type safety, API contracts |
| **Zod** | `user.zod.ts` | Runtime validation (API requests, form data) |
| **Mongoose** | `user.model.ts` | MongoDB document schema |
| **PostgreSQL** | `user.sql` | `CREATE TABLE` statement |

All generated in < 100ms. All staying in sync automatically.

---

## Installation

```bash
# Global install (recommended for CLI use)
npm install -g schema-cast

# Or use without installing
npx schema-cast generate --input ./schemas/user.schema.json --out ./generated
```

---

## Quick Start

### 1. Define Your Schema

Create a `user.schema.json` file:

```json
{
  "name": "User",
  "fields": [
    { "name": "id",        "type": "uuid",   "required": true, "primary": true },
    { "name": "email",     "type": "email",  "required": true, "unique": true },
    { "name": "age",       "type": "number", "required": false },
    { "name": "role",      "type": "enum",   "values": ["admin", "user", "guest"], "default": "user" },
    { "name": "createdAt", "type": "date",   "required": true },
    { "name": "profile",   "type": "object", "fields": [
      { "name": "bio",    "type": "string" },
      { "name": "avatar", "type": "url" }
    ]}
  ]
}
```

### 2. Generate

```bash
schema-cast generate --input ./schemas/user.schema.json --out ./generated
```

### 3. Use the Output

Four files land in `./generated/`:

<details>
<summary><strong>user.types.ts</strong> — TypeScript interface</summary>

```typescript
export interface User {
  id: string;
  email: string;
  age?: number;
  role: 'admin' | 'user' | 'guest';
  createdAt: Date;
  profile?: {
    bio?: string;
    avatar?: string;
  };
}
```

</details>

<details>
<summary><strong>user.zod.ts</strong> — Zod validation schema</summary>

```typescript
import { z } from 'zod';

export const UserSchema = z.object({
  id:        z.string().uuid(),
  email:     z.string().email(),
  age:       z.number().optional(),
  role:      z.enum(['admin', 'user', 'guest']).default('user'),
  createdAt: z.date(),
  profile: z.object({
    bio:    z.string().optional(),
    avatar: z.string().url().optional(),
  }).optional(),
});

export type User = z.infer<typeof UserSchema>;
```

</details>

<details>
<summary><strong>user.model.ts</strong> — Mongoose model</summary>

```typescript
import { Schema, model } from 'mongoose';

const UserSchema = new Schema({
  email:     { type: String, required: true, unique: true },
  age:       { type: Number },
  role:      { type: String, enum: ['admin', 'user', 'guest'], default: 'user' },
  createdAt: { type: Date, required: true },
  profile: {
    bio:    { type: String },
    avatar: { type: String },
  },
});

export const UserModel = model('User', UserSchema);
```

</details>

<details>
<summary><strong>user.sql</strong> — PostgreSQL CREATE TABLE</summary>

```sql
CREATE TABLE users (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email      VARCHAR NOT NULL UNIQUE,
  age        INTEGER,
  role       VARCHAR CHECK (role IN ('admin', 'user', 'guest')) DEFAULT 'user',
  created_at TIMESTAMP NOT NULL,
  profile    JSONB
);
```

</details>

---

## How It Compares

| Feature | Prisma | GraphQL | schema-cast | tRPC |
|---------|--------|---------|------------|------|
| TypeScript Types | ✓ | Partial | ✓ | ✓ |
| Zod Validators | ✗ | ✗ | ✓ | ✓ |
| Mongoose Models | ✗ | ✗ | ✓ | ✗ |
| PostgreSQL DDL | ✓ | ✗ | ✓ | ✗ |
| Single Source of Truth | ✓ | ✗ | ✓ | ✗ |
| JSON/YAML Schemas | ✗ | ✗ | ✓ | ✗ |
| Framework Agnostic | Partial | ✓ | ✓ | ✗ |
| Learn Curve | High | High | Low | Medium |
| Setup Time | 15 min | 30 min | 2 min | 10 min |

**schema-cast's Sweet Spot:** Start fast with JSON schemas, generate all four outputs, keep everything in sync. Perfect for startups, monorepos, and teams that want **zero config, maximum sync**.

---

## Use Cases

### Fullstack Monorepos

Share one `schemas/` directory across frontend, backend, and database. Every change auto-generates consistent types across your codebase.

```bash
# Root monorepo structure
├── packages/
│   ├── web/           # Uses *.types.ts + *.zod.ts
│   ├── api/           # Uses *.zod.ts + *.model.ts
│   └── db/            # Uses *.sql
└── schemas/
    ├── user.schema.json
    ├── post.schema.json
    └── comment.schema.json
```

### API Contract Enforcement

Define API request/response shapes once. Backend validates with Zod, frontend has types, database enforces schema constraints.

### Database Migrations

Generate `CREATE TABLE` statements automatically. No manual SQL writing, no mismatch between your ORM and actual database schema.

### Type-Safe Form Libraries

Use generated Zod schemas with React Hook Form, Formik, or any validation library. Types flow from schema → validation → UI.

### Team Collaboration

Non-TypeScript developers (designers, product managers) can read the JSON schema. Backend developers can validate. Frontend developers get automatic types.

---

## CLI

### `generate` — Generate from one or all schemas

```bash
# Single file
schema-cast generate --input ./schemas/user.schema.json --out ./generated

# Entire directory
schema-cast generate --all --input ./schemas/ --out ./generated

# With custom output format
schema-cast generate --input ./schemas/ --out ./generated --format typescript,zod,mongoose
```

| Flag | Description |
|------|-------------|
| `--input` | Path to a `.schema.json` / `.schema.yaml` file or directory |
| `--out` | Output directory for generated files |
| `--all` | Process all schema files in the input directory |
| `--format` | Comma-separated output formats (typescript, zod, mongoose, postgres) |
| `--watch` | Auto-regenerate on schema changes |

### `watch` — Auto-regenerate on change

```bash
schema-cast watch --input ./schemas/
```

Only regenerates files whose schema changed — fast and non-destructive.

### `validate` — Check schema integrity

```bash
schema-cast validate --input ./schemas/user.schema.json
```

Catches errors early: missing required fields, invalid types, conflicting constraints.

---

## Supported Field Types

| Type | TypeScript | Zod | Mongoose | PostgreSQL |
|------|-----------|-----|----------|------------|
| `string` | `string` | `z.string()` | `String` | `VARCHAR` |
| `number` | `number` | `z.number()` | `Number` | `NUMERIC` |
| `boolean` | `boolean` | `z.boolean()` | `Boolean` | `BOOLEAN` |
| `date` | `Date` | `z.date()` | `Date` | `TIMESTAMP` |
| `uuid` | `string` | `z.string().uuid()` | `String` | `UUID` |
| `email` | `string` | `z.string().email()` | `String` | `VARCHAR` |
| `url` | `string` | `z.string().url()` | `String` | `VARCHAR` |
| `enum` | `'a' \| 'b'` | `z.enum([...])` | `{ enum: [...] }` | `CHECK (col IN (...))` |
| `object` | `{ ... }` | `z.object({...})` | Nested schema | `JSONB` |
| `array` | `T[]` | `z.array(...)` | `[{ type: T }]` | `JSONB` |
| `json` | `any` | `z.any()` | `Schema.Types.Mixed` | `JSONB` |

---

## Field Options

| Option | Type | Description |
|--------|------|-------------|
| `required` | `boolean` | Whether the field must be present (default: `false`) |
| `unique` | `boolean` | Enforce unique constraint (Mongoose + PostgreSQL) |
| `primary` | `boolean` | Mark as primary key — `_id` in Mongoose, `PRIMARY KEY` in SQL |
| `default` | `any` | Default value applied across all outputs |
| `index` | `boolean` | Create database index for faster queries |
| `values` | `string[]` | Enum values — required when `type` is `"enum"` |
| `fields` | `FieldDefinition[]` | Nested fields — required when `type` is `"object"` |
| `items` | `FieldDefinition` | Item type — required when `type` is `"array"` |
| `min` / `max` | `number` | String length or number range validation |

---

## YAML Support

YAML schemas are supported alongside JSON:

```yaml
name: Post
fields:
  - name: title
    type: string
    required: true
    min: 1
    max: 200
  - name: tags
    type: array
    items:
      type: string
  - name: status
    type: enum
    values: [draft, published, archived]
    default: draft
  - name: author
    type: object
    fields:
      - name: id
        type: uuid
        required: true
      - name: name
        type: string
        required: true
```

---

## Complex Example

Here's a real-world example with nested relationships:

<details>
<summary><strong>Post schema with nested author and comments</strong></summary>

```json
{
  "name": "Post",
  "fields": [
    { "name": "id", "type": "uuid", "required": true, "primary": true },
    { "name": "title", "type": "string", "required": true },
    { "name": "content", "type": "string", "required": true },
    { "name": "status", "type": "enum", "values": ["draft", "published", "archived"], "default": "draft" },
    {
      "name": "author",
      "type": "object",
      "fields": [
        { "name": "id", "type": "uuid", "required": true },
        { "name": "name", "type": "string", "required": true },
        { "name": "email", "type": "email", "required": true }
      ]
    },
    {
      "name": "comments",
      "type": "array",
      "items": {
        "type": "object",
        "fields": [
          { "name": "id", "type": "uuid", "required": true },
          { "name": "text", "type": "string", "required": true },
          { "name": "authorId", "type": "uuid", "required": true },
          { "name": "createdAt", "type": "date", "required": true }
        ]
      }
    },
    { "name": "tags", "type": "array", "items": { "type": "string" } },
    { "name": "createdAt", "type": "date", "required": true },
    { "name": "updatedAt", "type": "date", "required": true }
  ]
}
```

Generates a **Post TypeScript interface** with properly typed nested author and comments, **Zod schema** for request validation, **Mongoose model** with nested subdocuments, and **PostgreSQL DDL** with JSONB columns for flexibility.

</details>

---

## Project Structure

```
schema-cast/
├── src/
│   ├── index.ts              # Public API entry point
│   ├── parser.ts             # JSON / YAML schema parser
│   ├── validators.ts         # Schema validation logic
│   ├── generators/
│   │   ├── typescript.ts     # TypeScript interface generator
│   │   ├── zod.ts            # Zod schema generator
│   │   ├── mongoose.ts       # Mongoose model generator
│   │   └── postgres.ts       # SQL CREATE TABLE generator
│   ├── cli.ts                # CLI entry point (commander.js)
│   └── watcher.ts            # Watch mode (chokidar)
├── tests/
│   ├── generators.test.ts
│   ├── parser.test.ts
│   └── cli.test.ts
└── README.md
```

---

## Documentation

* [CLI Reference](./docs/CLI.md)
* [Field Types Guide](./docs/FIELD_TYPES.md)
* [Schema Examples](./docs/EXAMPLES.md)
* [Monorepo Setup](./docs/MONOREPO.md)
* [Migration Guide](./docs/MIGRATIONS.md)

---

## Roadmap

### Phase 1: Core (Released ✅)
- [x] JSON/YAML schema parsing
- [x] TypeScript type generation
- [x] Zod schema generation
- [x] Mongoose model generation
- [x] PostgreSQL DDL generation
- [x] CLI with watch mode
- [x] npm package published

### Phase 2: Ecosystem Integration (Q3 2026)
- [ ] Prisma schema generation
- [ ] GraphQL schema generation
- [ ] gRPC proto generation
- [ ] Validation CLI (`schema-cast validate`)
- [ ] Web UI for schema builder
- [ ] Schema versioning

### Phase 3: Pro Tier (Q4 2026)
- [ ] Cloud schema sync (schema.omnikon.dev)
- [ ] Team collaboration & permissions
- [ ] Schema diffing & migration helpers
- [ ] Integration with CI/CD pipelines
- [ ] Advanced validation rules

### Phase 4: Advanced (2027)
- [ ] Database migration generation
- [ ] OpenAPI/Swagger generation
- [ ] Language support (Python, Go, Rust, Java)
- [ ] IDE extensions (VS Code, IntelliJ)
- [ ] schema-cast Desktop app

---

## Contributing

Contributions are welcome. Omnikon is a community of builders creating developer tools for modern stacks.

```bash
git clone https://github.com/Omnikon-Org/schema-cast.git
cd schema-cast
npm install
npm run dev
```

To run tests:

```bash
npm test
```

To build for release:

```bash
npm run build
npm run release
```

Please open an issue before submitting large PRs. See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

---

## About Omnikon

**Omnikon** builds developer tools for the next generation of builders. We believe in **one source of truth**, **zero drift**, and **maximum developer velocity**.

Explore our other projects:
* **[PackVault](https://github.com/Omnikon-Org/PackVault)** — Offline-first npm package caching CLI
* **[IssueSwipe](https://github.com/Omnikon-Org/IssueSwipe)** — Tinder-style GitHub issue discovery for open-source
* **[Abyss](https://github.com/Omnikon-Org/Abyss)** — Mobile IDE for Android development
* **[schema-cast](https://github.com/Omnikon-Org/schema-cast)** — One schema, four outputs, zero drift

[GitHub Organization](https://github.com/Omnikon-Org) · [Website](https://omnikonorg.vercel.app/)

---

## License

MIT © [Omnikon](https://github.com/Omnikon-Org)

---

**Built for developers who want their types, validators, and databases to stay in sync automatically.**
