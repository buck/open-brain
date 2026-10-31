# Open Brain — Workshop Planning Notes

Notes for a possible AI meetup workshop where attendees build Nate Jones's Open
Brain. Goal: find the cheapest setup that still gives a working Open Brain.
Researched 2026-10-30. **Do a dry run with fresh accounts long before the real
workshop.**

---

## Cost Model

| Component | Nate's guide | Cheapest workshop version |
|---|---|---|
| Coding agent | (none — copy/paste) | OpenCode with its free model |
| Database | Supabase + pgvector | Supabase free tier |
| Embedding | OpenRouter → `text-embedding-3-small` | Voyage AI `voyage-4-lite` (free tokens) |
| Classification / metadata | OpenRouter chat model | Free model, or skip it |

Possibly $0 total. The constraint is **speed, not money**.

### OpenRouter does two jobs in Nate's design
1. **Embedding** — `text-embedding-3-small`, 1536 dims.
2. **Classification** — an LLM call per thought for tags and metadata.

Voyage replaces only #1. Classification still needs an LLM — use a free model
(OpenCode's, or an OpenRouter `:free` model) or drop it. Search works on the
vectors alone.

---

## Voyage AI

Pricing as of 2026-10-30 (https://docs.voyageai.com/docs/pricing):

| Model | $/M tokens after free | Free tokens |
|---|---|---|
| `voyage-4-large` | $0.12 | 200M |
| `voyage-4` | $0.06 | 200M |
| `voyage-4-lite` | $0.02 | 200M |
| `voyage-context-4` | $0.12 | 200M |
| `voyage-code-4` | $0.12 | 200M |
| `voyage-finance-2`, `voyage-law-2` | $0.12 | 50M |
| `voyage-multimodal-3.5` | $0.12 + pixels | 200M text tokens |

- The free allowance is per account. The pricing page doesn't say whether it is
  one-time or monthly.
- 4-series: 32K context, 1024 dims by default (256/512/2048 also available).
- Batch API usage does **not** draw from the free tokens.
- Legacy models (`voyage-3.5`, `voyage-3-large`, …) get **no** free tokens.
- `voyage-medical-2` is retired; there is no medical model now.
- Open-weight `voyage-4-nano` is on Hugging Face as a possible local fallback.
- Pricing page inconsistency: its free-token sentence names `voyage-code-3`,
  but the table lists `voyage-code-4`.

### Open questions for the dry run
- **Rate limits with no card on file.** The docs only list Tier 1 ("payment
  method added"): `voyage-4-lite` 16M TPM / 2000 RPM, `voyage-4` 8M TPM / 2000
  RPM. Adding a card keeps the free tokens. Test a brand-new account without a
  card before telling people "no card needed."
- One-time vs monthly free allowance.

### Swapping Voyage into Nate's design

Table: change the vector size and record the model.
```sql
CREATE TABLE memories (
    id          bigint generated always as identity primary key,
    content     text,
    embedding   vector(1024),        -- was 1536 for text-embedding-3-small
    model       text default 'voyage-4-lite',
    created_at  timestamptz default now()
);
```

Python:
```python
import voyageai
vo = voyageai.Client()   # reads VOYAGE_API_KEY
vec = vo.embed([text], model="voyage-4-lite", input_type="document").embeddings[0]
# at search time: input_type="query"
```

Plain HTTP (e.g. for a Supabase Edge Function in TypeScript):
`POST https://api.voyageai.com/v1/embeddings` with
`{"input": [...], "model": "voyage-4-lite", "input_type": "document"}`.
The vector is at `data[0].embedding`, the same place as in OpenAI's response.

`input_type` is the one new concept: it tells Voyage whether the text is a
stored document or a search query. It is optional, but search is better with it.

Teaching point: the switch makes the deck's "re-embed, the text is safe" point
concrete — different dimensions, different vector space, same text.

TODO: only page 1 of Nate's guide is transcribed (`open_brain_guide.md`). Save
the full guide here to write the exact diff from his OpenRouter code to Voyage.

---

## Bulk Import Caveat

Nate's pipeline makes one embedding call and one LLM classification call per
thought. That's fine for typed thoughts. For thousands of items, free LLM rate
limits make it take hours. For bulk imports, **embed only and skip
classification**. Voyage takes up to 128 texts per request.

---

## Good Bounded Data Sources

What makes a source good for a room of people:
- They can export it during the session.
- The size is predictable.
- It's in a simple format.
- It's not too private. The data goes to Voyage and Supabase, and search
  results go to whatever AI tool they connect.
- It produces a "huh, I forgot about that" moment.

### Best for doing live
1. **Browser bookmarks** — top pick. One JSON file in the Chrome profile,
   available instantly. Usually hundreds to a few thousand entries. Everything
   in it was saved on purpose ("past me thought this was interesting"). Much
   less sensitive than history.
2. **Kindle highlights** (`My Clippings.txt`), Readwise, Goodreads, Letterboxd
   exports. Small, personal, well-formed. Not everyone has one.
3. **Obsidian vault or a folder of markdown notes.** Already plain text.

### Better as homework before the session
4. **ChatGPT or Claude conversation export.** Probably the strongest content:
   their own words about what they were working on. The export arrives by
   email, sometimes hours later. Embed only the person's own messages.
5. **Google Takeout of YouTube.** Can take hours or days to deliver — request
   a week ahead.
   - Search history is the best part: short, in their own words, shows how
     interests changed over time.
   - Watch history is titles only — thin content, but searchable. 42K watch
     rows ≈ 1M tokens, about 0.5% of the free allowance.
   - Takeout HTML has no charset declaration. Parse it as UTF-8 explicitly
     (see `~/projects/personalsearch/parse_history.py`).

### Avoid
- **Browser history.** Instant (copy the SQLite `History` file while Chrome is
  closed), but very sensitive (banking, medical, everything) and noisy (logins,
  redirects, repeat visits). Uncomfortable on a projector.
- **Arbitrary folders on their machine.** PDF/Word parsing, splitting into
  chunks, and the risk of a `taxes-2019.pdf` turning up. Too much for an
  evening, and too risky to send to the cloud without exclusion lists.
- **Email.** Too big and too private.

---

## Session Outline

1. Build Nate's core setup. Each person types a dozen thoughts and searches
   them. Proves the pipeline works.
2. Bulk-load their bookmarks. Proves the scale-up and gives the "I forgot about
   that" moment. This is where it stops being a toy, and where the deck's
   interestingness-filter idea becomes something people can try on their data.
3. Early finishers and homework-doers load a ChatGPT or YouTube export they
   requested beforehand.
