# 🎓 Recommended AI Courses & Learning Paths

A curated list of **reliable, trusted, and up-to-date** online courses for AI engineers, ML practitioners, and product teams. Every course here is verified as of **March 2026** — no fluff, no outdated MOOCs.

---

## Table of Contents

- [Foundation: LLMs & Transformers](#foundation)
- [RAG Pipelines](#rag)
- [Agentic AI & Multi-Agent Systems](#agents)
- [Context & Memory Management](#context-memory)
- [AI Evaluations & Observability](#evals)
- [Prompt Engineering & Context Engineering](#prompting)
- [Fine-tuning & Adaptation](#finetuning)
- [Inference Optimization & MLOps](#mlops)
- [AI Safety & Guardrails](#safety)
- [Coding Agents & Developer AI Tools](#coding-agents)
- [For PMs & Non-Engineers](#pm-track)
- [YouTube Channels & Free Content](#free)
- [Learning Path Suggestions](#paths)

---

## Foundation: LLMs & Transformers <a name="foundation"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[Neural Networks: Zero to Hero](https://www.youtube.com/playlist?list=PLAqhIrjkxbuWI23v9cThsA9GvCAUhRvKZ)** | Andrej Karpathy (YouTube) | Free | The definitive from-scratch series by an OpenAI/Tesla legend. Builds GPT from scratch. |
| **[CS324: Large Language Models](https://stanford-cs324.github.io/winter2022/)** | Stanford | Free | Stanford-quality lecture notes covering LLM fundamentals, scaling laws, alignment. |
| **[Generative AI with LLMs](https://www.coursera.org/learn/generative-ai-with-llms)** | DeepLearning.AI + AWS (Coursera) | ~$50 | Hands-on intro to LLMs, covering training, fine-tuning, RLHF. By Andrew Ng's team. |
| **[Practical Deep Learning for Coders](https://course.fast.ai/)** | fast.ai | Free | Bottom-up, code-first approach. Best for engineers who learn by doing. |

---

## RAG Pipelines <a name="rag"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[Building and Evaluating Advanced RAG](https://www.deeplearning.ai/short-courses/building-evaluating-advanced-rag/)** | DeepLearning.AI + LlamaIndex | Free | Covers sentence-window retrieval, auto-merging, RAG evaluation with TruLens. |
| **[Vector Databases: from Embeddings to Applications](https://www.deeplearning.ai/short-courses/vector-databases-embeddings-applications/)** | DeepLearning.AI + Weaviate | Free | Practical walkthrough of embeddings, vector stores, and hybrid search. |
| **[Building RAG Agents with LLMs](https://courses.nvidia.com/courses/course-v1:DLI+S-FX-15+V1/)** | NVIDIA Deep Learning Institute | Free | Enterprise-grade RAG with NVIDIA NIM. Covers chunking, reranking, evaluation. |
| **[LlamaIndex — Documentation: Learning](https://docs.llamaindex.ai/en/stable/understanding/)** | LlamaIndex | Free | Official LlamaIndex learning path — best for deep RAG pipeline mastery. |
| **[RAG Fundamentals (Haystack)](https://haystack.deepset.ai/tutorials)** | deepset / Haystack | Free | Hands-on tutorials for pipeline-based RAG using the Haystack framework. |

---

## Agentic AI & Multi-Agent Systems <a name="agents"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[AI Agents in LangGraph](https://www.deeplearning.ai/short-courses/ai-agents-in-langgraph/)** | DeepLearning.AI + LangChain | Free | LangGraph by the creators. Covers ReAct, persistence, human-in-the-loop, multi-agent. |
| **[Multi AI Agent Systems with crewAI](https://www.deeplearning.ai/short-courses/multi-ai-agent-systems-with-crewai/)** | DeepLearning.AI + crewAI | Free | Official CrewAI course. Covers Crews, Flows, and real-world business automations. |
| **[Building Agentic RAG with LlamaIndex](https://www.deeplearning.ai/short-courses/building-agentic-rag-with-llamaindex/)** | DeepLearning.AI + LlamaIndex | Free | Routing, tool-calling agents, and multi-document agentic retrieval. |
| **[Functions, Tools and Agents with LangChain](https://www.deeplearning.ai/short-courses/functions-tools-agents-langchain/)** | DeepLearning.AI + LangChain | Free | Tool-calling, OpenAI function calling, building from scratch. |
| **[Developing AI Agents using AutoGen](https://www.deeplearning.ai/short-courses/ai-agentic-design-patterns-with-autogen/)** | DeepLearning.AI + Microsoft | Free | AutoGen multi-agent patterns. Covers debate, tool-use, and code execution agents. |
| **[CS294/194-196: LLM Agents (Berkeley)](https://rdi.berkeley.edu/llm-agents/f24)** | UC Berkeley | Free | Graduate-level course on LLM agents. Covers memory, planning, safety, evaluation. |

---

## Context & Memory Management <a name="context-memory"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[Building Systems with the ChatGPT API](https://www.deeplearning.ai/short-courses/building-systems-with-chatgpt/)** | DeepLearning.AI + OpenAI | Free | Covers multi-turn conversation state, context management, moderation chains. |
| **[Prompt Engineering with Llama 2](https://www.deeplearning.ai/short-courses/prompt-engineering-with-llama-2/)** | DeepLearning.AI + Meta | Free | Shows context window tradeoffs and system prompt management with Llama 2. |
| **[Reasoning with o1](https://www.deeplearning.ai/short-courses/reasoning-with-o1/)** | DeepLearning.AI + OpenAI | Free | Deep dive into o1 reasoning, budget tokens, thinking modes. Directly applicable to o3 and Claude 3.7 Extended Thinking. |
| **[Mem0 Documentation](https://docs.mem0.ai/)** | Mem0 (official) | Free | Definitive reference for multi-tier memory in production agents. |

---

## AI Evaluations & Observability <a name="evals"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[Evals for AI: Maven Course](https://maven.com/hamel-shreya/evals-for-ai)** | Hamel Husain & Shreya Shankar (Maven) | Paid (~$400) | Industry gold standard for AI evals. Used in production at dozens of companies. Our repo's evals guides are based on this course. |
| **[Evaluating and Debugging Generative AI](https://www.deeplearning.ai/short-courses/evaluating-debugging-generative-ai/)** | DeepLearning.AI + W&B | Free | Covers tracing, evaluation with W&B Weave, and experiment tracking. |
| **[Quality and Safety for LLM Applications](https://www.deeplearning.ai/short-courses/quality-safety-llm-applications/)** | DeepLearning.AI + WhyLabs | Free | Covers hallucination detection, toxicity, bias evaluation, and drift monitoring. |
| **[LangSmith Evaluation Tutorials](https://docs.smith.langchain.com/evaluation)** | LangChain | Free | Official LangSmith docs are the best hands-on eval reference if you use the LangChain ecosystem. |
| **[Phoenix + Langfuse official docs](https://docs.arize.com/phoenix)** | Arize Phoenix | Free | Hands-on tutorials for open-source evals using Phoenix. |

> 📖 Also see this repo's companion guides:
> - [AI Evals: Comprehensive Study Guide (Phoenix + Langfuse)](ai_evals_comprehensive_study_guide.md)
> - [AI Evals: LangWatch + Langfuse Guide](ai_evals_complete_guide_langwatch_langfuse.md)

---

## Prompt Engineering & Context Engineering <a name="prompting"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[ChatGPT Prompt Engineering for Developers](https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/)** | DeepLearning.AI + OpenAI | Free | The foundational prompt engineering course. By Isa Fulford & Andrew Ng. |
| **[Prompting Fundamentals (Anthropic)](https://www.anthropic.com/learn)** | Anthropic | Free | Straight from the Claude team. Covers prompt design, XML tags, chain-of-thought. |
| **[DSPy: Building Optimizable Pipelines](https://github.com/stanfordnlp/dspy)** | Stanford NLP (GitHub) | Free | Not a course but the DSPy repo's notebooks are the best way to learn programmatic prompting. |
| **[Prompt Engineering Guide](https://www.promptingguide.ai/)** | DAIR.AI | Free | Comprehensive, community-maintained reference covering all major prompting techniques. |

---

## Fine-tuning & Adaptation <a name="finetuning"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[Finetuning Large Language Models](https://www.deeplearning.ai/short-courses/finetuning-large-language-models/)** | DeepLearning.AI + Lamini | Free | Covers LoRA, full fine-tuning, dataset preparation, evaluation. Concise and practical. |
| **[Reinforcement Learning from Human Feedback](https://www.deeplearning.ai/short-courses/reinforcement-learning-from-human-feedback/)** | DeepLearning.AI + AWS | Free | Deep dive into RLHF: reward models, PPO, preference datasets. |
| **[Hugging Face NLP Course](https://huggingface.co/learn/nlp-course/chapter1/1)** | Hugging Face | Free | The best free course for fine-tuning transformers with the HF ecosystem (Trainer, PEFT, etc). |
| **[How Diffusion Models Work](https://www.deeplearning.ai/short-courses/how-diffusion-models-work/)** | DeepLearning.AI | Free | For image model fine-tuning (stable diffusion, LoRA for images). |

---

## Inference Optimization & MLOps <a name="mlops"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[ML Engineering for Production (MLOps)](https://www.coursera.org/specializations/machine-learning-engineering-for-production-mlops)** | DeepLearning.AI (Coursera) | Paid | 4-course specialization on production ML: deployment, monitoring, pipelines. |
| **[Efficiently Serving LLMs](https://www.deeplearning.ai/short-courses/efficiently-serving-llms/)** | DeepLearning.AI + Predibase | Free | Covers vLLM, PagedAttention, quantization, LoRA serving. Exactly what this guide covers. |
| **[vLLM Documentation & Tutorial](https://docs.vllm.ai/en/latest/)** | vLLM | Free | The official vLLM docs are the most up-to-date reference for high-throughput serving. |

---

## AI Safety & Guardrails <a name="safety"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[Red Teaming LLM Applications](https://www.deeplearning.ai/short-courses/red-teaming-llm-applications/)** | DeepLearning.AI + Giskard | Free | Hands-on red teaming, prompt injection, jailbreak detection, bias testing. |
| **[AI Safety Fundamentals](https://aisafetyfundamentals.com/alignment-fast-track/)** | BlueDot Impact | Free | Most trusted free course on AI alignment and safety. Used by professionals at Anthropic, DeepMind. |
| **[NVIDIA AI Red Team (NEMO Guardrails)](https://github.com/NVIDIA/NeMo-Guardrails)** | NVIDIA | Free | Hands-on notebooks for building production guardrails with NeMo Guardrails. |

---

## Coding Agents & Developer AI Tools <a name="coding-agents"></a>

| Course | Provider | Cost | Why It's Trusted |
|--------|----------|------|-----------------|
| **[Claude Code — Official Docs](https://docs.anthropic.com/en/home)** | Anthropic | Free | The definitive starting point for Claude Code. Covers CLAUDE.md, SDK, and permissions. |
| **[Building Code Agents (Hugging Face)](https://huggingface.co/learn/agents-course/unit1/introduction)** | Hugging Face | Free | HuggingFace's official agents course — includes a unit on building code-execution agents. |
| **[Introduction to OpenHands](https://github.com/All-Hands-AI/OpenHands/blob/main/docs/getting-started.md)** | All-Hands AI | Free | Official getting-started guide for OpenHands autonomous coding agent. |

> 📖 Also see this repo's guides: [Claude Code Guide](09-frameworks-and-tools/09-claude-code.md) and [OpenCoder Landscape](09-frameworks-and-tools/10-opencoderguide.md)

---

## For PMs & Non-Engineers <a name="pm-track"></a>

These require no Python experience:

| Course | Provider | Cost | Why It's Good |
|--------|----------|------|--------------|
| **[AI for Everyone](https://www.coursera.org/learn/ai-for-everyone)** | DeepLearning.AI (Coursera) | Free | Andrew Ng's course for non-technical roles. Covers what AI can/can't do, project leadership. |
| **[Prompt Engineering for Everyone](https://learnprompting.org/)** | Learn Prompting | Free | Plain-English guide to prompt engineering for non-engineers. |
| **[Evals for AI (Maven)](https://maven.com/hamel-shreya/evals-for-ai)** | Hamel Husain & Shreya Shankar | Paid | Despite having code, this course is designed for PMs and QAs, not just engineers. Highly recommended. |
| **[AI Product Management](https://www.productschool.com/blog/product-management/ai-product-manager/)** | Product School | Free (blog) | Practical guide for PMs building AI-powered products. |
| **[Google: Introduction to Generative AI](https://cloud.google.com/learn/training/machinelearning-ai)** | Google Cloud Skills Boost | Free | No-code introduction to generative AI, LLMs, and responsible AI. |

---

## YouTube Channels & Free Content <a name="free"></a>

| Channel / Resource | Focus | Why Follow |
|-------------------|-------|------------|
| **[Andrej Karpathy](https://www.youtube.com/@AndrejKarpathy)** | Foundations, transformers | Best explanations of how LLMs actually work |
| **[Yannic Kilcher](https://www.youtube.com/@YannicKilcher)** | Paper reviews | Clear walkthroughs of latest ML research papers |
| **[Aleksa Gordić - The AI Epiphany](https://www.youtube.com/@TheAIEpiphany)** | Paper reviews | Deep technical paper breakdowns |
| **[AI Jason](https://www.youtube.com/@AIJasonZ)** | Agents, LangChain, practical | Great intro videos for agentic frameworks |
| **[Sam Witteveen](https://www.youtube.com/@samwitteveenai)** | Gemini, RAG, agents | One of the best practical AI YouTubers |
| **[Matt Wolfe](https://www.youtube.com/@mreflow)** | AI news, product demos | Best for staying current on AI news and tools |
| **[Hamel Husain (blog)](https://hamel.dev/)** | Evals, production AI, LLMs | Real production insights from the author of the evals maven course |
| **[Simon Willison (blog)](https://simonwillison.net/)** | LLM news, tools, coding | The most trustworthy daily AI news source |
| **[The Latent Space podcast](https://www.latent.space/)** | Technical AI interviews | Best technical AI podcast — deep dives with researchers |
| **[Lex Fridman Podcast](https://lexfridman.com/podcast/)** | Broad AI/ML interviews | Long-form interviews with leading AI researchers |

---

## Learning Path Suggestions <a name="paths"></a>

### 🛤️ Path: "I'm new to AI and want to build things fast"

```
Week 1: Prompt Engineering for Developers (DeepLearning.AI) — free, 2 hrs
Week 2: Building Systems with ChatGPT API (DeepLearning.AI) — free, 2 hrs
Week 3: Building and Evaluating Advanced RAG (DeepLearning.AI) — free, 2 hrs
Week 4: AI Agents in LangGraph (DeepLearning.AI) — free, 4 hrs
Month 2: Pick a real project, use this guide as reference
```

### 🛤️ Path: "I want to understand LLMs deeply"

```
Week 1-3: Neural Networks: Zero to Hero (Karpathy) — free, 12+ hrs
Week 4-6: CS324 Stanford LLMs — free, 30+ hrs
Month 2: Generative AI with LLMs (Coursera DeepLearning.AI)
Month 3: CS294 LLM Agents (Berkeley)
```

### 🛤️ Path: "I want to build production-ready AI evaluation"

```
Week 1: Evaluating and Debugging Generative AI (DeepLearning.AI + W&B) — free
Week 2: This repo's evals guides (Phoenix/Langfuse) — free  ← start here
Week 3-4: Quality and Safety for LLM Applications (DeepLearning.AI)
Month 2: Evals for AI (Maven, Hamel + Shreya) — paid, worth it
```

### 🛤️ Path: "I'm a PM learning to contribute to AI product quality"

```
Week 1: AI for Everyone (Coursera) — free
Week 2: Prompt Engineering for Everyone (learnprompting.org) — free
Week 3: AI Evals guide in this repo — free (especially Chapters 1-3 on error analysis)
Month 2: Evals for AI (Maven) — paid, has PM track
```

### 🛤️ Path: "I want to deploy coding agents in my team"

```
Day 1: Claude Code docs (anthropic.com) — free
Week 1: This repo's Claude Code Guide + OpenCoder Landscape Guide
Week 2: Building Code Agents (Hugging Face) — free
Month 1: Run Claude Code on a real project in CI
```

---

## How to Stay Current

AI moves fast. Beyond courses, these habits keep you current:

1. **Follow Simon Willison's blog** — daily, trustworthy AI news summaries
2. **Read Anthropic + OpenAI release notes** — primary sources beat second-hand summaries
3. **Watch the Latent Space podcast** — best technical depth
4. **Contribute to open source** — OpenHands, LlamaIndex, DSPy — real learning happens in PRs
5. **Star this repo** — we update it as the landscape changes ⭐

---

*Maintained by [Om Bharatiya](https://github.com/ombharatiya). PRs welcome for new course additions!*
