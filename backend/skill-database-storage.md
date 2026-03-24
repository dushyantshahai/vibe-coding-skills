# Skill: Database & Storage

```json
{
  "skill_id": "database-storage",
  "category": "Architecture & Backend",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>Database & Storage — Designing Reliable Data Models for AI Products</title>

<overview>

## What This Skill Enables
- Design a database schema that supports your app's features without needing to be rebuilt every time you add something new
- Choose the right database type (SQL, NoSQL, or Vector) for each kind of data your AI product needs
- Connect your backend routes to a real database safely and predictably

## Why It Matters for Vibe Coders
Your database is the memory of your app. If it is poorly designed, every new feature becomes a painful restructuring exercise. If the wrong database type is chosen, your AI features either won't work or will be unnecessarily slow and expensive. This skill gives you a decision framework and reusable schema patterns so you build the data layer right the first time.

## When to Use This Skill
- Before writing any route that reads or writes data
- When adding a new feature that requires storing new kinds of information
- When your queries are getting slow or your schema keeps needing changes
- When adding AI memory, semantic search, or RAG to your product

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "app_type": "[REPLACE: e.g. EdTech web app / Conversational reminder app]",
    "database_provider": "[REPLACE: e.g. Supabase / PlanetScale / MongoDB Atlas / Firebase]",
    "database_type": "[REPLACE: e.g. PostgreSQL / MySQL / MongoDB]",
    "orm_or_client": "[REPLACE: e.g. Prisma / Supabase JS client / Drizzle / SQLAlchemy]",
    "vector_db": "[REPLACE: e.g. Supabase pgvector / Pinecone / Weaviate / none yet]",
    "existing_tables_or_collections": [
      "[REPLACE: e.g. users, lessons, reminders]"
    ],
    "table_or_collection_to_design_today": "[REPLACE: ONE table/collection only]",
    "key_relationships": "[REPLACE: e.g. a user has many lessons, a lesson has many sections]",
    "ai_features_needing_storage": "[REPLACE: e.g. chat history, embeddings, generated lesson content]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About Databases

### Mental Model 1: The Filing Cabinet
Your database is a filing cabinet. Each table (or collection) is a drawer. Each row (or document) is a folder inside that drawer. A well-organised cabinet has one drawer per topic and clear labels. A poorly organised one has everything dumped in one drawer.

**Good schema design = one table per real-world concept.**  
Users in the Users table. Lessons in the Lessons table. Never mix concepts in one table to "save time."

### Mental Model 2: The Three Database Types
| Type | Analogy | Best For | Examples |
|---|---|---|---|
| **SQL (Relational)** | Spreadsheet with strict columns | Structured data with clear relationships | Users, orders, lessons, reminders |
| **NoSQL (Document)** | Folder of JSON files | Flexible, evolving data structures | User preferences, dynamic content configs |
| **Vector DB** | Library organised by meaning, not title | Semantic search, AI memory, RAG | Embeddings of lessons, chat history search |

**For most AI products:** Use SQL (PostgreSQL via Supabase) as your primary database. Add a vector store when you need semantic search or AI memory. You rarely need NoSQL if you start with SQL.

### Mental Model 3: Relationships Between Tables
| Relationship | Meaning | Example |
|---|---|---|
| **One-to-One** | One record maps to exactly one other | User → UserProfile |
| **One-to-Many** | One record maps to many others | User → many Reminders |
| **Many-to-Many** | Many records map to many others | Students ↔ Courses (via enrollment table) |

Always model relationships explicitly with foreign keys — do not store related IDs in comma-separated strings inside a single column.

</mental_models>

---

<system_design_breakdown>

## Data Layer Architecture for AI Products

```
APPLICATION LAYER (API Routes / Workflows)
           │
           ▼
┌──────────────────────────────────────────────┐
│           ORM / DATABASE CLIENT              │
│     (Prisma / Drizzle / Supabase JS)         │
│  Translates code into database queries       │
└───────────────────────┬──────────────────────┘
                        │
          ┌─────────────┼──────────────┐
          ▼             ▼              ▼
