# LLM Wiki + Vector Database Architecture — Expanded Design

## 0. Main idea

Маргоша needs a long-term knowledge system with two audiences:

1. **Human-readable wiki** — so Stanislav can open files, understand decisions, inspect sources, and edit knowledge manually.
2. **Agent-readable wiki** — so an AI agent can quickly orient itself, retrieve the right context, understand page purpose, trust level, relationships, freshness, and what to do next.

The key design principle:

> The wiki is not just a database for humans. It is an operational knowledge substrate for agents.

The vector database is not the brain. It is the semantic recall index. The Markdown wiki remains the curated source of truth.

---

## 1. Three-layer knowledge model

### Layer A — Durable Hermes memory

Purpose: compact behavioral facts that should be loaded into every future session.

Examples:

- User preferences.
- Stable environment facts.
- Long-lived project conventions.
- Tool quirks that prevent repeated mistakes.

Do not store:

- Temporary task status.
- PR numbers, issue numbers, commit SHAs.
- Logs.
- Raw research dumps.
- Secrets.

Memory is the **identity and preference layer**.

### Layer B — LLM Wiki

Purpose: curated, linked, auditable knowledge.

Examples:

- Architecture decisions.
- Research summaries.
- Project pages.
- Entity pages.
- Concept pages.
- Comparison pages.
- Operational runbooks.
- Source-backed notes.

The wiki is the **organized cortex**.

### Layer C — Vector database

Purpose: semantic search over both curated and raw knowledge.

Examples indexed into vectors:

- Wiki pages.
- Raw documents.
- Web extracts.
- Session excerpts.
- PDFs.
- Research notes.
- Code documentation.

The vector DB is the **associative recall index**.

---

## 2. Why the wiki must be both human-readable and agent-readable

A normal human wiki often optimizes for prose:

- Nice explanations.
- Long paragraphs.
- Loose headings.
- Implicit assumptions.
- Links that make sense to a person.

An agent needs more structure:

- What is this page about?
- Is this page canonical or draft?
- How fresh is it?
- What entities/concepts are mentioned?
- What sources support the claims?
- Which pages are related?
- What tasks can be performed from this page?
- What should be indexed?
- What should not be trusted?

So every wiki page should have two parts:

1. **Machine-facing frontmatter and control blocks**.
2. **Human-facing explanation and notes**.

---

## 3. Page format: agent-friendly Markdown

Every page should use YAML frontmatter:

```yaml
---
id: concept.llm-wiki-vector-architecture
title: LLM Wiki + Vector Database Architecture
type: concept
status: canonical
created: 2026-05-11
updated: 2026-05-11
owner: margosha
summary: >
  Architecture for combining a curated Markdown wiki, vector search,
  and Hermes durable memory into a long-term knowledge system.
tags:
  - hermes
  - memory
  - vector-db
  - llm-wiki
entities:
  - Margosha
  - Hermes Agent
  - Qdrant
  - Ollama
  - nomic-embed-text
related:
  - concepts/browser-automation.md
  - _meta/vector-index.md
source_paths:
  - raw/sessions/2026-05-11-llm-wiki-discussion.md
trust: high
freshness: stable
indexing:
  include: true
  chunk_strategy: heading
  priority: high
agent_hints:
  when_to_read: >
    Read this before designing memory, retrieval, wiki ingestion,
    or semantic recall features for Margosha.
  use_for:
    - architecture planning
    - implementation planning
    - explaining the memory stack
  do_not_use_for:
    - storing secrets
    - temporary task state
---
```

Then the body should follow a predictable structure:

```md
# Title

## Agent Brief
Short operational summary for agents.

## Human Summary
Plain-language explanation for humans.

## Canonical Claims
Claims that are considered current source of truth.

## Sources and Evidence
Where the claims came from.

## Related Pages
Explicit links.

## Open Questions
Unknowns and decisions still pending.

## Change Log
Small human-readable history.
```

---

## 4. Recommended wiki directory layout

