# 🧠 AI System Design Guide
### The Complete Interview & Production Reference

<p align="center">
  <a href="https://github.com/ombharatiya"><img src="https://img.shields.io/badge/GitHub-ombharatiya-181717?logo=github" alt="GitHub"></a>
  <a href="https://x.com/ombharatiya"><img src="https://img.shields.io/badge/Twitter-@ombharatiya-1DA1F2?logo=twitter" alt="Twitter"></a>
  <a href="https://linkedin.com/in/ombharatiya"><img src="https://img.shields.io/badge/LinkedIn-ombharatiya-0A66C2?logo=linkedin" alt="LinkedIn"></a>
</p>

<p align="center">
  <a href="#"><img src="https://img.shields.io/badge/Updated-March%202026-blue.svg" alt="Last Updated"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-green.svg" alt="License"></a>
  <a href="#-contributing"><img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs Welcome"></a>
  <a href="https://github.com/ombharatiya/ai-system-design-guide"><img src="https://img.shields.io/github/stars/ombharatiya/ai-system-design-guide?style=social" alt="Stars"></a>
</p>

> **The living reference for production AI systems.** Continuously updated. Interview-ready depth.

---

## 📚 Quick Navigation

| I want to... | Start here |
|--------------|------------|
| **Prepare for interviews** | [Question Bank](00-interview-prep/01-question-bank.md) → [Answer Frameworks](00-interview-prep/02-answer-frameworks.md) |
| **Learn AI systems fast** | [LLM Internals](01-foundations/01-llm-internals.md) → [RAG Fundamentals](06-retrieval-systems/01-rag-fundamentals.md) |
| **Build production RAG** | [Chunking](06-retrieval-systems/02-chunking-strategies.md) → [Vector DBs](06-retrieval-systems/04-vector-databases-comparison.md) → [Reranking](06-retrieval-systems/06-reranking-strategies.md) |
| **Design multi-tenant AI** | [Isolation Patterns](12-security-and-access/04-multi-tenant-rag-isolation.md) → [Case Study](16-case-studies/08-multi-tenant-saas.md) |
| **Build agents** | [Agent Fundamentals](07-agentic-systems/01-agent-fundamentals.md) → [MCP](07-agentic-systems/03-tool-use-and-mcp.md) → [LangGraph](09-frameworks-and-tools/02-langgraph-orchestration.md) |
| **Autonomous coding agents** | [Claude Code](09-frameworks-and-tools/09-claude-code.md) → [OpenCoder Landscape](09-frameworks-and-tools/10-opencoderguide.md) |
| **Pick the right model (2026)** | [Model Taxonomy](02-model-landscape/01-model-taxonomy.md) → [Pricing](02-model-landscape/03-pricing-and-costs.md) |
| **Evaluate AI in production** | [AI Evals Guide (Phoenix/Langfuse)](ai_evals_comprehensive_study_guide.md) → [AI Evals Guide (LangWatch/Langfuse)](ai_evals_complete_guide_langwatch_langfuse.md) |
| **Find the best courses to learn AI** | [Recommended Courses & Learning Paths](COURSES.md) |
| **Transition from my current role to AI** | [Role Transition Guide](TRANSITION_GUIDE.md) |

---

## 🎯 Why This Guide

**Traditional books are outdated before they ship.** This is a living document: when new models release, when patterns evolve, this updates.

| This Guide | Printed Books |
|------------|---------------|
| March 2026 models (Claude 3.7 Sonnet, GPT-4.5, o3, Gemini 2.0 Flash, Grok 3) | Stuck on GPT-4 |
| MCP, Claude Code, Agentic RAG, OpenCoder landscape | Does not exist |
| Real pricing with verification dates | Already wrong |
| Staff-level interview Q&A | Generic questions |

---

## 📖 Guide Structure

```
├── 00-interview-prep/           # Questions, frameworks, exercises
├── 01-foundations/              # Transformers, attention, embeddings
├── 02-model-landscape/          # Claude 3.7, GPT-4.5, o3, Gemini 2.0, DeepSeek
├── 03-training-and-adaptation/  # Fine-tuning, LoRA, DPO, distillation
├── 04-inference-optimization/   # KV cache, PagedAttention, vLLM
├── 05-prompting-and-context/    # CoT, Extended Thinking, DSPy, prompt injection
├── 06-retrieval-systems/        # RAG, chunking, GraphRAG, Agentic RAG
├── 07-agentic-systems/          # MCP 2.0, multi-agent, swarms, computer-use
├── 08-memory-and-state/         # L1-L3 memory tiers, Mem0, caching
├── 09-frameworks-and-tools/     # LangGraph, DSPy, LlamaIndex, Claude Code, OpenCoder
├── 10-document-processing/      # Vision-LLM OCR, multimodal parsing
├── 11-infrastructure-and-mlops/ # GPU clusters, LLMOps, cost management
├── 12-security-and-access/      # RBAC, ABAC, multi-tenant isolation
├── 13-reliability-and-safety/   # Guardrails, red-teaming
├── 14-evaluation-and-observability/ # RAGAS, LangSmith, drift detection
├── 15-ai-design-patterns/       # Pattern catalog, anti-patterns
├── 16-case-studies/             # Real-world architectures with diagrams
├── GLOSSARY.md                  # Every term defined
│
├── ai_evals_comprehensive_study_guide.md      # 🔬 Deep-dive: AI Evals (Phoenix + Langfuse)
└── ai_evals_complete_guide_langwatch_langfuse.md  # 🔬 Deep-dive: AI Evals (LangWatch + Langfuse)
└── COURSES.md                   # 🎓 Recommended courses & learning paths
└── TRANSITION_GUIDE.md          # 🔄 Transition from Backend/QA/PM/EM to AI roles
```