┌──────────────┐ ┌────────────┐ ┌───────────────┐
│  PRIMARY DB  │ │  VECTOR DB │ │  FILE STORAGE │
│ (PostgreSQL) │ │(pgvector / │ │ (Supabase     │
│ Users        │ │ Pinecone)  │ │  Storage /    │
│ Lessons      │ │ Embeddings │ │  S3)          │
│ Reminders    │ │ Semantic   │ │ Images, PDFs  │
│ etc.         │ │ search     │ │ Audio         │
└──────────────┘ └────────────┘ └───────────────┘
```

### Standard Tables Every AI Product Needs

```sql
-- 1. Users (always)
users: id, email, name, created_at, updated_at

-- 2. Sessions / Auth tokens (if not using Clerk/Supabase Auth)
sessions: id, user_id, token, expires_at

-- 3. AI Generations (track every LLM call)
ai_generations: id, user_id, prompt, response, model, tokens_used, created_at

-- 4. Your core domain table (app-specific)
-- EdTech: lessons, progress, topics
-- Reminder app: reminders, schedules, notification_logs
```

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Design one table at a time. Verify it exists with correct columns before building the next. -->

## Step 1 — Choose Your Database Stack
Use this decision table before writing any schema:

| Situation | Recommended Choice |
|---|---|
| Building on Next.js, want fast setup | **Supabase (PostgreSQL + Auth + Storage in one)** |
| Need full ORM control with migrations | **Prisma + PlanetScale or Supabase** |
| Python backend | **SQLAlchemy + PostgreSQL on Railway/Supabase** |
| Need offline-first mobile | **PocketBase or Firebase** |
| Need vector search for AI features | **Supabase pgvector (add-on) or Pinecone** |

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Confirm my database stack choice: [YOUR CHOICE]
List: 
1. The npm/pip packages I need to install
2. The environment variables I need to set
3. Any one-time setup steps before I can run my first query
Do not write any schema yet.
```

**Verify:** Packages installed. Environment variables set in `.env.local`. Database connection tested.

---

## Step 2 — Design the Schema for One Table
Start with the Users table (or your most central entity).

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Design the schema for the [TABLE NAME] table only.

This table needs to store: [DESCRIBE IN PLAIN ENGLISH what this table represents]
It relates to these other tables: [LIST RELATIONSHIPS]
Include: 
- Primary key (UUID preferred over auto-increment for AI products)
- Foreign keys for all relationships
- created_at and updated_at timestamps on every table
- Appropriate indexes for columns used in WHERE clauses
- Row Level Security rules if using Supabase

Return: the SQL migration file only. No application code yet.
```

**Verify:** Run the migration. Check the table exists in your database dashboard with all expected columns.

---

## Step 3 — Generate the ORM Model / Type
After the table exists in the database.

**Prompt your AI agent:**
```
The [TABLE NAME] table now exists with this schema: [PASTE SCHEMA]
Generate:
1. The Prisma model (or Drizzle schema / SQLAlchemy model) for this table
2. A TypeScript type (or Python Pydantic model) matching this table
3. A basic CRUD helper file: create, findById, findByUserId, update, delete
Each function must handle errors and return null (not throw) when a record is not found.
```

**Verify:** Import the helper in a test file and call `create` with test data. Check the database — the record should exist.

---

## Step 4 — Add Vector Storage (If AI Memory / Search Needed)
Only after core tables are working.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
I need to add vector storage for: [DESCRIBE USE CASE — e.g. semantic search over lessons, storing chat message embeddings]
Vector DB choice: [pgvector / Pinecone / other]

Generate:
1. The schema addition (new table or column) for storing embeddings
2. A utility function: generateEmbedding(text: string) → vector
3. A utility function: semanticSearch(query: string, userId: string, limit: number) → results[]
Use [AI_PROVIDER] embeddings model: [e.g. text-embedding-3-small]
```

