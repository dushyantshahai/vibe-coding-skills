# Skill: RAG — Retrieval Augmented Generation

```json
{
  "skill_id": "rag",
  "category": "AI-Specific",
  "version": "1.0",
  "compatible_with": ["Claude Code", "Cursor", "Codex", "Claude Projects"],
  "stack_agnostic": true,
  "last_updated": "2025"
}
```

---

<skill_document>

<title>RAG — Making Your AI Answer Questions From Your Own Data</title>

<overview>

## What This Skill Enables
- Build AI features that answer questions using your app's own content — not just the AI's training data
- Implement semantic search so users can find relevant lessons, reminders, or documents by meaning, not just keywords
- Give AI agents long-term memory by retrieving relevant past interactions before each response
- Understand chunking, embedding, and retrieval well enough to tune quality without being a machine learning engineer

## Why It Matters for Vibe Coders
RAG is the technology behind "chat with your PDF," "ask questions about your notes," "find lessons relevant to what I am struggling with." Without RAG, your AI can only use what OpenAI or Anthropic trained it on. With RAG, your AI can use your curriculum, your user's history, your knowledge base. For EdTech and AI assistant apps, this is the feature that makes the product feel genuinely personalised.

## When to Use This Skill
- When you need the AI to answer questions using content you own (lessons, docs, knowledge base)
- When semantic search would improve a core feature (find lessons by meaning, not just title match)
- When your AI agent needs memory across sessions (remember user preferences, past interactions)
- When a single AI call's context window is not big enough to include all relevant content

</overview>

---

<context_anchor>

## Project Context — Fill This In Before Every Session

```json
{
  "project_context": {
    "app_name": "[REPLACE: e.g. LearnFlow / RemindAI]",
    "framework": "[REPLACE: e.g. Next.js / FastAPI]",
    "vector_db": "[REPLACE: e.g. Supabase pgvector / Pinecone / Weaviate / not chosen yet]",
    "embedding_model": "[REPLACE: e.g. text-embedding-3-small / text-embedding-ada-002]",
    "content_to_index": "[REPLACE: describe what needs to be searchable — e.g. lesson content, user chat history, uploaded documents]",
    "rag_feature": "[REPLACE: what feature is RAG enabling — e.g. 'personalised lesson recommendations' / 'chat with curriculum']",
    "average_document_length": "[REPLACE: e.g. lessons are ~500 words / documents are 10-50 pages]",
    "existing_rag_files": "[REPLACE: list any existing embedding/search files]"
  }
}
```

</context_anchor>

---

<mental_models>

## How to Think About RAG

### Mental Model 1: The Smart Librarian
Imagine a library with millions of books. A regular search engine finds books that contain the exact words you searched for. A smart librarian (RAG) understands what you *mean* and finds the most relevant books even if they use different words.

RAG works in two phases:
- **Index phase:** Every book is read by the librarian, who writes a summary of its "meaning" as a list of numbers (embedding). These numbers are stored in the library index.
- **Query phase:** When you ask a question, the librarian encodes your question the same way, then finds the books whose "meaning numbers" are closest to your question's numbers.

### Mental Model 2: The Four Steps of RAG
```
1. CHUNK    Split your content into pieces that fit in a context window
            (too big = relevant parts diluted; too small = context lost)

2. EMBED    Convert each chunk to a vector (list of numbers representing meaning)
            using an embedding model

3. STORE    Save vectors + original text in a vector database

4. RETRIEVE At query time: embed the query, find nearest vectors, return their text
```

### Mental Model 3: Chunking Strategy Trade-offs
```
SMALL CHUNKS (100-300 tokens):
  ✅ Precise retrieval — returns exactly the relevant sentence
  ❌ Loses context — the answer makes no sense without surrounding text
  Best for: FAQs, structured data, short facts

MEDIUM CHUNKS (500-1000 tokens):
  ✅ Balances precision and context
  ✅ Good for most use cases
  Best for: lesson sections, documentation paragraphs, article sections

LARGE CHUNKS (2000+ tokens):
  ✅ Preserves full context
  ❌ Less precise — returns too much irrelevant content
  Best for: long-form content where chapter-level retrieval is sufficient
```

</mental_models>

---

<system_design_breakdown>

## RAG Architecture