```text
~/.hermes/wiki/
  SCHEMA.md                         # Rules for humans and agents
  index.md                          # Top-level map
  log.md                            # Chronological change log

  _meta/
    agent-reading-guide.md           # How agents should use the wiki
    ingestion-policy.md              # What gets indexed and how
    vector-index.md                  # Qdrant collection details
    trust-levels.md                  # canonical/draft/stale/deprecated
    taxonomy.md                      # Allowed page types and tags

  concepts/
    llm-wiki-vector-architecture.md
    browser-automation.md
    memory-model.md

  entities/
    people/
    projects/
    tools/
    organizations/

  projects/
    margosha-agent-docs.md
    amex-offer-buddy.md

  runbooks/
    browser-cdp-macos.md
    hermes-gateway-restart.md

  decisions/
    adr-0001-use-markdown-wiki-plus-qdrant.md

  comparisons/
    vector-db-qdrant-vs-chroma.md

  queries/
    2026-05-11-semantic-recall-design.md

  raw/
    sessions/
    web/
    papers/
    docs/
    transcripts/

  _generated/
    backlinks.json
    graph.json
    orphan-pages.md
    stale-pages.md
```

---

## 5. Page types

### concept
Explains an idea or architecture.

Examples:

- `concepts/memory-model.md`
- `concepts/browser-automation.md`

### entity
A durable object in the world.

Examples:

- person
- project
- tool
- company
- repository
- API

### runbook
Step-by-step procedure.

Examples:

- restarting Hermes gateway
- checking Chrome CDP
- backing up Qdrant

### decision / ADR
Architecture decision record.

Includes:

- context
- decision
- alternatives
- consequences
- rollback path

### comparison
Structured comparison between options.

Examples:

- Qdrant vs Chroma
- Ollama embeddings vs hosted embeddings

### query
Saved answer to an important question.

A query page should include:

- original question
- answer
- retrieved sources
- follow-up actions

### raw
Uncurated source material.

Raw files are searchable but not automatically trusted as canonical.

---

## 6. Knowledge lifecycle

```text
Capture → Normalize → Index → Retrieve → Synthesize → Curate → Maintain
```

### 1. Capture
Collect material from:

- chats/sessions
- web pages
- docs
- PDFs
- repositories
- user-provided notes

Save raw sources under `raw/` with metadata.

### 2. Normalize
Convert sources to clean Markdown with:

- source URL/path
- timestamp
- extraction method
- hash
- title
- tags

### 3. Index
Chunk and embed material into vector DB.

### 4. Retrieve
Use semantic search to find likely relevant chunks.

### 5. Synthesize
Agent reads retrieved chunks and writes a coherent answer or wiki page.

### 6. Curate
Important knowledge is promoted into canonical wiki pages.

### 7. Maintain
Run checks for:

- stale pages
- broken links
- missing sources
- orphan pages
- duplicate concepts
- low-trust claims

---

## 7. Ingestion pipeline

Input sources:

- Markdown wiki pages
- raw session excerpts
- web extracts
- PDFs converted to Markdown
- docs repos

Pipeline:

```text
File/Web/Session/PDF
        ↓
Normalize to Markdown
        ↓
Extract frontmatter + body
        ↓
Split into semantic chunks
        ↓
Generate embeddings
        ↓
Upsert chunks to Qdrant
        ↓
Record source/chunk hashes in SQLite
        ↓
Update generated graph/backlinks/staleness reports
```

Important rule:

> Qdrant stores chunks and metadata, but Markdown files remain the source of truth.

---

## 8. Query pipeline

When user asks a question:

```text
User question
    ↓
Classify intent
    ↓
Check Hermes durable memory
    ↓
Read wiki index + schema if needed
    ↓
Embed query
    ↓
Search Qdrant
    ↓
Rerank by trust, freshness, type, and score
    ↓
Read full source/wiki files
    ↓
Answer with grounded citations
    ↓
Optionally save valuable result into wiki
```

Search should not blindly trust vector chunks. It should use them as pointers to full files.

---

## 9. Retrieval ranking

Semantic score alone is not enough.

