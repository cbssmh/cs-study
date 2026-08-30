# 1. Title

Tencent Releases and Open-Sources Tencent Hy4 Preview

# 2. Source

- Author / Organization: Tencent
- Link: https://www.tencent.com/tencent-releases-and-open-sources-tencent-hy4-preview/
- Date: 2026-08-28
- Discussion: Hacker News — “Hy4 preview”

# 3. One-line Summary

Tencent's Hy4 preview combines a 770B-parameter MoE architecture, 1M+ token context, low-cost inference, and AI-assisted optimization of its own training and inference stack, signaling an early move toward recursive AI-assisted model development.

# 4. Key Points

- Hy4 preview has 770B total parameters but activates 49B parameters per inference, indicating a large Mixture-of-Experts-style architecture.
- Context length exceeds 1M tokens, targeting long-context coding, document analysis, research, and agentic workflows.
- Tencent positions the model around practical productivity: software engineering, office work, financial analysis, game development, and scientific research.
- In Tencent's internal blind evaluation of 203 engineering tasks by 163 experts, Hy4 scored 2.99/4.00 versus Kimi K3 at 2.94 and GLM-5.3 at 2.92.
- Hy4 participated in optimizing its own development pipeline, including training methods, data strategies, evaluation frameworks, and low-level operators.
- Tencent describes this human-supervised feedback loop as an early-stage form of "recursive self-improvement."
- Hy4 also analyzed inference bottlenecks and helped optimize operator fusion and communication, producing a reported 31.8% end-to-end throughput improvement over the baseline.
- API pricing is $0.834/M input tokens, $2.501/M output tokens, and $0.042/M cached tokens, making cache hits roughly 5% of normal input cost.
- Hacker News discussion highlights cache pricing, cache TTL, and provider routing as increasingly important costs for long-running agents.
- The community also debated compressed "caveman-style" reasoning traces and whether shorter internal representations can reduce inference cost without sacrificing reasoning quality.

# 5. Deep Dive (Structured Understanding)

## Problem

Frontier-scale LLM development faces two related bottlenecks.

First, improving models requires increasingly expensive human-led experimentation across data selection, training methods, evaluation, kernels, and inference infrastructure.

Second, agentic applications repeatedly process very large contexts. Model quality alone therefore does not determine usability; inference throughput, context length, caching behavior, and token cost become major constraints.

## Approach

Tencent scaled Hy4 across several dimensions:

- 770B total / 49B active parameters
- 1M+ token context
- expanded training data
- domain-specific expert data
- product-integrated training and evaluation
- aggressive inference and cache-cost optimization

More unusually, Tencent inserted Hy4 itself into parts of the model-development loop.

The model could propose optimization approaches, run experiments, inspect results, and use resulting code, logs, and feedback to inform subsequent experiments.

The same idea was applied below the model layer: Hy4 analyzed inference bottlenecks and proposed optimizations to low-level execution.

## Key Insight

The significant change is not simply that AI can write code used to build another AI.

The development loop is becoming:

Human defines objective and infrastructure  
→ AI proposes changes  
→ experiments execute  
→ measurements return  
→ AI interprets results  
→ another experiment is generated.

This converts parts of AI research and systems optimization into an increasingly automated search process.

A second insight emerges from the Hacker News discussion: for long-running agents, the economically relevant unit is no longer simply output-token price.

Repeated context processing means:

**effective agent cost ≈ inference price + cache hit rate + cache price + cache TTL + routing behavior**

This makes infrastructure efficiency increasingly important alongside benchmark intelligence.

## Result / Impact

Tencent reports a 31.8% improvement in end-to-end inference throughput from AI-assisted infrastructure optimization.

If this workflow generalizes, stronger models could accelerate experiments used to create subsequent models, shortening AI R&D iteration cycles.

At the deployment layer, inexpensive cached context could make persistent coding and productivity agents substantially cheaper to operate.

# 6. Why It Matters

Hy4 illustrates a shift from **AI as the product** toward **AI as part of the machinery that creates the next product**.

The important feedback loop is:

**better model → better AI-assisted R&D → faster experiments → better model**