```
INDEXING PIPELINE (runs when content is created/updated)
┌──────────────────────────────────────────────────────┐
│                                                      │
│  CONTENT SOURCE      CHUNK      EMBED       STORE   │
│  (DB / files)  →  (split text) → (API call) → (vector DB) │
│                                                      │
│  Trigger: content created/updated webhook or cron   │
└──────────────────────────────────────────────────────┘

RETRIEVAL PIPELINE (runs per user query)
┌──────────────────────────────────────────────────────┐
│                                                      │
│  USER QUERY   EMBED QUERY   VECTOR SEARCH   RESULTS  │
│  "fractions" → (API call) → (nearest N)  → [chunks] │
│                                                      │
│  Then: inject retrieved chunks into LLM context     │
└──────────────────────────────────────────────────────┘
```

## Stack Options by Use Case

| Situation | Recommended Stack |
|---|---|
| Already using Supabase | **pgvector** — add-on to your existing PostgreSQL, no extra service |
| Need large-scale production vector search | **Pinecone** — managed, blazing fast, built for scale |
| Need hybrid search (semantic + keyword) | **Weaviate** — native hybrid search |
| Python/FastAPI backend | **pgvector** or **ChromaDB** (local dev) → **Pinecone** (production) |

**Recommendation for most AI products:** Start with Supabase pgvector. It is free within your existing Supabase project, SQL-native, and handles up to ~500k vectors before you need to think about performance tuning.

</system_design_breakdown>

---

<step_by_step_execution>

<!-- INCREMENTAL BUILD RULE: Enable vector extension → build embedding utility → index one content type → build search → test quality → integrate into feature. -->

## Step 1 — Set Up Vector Database

**For Supabase pgvector (recommended starting point):**

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]

Set up pgvector in my Supabase project.
Generate:
1. SQL to enable the pgvector extension
2. The embeddings table schema (from skill-database-storage.md Pattern 3)
3. IVFFlat index for approximate nearest-neighbor search
4. A SQL function: match_embeddings(query_embedding vector, match_count int, filter_user_id uuid)
   that returns the top match_count most similar chunks for a given user

Also: confirm the embedding model dimensions
text-embedding-3-small = 1536 dimensions
text-embedding-ada-002 = 1536 dimensions
```

**Verify:** Run the SQL in Supabase dashboard. Table and index exist. Run the `match_embeddings` function with a test vector — it returns results without error.

---

## Step 2 — Build the Embedding Utility

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Vector DB is set up.

Create /lib/embeddings.ts with:
1. generateEmbedding(text: string) → number[]
   - Calls OpenAI text-embedding-3-small (or configured model)
   - Returns the embedding vector
   - Handles errors: returns null (does not throw)
   - Logs: [embedding] generated for text length X in Xms

2. generateAndStore(params: { entityId, entityType, text, userId? })
   - Generates embedding for text
   - Stores in embeddings table with entity reference
   - Handles: duplicate (upsert, not insert)
   - Returns: { success: boolean }

3. semanticSearch(params: { query, userId?, entityType?, limit: 5 })
   - Generates embedding for query
   - Calls match_embeddings SQL function
   - Returns: { id, entityId, entityType, similarity, text }[]

Track each embedding call in ai_generations table (from skill-database-storage.md).
```

**Verify:** Call `generateEmbedding("test text")`. Confirm it returns an array of 1536 numbers. Call `generateAndStore` — confirm a row appears in the embeddings table.

---

## Step 3 — Build the Indexing Pipeline

Index your existing content and set up automatic indexing for new content.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Embedding utility is working at /lib/embeddings.ts

Build the indexing pipeline for [CONTENT TYPE — e.g. lessons].

1. Chunking function: chunkContent(text: string, chunkSize: 600) → string[]
   - Split text into chunks of ~600 tokens with 100-token overlap
   - Preserve sentence boundaries (do not split mid-sentence)
   - Return array of chunk strings

2. Index one item: indexLesson(lessonId: string)
   - Fetch lesson from DB
   - Chunk the content
   - For each chunk: generateAndStore with entityId=lessonId, entityType="lesson"
   - Handle: already indexed (delete old chunks first, then re-index)

3. Bulk index: indexAllLessons()
   - Fetch all unindexed lessons (no row in embeddings table)
   - Index them in batches of 10 (rate limit protection)
   - Log progress

4. Auto-index webhook: after lesson create/update, call indexLesson(lessonId)
   Place this call in the lesson creation API route (after saving to DB).
