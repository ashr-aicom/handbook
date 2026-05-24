# OpenCode Go — Model Selection Guide

> **Important:** This is a point-in-time snapshot. Models are added and deprecated regularly. Run a model audit (`@lead audit models`) to detect drifts between this document and the live catalog at `https://opencode.ai/zen/go/v1/models`.
>
> **Pricing:** $5 first month, then $10/month. Limits: $12/5hr, $30/week, $60/month (in dollar value, not request count).
> Config format: `opencode-go/<model-id>` (e.g. `opencode-go/qwen3.6-plus`).

---

## 1. Quick Reference

| Model | Best For | Context | Requests / 5hr | Quality |
|---|---|---|---|---|
| **Qwen3.5 Plus** | Max volume, simple tasks | ~128K | **10,200** | ★★☆☆☆ |
| **Qwen3.6 Plus** | General default | ~256K | 3,300 | ★★★☆☆ |
| **MiniMax M2.5** | Cost-efficient coding, architect planning | ~200K | 6,300 | ★★★★☆ |
| **MiniMax M2.7** | Real-world SWE, self-evolution agents | ~200K | 3,400 | ★★★★☆ |
| **DeepSeek V4 Flash** | Budget high-quality coding, 1M ctx | 1M | 7,450 | ★★★★☆ |
| **DeepSeek V4 Pro** | Best raw coding, 1M ctx | 1M | 1,300 | ★★★★★ |
| **Kimi K2.5** | Strong reasoning, multilingual | ~256K | 1,850 | ★★★★☆ |
| **Kimi K2.6** | Agentic coding, swarm agents, vision | ~256K | 1,150 | ★★★★★ |
| **GLM-5** | Reasoning specialist, long-horizon | ~200K | 1,150 | ★★★★☆ |
| **GLM-5.1** | Max quality, 8hr autonomous tasks | ~200K | 880 | ★★★★★ |
| **MiMo-V2.5** | Multimodal on a budget, 1M ctx | 1M | 2,150 | ★★★☆☆ |
| **MiMo-V2.5-Pro** | Premium agentic coding, 1M ctx | 1M | 1,290 | ★★★★☆ |
| **MiMo-V2-Pro** | Agentic workflows, tool calls | 1M | 1,290 | ★★★★☆ |
| **MiMo-V2-Omni** | Full multimodal (image+video+audio) | ~256K | 2,150 | ★★★★☆ |

Requests per 5hr are estimates based on typical coding agent usage patterns. Higher-cost models yield fewer requests within the same $12 limit window.

---

## 2. Detailed Model Profiles

### GLM-5.1 — Maximum Quality, Maximum Endurance
- **Vendor:** Z.ai (formerly Zhipu AI)
- **Model ID:** `glm-5.1`
- **Architecture:** 754B total / 40B active MoE, 200K context, 128K max output
- **Strengths:** #1 on SWE-Bench Pro at release (58.4%). Can sustain autonomous execution for up to 8 hours — planning, testing, fixing, and iterating without human intervention. Strong on NL2Repo (42.7%, #1 globally) and CyberGym (68.7%).
- **Key benchmarks:** SWE-Bench Verified 77.8%, Terminal-Bench 2.0 63.5%, GPQA Diamond 86.2%, AIME 2026 95.3%
- **Best for:** Long-horizon autonomous coding (leave it running for hours), complex engineering optimization, sustained refactor pipelines, kernel optimization.
- **Trade-off:** Most expensive in the Go catalog (~880 requests per 5hr). 200K context is not class-leading. Text-only.

### GLM-5 — Strong Reasoning Specialist
- **Vendor:** Z.ai
- **Model ID:** `glm-5`
- **Architecture:** 744B total / 40B active MoE, 200K context
- **Strengths:** Best-in-class open-source on reasoning, coding, and agentic tasks. Uses DeepSeek Sparse Attention (DSA) for efficient long-context. Strong multilingual support.
- **Key benchmarks:** SWE-Bench Verified 77.8%, Terminal-Bench 2.0 56.2%, HLE 30.5, GPQA Diamond 86.0%
- **Best for:** Complex multi-step reasoning, systems engineering, algorithm design, planning-heavy tasks.
- **Trade-off:** Outperformed by GLM-5.1 on most agentic benchmarks. 200K context.

