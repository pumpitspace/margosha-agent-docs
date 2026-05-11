# LLM Wiki + Vector Database Architecture

## Goal

Build a long-term knowledge system for Маргоша that combines:

- **LLM Wiki**: curated, human-readable, interlinked Markdown knowledge.
- **Vector database**: fast semantic recall over raw sources, sessions, docs, and wiki pages.
- **Operational memory**: compact durable facts loaded into Hermes prompts.

The point is not to replace memory with embeddings. The point is to create three complementary layers.

## Why three layers

### 1. Durable memory

Use for compact facts that should affect future behavior:

- User preferences.
- Stable environment facts.
- Long-lived conventions.

Do not store temporary task status, PR numbers, secrets, or stale operational logs.

### 2. LLM Wiki

Use for curated knowledge that should compound over time:

- Architecture decisions.
- Research syntheses.
- Entity/concept pages.
- Comparisons and plans.
- Source-backed notes with cross-links.

The wiki is authoritative for synthesized knowledge because it is readable, edited, linked, and auditable.

### 3. Vector DB

Use for retrieval and discovery:

- Raw source snippets.
- Past session fragments.
- Web extracts.
- PDFs and docs.
- Wiki chunks.

The vector DB is not the source of truth. It is an index over source-of-truth documents.

## Proposed local stack

Recommended first implementation:

- Wiki: Markdown at `~/wiki` or `~/.hermes/wiki`.
- Embeddings: local Ollama `nomic-embed-text` or a small hosted embedding model.
- Vector store: Qdrant via Docker/Colima.
- Metadata store: SQLite for source IDs, hashes, chunk provenance, and ingest state.
- Agent integration: Hermes skill/tool wrapper for ingest, query, and lint.

## Directory layout

```text
wiki/
  SCHEMA.md
  index.md
  log.md
  raw/
    articles/
    papers/
    transcripts/
    sessions/
  entities/
  concepts/
  comparisons/
  queries/
  _meta/
    vector-index.md
    ingestion-policy.md
```

## Vector schema

Each vector point should include:

```json
{
  "id": "stable-content-hash-or-source-id:chunk-number",
  "vector": [0.0],
  "payload": {
    "source_path": "raw/articles/example.md",
    "wiki_page": "concepts/browser-automation.md",
    "chunk_index": 3,
    "chunk_text": "...",
    "sha256": "...",
    "created": "YYYY-MM-DD",
    "updated": "YYYY-MM-DD",
    "type": "raw|wiki|session|doc",
    "tags": ["browser", "hermes"]
  }
}
```

## Ingestion flow

1. Capture raw material into `raw/` with frontmatter and SHA-256.
2. Chunk into semantic blocks: headings, paragraphs, code blocks preserved.
3. Embed chunks.
4. Upsert into Qdrant with metadata.
5. Synthesize or update LLM Wiki pages.
6. Add/update `index.md` and append to `log.md`.

## Query flow

1. User asks a question.
2. Agent checks durable memory for stable preferences.
3. Agent reads `SCHEMA.md` and `index.md` to orient.
4. Agent uses vector search for semantically relevant raw/wiki/session chunks.
5. Agent reads full wiki pages and source files for final grounding.
6. Agent answers with citations to wiki pages/source paths.
7. If the answer is valuable, save it as a wiki query or concept page.

## Sync strategy

- Wiki files are Git-friendly and can be versioned.
- Vector DB can be rebuilt from wiki/raw/session sources.
- Therefore, backup priority is:
  1. Markdown wiki.
  2. Raw source files.
  3. SQLite ingest metadata.
  4. Qdrant snapshots, optional.

## Implementation plan

### Phase 1 — Wiki foundation

- Create `~/wiki` or `~/.hermes/wiki`.
- Add `SCHEMA.md`, `index.md`, `log.md`.
- Start with pages about Hermes, browser automation, voice, memory, and this architecture.

### Phase 2 — Local vector store

- Start Qdrant with Docker/Colima.
- Create collection `margosha_knowledge`.
- Use Ollama `nomic-embed-text` for local embeddings.
- Build a small Python ingester for Markdown files.

### Phase 3 — Hermes integration

- Add scripts:
  - `scripts/wiki_ingest.py`
  - `scripts/wiki_query.py`
  - `scripts/wiki_lint.py`
- Wrap them as a Hermes skill/tool workflow.
- Add cron/watchdog for periodic indexing.

### Phase 4 — Quality gates

- Lint broken wikilinks.
- Detect orphan pages.
- Detect stale or drifted sources by hash.
- Require provenance for claims sourced from raw files.

## Design principle

The wiki is the brain's **organized cortex**. The vector DB is the **associative recall index**. Hermes durable memory is the **working identity and preferences layer**.

Do not let vector search become an uncurated junk drawer. Use it to find material, then consolidate important knowledge into the wiki.