**Verify:** Generate one test embedding. Store it. Run a semantic search query. Confirm results are returned.

---

## Step 5 — Set Up Row Level Security (Supabase Only)
Critical — without this, any authenticated user can read any other user's data.

**Prompt your AI agent:**
```
I am using Supabase. Add Row Level Security (RLS) policies for the [TABLE NAME] table.
Rules:
- Users can only SELECT their own rows (where user_id = auth.uid())
- Users can only INSERT rows where user_id = auth.uid()
- Users can only UPDATE and DELETE their own rows
- Service role (backend) can bypass RLS for admin operations
Return the SQL for these policies only.
```

**Verify:** Test with two different user accounts. Confirm User A cannot read User B's data.

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am working on the database layer for my app.
Project context: [PASTE context_anchor JSON]
Existing tables already created: [LIST TABLES]
Today I am designing: [ONE TABLE OR FEATURE ONLY]
Before writing anything, confirm the exact file paths you will create or modify.
```

### Schema Design Prompt
```
Design a PostgreSQL schema for a [TABLE NAME] table in a [APP TYPE] app.
This table stores: [PLAIN ENGLISH DESCRIPTION]
Relationships:
- [TABLE] has many [OTHER TABLE] (foreign key on [OTHER TABLE])
- [TABLE] belongs to [OTHER TABLE] (foreign key on this table)
Requirements:
- UUID primary keys
- created_at / updated_at on every table
- Indexes on: [LIST COLUMNS USED IN FILTERS]
- Add a deleted_at column for soft deletes (do not hard delete records)
Return only the SQL migration. No application code.
```

### CRUD Helper Prompt
```
Generate a [TypeScript / Python] CRUD helper for the [TABLE NAME] table.
ORM: [Prisma / Drizzle / SQLAlchemy / Supabase client]
Functions needed:
- create[Entity](data): returns created record
- get[Entity]ById(id, userId): returns record or null
- getAll[Entities]ByUser(userId, options?: { limit, offset }): returns array
- update[Entity](id, userId, data): returns updated record or null
- softDelete[Entity](id, userId): sets deleted_at, returns boolean
Each function: handles errors, never throws to the caller, logs errors internally.
```

### Embedding + Semantic Search Prompt
```
Add semantic search to my [APP TYPE] app.
Context: [PASTE context_anchor JSON]
I want to search over: [WHAT DATA — e.g. lesson content, reminder text]
Stack: [pgvector on Supabase / Pinecone / Weaviate]
Embedding model: [text-embedding-3-small / text-embedding-ada-002]

Generate:
1. Schema: embeddings table with columns (id, entity_id, entity_type, embedding vector(1536), created_at)
2. generateAndStoreEmbedding(entityId, entityType, text) function
3. semanticSearch(queryText, userId, limit) function that returns ranked results
4. A hook: whenever a [ENTITY] is created or updated, automatically generate its embedding
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "SQL vs NoSQL — which should I use?"
For 95% of AI products being built today: **use SQL (PostgreSQL)**.

NoSQL (MongoDB) was popular when data structures were unpredictable. But modern PostgreSQL handles JSON columns natively, so you get the flexibility of NoSQL when you need it inside a SQL database. The structure of SQL also makes AI agent queries more reliable — AI agents understand SQL schemas better than they understand NoSQL document structures.

**Only choose NoSQL if:** Your entire data model is genuinely schema-less and changes constantly. This is rare for most apps.

---

### "What is a vector database and do I actually need one?"
A vector database stores the *meaning* of text as numbers (called embeddings). Instead of searching for exact words, it finds content that is *semantically similar*.

**You need a vector DB if your app does any of these:**
- "Find lessons similar to this topic" (semantic search)
- "Remember what we talked about in previous conversations" (AI memory)
- "Answer questions based on my uploaded documents" (RAG)

