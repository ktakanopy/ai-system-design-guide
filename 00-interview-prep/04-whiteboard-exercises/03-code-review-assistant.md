# Exercise 3: Code Review Assistant

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

```mermaid
flowchart TD
    subgraph VCS["VCS Integration (GitHub / GitLab)"]
        PR["Pull Request Event\n· pr.opened / pr.synchronized\n· 50,000 PRs / day"]
        WEBHOOK["Webhook / API\n· Delivers PR metadata\n· diff, file list, branch info"]
        PR --> WEBHOOK
    end

    subgraph QUEUE["Job Queue"]
        MQ["Message Queue\n(Kafka / SQS)\n· One message per PR\n· Priority by repo tier\n· Backpressure on overload"]
    end

    subgraph CONTEXT["Context Assembly Service"]
        DIFF["Diff Extractor\n· Changed lines per file\n· Hunk metadata (line numbers)"]
        FILE_FETCH["File Fetcher\n· Full file content\n· Related files: imports,\n  tests, type definitions"]
        CONV_CACHE[("Convention Cache\n· .eslintrc, .editorconfig\n· .prettierrc, pyproject.toml\n· Per-repo, Redis TTL=1h")]
        HIST["Review History Loader\n· Previous PR comments\n· Accepted / rejected suggestions\n· Per-repo learning signal"]
        CTX_BUILD["Context Builder\n· Assembles per-file context:\n  diff + full file + related\n  + conventions + history\n· Truncates to model window\n· Parallel per changed file"]
    end

    subgraph ANALYSIS["Analysis Pipeline (parallel per file)"]
        STATIC["Static Analysis\n· Linters (ESLint, Ruff, etc.)\n· CodeQL (security)\n· Returns line-level findings"]
        LLM_PRIMARY["Primary LLM\nClaude 3.5 Sonnet\n· Code understanding\n· bug_risk, security,\n  performance, maintainability,\n  style, test_coverage\n· Suggests fixes inline"]
        LLM_FALLBACK["Fallback LLM\nGPT-4o\n· Used if primary fails\n  or rate-limited"]
        MERGE["Result Merger\n· Dedup linter + LLM findings\n· Assign severity:\n  critical | suggestion | nit\n· Attach line references"]
    end

    subgraph OUTPUT["Output & Delivery"]
        FORMATTER["Review Formatter\n· Grouped by severity\n· Code diff blocks for fixes\n· Inline comment positions"]
        STREAM["Streaming Publisher\n· Post comments as ready\n  (no wait for full review)\n· Target: first comment < 30s\n· Full review < 2 min"]
        REVIEW_DB[("Review Store\n· All reviews persisted\n· Used for history + retraining\n· Per-repo analytics")]
        METRICS["Metrics\n· Review latency p50/p95\n· Comment acceptance rate\n· False positive rate\n· Cost per PR"]
    end

    WEBHOOK --> MQ
    MQ --> DIFF & FILE_FETCH & HIST
    DIFF & FILE_FETCH --> CTX_BUILD
    CTX_BUILD <-->|"cache read/write"| CONV_CACHE
    HIST --> CTX_BUILD
    CTX_BUILD --> STATIC & LLM_PRIMARY
    LLM_PRIMARY -->|"on failure"| LLM_FALLBACK
    STATIC & LLM_PRIMARY & LLM_FALLBACK --> MERGE
    MERGE --> FORMATTER
    FORMATTER --> STREAM
    STREAM -->|"post inline comments"| VCS
    STREAM --> REVIEW_DB
    REVIEW_DB --> METRICS
```

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
