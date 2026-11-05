# Living Brain vs. DIY Open Brain

Background only: the workshop is a simple version of products like this one
and isn't trying to match its features. Feature list from
https://livingbrain.com/pricing, captured 2026-11-05. Living Brain is by
ChatAndBuild. The plan tables below are copied from the page; the page may
change.

The **DIY** column is my first pass, based on the design in
`workshop_prompt.md`, so it describes the plan, not a tested build:

- **Yes**: in the core workshop build
- **Option**: an attendee's choice during the build, or a stretch goal
- **Plan**: not built, but the design leaves room for it
- **Free**: comes with Supabase or the other services, no build needed
- **No**: not in the design
- **?**: what Living Brain means by this isn't clear from the pricing page

---

## Plans

| | Living Brain API | Living Brain Workspace | Business |
|---|---|---|---|
| Price | $8/M input + $0.50/M output tokens | $99/month | Custom |
| Billing | Prepaid balance, no monthly minimum | Monthly subscription | Contract |
| Pitch | "Prepaid memory for your applications and agents" | "Create, organise and run your team's Brains and agents in ChatChat" | "Connected memory for your organization" |

- API balance is shared with ChatChat. Minimum top-up is $5. Auto top-up with a
  monthly cap is optional.
- Workspace is "included at no additional cost with ChatChat Pro+". On
  2026-11-05 its button read "Checkout unavailable".
- Workspace includes one shared credit pool: $80 per cycle for the first three
  cycles, then $50. Model processing is included and doesn't draw on it.
- Seats are people managing the integration, not an application's end users.
- Token counts are provider-reported. Reasoning tokens are included in output.
  File extraction that doesn't call a model isn't token-billed.

---

## Feature Comparison

### Pricing and billing

| Capability | API | Workspace | Business | DIY |
|---|---|---|---|---|
| Measured model tokens | Charged from balance | Within plan allowances | Within contracted allowance | Paid directly, or free tiers |
| Embedding usage | Billed as model input tokens | Included | Included | Voyage: 200M free tokens, then $0.02/M |
| ChatChat Pro+ access and monthly credits | Not included | Included, one pool for all three seats | By agreement | No |

### Find and understand knowledge

| Capability | API | Workspace | Business | DIY |
|---|---|---|---|---|
| Semantic search | Included | Included | Included | Yes |
| Connection discovery | Included | Included | Included | Plan |
| Brain scores and suggestions | Included | Included | Included | ? |
| Daily briefs | Scheduled | Scheduled | Scheduled | Plan |
| Page version history | 30 days | 90 days | Agreed retention | No |

### Capture and organize context

| Capability | API | Workspace | Business | DIY |
|---|---|---|---|---|
| Notes and URL capture | Included | Included | Included | Yes for notes; URLs via bookmark import only |
| Document extraction: PDF, DOCX, PPTX, HTML, text | Up to 100 MB/file | Up to 100 MB/file | Agreed file limits | No (deliberately avoided for the workshop) |
| Audio transcription | Included, no model-token charge | Available with allowance | Contracted allowance or rates | No |
| Automatic chat-turn classification | Included, charged by measured tokens | Within processing allowance | Within contracted allowance | Option (auto-tagging of typed thoughts) |
| Batch capture | 100 items/request | 100 items/request | Agreed batch limits | Option (bulk import, 128 texts per embedding call) |
| Telegram capture bot | Included | Included | Where connectivity permits | Option (runs only while the laptop is on) |

### Connect applications and agents

| Capability | API | Workspace | Business | DIY |
|---|---|---|---|---|
| REST API | Included | Included | Included | Plan (Supabase REST exposes the table, no search API of our own) |
| SDK and CLI | Included | Included | Included | CLI yes (`brain.py`); no SDK |
| MCP read tools: search, pages, index, connections | Included | Included | Included | Search only (`search_brain`) |
| MCP capture tool | Included | Included | Included | Yes (`add_thought`) |
| MCP create tools: brains and projects | Within plan limits | Within plan limits | Within agreement | No (single brain) |
| MCP resources: index, pages, profile | Included | Included | Included | No |
| Manage Brains and agents in ChatChat | Not included | Included | By agreement | No |
| Webhook subscriptions | 5 | 20 | Agreed allocation | No |
| Markdown export | Included | Included | Included | Plan (a short script; data is plain Postgres) |