**You do not need one yet if:** Your app just stores and retrieves structured data (users, reminders, lesson records). Add it when a feature specifically requires semantic search.

**Easiest starting point:** Supabase pgvector — it adds vector search to your existing PostgreSQL database. No separate service to manage.

---

### "Should I use Prisma or just the Supabase client?"

| | Prisma | Supabase JS Client |
|---|---|---|
| **Best for** | Complex queries, multiple DB providers, type safety | Supabase-only apps, simpler setup |
| **Setup effort** | Medium (schema file + migrations) | Low (works immediately) |
| **AI agent friendliness** | Very high — agents understand Prisma schema well | High |
| **Recommendation** | Use if building something substantial | Use for MVPs and fast prototypes |

When in doubt for a new project: **start with Supabase client, migrate to Prisma when queries get complex.**

</vibe_coder_bridge>

---

<testing_and_qa>

## How to Verify Your Database Layer

### After Every Migration
```sql
-- Run in Supabase SQL editor or psql
-- Verify table exists with correct columns
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'your_table_name'
ORDER BY ordinal_position;

-- Verify indexes exist
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'your_table_name';
```

### After Building CRUD Helpers
```typescript
// Quick integration test — run once, then delete
import { createReminder, getReminderById, softDeleteReminder } from "@/lib/db/reminders";

async function testReminderCRUD() {
  // Create
  const created = await createReminder({
    userId: "test-user-id",
    text: "Test reminder",
    scheduledAt: new Date().toISOString(),
  });
  console.assert(created?.id, "Create failed");

  // Read
  const fetched = await getReminderById(created.id, "test-user-id");
  console.assert(fetched?.text === "Test reminder", "Read failed");

  // Soft delete
  const deleted = await softDeleteReminder(created.id, "test-user-id");
  console.assert(deleted === true, "Delete failed");

  // Verify soft deleted (should return null)
  const afterDelete = await getReminderById(created.id, "test-user-id");
  console.assert(afterDelete === null, "Soft delete not working");

  console.log("All CRUD tests passed ✅");
}

testReminderCRUD();
```

### Debugging Loop
```
Database error received →
  1. Read the full error — Postgres errors are descriptive
  2. Common error patterns:
     - "relation does not exist" → migration not run, or wrong database env
     - "null value in column violates not-null" → required field missing in insert
     - "duplicate key value violates unique constraint" → trying to create a duplicate
     - "foreign key constraint violated" → referenced record does not exist yet
  3. Paste error + your schema + the failing query to AI agent:
     "This query fails with: [ERROR]
      Schema: [PASTE RELEVANT TABLE SQL]
      Query: [PASTE FAILING CODE]
      Fix only this query. Do not change the schema."
  4. Re-run the test after fix
```

### Common Errors and Fixes

| Error | Meaning | Fix |
|---|---|---|
| `relation "table_name" does not exist` | Migration not applied | Run `prisma migrate dev` or apply SQL in Supabase dashboard |
| `column "x" does not exist` | Schema mismatch between code and DB | Check column name spelling; run migration if column was just added |
| `JWT expired` | Supabase auth token stale in test | Refresh the session token in your test setup |
| `permission denied for table` | RLS blocking the query | Check RLS policies; use service role key for backend operations |
| `value too long for type character varying` | Input exceeds column max length | Increase column length in migration or truncate input |

</testing_and_qa>

---

<common_patterns>

## Reusable Schema Patterns

