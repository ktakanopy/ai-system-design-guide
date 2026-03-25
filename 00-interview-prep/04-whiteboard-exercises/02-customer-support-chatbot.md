# Exercise 2: Customer Support Chatbot

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
