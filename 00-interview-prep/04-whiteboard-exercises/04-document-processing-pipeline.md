# Exercise 4: Document Processing Pipeline

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