### Pattern 1: Universal Base Columns (Add to Every Table)
```sql
CREATE TABLE your_table (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at  TIMESTAMPTZ  -- NULL means not deleted (soft delete)
);

-- Auto-update updated_at on every row change
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN NEW.updated_at = NOW(); RETURN NEW; END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_updated_at
BEFORE UPDATE ON your_table
FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

### Pattern 2: AI Generations Audit Table
```sql
-- Track every LLM call for cost monitoring and debugging
CREATE TABLE ai_generations (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      UUID REFERENCES users(id) ON DELETE SET NULL,
  feature      TEXT NOT NULL,       -- e.g. 'lesson_generation', 'intent_extraction'
  model        TEXT NOT NULL,       -- e.g. 'gpt-4o', 'claude-3-5-sonnet'
  prompt_tokens    INTEGER,
  completion_tokens INTEGER,
  total_tokens     INTEGER,
  latency_ms       INTEGER,
  status       TEXT NOT NULL DEFAULT 'success', -- 'success' | 'failed' | 'timeout'
  created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_ai_generations_user_id ON ai_generations(user_id);
CREATE INDEX idx_ai_generations_feature ON ai_generations(feature);
CREATE INDEX idx_ai_generations_created_at ON ai_generations(created_at DESC);
```

### Pattern 3: Embeddings Table (pgvector)
```sql
-- Enable pgvector extension (Supabase: Extensions tab, or run once)
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE embeddings (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  entity_id    UUID NOT NULL,
  entity_type  TEXT NOT NULL,  -- e.g. 'lesson', 'reminder', 'message'
  embedding    vector(1536),   -- 1536 for OpenAI text-embedding-3-small
  created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_embeddings_entity ON embeddings(entity_id, entity_type);
-- IVFFlat index for fast approximate nearest-neighbor search
CREATE INDEX idx_embeddings_vector ON embeddings
  USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

### Pattern 4: Prisma Schema Starter
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  reminders Reminder[]
  lessons   Lesson[]

  @@map("users")
}

model Reminder {
  id          String    @id @default(cuid())
  userId      String    @map("user_id")
  text        String
  scheduledAt DateTime  @map("scheduled_at")
  isComplete  Boolean   @default(false) @map("is_complete")
  deletedAt   DateTime? @map("deleted_at")
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([scheduledAt])
  @@map("reminders")
}
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE — apply to every database interaction -->

### Rule 1: Always Use Row Level Security on Supabase
Every table must have RLS enabled and policies set before going to production. Without RLS, any authenticated user can query any row.

```sql
-- Enable RLS on every table
ALTER TABLE reminders ENABLE ROW LEVEL SECURITY;

-- Users can only see their own data
CREATE POLICY "users_own_data" ON reminders
  FOR ALL USING (user_id = auth.uid());
```

### Rule 2: Never Use the Supabase Service Key on the Client
```typescript
// ❌ NEVER expose this to the browser
const supabase = createClient(url, process.env.SUPABASE_SERVICE_ROLE_KEY);

// ✅ Use anon key on client, service key only on server
// Client-side:
const supabase = createClient(url, process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY);
// Server-side (API routes only):
const supabaseAdmin = createClient(url, process.env.SUPABASE_SERVICE_ROLE_KEY);
```

### Rule 3: Always Use Parameterised Queries
Never concatenate user input into SQL strings.
```typescript
// ❌ SQL injection risk
const result = await db.query(`SELECT * FROM users WHERE email = '${email}'`);

// ✅ Parameterised — safe
const result = await db.query("SELECT * FROM users WHERE email = $1", [email]);
// Or with Prisma (automatically safe):
const result = await prisma.user.findUnique({ where: { email } });
```

### Rule 4: Soft Delete, Never Hard Delete
```sql
-- ❌ Hard delete — data gone forever, no audit trail
DELETE FROM reminders WHERE id = $1;

-- ✅ Soft delete — data preserved, recoverable
UPDATE reminders SET deleted_at = NOW() WHERE id = $1;
-- Always filter soft-deleted records in queries:
SELECT * FROM reminders WHERE deleted_at IS NULL AND user_id = $2;
```

### Rule 5: Never Store Raw Sensitive Data
- Passwords: never store plain text — use your auth provider (Clerk/Supabase Auth handles this)
- Payment info: never store card numbers — use Stripe, which tokenises everything
- PII: if you must store sensitive fields, encrypt at the application layer before saving

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Designing the Entire Schema Upfront
You try to design 15 tables before building anything. Half of them change when you actually start building features.  
**Fix:** Design one table per feature, at the time you need it. Start with Users + your core domain table.

### ❌ Using Auto-Increment IDs Instead of UUIDs
```sql
-- ❌ Sequential IDs expose data volume and are guessable
id SERIAL PRIMARY KEY  -- user id=1, id=2 ... attacker can enumerate

-- ✅ UUIDs are non-guessable and work across distributed systems
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
```

### ❌ No Indexes on Foreign Keys and Filter Columns
Every column used in a `WHERE`, `JOIN`, or `ORDER BY` should have an index. Without them, queries become full table scans that slow down as data grows.

### ❌ Storing Arrays as Comma-Separated Strings
```sql
-- ❌ Unqueryable, fragile
tags TEXT  -- stored as "math,science,beginner"

-- ✅ Proper array or junction table
tags TEXT[]  -- PostgreSQL native array
-- Or for complex relationships: a separate tags table with foreign keys
```

### ❌ Not Tracking AI Generation Costs
You make thousands of AI calls through your app and have no idea which features are burning your API budget.  
**Fix:** Always insert a row into `ai_generations` after every LLM call. Review weekly.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your Data Layer

### Add Database Connection Pooling
At scale, each serverless function opening its own DB connection exhausts PostgreSQL's connection limit.
```
Ask your AI agent: "Add PgBouncer connection pooling via Supabase's pooler URL.
Switch DATABASE_URL to the pooler endpoint (port 6543, not 5432)."
```

### Add Read Replicas for Heavy Read Workloads
When your AI features generate content once but read it thousands of times, a read replica takes the load off the primary database. Supabase supports this natively.

### Implement Database Branching for Safe Migrations
```
Use Supabase branching or PlanetScale branching:
- Main branch = production data
- Feature branch = safe to run migrations against
- Never run untested migrations on production directly
```

### Add Full-Text Search Before Reaching for Vector Search
For simple keyword-based search, PostgreSQL's built-in full-text search is fast and free — no extra service needed:
```sql
-- Add a search index
ALTER TABLE lessons ADD COLUMN search_vector tsvector
  GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || content)) STORED;
CREATE INDEX idx_lessons_search ON lessons USING GIN(search_vector);

-- Query it
SELECT * FROM lessons
WHERE search_vector @@ plainto_tsquery('english', 'algebra beginner')
ORDER BY ts_rank(search_vector, plainto_tsquery('english', 'algebra beginner')) DESC;
```
Add vector/semantic search only when keyword search is insufficient.

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: Reminder App Schema Evolution
```
v1 Schema (MVP — 2 tables):
  users: id, email, name, created_at
  reminders: id, user_id, text, scheduled_at, is_complete, created_at

v2 Schema (after adding recurrence):
  + recurrence_rules: id, reminder_id, pattern, next_occurrence, end_date

v3 Schema (after adding AI parsing audit):
  + ai_generations: id, user_id, feature, model, tokens_used, created_at
  + reminders: + raw_input TEXT (store original user text before AI parsing)

Key lesson: schema evolved one feature at a time. 
No big-bang redesign was ever needed because each table had a single clear responsibility.
```

### Case Study 2: EdTech App — Adding Semantic Search to Lessons
**Problem:** Students search for "how to multiply fractions" but lessons are titled "Fraction Multiplication — Module 3". Keyword search misses them.

**Solution:** Added pgvector semantic search in 3 steps:
```
Step 1: Enable pgvector extension in Supabase (1 click in dashboard)
Step 2: Add embeddings table (Pattern 3 above)
Step 3: Background job — when a lesson is created, generate embedding and store it
Step 4: Search endpoint — embed the user's query, find top 5 nearest lesson embeddings

Result: Search quality improved dramatically. 
"how to multiply fractions" correctly surfaces "Fraction Multiplication — Module 3" 
even though no words match, because the meanings are similar.
```

</real_world_examples>

</skill_document>