This does not constitute autonomous recursive self-improvement in the strong sense: humans still control objectives, infrastructure, training, and deployment.

However, automating more of the experimentation loop can still compress development cycles without requiring fully autonomous self-modification.

Hy4 also reflects another industry transition: model competition is expanding beyond benchmark scores toward total agent economics.

Context caching, inference throughput, active parameter count, provider routing, and cache persistence can materially determine the cost of running long-lived agents.

# 7. Critical Analysis

- Tencent's benchmark results are internally produced. Independent evaluations are necessary before treating small differences such as 2.99 versus 2.94 as meaningful.
- "Recursive self-improvement" is potentially overstated. Hy4 did not independently redesign and retrain itself; it participated inside a human-designed experimentation pipeline.
- The reported 31.8% throughput gain demonstrates useful optimization but does not prove accelerating improvements in model intelligence.
- Model-generated reasoning traces should not automatically be interpreted as faithful representations of the model's internal computation.
- Claims that compressed "caveman" reasoning improves efficiency remain speculative in the Hacker News discussion; shorter reasoning could also remove useful exploration or verification.
- Tencent calls Hy4 "open-source," but the community disputes whether released weights without the complete training data and reproducible training pipeline should instead be described as open weights.
- A 1M+ context window describes capacity, not necessarily reliable reasoning across the entire context.
- Cache price alone is insufficient for estimating real agent costs. Cache TTL, hit rate, provider consistency, and routing behavior can dominate actual spending.
- Preview-model performance may not predict production reliability; Hacker News users reported provider availability, timeout, and rate-limit concerns around current open-model infrastructure.

# 8. Connections

### 1. Recursive Self-Improvement and Automated AI Research

Hy4 connects to the broader idea of AI systems participating in AI R&D.

The important intermediate stage before fully autonomous self-improvement is automated experimentation: models generate hypotheses, modify code, run evaluations, inspect measurements, and repeat.

This resembles an AI-driven optimization/search loop more than an AI directly rewriting its own intelligence.

### 2. Coding Agents and Software Engineering Automation

Coding agents already follow a similar structure:

**inspect → plan → modify → test → observe → retry**

Hy4 extends the same agentic pattern from application software toward the infrastructure and training systems used to build AI models themselves.

The distinction between "coding agent" and "AI researcher" therefore becomes increasingly dependent on the tools, evaluation environment, and autonomy provided to the model.

### 3. Mixture of Experts and Inference Economics

Hy4's 770B total versus 49B active parameters reflects the Mixture-of-Experts strategy: maintain very large model capacity while activating only a subset during each inference step.

This connects model architecture directly to serving economics.

Future competition may increasingly optimize:

**capability / active compute / latency / dollar**

rather than raw parameter count.

### 4. Prompt Caching and Persistent Agents

Traditional chatbot interactions contain relatively small contexts.

Agentic coding workflows may repeatedly process repositories, specifications, tool outputs, and long conversation histories.

Consequently, prompt caching changes from a minor API feature into an infrastructure primitive for persistent agents.

A cheap model with poor cache persistence can potentially cost more than a nominally expensive model with effective caching.

### 5. Compressed Reasoning and Representation Efficiency

The discussion around Hy4's terse reasoning traces connects to a deeper question: natural language may be an inefficient intermediate representation for machine reasoning.

If models can learn denser internal or generated representations, they may perform equivalent reasoning with fewer sequential tokens.

The unresolved trade-off is whether compression preserves reasoning depth or removes useful exploratory computation.

# 9. Keywords

- Tencent Hy4
- Mixture of Experts (MoE)
- Recursive Self-Improvement
- AI-Assisted R&D
- Agentic AI
- Inference Optimization
- Prompt Caching
- Long Context
- Reasoning Compression
- Open Weights

# 10. TL;DR

Tencent Hy4 preview combines 770B total parameters, 49B active parameters, 1M+ context, and aggressive inference economics.
Its most notable feature is participation in its own development and inference optimization loop, including a reported 31.8% throughput improvement.
The broader shift is from simply building better LLMs toward using LLMs to accelerate the engineering and research processes that build their successors.
