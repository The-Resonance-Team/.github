# ADR 0005: PostgreSQL + dedicated vector database

- **Date:** 2026-06-24
- **Status:** accepted

## Context

AI applications need both relational data (users, documents, orders) and
vector search (embeddings, semantic retrieval, RAG). PostgreSQL's `pgvector`
extension handles both in one database, simplifying the stack to a single
datastore.

## Decision

Use **PostgreSQL** for relational data and **Qdrant** as a dedicated vector
database. Do NOT use `pgvector` as the sole vector store — its ANN index
performance trails dedicated vector DBs at scale, and the team's AI workloads
expect heavy vector search from day one.

**Qdrant** is the default; Milvus and Weaviate are documented as scale
alternatives for very large vector collections.

## Consequences

- **Positive:** PostgreSQL remains the single source of truth for
  business data; Qdrant is optimized for its one job (vector search).
- **Positive:** Qdrant's Python SDK is mature, self-hosting is a single
  Docker container, and it handles filtering + hybrid search well.
- **Negative:** Two databases to operate instead of one. Each needs
  backup, monitoring, and connection management. Worth it for the perf
  delta on vector search.
- **Negative:** `pgvector` would have reduced ops complexity. If a
  product's vector workload is light, this decision can be revisited
  via a project-level ADR superseding this one.