Final retrieval ranking should combine:

- Vector similarity.
- Page type.
- Trust level.
- Freshness.
- Source quality.
- Whether the page is canonical.
- Whether the page has explicit agent hints.
- Whether the page is linked from `index.md`.

Example scoring idea:

```text
final_score =
  vector_score
  + trust_bonus
  + canonical_bonus
  + freshness_bonus
  + page_type_bonus
  + linked_from_index_bonus
  - stale_penalty
  - deprecated_penalty
```

---

## 10. Vector database schema

Collection name:

```text
margosha_knowledge
```

Point payload:

```json
{
  "id": "sha256:chunk_index",
  "source_path": "concepts/browser-automation.md",
  "source_uri": "file:///Users/minibot/.hermes/wiki/concepts/browser-automation.md",
  "title": "Browser Automation",
  "type": "concept",
  "status": "canonical",
  "trust": "high",
  "freshness": "stable",
  "tags": ["browser", "hermes", "cdp"],
  "entities": ["Hermes Agent", "Chrome CDP"],
  "chunk_index": 3,
  "chunk_heading": "Chrome CDP on macOS",
  "chunk_text": "...",
  "sha256": "...",
  "created": "2026-05-11",
  "updated": "2026-05-11",
  "index_priority": "high"
}
```

SQLite metadata should track:

- file path
- file hash
- chunk hash
- last indexed time
- embedding model
- vector collection
- page status
- errors

---

## 11. Agent-readable features

### Agent Brief
Every important page should contain:

```md
## Agent Brief
Use this page when ...
Do not use this page for ...
Canonical answer: ...
```

This lets an agent quickly determine whether the page is useful.

### Canonical Claims
A section for stable claims:

```md
## Canonical Claims
- Hermes durable memory should only contain compact stable facts.
- The vector DB is an index, not the source of truth.
- Wiki pages should carry provenance and trust metadata.
```

### Agent Actions
Optional executable guidance:

```md
## Agent Actions
- If Chrome CDP fails, read `runbooks/browser-cdp-macos.md`.
- If a query produces a reusable answer, save it under `queries/`.
- If a page becomes stable, promote `status: draft` to `status: canonical`.
```

### Machine-readable backlinks
Generated JSON can help agents avoid expensive full-wiki scans:

```json
{
  "concepts/browser-automation.md": {
    "links_to": ["runbooks/browser-cdp-macos.md"],
    "linked_from": ["index.md", "concepts/memory-model.md"]
  }
}
```

---

## 12. Trust and freshness model

### status

- `draft` — useful but not fully reviewed.
- `canonical` — preferred current source of truth.
- `stale` — may be outdated.
- `deprecated` — do not use except for history.

### trust

- `high` — sourced, tested, or user-confirmed.
- `medium` — plausible synthesis, needs occasional review.
- `low` — raw/unverified material.

### freshness

- `stable` — unlikely to change soon.
- `volatile` — may change quickly.
- `expires: YYYY-MM-DD` — must be reviewed after date.

---

## 13. Human readability rules

The wiki should still feel natural for a person:

- Clear titles.
- Short summaries.
- Diagrams where useful.
- No giant JSON blobs in the main body.
- Frontmatter should be helpful but not overwhelming.
- Important decisions should be written in normal language.
- Every page should answer: “Why does this page exist?”

---

## 14. Agent readability rules

The wiki should be easy for agents to operate:

- Stable page IDs.
- Consistent frontmatter.
- Predictable headings.
- Explicit page type.
- Explicit trust/status/freshness.
- Explicit source paths.
- Explicit related pages.
- Agent Brief near the top.
- Canonical Claims section.
- Open Questions section.

---

## 15. Proposed local stack

Recommended implementation for this Mac mini:

- Wiki: `~/.hermes/wiki`
- Embeddings: Ollama `nomic-embed-text`
- Vector DB: Qdrant in Docker/Colima
- Metadata: SQLite
- Scripts:
  - `wiki_ingest.py`
  - `wiki_query.py`
  - `wiki_lint.py`
  - `wiki_graph.py`
