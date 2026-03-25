# Whiteboard Exercises for AI System Design

This chapter provides detailed walkthroughs of system design exercises commonly asked in AI-focused interviews. Each exercise includes the full problem statement, a structured solution approach, and discussion points that distinguish strong candidates.

## Table of Contents

- [Exercise 1: Enterprise RAG System](#exercise-1-enterprise-rag-system)
- [Exercise 2: Customer Support Chatbot](#exercise-2-customer-support-chatbot)
- [Exercise 3: Code Review Assistant](#exercise-3-code-review-assistant)
- [Exercise 4: Document Processing Pipeline](#exercise-4-document-processing-pipeline)
- [Exercise 5: Real-Time Content Moderation](#exercise-5-real-time-content-moderation)
- [Exercise 6: Multi-Tenant AI Platform](#exercise-6-multi-tenant-ai-platform)
- [Exercise 7: Semantic Search at Scale](#exercise-7-semantic-search-at-scale)
- [Tips for Whiteboard Exercises](#tips-for-whiteboard-exercises)

---

## Exercise 1: Enterprise RAG System

### Problem Statement

Design a RAG-based knowledge assistant for a large enterprise with the following requirements:

- 10 million documents from multiple sources (SharePoint, Confluence, Google Drive, internal wikis)
- 50,000 employees with role-based access
- Documents update continuously
- Must respect document permissions at query time
- Sub-3 second response time for 95% of queries
- Support for multiple languages (English, Spanish, Mandarin)

### Time Allocation (35 minutes)

| Phase | Time | Focus |
|-------|------|-------|
| Clarification | 3 min | Scope, priorities, constraints |
| High-level architecture | 7 min | Components and data flow |
| Data pipeline | 8 min | Ingestion, chunking, indexing |
| Query pipeline | 8 min | Retrieval, generation, permissions |
| Reliability and scale | 5 min | Failure handling, scaling |
| Evaluation | 4 min | Metrics and monitoring |

### Solution Walkthrough

#### Clarification Questions

```
1. What is the document size distribution? (PDFs, wikis, code?)
2. How often do permissions change? (Impacts caching strategy)
3. Is conversation history required or single-turn Q&A?
4. What is the accuracy bar? (Can we say "I don't know"?)
5. Are there compliance requirements? (Audit, data residency)
```

#### High-Level Architecture

```mermaid
flowchart TD
    subgraph DATA_PLANE["DATA PLANE"]
        direction LR

        subgraph CONNECTORS["Connectors"]
            SP["SharePoint\nGraph API + delta sync"]
            CONF["Confluence\nREST API + webhooks"]
            GD["Google Drive\nDrive API + push notifications"]
        end

        subgraph PROCESSOR["Processor"]
            CHUNK["Chunking\n· Markdown/HTML → semantic by headers\n· PDFs → layout-aware via Document AI\n· Wiki → section-based\n· 512 tokens, 50 overlap"]
            EMBED["Embedding Service\n· Cohere embed-v3 (multilingual)\n· Batches of 100 chunks\n· Exponential backoff on rate limit"]
        end

        PERM_SVC["Permission Service\n· Syncs user/group grants from source\n· Attaches visibility: private|internal|public\n· Triggers on permission change events"]

        VDB[("Vector Database\nPinecone / Qdrant\n50M vectors (10M docs × 5 chunks)\n· Dense + sparse hybrid search\n· Metadata: doc_id, chunk_id,\n  language, permissions, source")]

        SP & CONF & GD -->|"normalized doc schema\n(doc_id, content, permissions, metadata)"| CHUNK
        CHUNK -->|"chunks with inherited permissions"| EMBED
        EMBED -->|"vectors + metadata"| VDB
        PERM_SVC -->|"permission sync"| VDB
        SP & CONF & GD -->|"permission events"| PERM_SVC
    end

    subgraph QUERY_PLANE["QUERY PLANE"]
        direction TD

        USER["User Interface\n(Web / API / Slack)"]
        QAPI["Query API\n· Auth + rate limiting\n· Conversation history (optional)"]
        LANGDET["Language Detection\n· Detect: en / es / zh\n· Boosts same-language results"]
        REDIS[("Redis Cache\n· permissions:{user_id} TTL=5min\n· response cache for repeated queries")]
        PERM_RES["Permission Resolver\n· Expands group memberships\n· Builds $or filter:\n  public | user grant | group grant"]
        RETRIEVER["Retriever\n· Hybrid search (dense + sparse)\n· Permission filter applied\n· top_k = 20 candidates"]
        RERANKER["Reranker\n· Model: bge-reranker-v2-m3\n· Cross-encoder, multilingual\n· top-20 → top-5\n· Latency budget: ~150ms"]
        GEN["Generator\n· Model: GPT-4o (temp=0.1)\n· Grounded strictly on context\n· Falls back to secondary LLM\n· Adds [1],[2] citations"]
        RESP["Response\n· Answer + cited sources\n· Source links\n· Degraded warning if partial"]
        FEEDBACK["User Feedback\n· Thumbs up / down\n· Query reformulation rate\n· Citation click-through"]
        MONITOR["Monitoring\n· Latency dashboards (p50/p95/p99)\n· Permission filter hit rate\n· Empty result rate by source\n· Cost per query"]

        USER -->|"natural language query"| QAPI
        QAPI --> LANGDET
        QAPI -->|"user_id"| PERM_RES
        PERM_RES <-->|"cache read/write"| REDIS
        LANGDET & PERM_RES -->|"embedded query + filter"| RETRIEVER
        RETRIEVER -->|"top-20 chunks"| RERANKER
        RERANKER -->|"top-5 chunks"| GEN
        GEN -->|"grounded answer"| RESP
        RESP --> USER
        RESP --> FEEDBACK
        QAPI & GEN & RETRIEVER --> MONITOR
    end

    VDB -->|"ranked chunks + metadata"| RETRIEVER
```

#### Data Pipeline Deep Dive

**1. Connectors:**
```
Each source has a dedicated connector:
- SharePoint: Graph API with delta sync
- Confluence: REST API with webhooks
- Google Drive: Drive API with push notifications

Connector responsibilities:
- Fetch document content and metadata
- Track change events (create, update, delete)
- Extract permissions from source system
- Normalize to common document schema
```

**2. Document Schema:**
```json
{
  "doc_id": "uuid",
  "source": "sharepoint|confluence|gdrive",
  "source_id": "original_id_in_source",
  "title": "string",
  "content": "string",
  "content_type": "pdf|html|docx|md",
  "language": "en|es|zh",
  "permissions": {
    "users": ["user_id_1", "user_id_2"],
    "groups": ["group_id_1"],
    "visibility": "private|internal|public"
  },
  "metadata": {
    "author": "string",
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "path": "folder/path"
  }
}
```

**3. Chunking Strategy:**
```
Given mixed document types, use adaptive chunking:

- Markdown/HTML: Semantic chunking by headers
- PDFs: Layout-aware chunking using document AI
- Wiki pages: Section-based chunking

Chunk parameters:
- Target size: 512 tokens
- Overlap: 50 tokens
- Preserve: headers, tables, code blocks

Each chunk inherits parent document permissions.
```

**4. Embedding:**
```
Multilingual requirement suggests:
- Model: Cohere embed-v3 (multilingual, good quality)
- Alternative: OpenAI text-embedding-3-large

Batch embedding:
- Process in batches of 100 chunks
- Rate limit handling with exponential backoff
- Store embedding with chunk in vector DB
```

**5. Vector Database Choice:**
```
Pinecone or Qdrant for this scale.

Selection criteria:
- Metadata filtering: Critical for permissions
- Scale: 10M docs × 5 chunks = 50M vectors
- Hybrid search: Needed for keyword queries

Schema:
- Vector: embedding
- Metadata: doc_id, chunk_id, language, permissions, source
```

#### Query Pipeline Deep Dive

**1. Permission Resolution:**
```python
def get_user_permissions(user_id: str) -> PermissionSet:
    """
    Resolve all documents user can access.
    Returns set of:
    - Direct user grants
    - Group memberships expanded
    - Public document access
    
    CACHED with 5-minute TTL since permissions change infrequently.
    """
    cache_key = f"permissions:{user_id}"
    if cached := cache.get(cache_key):
        return cached
    
    perms = permission_service.resolve(user_id)
    cache.set(cache_key, perms, ttl=300)
    return perms
```

**2. Retrieval with Filtering:**
```python
def retrieve(query: str, user_id: str, top_k: int = 20) -> List[Chunk]:
    perms = get_user_permissions(user_id)
    
    # Detect language for query
    lang = detect_language(query)
    
    # Build permission filter
    # User can see: public docs, their own, or groups they belong to
    filter = {
        "$or": [
            {"visibility": "public"},
            {"users": {"$in": [user_id]}},
            {"groups": {"$in": perms.groups}}
        ]
    }
    
    # Optional: boost same-language content
    if lang != "en":
        filter["language"] = lang
    
    results = vector_db.search(
        query_embedding=embed(query),
        top_k=top_k,
        filter=filter
    )
    return results
```

**3. Reranking:**
```
Rerank top-20 to get top-5 with cross-encoder.
Model: bge-reranker-v2-m3 (multilingual)
Latency budget: ~100ms
```

**4. Generation:**
```python
def generate(query: str, chunks: List[Chunk], user_id: str) -> Response:
    context = format_chunks_with_citations(chunks)
    
    prompt = f"""You are a knowledge assistant for [Company].
Answer the question using ONLY the provided context.
If the context does not contain the answer, say "I could not find information about that in our knowledge base."
Always cite sources using [1], [2] format.

CONTEXT:
{context}

QUESTION: {query}
"""
    
    response = llm.generate(
        prompt=prompt,
        model="gpt-4o",
        temperature=0.1
    )
    
    return format_with_source_links(response, chunks)
```

#### Scaling and Reliability

**Latency Budget (p95 < 3s):**
```
Permission resolution:   50ms  (cached)
Embedding:              100ms
Vector search:          100ms
Reranking:              150ms
LLM generation:        1500ms
Network/overhead:       100ms
─────────────────────────────
Total:                 2000ms (buffer for P95)
```

**Scaling Considerations:**
```
- Vector DB: Sharded by source or hash
- Embedding service: Horizontal scale, stateless
- LLM calls: Multiple providers for redundancy
- Cache: Redis cluster for permissions and responses
```

**Failure Handling:**
```
- Vector DB down: Return cached results + degraded warning
- LLM down: Fallback to secondary provider
- Rate limiting: Queue with backpressure
- Embedding service: Batch retries with circuit breaker
```

#### Evaluation Approach

**Offline Metrics:**
```
- Retrieval: Precision@5, Recall@5, MRR
- Generation: RAGAS (faithfulness, relevance)
- End-to-end: Answer correctness on test set
```

**Online Metrics:**
```
- User feedback: Thumbs up/down
- Query reformulation rate: User rephrasing indicates failure
- Citation click-through: Are sources useful?
```

**Monitoring:**
```
- Latency dashboards by percentile
- Permission filter hit rate
- Empty result rate by source
- Cost per query
```

---

## Exercise 2: Customer Support Chatbot

### Problem Statement

Design an AI-powered customer support system for an e-commerce company:

- Handle 10,000 conversations per day
- Access to product catalog (1M products), order history, FAQs
- Goal: Resolve 70% of tickets without human handoff
- Support order lookup, returns, product questions
- Multilingual support (3 languages)
- Integration with existing Zendesk ticketing

### Solution Highlights

**Key Architectural Decisions:**

1. **Agent Architecture with Flow Control:**
```mermaid
flowchart TD
    subgraph EXTERNAL["External Integrations"]
        ZD_IN["Zendesk Webhook\nNew ticket arrives"]
        ZD_OUT_CLOSE["Zendesk: Close Ticket\n(resolved by AI)"]
        ZD_OUT_ESC["Zendesk: Assign to Queue\n(context summary attached)"]
    end

    subgraph DATASTORES["Data Stores"]
        ORDERS[("Order History DB\norder_id, status,\nitems, shipment")]
        PRODUCTS[("Product Catalog\n1M products\ncategory, price, specs")]
        FAQ[("FAQ / Knowledge Base\ncommon questions\nreturn policies")]
        CONV_MEM[("Conversation Memory\nmulti-turn history\nper session_id")]
    end

    subgraph AGENT["AI Agent System"]
        INTAKE["Intake\n· Language detection (3 langs)\n· Session init / history load\n· PII extraction (email, order_id)"]
        CLASSIFY["Intent Classifier\n· order_inquiry | product_question\n· return_request | complaint\n· payment_dispute | other"]
        SENTIMENT["Sentiment Analyzer\n· Detects highly negative tone\n· Flags payment disputes\n· Triggers escalation threshold"]
        ROUTER["Router\n· Maps intent + sentiment to flow\n· Escalates if confidence < threshold\n  after 2 attempts"]

        subgraph FLOWS["Specialized Flows"]
            ORDER_FLOW["Order Flow\n· lookup_order(order_id | email)\n· shipment status, ETA\n· order modifications"]
            PRODUCT_FLOW["Product Q&A\n· search_products(query, category,\n  price_range)\n· Compare products\n· Answer from FAQ"]
            RETURN_FLOW["Returns Flow\n· Verify eligibility\n· create_return(order_id,\n  reason, items)\n· Confirm policy"]
            HUMAN_FLOW["Escalate to Human\n· escalate_to_human(reason, priority)\n· Triggers on:\n  - explicit request\n  - negative sentiment\n  - payment dispute\n  - low confidence (2 attempts)\n  - refund above threshold\n  - complex multi-order"]
        end

        RESPGEN["Response Generator\n· GPT-4o grounded on tool results\n· Multilingual reply\n· Adds order/product links\n· Logs to ticket timeline"]
        ANALYTICS["Analytics & Logging\n· Resolution rate (target: 70%)\n· Escalation rate\n· Intent distribution\n· Avg turns to resolve"]
    end

    ZD_IN -->|"ticket payload"| INTAKE
    INTAKE --> CLASSIFY
    CLASSIFY --> SENTIMENT
    SENTIMENT --> ROUTER

    ROUTER --> ORDER_FLOW
    ROUTER --> PRODUCT_FLOW
    ROUTER --> RETURN_FLOW
    ROUTER --> HUMAN_FLOW

    ORDER_FLOW <-->|"lookup_order"| ORDERS
    PRODUCT_FLOW <-->|"search_products"| PRODUCTS
    PRODUCT_FLOW <-->|"FAQ lookup"| FAQ
    RETURN_FLOW <-->|"verify + create_return"| ORDERS

    INTAKE <-->|"read/write session"| CONV_MEM

    ORDER_FLOW & PRODUCT_FLOW & RETURN_FLOW --> RESPGEN
    RESPGEN -->|"resolved"| ZD_OUT_CLOSE
    HUMAN_FLOW -->|"context summary + priority"| ZD_OUT_ESC
    RESPGEN --> ANALYTICS
```

2. **Tool Design:**
```python
tools = [
    {
        "name": "lookup_order",
        "description": "Look up order details by order ID or customer email",
        "parameters": {
            "order_id": "optional string",
            "email": "optional string"
        }
    },
    {
        "name": "search_products",
        "description": "Search product catalog",
        "parameters": {
            "query": "string",
            "category": "optional string",
            "price_range": "optional tuple"
        }
    },
    {
        "name": "create_return",
        "description": "Initiate a return for an order",
        "parameters": {
            "order_id": "string",
            "reason": "string",
            "items": "list of item IDs"
        }
    },
    {
        "name": "escalate_to_human",
        "description": "Transfer to human agent",
        "parameters": {
            "reason": "string",
            "priority": "low|medium|high"
        }
    }
]
```

3. **Escalation Criteria:**
```
Escalate to human when:
- Customer explicitly requests human
- Sentiment is highly negative (detected by classifier)
- Issue involves payment disputes
- Agent confidence is low after 2 attempts
- Complex multi-order issues
- Refund above threshold amount
```

4. **Integration Pattern:**
```
Zendesk integration:
- Webhook receives new tickets
- AI handles via API
- Resolution → close ticket
- Escalation → assign to queue with context summary
- All interactions logged to ticket timeline
```

---

## Exercise 3: Code Review Assistant

### Problem Statement

Design a code review assistant for a development platform:

- Reviews pull requests automatically
- Provides specific, actionable feedback
- Respects repository style guides and conventions
- Can suggest code fixes
- Integration with GitHub/GitLab
- Handles 50,000 PRs per day

### Solution Highlights

**Key Technical Choices:**

1. **Context Assembly:**
```
For each changed file, assemble context:
- The diff (changed lines)
- Full file content (for understanding)
- Related files (imports, tests, types)
- Repository conventions (.eslintrc, .editorconfig)
- Previous review comments (learn from feedback)
```

2. **Review Categories:**
```python
review_types = [
    "bug_risk",           # Potential bugs
    "security",           # Security issues
    "performance",        # Performance concerns
    "maintainability",    # Code quality
    "style",              # Style guide violations
    "test_coverage"       # Missing tests
]
```

3. **Model Selection:**
```
Primary: Claude 3.5 Sonnet (best for code understanding)
Fallback: GPT-4o

Specialized models:
- Security scanning: CodeQL + LLM review
- Style: Linters + LLM explanation
```

4. **Output Format:**
```markdown
## Review Summary

### Critical Issues (must fix)
- **Line 45**: SQL injection vulnerability in user query
  ```python
  # Instead of:
  query = f"SELECT * FROM users WHERE id = {user_id}"
  # Use:
  query = "SELECT * FROM users WHERE id = ?"
  cursor.execute(query, (user_id,))
  ```

### Suggestions (consider fixing)
- **Line 78-82**: This loop could be simplified using list comprehension
...
```

5. **Latency Strategy:**
```
Target: Review ready within 2 minutes of PR creation

Strategy:
- Queue PR for processing
- Parallel processing of files
- Stream results as available
- Cache repository conventions
```

---

## Exercise 4: Document Processing Pipeline

### Problem Statement

Design a document processing pipeline for financial services:

- Process 100,000 documents per day (invoices, contracts, forms)
- Extract structured data with 99% accuracy
- Handle PDFs, scanned documents, handwritten notes
- HIPAA/SOC2 compliance
- Human review for low-confidence extractions

### Solution Highlights

**Pipeline Architecture:**

```mermaid
flowchart TD
    subgraph COMPLIANCE["Compliance Boundary (HIPAA / SOC2)"]
        direction TB

        subgraph INGEST["Ingestion"]
            SRC["Document Sources\nPDFs · Scanned docs · Handwritten notes\nInvoices · Contracts · Forms"]
            PREPROC["Pre-processing\n· OCR for scanned / handwritten\n· PDF text extraction\n· Format normalization\n· AES-256 encryption at rest\n· PHI detection & masking"]
            AUDIT[("Audit Log\n· All access & changes\n· Timestamp + user_id\n· SOC2 required")]
        end

        subgraph CLASSIFY_STAGE["Classification"]
            CLASSIFIER["Document Classifier\n· Model: LayoutLMv3 / fine-tuned ViT\n· Types: Invoice | Contract | Receipt\n         Form | ID | Other\n· Confidence threshold: 0.95 auto-route"]
        end

        subgraph EXTRACT_STAGE["Extraction (Tiered)"]
            TIER1["Tier 1: Document AI\nAWS Textract / Azure Form Recognizer\n· Structured forms (fast + cheap)\n· Returns per-field confidence scores"]
            TIER2["Tier 2: Vision LLM\nGPT-4V / Claude Vision\n· Complex / unstructured layouts\n· Better semantic understanding\n· Higher cost, fallback only"]
            CROSSVAL["Cross-Validation\n· Combine Tier 1 + Tier 2 outputs\n· Resolve conflicts by confidence\n· Aggregate field-level confidence"]
        end

        subgraph VALIDATE_STAGE["Validation"]
            RULES["Validation Rules Engine\nInvoice: total > 0, valid date,\n  tax ID format, line items sum\nContract: ≥2 parties, valid date,\n  signature present"]
            CONF_GATE{{"Confidence\n& Rules\nGate"}}
        end

        subgraph OUTCOMES["Outcomes"]
            AUTO["Auto-pass\n· confidence ≥ 0.99\n· all rules pass\n→ write to Structured Store"]
            REVIEW["Human Review Queue\n· 0.85 ≤ confidence < 0.99\n· or rule warnings\nReviewer sees:\n· original doc image\n· fields + confidence scores\n· validation errors highlighted\n· LLM-suggested corrections\n· one-click approve or field edit"]
            REJECT["Reject / Dead Letter\n· confidence < 0.85\n· critical rule failures\n→ retry queue or manual triage"]
        end

        STRUCTURED[("Structured Data Store\nExtracted fields\nper doc type schema")]
        DLQ[("Dead Letter Queue\nFailed docs\nretry + alert")]
    end

    SRC -->|"raw documents\nTLS 1.3 in transit"| PREPROC
    PREPROC -->|"normalized + encrypted"| CLASSIFIER
    PREPROC --> AUDIT
    CLASSIFIER -->|"doc type + confidence"| TIER1
    CLASSIFIER -->|"low conf or complex layout"| TIER2
    TIER1 & TIER2 --> CROSSVAL
    CROSSVAL -->|"fields + confidence"| RULES
    RULES --> CONF_GATE
    CONF_GATE -->|"high confidence + rules pass"| AUTO
    CONF_GATE -->|"medium confidence or warnings"| REVIEW
    CONF_GATE -->|"low confidence or rule failure"| REJECT
    AUTO --> STRUCTURED
    REVIEW -->|"approved corrections"| STRUCTURED
    REVIEW --> AUDIT
    REJECT --> DLQ
```

**Key Components:**

1. **Document Classification:**
```
Fine-tuned classifier on document types:
- Invoice, Contract, Receipt, Form, ID, Other

Model: LayoutLMv3 or fine-tuned ViT
Confidence threshold: 0.95 for auto-routing
```

2. **Extraction Strategy:**
```
Tiered extraction based on document type:

Tier 1: Document AI (Textract/Azure)
- Good for structured forms
- Fast and cheap
- Returns confidence scores

Tier 2: Vision LLM (GPT-4V/Claude)
- Fallback for complex layouts
- Better for unstructured text
- More expensive

Combine outputs and cross-validate.
```

3. **Validation Rules:**
```python
validation_rules = {
    "invoice": [
        ("total", lambda x: x > 0, "Total must be positive"),
        ("date", lambda x: parse_date(x), "Invalid date format"),
        ("vendor_id", lambda x: regex_match(x, TAX_ID_PATTERN), "Invalid tax ID"),
        ("line_items", lambda x: sum(i.amount for i in x) == total, "Line items must sum to total")
    ],
    "contract": [
        ("parties", lambda x: len(x) >= 2, "Contract must have at least 2 parties"),
        ("effective_date", lambda x: parse_date(x), "Invalid date"),
        ("signature_present", lambda x: x == True, "Signature required")
    ]
}
```

4. **Human Review Interface:**
```
Reviewer sees:
- Original document image
- Extracted fields with confidence scores
- Validation errors highlighted
- Suggested corrections from LLM
- One-click approval or field-level corrections
```

5. **Compliance Measures:**
```
HIPAA/SOC2 requirements:
- All documents encrypted at rest (AES-256)
- TLS 1.3 in transit
- Audit log for all access and changes
- PHI detection and masking
- Retention policies enforced
- Access controls with MFA
```

---

## Exercise 5: Real-Time Content Moderation

### Problem Statement

Design a content moderation system for a social platform:

- 1 million posts per day (text, images, video)
- Latency requirement: under 500ms for posts to be visible
- Detect: hate speech, violence, adult content, spam
- Appeal workflow for false positives
- Support 10 languages

### Solution Highlights

**Architecture Pattern: Multi-Stage Cascade**

```mermaid
flowchart TD
    subgraph INGEST["Content Ingestion"]
        POST["Incoming Post\ntext · image · video"]
        SPLIT["Content Type Router"]
        TEXT_IN["Text Pipeline"]
        IMAGE_IN["Image Pipeline"]
        VIDEO_IN["Video Pipeline"]
        POST --> SPLIT
        SPLIT --> TEXT_IN & IMAGE_IN & VIDEO_IN
    end

    subgraph STAGE1["Stage 1: Fast Filters (20ms)"]
        FF["Fast Filters\n· Regex pattern matching\n· PhotoDNA hash matching (CSAM)\n· Blocklist lookup (known bad actors)\n· Exact-match spam signatures"]
        DEC1{{"Decision"}}
        FF --> DEC1
    end

    subgraph STAGE2["Stage 2: ML Classifiers (80ms)"]
        ML_TEXT["Text Classifier\nBERT (multilingual, 10 langs)\nhate speech · spam · threats"]
        ML_IMAGE["Image Classifier\nCLIP\nadult · violence · symbols"]
        ML_VIDEO["Video Classifier\nX3D (temporal)\nscene-level violence · adult"]
        MERGE2["Merge Signals\n· Combine scores per category\n· Apply thresholds:\n  hate_speech: block≥0.95, limit≥0.80\n  adult_content: block≥0.98, limit≥0.90\n· Batched GPU inference"]
        DEC2{{"Decision"}}
        ML_TEXT & ML_IMAGE & ML_VIDEO --> MERGE2 --> DEC2
    end

    subgraph STAGE3["Stage 3: LLM Analysis (400ms async — ~5% of content)"]
        LLM_MOD["LLM Moderator\n· Context-aware nuanced decisions\n· Handles satire, cultural context,\n  edge cases across 10 languages\n· Returns: decision + reasoning"]
        DEC3{{"Decision"}}
        LLM_MOD --> DEC3
    end

    subgraph STAGE4["Stage 4: Human Review Queue (~0.5% of content)"]
        HUMAN["Human Reviewer\n· Reviews flagged content\n· Sees LLM reasoning + confidence\n· Logs decision with rationale"]
        DEC4{{"Decision"}}
        HUMAN --> DEC4
    end

    subgraph OUTCOMES["Outcomes"]
        BLOCK["BLOCK\nHigh-confidence violation\nRemove immediately"]
        LIMIT["LIMIT\nReduce distribution\nDe-amplify in feed"]
        ALLOW["ALLOW\nHigh-confidence safe\nPost goes live"]
        REVIEW_Q["Human Review Queue"]
    end

    subgraph APPEAL["Appeal Workflow"]
        APPEAL_IN["User Submits Appeal"]
        BLIND_REV["Blind Review\n(different reviewer)"]
        APPEAL_DEC{{"Overturn?"}}
        RESTORE["Restore Content"]
        UPHOLD["Uphold Decision"]
        TRAIN_DATA[("Training Data\nNegative example added")]
        APPEAL_IN --> BLIND_REV --> APPEAL_DEC
        APPEAL_DEC -->|"yes"| RESTORE
        APPEAL_DEC -->|"no"| UPHOLD
        RESTORE -->|"overturn logged"| TRAIN_DATA
    end

    subgraph OPS["Ops & Retraining"]
        METRICS["Metrics Dashboard\n· False positive rate\n· Appeal overturn rate\n· Latency per stage\n· Coverage per violation type"]
        RETRAIN["Periodic Model Retraining\n· Human review corrections\n· Overturned appeal decisions\n· New violation patterns"]
        TRAIN_DATA --> RETRAIN
    end

    TEXT_IN & IMAGE_IN & VIDEO_IN --> FF

    DEC1 -->|"known violation"| BLOCK
    DEC1 -->|"known safe (95%)"| ALLOW
    DEC1 -->|"uncertain"| ML_TEXT & ML_IMAGE & ML_VIDEO

    DEC2 -->|"block score ≥ threshold"| BLOCK
    DEC2 -->|"limit score ≥ threshold"| LIMIT
    DEC2 -->|"safe score ≥ threshold"| ALLOW
    DEC2 -->|"uncertain (5%)"| LLM_MOD

    DEC3 -->|"block"| BLOCK
    DEC3 -->|"limit"| LIMIT
    DEC3 -->|"allow"| ALLOW
    DEC3 -->|"still uncertain (0.5%)"| REVIEW_Q

    REVIEW_Q --> HUMAN
    DEC4 -->|"violation"| BLOCK
    DEC4 -->|"safe"| ALLOW

    BLOCK --> APPEAL_IN
    HUMAN & DEC3 & DEC4 --> METRICS
```

**Key Design Decisions:**

1. **Latency Optimization:**
```
Target: 500ms total

Stage 1 (Fast): 20ms
- Regex patterns
- Known hash matching (PhotoDNA)
- Blocklist lookup

Stage 2 (ML): 80ms
- Batched inference on GPU
- Small specialized models
- Parallel text/image processing

Stage 3 (LLM): 400ms (async for borderline)
- Only 5% of content reaches here
- Used for nuanced decisions
```

2. **Threshold Strategy:**
```python
class ModerationDecision:
    BLOCK = "block"          # High confidence violation
    ALLOW = "allow"          # High confidence safe
    LIMIT = "limit"          # Reduce distribution
    REVIEW = "human_review"  # Queue for human

thresholds = {
    "hate_speech": {
        "block": 0.95,
        "limit": 0.80,
        "review": 0.60
    },
    "adult_content": {
        "block": 0.98,  # Higher threshold, legal implications
        "limit": 0.90,
        "review": 0.70
    }
}
```

3. **Appeal Workflow:**
```
1. User submits appeal
2. Content queued for human review
3. Different reviewer than original (blind review)
4. Decision logged with reasoning
5. If overturned:
   - Content restored
   - Original decision added to training data as negative
   - Model retrained periodically
```

---

## Exercise 6: Multi-Tenant AI Platform

### Problem Statement

Design a multi-tenant AI platform (AI-as-a-Service):

- Serve 500+ enterprise customers
- Each customer has their own documents and models
- Complete data isolation between tenants
- Per-tenant usage tracking and billing
- Different pricing tiers with different capabilities
- SOC2 compliance required

### Solution Highlights

**Tenant Isolation Architecture:**

```mermaid
flowchart TD
    subgraph SOC2["SOC2 Compliance Boundary"]
        direction TB

        subgraph API_LAYER["API Gateway Layer"]
            CLIENT["Tenant Client\nSDK / REST / Web UI"]
            GW["API Gateway\n· TLS termination\n· JWT validation\n· Extract tenant_id from token"]
            AUTH["Auth Service\n· Verify JWT\n· Check tenant active\n· MFA enforcement"]
            TENANT_CFG[("Tenant Config Store\n· tier: starter|pro|enterprise\n· feature flags\n· rate limit quotas\n· custom model config")]
            RATE["Rate Limiter\n· Per-tenant token budget\n· Redis counter:\n  usage:{tenant_id}:{today}\n· 429 on quota exceeded"]
            ROUTER_GW["Request Router\n· Route to service\n· Inject TenantContext\n  (tenant_id, user_id, tier)"]
            AUDIT_LOG[("Audit Log DB\n· Every request logged\n· tenant_id + user_id\n· operation + timestamp\n· SOC2 required")]

            CLIENT -->|"Bearer JWT"| GW
            GW --> AUTH
            AUTH -->|"tier lookup"| TENANT_CFG
            TENANT_CFG --> RATE
            RATE -->|"TenantContext injected"| ROUTER_GW
            GW & ROUTER_GW --> AUDIT_LOG
        end

        subgraph SERVICE_LAYER["Tenant-Aware Service Layer"]
            SVC["Core AI Service\n· All ops scoped to tenant_id\n· Cache keys: {tenant_id}:*\n· Retrieval filters by tenant\n· No cross-tenant data leakage"]
            REDIS_CACHE[("Redis Cache\n· Per-tenant prefix\n· Permissions, embeddings,\n  query results\n· Eviction by tenant tier")]
        end

        subgraph INFRA["Shared Infrastructure (Isolated by tenant_id)"]
            VDB[("Vector Database\nShared cluster\n· tenant_id in ALL metadata\n· Query filter: tenant_id=X\n· No cross-tenant results")]
            LLM["Shared LLM Pool\nGPT-4o / Claude\n· No tenant data in\n  prompt history\n· Stateless per request\n· No conversation leakage"]
            OBJ[("Object Storage\nS3 / GCS\n· Path: s3://bucket/{tenant_id}/\n· Bucket policy enforced\n· AES-256 encryption at rest")]
        end

        subgraph BILLING_OPS["Billing & Operations"]
            USAGE_SVC["Usage Tracking Service\n· operation: embed|retrieve|generate\n· model, tokens_in, tokens_out\n· latency_ms, cost_cents"]
            TSDB[("Time-Series DB\nper-tenant usage events\ntenant_id + timestamp")]
            BILLING["Billing Service\n· Aggregate monthly usage\n· Apply tier pricing\n· Generate invoices\n· Overage alerts"]
            DASH["Usage Dashboard\n· Real-time token consumption\n· Cost by operation type\n· Latency percentiles\n· Quota utilization"]
        end
    end

    ROUTER_GW --> SVC
    SVC <-->|"cached reads/writes"| REDIS_CACHE
    SVC <-->|"tenant_id filtered query"| VDB
    SVC <-->|"stateless request"| LLM
    SVC <-->|"tenant-scoped path"| OBJ
    SVC -->|"usage event"| USAGE_SVC
    USAGE_SVC --> TSDB
    TSDB --> BILLING & DASH
    REDIS_CACHE -->|"cache miss"| VDB
```

**Critical Isolation Points:**

```python
class TenantContext:
    tenant_id: str
    user_id: str
    tier: str  # "starter" | "pro" | "enterprise"
    
    def __enter__(self):
        # Set tenant context for all downstream calls
        _tenant_context.set(self)
        
    def __exit__(self, *args):
        _tenant_context.set(None)

# Middleware ensures tenant context on every request
@middleware
def enforce_tenant_context(request, call_next):
    tenant_id = extract_tenant_from_token(request.headers["Authorization"])
    with TenantContext(tenant_id=tenant_id, ...):
        verify_tenant_access(tenant_id, request.path)
        response = call_next(request)
        add_tenant_to_audit_log(tenant_id, request, response)
    return response
```

**Billing and Usage Tracking:**

```python
usage_schema = {
    "tenant_id": "string",
    "timestamp": "datetime",
    "operation": "embed|retrieve|generate",
    "model": "string",
    "tokens_in": "int",
    "tokens_out": "int",
    "latency_ms": "int",
    "cost_cents": "decimal"
}

# Real-time usage aggregation
async def track_usage(tenant_id: str, operation: Usage):
    # Append to time-series DB
    await timeseries.write("usage", {
        "tenant_id": tenant_id,
        **operation.dict()
    })
    
    # Update real-time counter for rate limiting
    await redis.incr(f"usage:{tenant_id}:{today()}", operation.tokens)
```

---

## Exercise 7: Semantic Search at Scale

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

## Tips for Whiteboard Exercises

### Drawing Tips

1. **Start with boxes and labels** before connecting with arrows
2. **Use consistent notation**: rectangles for services, cylinders for databases, arrows for data flow
3. **Label data on arrows**: what flows between components
4. **Leave space** for additions as you discuss

### Common Patterns to Know

| Pattern | When to Use | Draw As |
|---------|-------------|---------|
| Load balancer + service fleet | Any scaled service | LB → multiple boxes |
| Queue + workers | Async processing | Queue → worker pool |
| Cache layer | Read-heavy, latency-sensitive | Diamond before service |
| CDC/streaming | Real-time updates | Kafka/stream icon |
| Sidecar | Cross-cutting concerns | Small box attached to service |

### Phrases That Signal Strong Candidates

- "Before I design this, let me understand the scale..."
- "The tradeoff here is..."
- "In production, we would also need..."
- "One failure mode to consider is..."
- "Let me walk you through the latency budget..."
- "For evaluation, I would measure..."

### Time Management

- Do not spend more than 5 minutes on clarification
- Draw the complete high-level picture before deep diving
- Leave time for reliability and evaluation
- Check in with the interviewer on focus areas

---

*See also: [Question Bank](01-question-bank.md) | [Answer Frameworks](02-answer-frameworks.md) | [Common Pitfalls](03-common-pitfalls.md)*