### Kimi K2.6 — Best Agentic Coding, Vision Support
- **Vendor:** Moonshot AI
- **Model ID:** `kimi-k2.6`
- **Architecture:** 1T total / 32B active MoE, 256K context, text + image input
- **Strengths:** Excellent long-horizon agents, native 300-agent swarms, multimodal input (image + video via MoonViT vision encoder). SWE-Bench Verified 80.2%. Strong on web research and multilingual coding.
- **Key benchmarks:** SWE-Bench Verified 80.2%, Terminal-Bench 2.0 66.7%, LiveCodeBench 89.6%
- **Best for:** Autonomous multi-agent systems, vision-grounded UI development, long-running agent sessions, multilingual projects.
- **Trade-off:** 256K context is smaller than DeepSeek's 1M. Priced higher than DeepSeek V4 Flash in the Go catalog (~1,150 requests per 5hr).

### Kimi K2.5 — Strong Reasoning, Good Value
- **Vendor:** Moonshot AI
- **Model ID:** `kimi-k2.5`
- **Architecture:** 1T total / 32B active MoE, 256K context, text + image input
- **Strengths:** Predecessor to K2.6. Strong reasoning for complex tasks, multilingual coding.
- **Key benchmarks:** SWE-Bench Verified 65.8%, SWE-Bench Multilingual 47.3%
- **Best for:** Cost-effective alternative to K2.6 for complex reasoning when vision is not needed. Higher request count than K2.6 (~1,850 per 5hr vs 1,150).
- **Trade-off:** Weaker than K2.6 and DeepSeek V4 Pro on coding benchmarks.

### DeepSeek V4 Pro — Best Raw Coding Performance
- **Vendor:** DeepSeek
- **Model ID:** `deepseek-v4-pro`
- **Architecture:** 1.6T total / 49B active MoE, 1M context, text-only
- **Strengths:** SWE-Bench Verified 80.6%, LiveCodeBench 93.5%, Codeforces rating 3206. The 1M token context window enables full-codebase analysis without chunking. Hybrid CSA+HCA attention makes 1M context inference economically viable. 384K max output.
- **Key benchmarks:** SWE-Bench Verified 80.6%, LiveCodeBench 93.5%, Terminal-Bench 2.0 67.9%, HLE 37.7%, GPQA Diamond 90.1%, MMLU-Pro 87.5
- **Best for:** Agentic coding, competitive programming, full-codebase reasoning, long-context processing where top quality matters.
- **Trade-off:** Premium cost (~1,300 requests per 5hr). Text-only — no multimodal support.

