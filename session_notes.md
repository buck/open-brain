# Open Brain — Architecture Session Notes

## Source Material
- **Nate B Jones** — "Build Your Open Brain" companion guide (Substack)
- First page transcribed to `open_brain_guide.md`
- Nate's challenge: he showed enough in his video that you could vibe code your own solution

---

## What Open Brain Is

An infrastructure layer for personal memory. Not a notes app — a database with vector search and an open protocol so that every AI tool you use shares the same persistent memory of you. Claude, ChatGPT, Cursor, Claude Code — one brain, all of them.

---

## The Stack (Nate's Version → Our Version)

| Component | Nate's Guide | Our Plan |
|---|---|---|
| Capture UI | Slack | Telegram (already have bot on OpenClaw) |
| AI Gateway | OpenRouter | OpenRouter |
| Database | Supabase + pgvector | Supabase + pgvector |
| Hosting | (unspecified) | OpenClaw (already running 24/7) |
| Language | (unspecified) | Python |

---

## How Embeddings Work

1. Send text to an embedding model API (HTTP POST)
2. Get back a vector — a plain list of floats (e.g. 1536 numbers)
3. Store both the original text AND the vector in Postgres

```python
response = client.embeddings.create(
    model="text-embedding-3-small",
    input="I want to learn piano"
)
vector = response.data[0].embedding  # list of 1536 floats
```

The vector is not human-readable. You never look at it — you just file it away.

**Search works by vector proximity:** embed the query, find rows whose vectors are geometrically close. Matches by meaning, not keywords. "Music hobbies" finds notes about piano.

---

## Embedding Model Choice

- **OpenAI** `text-embedding-3-small` — 1536 dims, cheap, most tutorials use it
- **OpenAI** `text-embedding-3-large` — 3072 dims, better recall, costs more
- **Cohere** `embed-v3` — good at retrieval tasks specifically
- **Voyage AI** — Anthropic recommends for use with Claude
- **Open source** (nomic-embed, bge-m3) — run locally, no API cost, no data leaving machine

For a personal brain: `text-embedding-3-small` is the obvious starting point.

### The Consistency Rule
You must use the **same model** for storing and searching. Vectors from different models live in incompatible geometric spaces.

### Portability
Embeddings are **derived data** — not precious. Your real data is the text. If you switch models, you can re-embed everything overnight with a script. Add a `model` column to the table so you know which rows need regenerating.

```sql
CREATE TABLE memories (
    id          bigint primary key,
    content     text,
    embedding   vector(1536),
    model       text,
    created_at  timestamptz
);
```

---

## The MCP Server — Why Any Model Can Use It

The MCP server hides the embedding machinery. A calling model sends plain text, gets plain text back:

```
search_brain("what have I said about learning music?")
→ [returns relevant memories as text]
```

Internally the MCP server:
1. Calls the embedding API → gets a vector
2. Does similarity search in Postgres
3. Returns plain text results

The caller never sees a vector. The embedding model is an implementation detail. Any AI (Claude, ChatGPT, Cursor) can use the MCP tool without knowing or caring which embedding model is in use.

---

## Build Order

1. **Supabase setup** — enable pgvector extension, create memories table
2. **Ingest handler** — Telegram message → OpenRouter embedding → Supabase insert → confirmation reply
3. **MCP server** — accepts text query, embeds it, similarity search, returns results

Steps 1 and 2 = working capture pipeline. Step 3 = the payoff.

---

## Scaling Up — The "Interestingness Filter"

Context: 20 years of saved web pages + years of ChatGPT history + ongoing Telegram captures.

### The Key Insight
Saved pages are an implicit training set with no labeling work required.

**Pipeline:**
1. Embed all saved HTML pages (strip tags, chunk, embed) → store in Open Brain
2. For a new HN post, embed title + summary
3. Find nearest neighbors in saved pages corpus
4. Average similarity score = interestingness score

No training step. No labels. Your 20-year curation history *is* the model.

### Negative Signal Problem
You have "I liked this" signal but no "I rejected this" signal. Options:
- Accept it and use a similarity threshold
- Add a browser extension with thumbs-down for explicit negatives

### LLM Scoring Upgrade (RAG for taste prediction)
Retrieve 5 most similar saved pages, then ask a model:
```
Given that this person saved these articles in the past:
[retrieved examples]
Rate this new post's likely interest to them: [post title + summary]
```

### ChatGPT Export
Settings → Data Controls → Export Data → `conversations.json`
- Years of heavy use typically lands under a few hundred MB
- Embed only your messages (what *you* typed) for purest personal brain signal
- Or both sides if you want to recall advice you received
- Cost to embed it all: probably under $1

---

## Chunk Size Reference

| Content | Approx tokens | Chunks needed |
|---|---|---|
| Tweet / short thought | ~50 | 1 |
| Article | ~2,000 | 4–5 |
| Long essay | ~10,000 | 20 |
| Book chapter | ~50,000 | 100 |

`text-embedding-3-small` cap: 8,192 tokens (~6,000 words, ~32KB plain text).
Chunking strategy: split by conversation turn for ChatGPT exports, overlapping windows for long documents.

---

## Privacy / Trust Model

**Open Brain sends text to two third parties:**
- OpenRouter (for embedding)
- Supabase (for storage)

**Fully local alternative is buildable:**
- `ollama` for local embeddings (some quality tradeoff)
- Postgres on OpenClaw for storage
- Fully air-gapped, nothing leaves your machine

**Historical context:** Google Desktop Search (2004–2011) was the previous version of this idea. Killed in 2011 — reasons likely included privacy distrust and the rise of cloud storage. Open Brain is the modern equivalent but semantically richer (meaning-based vs keyword index), more open (any AI can query via MCP), and ownable (you control the full stack).

---

## Infrastructure Notes

- **OpenClaw** — user's home server, already running 24/7, already hosts a Telegram bot
- **Supabase** — user has an existing free account
- Railway/Render free tiers: avoid for always-on bots (cold start on inactivity)
- pgvector is just a Postgres extension — Supabase is regular Postgres, not a dedicated vector DB

---

## Next Steps (When Ready)

1. Log into Supabase, enable `pgvector` extension
2. Create `memories` table (SQL above)
3. Write Telegram ingest handler
4. Write MCP server
5. (Later) Bulk ingest saved web pages
6. (Later) Bulk ingest ChatGPT export
7. (Later) Build HN interestingness filter on top