```

**Verify:** Run `indexLesson(testLessonId)`. Check embeddings table — rows should exist for each chunk. Run `indexAllLessons()` — all lessons should be indexed.

---

## Step 4 — Build the RAG Query Function

Bring retrieval and generation together.

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
Indexing pipeline is working.

Build the RAG query function at /lib/rag/[feature]-rag.ts:

async function answerWithRAG(params: {
  question: string;
  userId: string;
  entityType: "lesson" | "memory";
  maxChunks: 5;
}): Promise<{ answer: string; sourceLessonIds: string[] }>

Steps:
1. Semantic search for top maxChunks chunks relevant to the question
2. If no relevant chunks found: answer from model knowledge alone (fallback)
3. Build prompt: inject retrieved chunks as context before the question
4. Call LLM with context-enriched prompt
5. Return: answer text + list of source entity IDs (for citation)

Prompt structure:
"Use the following content to answer the question.
If the answer is not in the content, say so clearly.
---
[RETRIEVED CHUNKS]
---
Question: [USER QUESTION]"
```

**Verify:** Ask a question that is answered by content in your index. Confirm the response references specific details from your indexed content — not just generic knowledge.

---

## Step 5 — Evaluate Retrieval Quality

**Prompt your AI agent:**
```
Project context: [PASTE context_anchor JSON]
RAG query function is working.

Help me evaluate retrieval quality.
Generate a test suite:
1. 5 questions whose answers ARE in the indexed content
2. 3 questions whose answers are NOT in the content (should say "I don't know")
3. 2 ambiguous questions (partial information in content)

For each question, I will run the RAG pipeline and evaluate:
- Did it retrieve the right chunks? (check sourceLessonIds)
- Did it answer correctly?
- Did it correctly say "I don't know" when the answer was not available?

Acceptable quality threshold: 8/10 correct (80%)
```

**Verify:** Run all 10 test questions. Score the results. If below 80% — identify which retrieval failures are most common (wrong chunk retrieved vs correct chunk but poor answer).

</step_by_step_execution>

---

<ai_agent_prompts>

## Ready-to-Use Prompts

### Session Start Prompt
```
I am building RAG functionality for my app.
Project context: [PASTE context_anchor JSON]

Already completed:
- Vector DB: [set up / not yet]
- Embedding utility: [built / not yet]
- Indexing pipeline: [built / not yet]
- RAG query: [built / not yet]

Today I am building: [ONE STEP]
Before writing code, tell me which files you will create and what functions they will contain.
```

### Chunking Strategy Prompt
```
I need to chunk [CONTENT TYPE — e.g. lesson markdown, PDF pages, user notes] for RAG indexing.
Average content length: [DESCRIBE]
Use case: [DESCRIBE — e.g. answer questions about specific topics / find relevant sections]

Recommend and implement a chunking strategy:
1. Chunk size (in tokens) and why
2. Overlap amount and why
3. How to handle: titles/headings, code blocks, lists, tables
4. How to handle very short documents (shorter than chunk size)

Create: /lib/rag/chunker.ts with chunkContent(text, options) function
Include tests showing how different content types are chunked.
```

### Retrieval Quality Improvement Prompt
```
My RAG retrieval quality is lower than expected.
Context: [PASTE context_anchor JSON]
Current accuracy: [X/10 test questions answered correctly]

Common failures observed:
[DESCRIBE — e.g. "retrieves the right lesson but wrong section" / "completely misses relevant content"]

Suggest and implement improvements to try in order:
1. Quick wins: adjust chunk size, overlap, or top-k count
2. Medium effort: add metadata filtering (entityType, userId, date range)
3. Advanced: hybrid search (semantic + keyword BM25) or reranking

Try one improvement at a time. Show me the test to re-run after each change.
```

### Add RAG to Existing Chat Prompt
```
I have a working chat interface using Vercel AI SDK useChat.
API route: /api/chat
Context: [PASTE context_anchor JSON]

Add RAG to this chat: before passing the user's message to the LLM,
retrieve the top 5 relevant chunks from the embeddings table.

Modify /app/api/chat/route.ts to:
1. Extract the user's latest message from the messages array
2. Call semanticSearch({ query: latestMessage, userId, entityType: "[TYPE]", limit: 5 })
3. If results found: prepend them to the system prompt as "Relevant context:"
4. If no results: continue without context (graceful fallback)
5. Track the retrieval: log how many chunks were found and their avg similarity score

Do not change the streaming response or useChat configuration.
```

</ai_agent_prompts>

---

<vibe_coder_bridge>

## Plain-English Decision Guide

### "What is an embedding and why is it a list of numbers?"