- Hermes integration:
  - skill for wiki workflows
  - optional tool wrapper later
  - cron job for periodic index refresh

---

## 16. Implementation phases

### Phase 1 — Wiki schema

Create and enforce:

- `SCHEMA.md`
- `_meta/agent-reading-guide.md`
- `_meta/taxonomy.md`
- `_meta/trust-levels.md`
- starter concept/runbook/project pages

### Phase 2 — Local indexing

Build:

- Markdown parser
- frontmatter parser
- chunker
- Ollama embedding client
- Qdrant upsert
- SQLite metadata DB

### Phase 3 — Query CLI

Build:

```bash
wiki query "How does browser CDP work in Margosha?"
```

Expected output:

- direct answer
- top matching pages
- source paths
- confidence/trust notes
- suggested wiki updates

### Phase 4 — Agent workflow

Create Hermes skill:

- when to use wiki
- how to query
- how to create/update pages
- how to avoid storing junk
- how to cite sources

### Phase 5 — Maintenance automation

Add:

- broken wikilink lint
- orphan detection
- stale page detection
- duplicate concept detection
- periodic reindex
- backup/export

---

## 17. Core diagrams

### Knowledge layers

```text
┌─────────────────────────────────────────────────────────────┐
│                    Hermes Prompt Context                    │
│  Compact facts loaded every session: preferences, identity   │
└──────────────────────────────▲──────────────────────────────┘
                               │
                        Durable Memory
                               │
┌──────────────────────────────┴──────────────────────────────┐
│                         LLM Wiki                             │
│  Curated Markdown: concepts, projects, runbooks, decisions    │
│  Human-readable + agent-readable + source-backed              │
└──────────────────────────────▲──────────────────────────────┘
                               │ points to / indexed by
┌──────────────────────────────┴──────────────────────────────┐
│                         Vector DB                            │
│  Semantic recall over chunks from wiki, raw docs, sessions    │
└──────────────────────────────▲──────────────────────────────┘
                               │ indexes
┌──────────────────────────────┴──────────────────────────────┐
│                       Raw Sources                            │
│  Sessions, web pages, PDFs, repos, transcripts, docs          │
└─────────────────────────────────────────────────────────────┘
```

### Human + agent page design

```text
┌─────────────────────────────────────────────────────────────┐
│ Markdown Wiki Page                                           │
├─────────────────────────────────────────────────────────────┤
│ YAML frontmatter                                             │
│ - id, type, status, trust, freshness                         │
│ - tags, entities, related pages                              │
│ - source paths                                               │
│ - indexing policy                                            │
│ - agent hints                                                │
├─────────────────────────────────────────────────────────────┤
│ Agent Brief                                                  │
│ - when to read                                               │
│ - what this page is canonical for                            │
│ - what not to use it for                                     │
├─────────────────────────────────────────────────────────────┤
│ Human Summary                                                │
│ - normal explanation                                         │
│ - context and decisions                                      │
│ - diagrams                                                   │
├─────────────────────────────────────────────────────────────┤
│ Canonical Claims + Sources + Related Pages + Open Questions  │
└─────────────────────────────────────────────────────────────┘
```

### Retrieval loop

```text
User asks
   ↓
Agent classifies intent
   ↓
Check durable memory
   ↓
Search vector DB
   ↓
Rerank by score + trust + freshness
   ↓
Read full wiki/source files
   ↓
Answer with citations
   ↓
If valuable: write/update wiki page
   ↓
Reindex changed pages
```

---

## 18. Non-goals

This system should not become:

- A dumping ground for every chat message.
- A replacement for concise Hermes memory.
- A black-box vector-only memory.
- A place for secrets.
- A system that requires cloud infrastructure to work.

---

## 19. One-sentence architecture

Маргоша should use **Hermes memory for compact identity and preferences**, **LLM Wiki for curated human/agent-readable knowledge**, and **Qdrant vector search for semantic recall over all indexed sources**, with Markdown remaining the canonical source of truth.
