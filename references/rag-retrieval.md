# RAG / Retrieval Engineering

A production RAG is NOT "dump documents into a vector DB and similarity-search." It is a multi-stage retrieval pipeline. This file covers hybrid search, chunking, metadata, reranking, query rewrite, and no-answer handling.

## Hybrid Search: Never Vector-Only

Vector search handles semantic similarity, synonyms, and concept relevance. But it misses exact keyword hits — code symbols, error codes, config keys, proper nouns. BM25 (the classic keyword algorithm behind Elasticsearch/Lucene/OpenSearch) handles those.

### The Failure Mode

User asks: "What is Android 17's `memory.high`?"

- Vector-only finds: "Android memory optimization", "low-memory management", "background process limits" — semantically close but misses the exact key.
- BM25 hits: `memory.high`, `memory.swap.max`, `memory.events`, `pmgd/config.json` — exact matches.

Use both. Merge, dedupe, normalize scores, then rerank.

### Prompt phrasing

> Do not implement vector-only retrieval. Implement Hybrid Search:
> 1. Vector retrieval for semantic recall.
> 2. BM25 for keyword, code-symbol, error-code, and proper-noun recall.
> 3. Merge, dedupe, and normalize scores across both result sets.
> 4. Rerank candidates with a reranker.
> 5. Every result must carry `source`, `chunk_id`, `score`, and `metadata` for citation and debugging.

## Chunking Matters More Than the Vector DB

Most "RAG quality is bad" complaints trace back to bad chunking, not a bad model. The bottleneck is rarely reached before chunking is the problem.

### Chunking Strategies

| Type | Description | Best for |
|------|-------------|----------|
| Fixed-length | Split every 500/1000 tokens | Plain articles |
| Recursive | Split by heading → paragraph → sentence | Markdown, docs |
| Semantic | Split on semantic boundaries | Long-form, knowledge bases |
| Code | Split by class / function / method | Code repositories |
| Parent-child | Small chunk for retrieval, large chunk for answer | Technical docs, books |

### The Bad-Chunk Example

```
Chunk 1: Android 17 introduces PMGD, which can...
Chunk 2: /vendor/etc/pmgd/config.json configures memory.high...
```

User searches "PMGD" — chunk 1 has the concept, chunk 2 has the config, but the context is split across both.

### The Parent-Child Fix

- Small chunks for search: "PMGD is what", "config.json config", "memory.high", "memory.events".
- Parent chunk for the answer: the entire PMGD section.

### Prompt phrasing

> Do not chunk by character count. Perform semantic chunking:
> 1. Markdown: split by heading hierarchy.
> 2. Code: split by function/class.
> 3. Every chunk retains `parent_id`.
> 4. Retrieve with small chunks; return the parent or adjacent chunks for the answer.
> 5. Each chunk stores `source`, `heading_path`, `created_at`, `updated_at`, `token_count`.

## Metadata: Required, Not Optional

Every chunk needs structured metadata. Without it the retriever degrades to "search the whole DB blindly."

### Example metadata

```json
{
  "source": "android_17_memory.md",
  "type": "technical_note",
  "created_at": "2026-06-20",
  "section": "PMGD",
  "tags": ["Android", "memory", "cgroup"],
  "language": "zh-CN",
  "project": "Android 17"
}
```

### Why metadata matters

Most real queries are filter queries, not full-text:
- Only docs after 2026.
- Only Android 17.
- Only my own notes.
- Only official docs.
- Only code files.
- Exclude archived docs.

### Prompt phrasing

> The RAG store must not save only `text` and `embedding`. Every chunk must store metadata: `source`, `title`, `section`, `tags`, `language`, `created_at`, `updated_at`, `doc_type`, `project`, `is_archived`. Retrieval must support metadata filtering.

## Reranker: Re-rank After Recall

Vector + BM25 recall broadly ("find more relevant stuff"), but their ordering is not precise. A reranker re-judges: how relevant is each candidate chunk to the user's actual question?

### Example

User: "How to configure `publicHeadersPath` for Flutter iOS SPM migration?"

Vector recall might surface: "Flutter iOS build", "Swift Package Manager", "iOS public header", "CocoaPods migration". But the most relevant chunk contains `publicHeadersPath`, `Package.swift`, `target`, `headers`. The reranker pushes it to the top.

### Prompt phrasing

> The retrieval pipeline must have a rerank stage. Hybrid-search recalls 30–50 candidates, the reranker reorders them, and only the top 5–10 are passed to the LLM. Never pipe raw vector Top-K straight into the prompt.

## Query Rewrite: Never Search the Raw User Input

Raw user input is noisy. "Why doesn't this feature work?" as a raw search destroys the Agent. Rewrite into multiple retrieval queries first.

### Example

User: "Does Android 17's background memory limit have any exemption?"

Rewrite into:
- `Android 17 background memory limit exemption`
- `Android 17 PMGD memory.high allowlist`
- `Android 17 cgroup memory events vendor process`
- `Android 17 后台 内存 限制 豁免`

Merge the results from all rewritten queries.

### Prompt phrasing

> Implement query rewrite. Do not search with the user's raw sentence. Generate 3–5 queries:
> 1. Original natural-language query.
> 2. Keyword query.
> 3. English technical-term query.
> 4. Code/API/config-name query.
> 5. Synonym query.
>
> Merge search results across all rewritten queries.

## No-Answer Handling

Retrieval will sometimes return nothing relevant. The Agent must NOT hallucinate an answer. It must explicitly return "no sufficient evidence found" and, if appropriate, ask the user to clarify or supply more context.

### Prompt phrasing

> If no chunk exceeds the relevance threshold after rerank, return a structured "no answer" result with `status: "no_evidence"` and list the queries that were tried. Do not fabricate an answer from low-score chunks.