### Capacity and throughput

| Capability | API | Workspace | Business | DIY |
|---|---|---|---|---|
| Projects | 1 | 1 production | Agreed | Separate Supabase project per brain |
| Brains | Multiple within allowances | Multiple within allowances | Agreed | 1 |
| Team seats | 1 | 3 | Agreed | 1 (single user, service_role key) |
| Original file and note storage | 1 GB | 10 GB | Contracted | Supabase free-tier limits |
| Monthly API, MCP and Telegram requests | 100,000 | 500,000 | Contracted | No limit of its own; Supabase and Voyage limits apply |
| API keys | 3 | 3 | Agreed | Not applicable |
| API rate limit | 300/min | 600/min | Agreed | Voyage: 2000 RPM once a card is on file |
| MCP rate limit | 120/min | 240/min | Agreed | None (local stdio) |

### Team and support

| Capability | API | Workspace | Business | DIY |
|---|---|---|---|---|
| Team membership | Single user | Up to 3 including owner | Agreed | Single user |
| Support | Documentation and self-service | Community Slack and standard email | Priority, agreed coverage | Yourself |
| Slack integration and support | Integration, self-service | Integration plus community Slack | Integration plus private support | Option (Socket Mode, more setup) |
| Custom onboarding | Not included | Not included | Defined package | Not applicable |
| Custom SLA and contract | Not included | Not included | By agreement | Not applicable |

### Deployment and control

| Capability | API | Workspace | Business | DIY |
|---|---|---|---|---|
| Hosting | Standard hosted | Standard hosted | Scoped deployment | Supabase cloud; local scripts |
| Dedicated managed instance | Not included | Not included | Technical validation | Your own Supabase project |
| Customer cloud or on-premises | Not included | Not included | Architecture assessment | Plan (self-host Postgres + pgvector) |
| Fully air-gapped operation | Not included | Not included | Architecture and model assessment | Plan (needs local embeddings, e.g. `voyage-4-nano`) |
| Data residency and retention | Standard policy | Standard policy | Agreed | You choose the Supabase region and keep everything |
| Identity and access integration | Standard controls | Standard controls | Assessed | No |
| Audit and operational logging | Standard capabilities | Standard capabilities | Assessed | Whatever Supabase logs |
| Model processing and outbound connectivity | Standard configuration | Standard configuration | Agreed boundary | You choose the models |
| SLA, incident support and maintenance | Standard terms | Standard terms | Contracted | Yourself |

---

## Takeaways

**What the commercial product adds**
1. Connection discovery. Possibly the most interesting one: nearest neighbors
   between stored memories, which pgvector already makes possible.
2. Scheduled daily briefs. Needs a scheduler, an LLM call, and an always-on
   machine.
3. Document and URL capture. The workshop avoids this on purpose (see
   `workshop_notes.md`), but it's the largest functional gap.
4. Always-on capture. The Telegram bot stops when the laptop does.

**Where DIY is ahead**
- Embedding cost. Living Brain bills embeddings as model input at $8/M.
  `voyage-4-lite` is $0.02/M, 400 times cheaper, with 200M tokens free. At
  $8/M, the 200M free tokens would cost $1,600. The comparison ignores
  whatever else Living Brain does with those tokens.
- Control. Our text sits in a plain Postgres table we can query, export, and
  re-embed. Air-gapped or on-premises operation is Business-only there.

**Check before quoting**
- $8 input against $0.50 output is unusual, since output normally costs more
  than input. The page states it consistently, so it may be deliberate, but it
  could also be swapped. Watch for a correction.
- "Brain scores and suggestions" and "pages" aren't defined on the pricing
  page. The API docs may explain them.

---

## Related Project

`~/projects/personalsearch` is the local-only counterpart: a Google Desktop-style
search over personal data, using hybrid keyword and semantic retrieval with CPU
embeddings (`bge-small-en-v1.5`) in SQLite. See its `TRACER_BULLET_BRIEF.md`,
`PLAN.md`, and `LANDSCAPE.md`, which covers adjacent open-source tools such as
Hister. As of 2026-11-05 the embedding work is planned but not built; only the
YouTube Takeout parsers exist.
