# Exercise 7: Semantic Search at Scale

### Problem Statement

Design a semantic search system for an e-commerce site:

- 50 million products
- 100 million queries per day
- P99 latency under 100ms
- Support filters (price, category, brand, ratings)
- Personalization based on user history
- Real-time inventory updates

### Solution Highlights

**Key Challenge: 100ms at 100M queries/day**

```
100M queries/day = 1,157 QPS average
Peak: 5,000-10,000 QPS

At 100ms latency, need:
- Edge caching
- Pre-computed embeddings
- Optimized retrieval
- Minimal LLM involvement
```

**Architecture:**

```mermaid
flowchart TD
    subgraph CLIENT["Client Layer"]
        USER["User Browser / App\n100M queries/day\nPeak: 5,000–10,000 QPS"]
    end

    subgraph EDGE["Edge Layer"]
        CDN["CDN / Edge Cache\n· ~30% cache hit rate\n· Cached: popular query results\n· TTL tuned per category\n· 5ms check"]
    end

    subgraph QUERY_SVC["Query Service (stateless, horizontally scaled)"]
        EMBED_SVC["Embedding Service\n· Small bi-encoder model\n· Common query cache (Redis)\n  10ms cached / 30ms compute\n· Pre-computed for top-10k queries"]
        QUERY_TYPE{{"Query Type\nDetection"}}
        SPARSE_W["Keyword-heavy query\ne.g. 'nike air max 90 size 10'\nsparse_weight=0.7, dense=0.3"]
        DENSE_W["Semantic query\ne.g. 'comfortable shoes for flat feet'\ndense_weight=0.7, sparse=0.3"]
        FILTER_ENGINE["Filter Engine\n· Price range\n· Category\n· Brand\n· Ratings\n· In-stock flag\n· Roaring bitmap indexes\n· 10ms"]
        RRF["Reciprocal Rank Fusion\n· Merges dense + sparse results\n· Dynamic weight by query type\n· top-100 → merged → top-20\n· 10ms"]
        PERS_SVC["Personalization Service\n· User history-based boost\n· Category affinity scores\n· Brand preferences\n· Recent views / purchases\n· 10ms"]
    end

    subgraph RETRIEVAL["Retrieval Layer"]
        VDB[("Vector DB Cluster\nDense / ANN Search\n· 50M product embeddings\n· Sharded by category\n· HNSW index, ef_search tuned\n· 30ms\n· top-100 candidates")]
        ES[("Elasticsearch\nSparse / Keyword Search\n· BM25 + product attributes\n· Filters via inverted index\n· 30ms\n· top-100 candidates")]
        USER_PROFILE[("User Profile Store\n· Purchase history\n· Click / view events\n· Category affinities\n· Brand preferences")]
        EMBED_CACHE[("Embedding Cache\n· Redis\n· Pre-computed embeddings\n· Popular product vectors")]
    end

    subgraph UPDATES["Real-Time Update Pipeline"]
        CATALOG[("Product Catalog DB\nSource of truth\n50M products")]
        KAFKA["Kafka Event Stream\n· price_changed\n· inventory_updated\n· product_added\n· description_changed"]
        META_CONSUMER["Metadata Consumer\n· Price / inventory changes\n· Updates vector DB metadata only\n· No re-embed needed\n· Reflects within seconds"]
        REINDEX_JOB["Async Reindex Job\n· Description / image changes\n· Full re-embed required\n· Blue/green index swap\n· Zero-downtime cutover"]
        INVENTORY["Inventory Service\n· Real-time stock levels\n· Reserve on order\n· Feed to filter engine"]
    end

    USER -->|"search query"| CDN
    CDN -->|"cache miss"| EMBED_SVC
    EMBED_SVC --> QUERY_TYPE
    QUERY_TYPE --> SPARSE_W & DENSE_W
    SPARSE_W & DENSE_W -->|"parallel retrieval"| VDB & ES
    EMBED_SVC <-->|"cached vector lookup"| EMBED_CACHE
    VDB & ES -->|"top-100 each"| RRF
    RRF --> FILTER_ENGINE
    FILTER_ENGINE -->|"filtered candidates"| PERS_SVC
    PERS_SVC <-->|"user history"| USER_PROFILE
    PERS_SVC -->|"ranked results"| CDN

    CATALOG --> KAFKA
    KAFKA --> META_CONSUMER & REINDEX_JOB
    META_CONSUMER -->|"metadata patch"| VDB
    REINDEX_JOB -->|"new index swap"| VDB
    INVENTORY -->|"in-stock filter"| FILTER_ENGINE
```

**Latency Budget:**

```
Edge cache check:    5ms
Embedding lookup:   10ms (cached) or 30ms (compute)
Vector search:      30ms
Filtering:          10ms
Personalization:    10ms
Serialization:      10ms
Network overhead:   25ms
─────────────────────
Total:              100ms target (with cache hit)
```

**Hybrid Search Strategy:**

```python
def search(query: str, filters: dict, user_id: str) -> List[Product]:
    # Determine search strategy based on query
    if is_keyword_heavy(query):
        # "nike air max 90 size 10"
        sparse_weight = 0.7
        dense_weight = 0.3
    else:
        # "comfortable running shoes for flat feet"
        sparse_weight = 0.3
        dense_weight = 0.7
    
    # Parallel retrieval
    dense_results = vector_db.search(embed(query), top_k=100, filter=filters)
    sparse_results = elastic.search(query, top_k=100, filter=filters)
    
    # Reciprocal rank fusion
    combined = rrf_merge(
        [dense_results, sparse_results],
        weights=[dense_weight, sparse_weight]
    )
    
    # Personalization boost
    personalized = apply_user_preferences(combined, user_id)
    
    return personalized[:20]
```

**Real-Time Updates:**

```
Product updates (price, inventory) flow:
1. Change event published to Kafka
2. Consumer updates vector DB metadata
3. Search reflects change within seconds

Reindexing (description changes):
1. Full re-embed required
2. Run as async job
3. Swap index when complete
```

---
