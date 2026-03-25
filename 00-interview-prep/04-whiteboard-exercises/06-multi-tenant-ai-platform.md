# Exercise 6: Multi-Tenant AI Platform

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