An embedding is a mathematical representation of the *meaning* of a piece of text. The AI takes your text and converts it into a list of 1,536 numbers (for OpenAI's embedding model). These numbers encode where the text sits in a 1,536-dimensional "meaning space."

Text that means similar things (even if worded differently) ends up near each other in this space. "How do I add fractions?" and "tutorial on fraction addition" would produce vectors that are very close to each other — even though they share no common words.

**The practical implication:** You can find relevant content even when the user's exact words do not appear in the content. This is why RAG is so powerful for search and personalised recommendations.

---

### "pgvector vs Pinecone — which should I use?"

**pgvector (Supabase add-on):**
- Free if you are already on Supabase
- Works with your existing PostgreSQL data — join embeddings with your regular tables
- Simpler architecture (one less service)
- Good up to ~500k embeddings before needing performance tuning
- **Use for:** most AI products at early stage

**Pinecone:**
- Dedicated vector DB — built for nothing else, extremely fast
- Scales to billions of vectors without configuration
- More expensive (starts at ~$70/month for production)
- More complex (separate service to manage)
- **Use for:** when pgvector hits performance limits, or when vector search is the core product

**Decision:** Start with pgvector. Migrate to Pinecone when you have >500k vectors or when vector search queries take more than 200ms.

---

### "My RAG answers are not very good. What should I fix first?"

Work through this checklist in order:

1. **Check retrieval, not generation first.** Log the chunks that were retrieved. Are they actually relevant to the question? If the wrong chunks are retrieved, better generation cannot fix it.

2. **Adjust chunk size.** If retrieved chunks are too small (single sentences), increase chunk size. If they contain too much irrelevant context, decrease it.

3. **Increase top-k.** Retrieve 8–10 chunks instead of 3–5. Give the LLM more context to work with.

4. **Add metadata filters.** If searching lessons for a specific user's level, filter by level. Irrelevant results from other levels dilute quality.

5. **Check your embedding model.** `text-embedding-3-small` is much better than `ada-002` for most languages. If using ada-002, consider upgrading.

6. **Improve the generation prompt.** If retrieval is correct but generation is wrong, the prompt needs to be more specific about how to use the retrieved context.

</vibe_coder_bridge>

---

<testing_and_qa>

## Testing RAG Quality

### Retrieval Precision Test
```typescript
// /tests/rag/retrieval-precision.test.ts
// Run manually — real embedding calls, not mocked

const GROUND_TRUTH = [
  {
    query: "how to add fractions with different denominators",
    expectedLessonTitle: "Fraction Addition — Module 3",
    expectedMinSimilarity: 0.75,
  },
  {
    query: "what is the pythagorean theorem",
    expectedLessonTitle: "Geometry Fundamentals",
    expectedMinSimilarity: 0.80,
  },
];

for (const test of GROUND_TRUTH) {
  const results = await semanticSearch({ query: test.query, limit: 5 });
  const topResult = results[0];
  const correctLessonReturned = topResult?.entityId === getIdByTitle(test.expectedLessonTitle);
  const similarityMet = topResult?.similarity >= test.expectedMinSimilarity;

  console.log(`${correctLessonReturned && similarityMet ? "✅" : "❌"} "${test.query}"`);
  console.log(`  Top result: ${topResult?.entityId} (similarity: ${topResult?.similarity})`);
}
```

### Common RAG Issues and Fixes

| Issue | Symptom | Fix |
|---|---|---|
| Wrong chunks retrieved | Answer is generic, not from your content | Increase top-k; add metadata filter |
| Retrieval too slow | Search takes >500ms | Ensure IVFFlat index exists; consider Pinecone |
| Hallucination despite RAG | AI ignores context and answers from training | Strengthen "only use provided context" constraint in prompt |
| Duplicate chunks indexed | Same content indexed multiple times | Use upsert not insert; delete old chunks before re-indexing |
| Embeddings cost too high | Large bill from embedding API | Cache embeddings; only re-embed on content change |
| Short queries return poor results | Single-word queries miss relevant content | Use query expansion: ask LLM to rewrite short queries into full questions first |

</testing_and_qa>

---

<common_patterns>

## Reusable RAG Patterns

### Pattern 1: Complete Semantic Search Implementation
```typescript
// /lib/rag/search.ts
import { generateEmbedding } from "@/lib/embeddings";
import { db } from "@/lib/db";

export interface SearchResult {
  entityId: string;
  entityType: string;
  content: string;
  similarity: number;
}

export async function semanticSearch(params: {
  query: string;
  userId?: string;
  entityType?: string;
  limit?: number;
  minSimilarity?: number;
}): Promise<SearchResult[]> {
  const { query, userId, entityType, limit = 5, minSimilarity = 0.5 } = params;

  const queryEmbedding = await generateEmbedding(query);
  if (!queryEmbedding) return [];

  // Use Supabase RPC call to the match_embeddings function
  const { data, error } = await supabase.rpc("match_embeddings", {
    query_embedding: queryEmbedding,
    match_count: limit,
    filter_user_id: userId ?? null,
    filter_entity_type: entityType ?? null,
    min_similarity: minSimilarity,
  });

  if (error) {
    console.error("[semanticSearch]", error);
    return [];
  }

  return data as SearchResult[];
}
```

### Pattern 2: Overlap-Aware Text Chunker
```typescript
// /lib/rag/chunker.ts
export function chunkText(text: string, chunkSize = 600, overlap = 100): string[] {
  const words = text.split(/\s+/);
  const chunks: string[] = [];

  let start = 0;
  while (start < words.length) {
    const end = Math.min(start + chunkSize, words.length);
    const chunk = words.slice(start, end).join(" ");

    // Only add non-trivial chunks
    if (chunk.trim().length > 50) {
      chunks.push(chunk.trim());
    }

    // Move forward by (chunkSize - overlap) to create overlap
    start += chunkSize - overlap;
  }

  return chunks;
}
```

### Pattern 3: RAG-Enhanced LLM Call
```typescript
// /lib/rag/answer.ts
import { semanticSearch } from "./search";
import { generateText } from "@/lib/openai";

export async function answerWithContext(params: {
  question: string;
  userId: string;
  entityType: string;
}): Promise<{ answer: string; sourceIds: string[] }> {

  const chunks = await semanticSearch({
    query: params.question,
    userId: params.userId,
    entityType: params.entityType,
    limit: 5,
    minSimilarity: 0.6,
  });

  const sourceIds = [...new Set(chunks.map(c => c.entityId))];

  const context = chunks.length > 0
    ? chunks.map((c, i) => `[Source ${i + 1}]: ${c.content}`).join("\n\n")
    : "No relevant content found in the knowledge base.";

  const result = await generateText(
    params.question,
    `You are a helpful tutor. Answer questions using ONLY the provided context.
If the answer is not in the context, say: "I don't have specific information about that yet."
Do not use outside knowledge.

Context:
${context}`
  );

  return {
    answer: result.data?.content ?? "I was unable to generate an answer.",
    sourceIds,
  };
}
```

</common_patterns>

---

<security_guardrails>

<!-- NON-NEGOTIABLE -->

### Rule 1: Always Filter by UserId in Vector Search
If your embeddings contain user-specific content (their notes, their history), always filter the search to the current user's data:
```typescript
// ❌ Returns any user's relevant content
await semanticSearch({ query, entityType: "note" });

// ✅ Returns only this user's relevant content
await semanticSearch({ query, entityType: "note", userId: authenticatedUserId });
```

### Rule 2: Never Include PII in Embeddings Without Consent
Embedding user messages and storing them creates a queryable record of everything the user said. Ensure your privacy policy covers this and provide an option to delete indexed user content.

### Rule 3: Validate Chunk Content Before Indexing
```typescript
// Sanitise before embedding — prevents injection of malicious content into retrieval
function sanitiseForEmbedding(text: string): string {
  return text
    .slice(0, 8000)       // Limit length
    .replace(/<[^>]*>/g, "") // Remove HTML
    .trim();
}
```

### Rule 4: Set Rate Limits on Embedding Generation
Embedding API calls cost money. Without limits, a single user uploading many large documents can create unexpectedly large costs:
```typescript
const MAX_CHUNKS_PER_DOCUMENT = 100;
const MAX_DOCUMENTS_PER_HOUR_PER_USER = 10;
```

</security_guardrails>

---

<mistakes_to_avoid>

### ❌ Indexing Content Without Chunking
You embed an entire 2,000-word lesson as one vector. The embedding captures the "average meaning" of the entire lesson — retrieval becomes imprecise because the specific relevant section's meaning is diluted.  
**Fix:** Always chunk before embedding. 500-700 token chunks work for most content.

### ❌ Never Re-Indexing When Content Changes
A lesson is updated but the old embedding stays. Users search and get results based on the old version.  
**Fix:** Trigger re-indexing on every content update. Delete old chunks, re-generate.

### ❌ Using Similarity Threshold of 0
With a threshold of 0, every query returns results even when nothing is relevant. The AI receives low-quality context and produces confusing answers.  
**Fix:** Set `minSimilarity: 0.6` as a starting point. Tune up or down based on your test results.

### ❌ Embedding Short Strings (Under 50 Characters)
Short strings do not have enough semantic content to embed meaningfully. The resulting vectors are noisy and retrieval quality suffers.  
**Fix:** Skip embedding strings shorter than 50 characters. Filter them out in your chunker.

### ❌ Not Testing Retrieval Separately From Generation
The RAG answer is wrong. Is retrieval failing or is generation failing? You change the generation prompt without checking if the right chunks were even retrieved.  
**Fix:** Always log and inspect retrieved chunks before debugging the generation step.

</mistakes_to_avoid>

---

<advanced_extensions>

## Scaling Your RAG System

### Add Hybrid Search (Semantic + Keyword)
Pure semantic search misses exact phrase matches. Hybrid search combines both:
```sql
-- PostgreSQL full-text search + vector search combined
-- Weighted: 70% semantic + 30% keyword
SELECT *, 
  (0.7 * vector_similarity + 0.3 * text_rank) as hybrid_score
FROM search_hybrid($query_embedding, $query_text)
ORDER BY hybrid_score DESC
LIMIT 5;
```

### Add Reranking for Better Precision
After retrieving top-20 candidates, use a cross-encoder model to re-rank them more precisely before passing to the LLM:
```
Tool: Cohere Rerank API
Ask your AI agent: "Add Cohere reranking after the initial pgvector retrieval.
Retrieve top 20, rerank to top 5 before passing to the LLM."
```

### Add Query Expansion for Short Queries
Single-word queries produce poor embeddings. Expand them first:
```typescript
// Before embedding the query, ask the LLM to expand it
const expandedQuery = await generateText(
  `User query: "${shortQuery}"
   Rewrite as a complete question that would be answered by an educational lesson.
   Return ONLY the rewritten question, nothing else.`,
  "You expand short queries into complete educational questions."
);
// Use expandedQuery for embedding, original shortQuery for display
```

</advanced_extensions>

---

<real_world_examples>

## Mini Case Studies

### Case Study 1: EdTech App — Semantic Lesson Search
**Before RAG:** Students typed "fractions" in search — only returned lessons with "fractions" in the title. Missed "Dividing Pizza," "Ratio Fundamentals," and "Decimal Conversion" — all highly relevant.

**After RAG:**
```
Query: "fractions"
Retrieved chunks:
  - "Fraction Basics" (similarity: 0.91) ✅
  - "Dividing Pizza — Understanding Parts of a Whole" (similarity: 0.84) ✅
  - "Ratio and Proportion" (similarity: 0.78) ✅
  - "Decimal to Fraction Conversion" (similarity: 0.76) ✅

Implementation:
  Chunk size: 500 tokens (lesson sections)
  Overlap: 100 tokens
  Embedding model: text-embedding-3-small
  Vector DB: Supabase pgvector
  Min similarity: 0.65

Search quality: 73% of searches returned all relevant lessons (vs 41% with keyword search)
```

### Case Study 2: Reminder App — Agent Long-Term Memory
**Problem:** Agent forgot user's preferences between sessions. Every new session: "What time do you usually want reminders?" Same question, every time.

**RAG Memory Implementation:**
```typescript
// After each conversation, extract and embed key preferences
const preferences = await extractPreferences(conversationHistory);
// e.g. "User prefers reminders in the morning, around 8-9am"
// "User calls their mother every Sunday — this is a recurring need"

await generateAndStore({
  entityId: userId,
  entityType: "user_memory",
  text: preferences,
  userId,
});

// At agent startup, retrieve relevant past context
const relevantMemory = await semanticSearch({
  query: currentUserMessage,
  userId,
  entityType: "user_memory",
  limit: 3,
});

// Inject into system prompt
const systemPrompt = `
${BASE_SYSTEM_PROMPT}
${relevantMemory.length > 0 ? `\nKnown about this user:\n${relevantMemory.map(m => m.content).join("\n")}` : ""}
`;
```

**Result:** Agent remembered user preferences across sessions. "What time do you usually want reminders?" dropped from being asked in 100% of new sessions to less than 5%.

</real_world_examples>

</skill_document>