---

## 🔥 Featured Case Studies

Real interview problems with complete solutions and diagrams:

| Case Study | Problem | Key Patterns |
|------------|---------|--------------|
| [Real-Time Search](16-case-studies/06-real-time-search.md) | 5-minute data freshness at scale | Streaming + Hybrid Search |
| [Coding Agent](16-case-studies/07-autonomous-coding-agent.md) | Autonomous multi-file changes | Sandboxing + Self-Correction |
| [Multi-Tenant SaaS](16-case-studies/08-multi-tenant-saas.md) | Coca-Cola and Pepsi on same infra | Defense-in-Depth Isolation |
| [Customer Support](16-case-studies/09-customer-support-automation.md) | 60% auto-resolution rate | Tiered Routing + Escalation |
| [Document Intelligence](16-case-studies/10-document-intelligence.md) | 50K contracts/month extraction | Vision-LLM + Parallel Extractors |
| [Recommendation Engine](16-case-studies/11-recommendation-engine.md) | Personalized explanations at 50M users | ML Ranking + LLM Explanations |
| [Compliance Automation](16-case-studies/12-compliance-automation.md) | FDA regulation pre-screening | Claim Extraction + Precedent DB |
| [Voice Healthcare](16-case-studies/13-voice-ai-healthcare.md) | Real-time clinical note generation | On-Prem ASR + HIPAA |
| [Fraud Detection](16-case-studies/14-fraud-detection.md) | 100ms decision with explainability | ML + Rules Hybrid |
| [Knowledge Management](16-case-studies/15-knowledge-management.md) | 2M docs with access control | Permission-Aware RAG |

---

## 🔬 Bonus Deep-Dive Guides

Two companion guides (3,000+ lines each) covering AI evaluation end-to-end — for Engineers, PMs, and QAs:

| Guide | Platforms Covered | What's Inside |
|-------|------------------|---------------|
| [AI Evals: Comprehensive Study Guide](ai_evals_comprehensive_study_guide.md) | Arize Phoenix + Langfuse | LLM-as-a-Judge, RAG eval, multi-turn eval, production safety, statistical correction with `judgy`, 30-day learning path |
| [AI Evals: LangWatch + Langfuse Guide](ai_evals_complete_guide_langwatch_langfuse.md) | LangWatch + Langfuse | Same syllabus with LangWatch's 40+ built-in evaluators, side-by-side platform comparisons, platform choice guidance |

**Topics covered across both guides:**
- Tracing and observability setup (Phoenix, LangWatch, Langfuse)
- Error analysis: open coding → axial coding → failure mode taxonomy
- Building LLM judges with Train/Dev/Test split and ground truth calibration
- Code-based evaluators (regex, JSON schema, format validators)
- RAG-specific evals: faithfulness, context recall, answer relevance
- Multi-step pipeline evaluation and multi-turn conversation eval
- Production guardrails, safety monitoring, real-time drift detection
- Statistical correction with `judgy` library
- Human annotation best practices and inter-rater reliability
- Cost/latency optimization for eval pipelines at scale

---

## 🎓 For Interview Prep

AI system design interviews ask questions like:

> "Design a multi-tenant RAG system where competitors cannot see each other's data."

> "Your agent takes 15 steps for a 3-step task. How do you debug it?"

This guide gives you **concrete patterns**, **real tradeoffs**, and **production failure modes**: the depth interviewers expect at senior levels.

➡️ Start with [Interview Prep](00-interview-prep/)

---

## 🔄 Living Book

This guide tracks:
- New model releases and real-world performance
- Emerging patterns (MCP, Agentic RAG, Flow Engineering)
- Updated pricing and rate limits
- Deprecations and best practice changes

**⭐ Star and Watch** to get notified when updates are pushed.

---

## 🤝 Contributing

Found outdated info? Have production experience to share? PRs welcome.
See [Contributing Guide](CONTRIBUTING.md).

---

## 📄 License

MIT License. See [LICENSE](LICENSE).

---

<p align="center">
  <b>Built by <a href="https://github.com/ombharatiya">Om Bharatiya</a></b><br/>
  <a href="https://github.com/ombharatiya"><img src="https://img.shields.io/badge/GitHub-Follow-181717?logo=github" alt="GitHub"></a>
  <a href="https://x.com/ombharatiya"><img src="https://img.shields.io/badge/Twitter-Follow-1DA1F2?logo=twitter" alt="Twitter"></a>
  <a href="https://linkedin.com/in/ombharatiya"><img src="https://img.shields.io/badge/LinkedIn-Connect-0A66C2?logo=linkedin" alt="LinkedIn"></a>
</p>

<p align="center"><i>Last updated: March 2026</i></p>