### DeepSeek V4 Flash — Best Value for High-Quality Coding
- **Vendor:** DeepSeek
- **Model ID:** `deepseek-v4-flash`
- **Architecture:** 284B total / 13B active MoE, 1M context, text-only
- **Strengths:** Flash@max ≈ Pro@high on reasoning tasks, but output tokens are ~12.4x cheaper at sticker. Same 1M context as Pro. Fast inference (~81.3 tok/s vs Pro's ~35.6). SWE-Bench Verified ~79.0%.
- **Key benchmarks:** SWE-Bench Verified ~79.0%, LiveCodeBench 91.6%, Terminal-Bench 2.0 56.9%, Codeforces 3,052
- **Best for:** High-volume code generation, PR review automation, CI pipelines, sub-agent workloads, latency-sensitive IDE integration.
- **Trade-off:** Smaller gap vs Pro on Terminal-Bench (56.9 vs 67.9). Weaker on complex agent tasks, BrowseComp, and frontier reasoning benchmarks.

### MiMo-V2.5-Pro — Flagship Xiaomi Agent
- **Vendor:** Xiaomi
- **Model ID:** `mimo-v2.5-pro`
- **Architecture:** 1.02T total / 42B active MoE, 1M context, hybrid SWA+GA attention
- **Strengths:** Significant improvement over V2-Pro. 40-60% fewer tokens for same tasks. Sustains thousands of tool calls with goal coherence. Harness-aware — adapts to execution environment. 3 built-in MTP modules for ~3x decode throughput.
- **Key benchmarks:** SWE-bench Pro 57.2%, HumanEval+ 75.6%, MMLU 89.4%
- **Best for:** Long-horizon agents with 100+ tool calls per session, whole-codebase analysis, complex multi-file editing.
- **Trade-off:** Relatively new — fewer independent benchmark reproductions available.

### MiMo-V2.5 — Budget Multimodal with 1M Context
- **Vendor:** Xiaomi
- **Model ID:** `mimo-v2.5`
- **Architecture:** 310B total / 15B active MoE, 1M context
- **Strengths:** Full-modal understanding (text + image + video + audio) in a single model. Large 1M context. Good cost efficiency (~2,150 requests per 5hr).
- **Best for:** Applications needing multimodal understanding at lower cost, long-document RAG, mixed-media coding tasks.
- **Trade-off:** Smaller active parameter count than Pro variants — lower reasoning quality.

### MiMo-V2-Pro — Agentic Coding Workhorse
- **Vendor:** Xiaomi
- **Model ID:** `mimo-v2-pro`
- **Architecture:** 1T+ total / 42B active MoE, 1M context, text + image input
- **Strengths:** Coding surpasses Claude Sonnet 4.6. Strong tool-call stability and accuracy. ClawEval 61.5, approaching Opus 4.6 (66.3). 7:1 hybrid attention ratio for efficient long-context.
- **Best for:** Autonomous coding agents, multi-step workflow orchestration, code review and refactoring, tool-heavy agent pipelines.
- **Trade-off:** Degrades past 128K context on complex tasks (improved in V2.5-Pro). Text-focused (V2-Omni for multimodal needs).

### MiMo-V2-Omni — Full Multimodal Agent
- **Vendor:** Xiaomi
- **Model ID:** `mimo-v2-omni`
- **Architecture:** 256K context, unified multimodal (text + image + video + audio)
- **Strengths:** Natively processes all modalities — not separate pipelines bolted together. 10+ hours continuous audio understanding. Benchmarks surpass Gemini 3 Pro on perceptual tasks. Structured tool calling and UI grounding built in.
- **Key benchmarks:** BigBench Audio 94.0, MMAU-Pro 69.4, image understanding surpasses Opus 4.6
- **Best for:** Browser automation with visual grounding, video analysis agent workflows, audio transcription and analysis, multimodal document parsing.
- **Trade-off:** Smaller context (256K) than Pro variants. Lower pure-reasoning performance than text-only models.

### MiniMax M2.7 — Real-World Engineering, Self-Evolving
- **Vendor:** MiniMax
- **Model ID:** `minimax-m2.7`
- **Architecture:** 230B total / 10B active MoE, ~200K context, Lightning Attention
- **Strengths:** Strong real-world software engineering — log analysis, bug troubleshooting, SRE-level decisions. SWE-Pro 56.22% (matches GPT-5.3-Codex). VIBE-Pro 55.6%. Supports Agent Teams for multi-agent collaboration. Self-evolution capability during development.
- **Key benchmarks:** SWE-Pro 56.22%, VIBE-Pro 55.6%, Terminal-Bench 2 57.0%, GDPval-AA 1495 ELO (#1 open-weight), MLE Bench Lite 66.6%
- **Best for:** Production incident resolution, end-to-end project delivery, ML competition tasks, Office document generation (Excel/PPT/Word), multi-agent systems.
- **Trade-off:** Smaller active parameters (10B). Context is 200K, not 1M.

### MiniMax M2.5 — SOTA Coding, Best Value Among High-Quality
- **Vendor:** MiniMax
- **Model ID:** `minimax-m2.5`
- **Architecture:** MoE with Lightning Attention, ~200K context, 8 experts per token
- **Strengths:** SWE-Bench Verified 80.2% (SOTA open-source at release). Architect-level planning — decomposes tasks before coding. 37% faster than M2.1 on SWE-Bench. Outperforms Opus 4.6 on Droid (79.7 vs 78.9) and OpenCode (76.1 vs 75.9) scaffolds. Trained on 13+ languages.
- **Key benchmarks:** SWE-Bench Verified 80.2%, BrowseComp 76.3%, Multi-SWE-Bench 51.3%, IFBench 70.0
- **Best for:** Full-stack development across Web/Android/iOS/Windows, architect-level project planning, cost-efficient high-quality coding.
- **Trade-off:** 200K context (less than DeepSeek/GLM). No official parameter count disclosure.

### Qwen3.6 Plus — Recommended All-Round Default
- **Vendor:** Alibaba (Qwen)
- **Model ID:** `qwen3.6-plus`
- **Architecture:** Hybrid linear attention + MoE, up to ~256K context (text-only), OpenAI-compatible endpoint
- **Strengths:** Best quality-to-cost ratio for general coding. 3,300 requests per 5hr. Improved over Qwen3.5 Plus with stronger reasoning.
- **Best for:** General-purpose coding default, daily development tasks, when you want good enough quality with a large request budget.
- **Trade-off:** Outperformed by DeepSeek V4 Pro, GLM-5.1, and Kimi K2.6 on complex coding benchmarks.

### Qwen3.5 Plus — Volume King
- **Vendor:** Alibaba (Qwen)
- **Model ID:** `qwen3.5-plus`
- **Architecture:** MoE, ~128K context
- **Strengths:** **Best value by far** — 10,200 requests per 5hr! Suitable for high-volume simple-to-moderate coding tasks where you need massive throughput.
- **Best for:** Simple code edits, boilerplate generation, quick one-shot questions, when you need 3-10x more requests than other models.
- **Trade-off:** Weakest on complex coding benchmarks. Limited context (128K). Not suitable for agentic workflows or complex multi-step tasks.

---

## 3. Decision Guide

### By Task Type

| Task | Primary Pick | Budget Alternative |
|---|---|---|
| **Agentic coding (complex, multi-step)** | DeepSeek V4 Pro | DeepSeek V4 Flash |
| **Long-horizon autonomous tasks (hours)** | GLM-5.1 | MiniMax M2.7 |
| **General daily coding** | Qwen3.6 Plus | Qwen3.5 Plus |
| **Full-codebase analysis (large repos)** | DeepSeek V4 Pro (1M ctx) | DeepSeek V4 Flash (1M ctx) |
| **Architect-level design + implementation** | MiniMax M2.5 | GLM-5 |
| **Multi-agent swarm systems** | Kimi K2.6 | MiniMax M2.7 |
| **Vision/multimodal coding (UI, images)** | MiMo-V2-Omni (text+image+video+audio) | Kimi K2.6 (text+image) |
| **Browser automation / web agents** | MiMo-V2-Omni | Kimi K2.6 |
| **Production debugging, SRE, logs** | MiniMax M2.7 | MiniMax M2.5 |
| **Maximum volume, simple tasks** | Qwen3.5 Plus | — |
| **ML/data science tasks** | MiniMax M2.7 | GLM-5.1 |
| **Competitive programming** | DeepSeek V4 Pro | DeepSeek V4 Flash |
| **Long-document RAG** | DeepSeek V4 Flash (1M + low cost) | MiMo-V2.5 (1M + multimodal) |

### By Budget Priority

| Priority | Model | Requests / 5hr |
|---|---|---|
| **Absolute cheapest** | Qwen3.5 Plus | 10,200 |
| **Best value quality** | Qwen3.6 Plus | 3,300 |
| **Quality on a budget** | MiniMax M2.5 | 6,300 |
| **Quality + 1M context** | DeepSeek V4 Flash | 7,450 |
| **Max quality** | DeepSeek V4 Pro / GLM-5.1 | 1,300 / 880 |

### By Context Window Needs

| Need | Model | Context |
|---|---|---|
| **1M-token context** | DeepSeek V4 Pro, DeepSeek V4 Flash, MiMo-V2.5-Pro, MiMo-V2.5 | 1M |
| **256K context** | Kimi K2.5, Kimi K2.6, MiMo-V2-Omni, Qwen3.6 Plus | 256K |
| **200K context** | GLM-5, GLM-5.1, MiniMax M2.5, MiniMax M2.7 | 200K |
| **128K context** | Qwen3.5 Plus | 128K |

---

## 4. Usage Limits & Request Budgeting

| Model | Requests / 5hr | Requests / Week | Requests / Month |
|---|---|---|---|
| GLM-5.1 | 880 | 2,150 | 4,300 |
| GLM-5 | 1,150 | 2,880 | 5,750 |
| Kimi K2.6 | 1,150 | 2,880 | 5,750 |
| Kimi K2.5 | 1,850 | 4,630 | 9,250 |
| MiMo-V2.5-Pro | 1,290 | 3,225 | 6,450 |
| MiMo-V2-Pro | 1,290 | 3,225 | 6,450 |
| MiMo-V2.5 | 2,150 | 5,450 | 10,900 |
| MiMo-V2-Omni | 2,150 | 5,450 | 10,900 |
| MiniMax M2.7 | 3,400 | 8,500 | 17,000 |
| MiniMax M2.5 | 6,300 | 15,900 | 31,800 |
| Qwen3.6 Plus | 3,300 | 8,200 | 16,300 |
| Qwen3.5 Plus | 10,200 | 25,200 | 50,500 |
| DeepSeek V4 Pro | 1,300 | 3,250 | 6,500 |
| DeepSeek V4 Flash | 7,450 | 18,600 | 37,300 |

**Note:** Limits are in dollar value ($12/5hr, $30/week, $60/month). Actual request counts vary with your token usage patterns. Estimates above use typical coding agent request shapes (input + cached + output).

---

## 5. Endpoints & API Compatibility

| Endpoint Format | Models |
|---|---|
| **OpenAI-compatible** (`/v1/chat/completions`) | GLM-5, GLM-5.1, Kimi K2.5, Kimi K2.6, DeepSeek V4 Pro, DeepSeek V4 Flash, MiMo-V2-Pro, MiMo-V2-Omni, MiMo-V2.5-Pro, MiMo-V2.5, Qwen3.6 Plus, Qwen3.5 Plus |
| **Anthropic-compatible** (`/v1/messages`) | MiniMax M2.5, MiniMax M2.7 |

All models accessible via: `https://opencode.ai/zen/go/v1/`

---

## 6. Key Takeaways

1. **Daily general coding → Qwen3.6 Plus**. Best quality-to-cost ratio for most tasks.
2. **High-volume simple tasks → Qwen3.5 Plus**. You get 3-10x more requests.
3. **Complex agentic coding → DeepSeek V4 Pro**. Best raw coding benchmarks at 1M context.
4. **Budget high-quality coding → DeepSeek V4 Flash**. Near-Pro quality, 5.7x more requests.
5. **Autonomous long tasks → GLM-5.1**. The 8-hour sustained execution is unique.
6. **Multimodal needs → MiMo-V2-Omni** (all modalities) or **Kimi K2.6** (text+image).
7. **Architect + full-stack → MiniMax M2.5**. Design-then-implement workflow, 80.2% SWE-bench.
8. **Production ops/SRE → MiniMax M2.7**. Log analysis, debugging, incident resolution.
