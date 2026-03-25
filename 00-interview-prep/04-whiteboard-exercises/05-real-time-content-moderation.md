# Exercise 5: Real-Time Content Moderation

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
