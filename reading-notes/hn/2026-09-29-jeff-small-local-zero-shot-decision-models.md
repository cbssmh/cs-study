## 1. Title

Jeff — Small Local Zero-Shot Decision Models with ~30 ms Inference

## 2. Source

- Author / Organization: firelex / Jeff
- Link: https://github.com/firelex/jeff
- Discussion: https://news.ycombinator.com/
- Date: 2026-09-29
- Source context: GitHub project README and Hacker News discussion. 

## 3. One-line Summary

Jeff fine-tunes 0.8B–2B Qwen and Gemma models into fast, local, non-generative zero-shot classifiers that return calibrated option probabilities in one forward pass, trading large-model reasoning ability for roughly 20–60 ms decision latency.

## 4. Key Points

- Jeff turns small Qwen3.5 and Gemma 4 models into Jev-compatible decision models: provide a situation plus arbitrary textual options and receive probabilities rather than generated text.
- The models target zero-shot classification, allowing labels such as support queues, intents, moderation categories, commands, or actions without requiring those exact categories in training data.
- Jeff-Qwen3.5-0.8B reaches 79.1% and the 2B model 83.1% across five benchmarks; Jev's separately published result is 83.0%, but the samples were not identical.
- Performance is highly task-dependent: Jeff performs strongly on Financial PhraseBank and RAGTruth but remains substantially behind larger models on reasoning-heavy BBH, JudgeBench, and JevBench Hard.
- Jeff-Qwen3.5-0.8B reports median decision latency of 22 ms on an RTX PRO 6000 and 28 ms on an Apple M4 Max; CPU inference is much slower at 463 ms.
- Domain-specific fine-tuning can radically outperform zero-shot behavior: a voice-navigation experiment reportedly improved held-out accuracy from 31.7% to 95.8% using roughly 11,000 examples and about 30 minutes of GPU training.
- The entire training pipeline used local hardware, including one RTX PRO 6000 for training and open-model-generated synthetic data; closed-model output was excluded from training data.
- Model size does not monotonically predict practical performance: the 0.8B model performs better than the 2B variant in several game experiments despite lower aggregate benchmark accuracy.
- Prompt wording remains a major variable, meaning the system behaves less like a conventional fixed classifier than its simple API may suggest.
- Released models support at most 26 options because training did not teach them reliable two-letter answer codes beyond A–Z.

## 5. Deep Dive (Structured Understanding)

### Problem

General-purpose LLMs are frequently used for routing, intent detection, moderation, ranking, and other classification-like tasks even though these workloads usually require neither long-form generation nor extensive reasoning.

Traditional classifiers solve such problems cheaply once trained, but usually require task-specific labeled data and training. General-purpose LLMs provide much stronger zero-shot flexibility but introduce larger models, autoregressive decoding, API dependencies, and higher latency.

Jeff explores the middle ground: **Can a very small language model preserve useful zero-shot semantic generalization while behaving operationally like a conventional classifier?**

### Approach

Jeff starts from small Qwen3.5 and Gemma 4 models and fine-tunes them using the open AutoJev recipe.

Instead of generating an explanation or structured response token-by-token, the model evaluates predefined textual alternatives and produces their probabilities through a single forward pass.

The training recipe uses full-weight fine-tuning, cross-entropy over option labels, and fitted temperature scaling for probability calibration. Synthetic examples are produced locally by an open model, while development data rather than benchmark-panel results determines checkpoint selection.

The resulting architecture is intended to separate responsibilities:

**application code → reasoning/state construction → Jeff → fast decision**

rather than expecting the model itself to plan or perform multi-step reasoning.

### Key Insight

The interesting optimization is not simply "smaller LLM."

It is **removing generation from workloads where generation is unnecessary**.

For many application hot paths, the required output is one decision among known alternatives. A model optimized around that constraint can retain language understanding while avoiding much of the machinery and latency associated with generating arbitrary text.

The project also demonstrates that zero-shot capability and domain-specific classification are different operating points. Jeff can start as a generic decision model, then be fine-tuned when application-specific accuracy matters more than universal zero-shot flexibility.

### Result / Impact

Jeff-Qwen3.5-2B nearly matches Jev's published aggregate benchmark number, while the 0.8B variant reaches tens-of-milliseconds latency on modern local hardware. However, reasoning-heavy evaluations reveal a substantial gap from larger models.

The practical implication is therefore not "tiny models replace LLMs," but a more specialized architecture:

**deterministic code for explicit logic + small decision model for ambiguous semantic choices + larger LLM only when genuine reasoning or generation is required.**

## 6. Why It Matters

Jeff represents a broader shift from using one general-purpose generative model for every AI workload toward **heterogeneous inference systems**.

LLMs made zero-shot natural-language interfaces convenient enough that developers began using generation for tasks historically handled by classifiers. Decision models attempt to preserve that flexible interface while recovering classifier-like latency, cost, and deployability.

This is especially relevant to request-time infrastructure such as intent routing, agent tool selection, moderation, RAG routing, UI actions, policy selection, and context selection, where adding hundreds of milliseconds to every request can dominate application latency.

Local execution also changes the deployment model. A sub-billion-parameter decision model can potentially keep high-frequency semantic decisions on-device or inside private infrastructure instead of turning every decision into a remote API request.

## 7. Critical Analysis

The headline benchmark comparison with Jev requires caution. Jeff and Jev were evaluated on different samples of the same benchmark families, so 83.1% versus 83.0% is not a controlled head-to-head result. The project's author explicitly acknowledges this limitation.

Aggregate accuracy also hides large capability differences. Jeff performs extremely well on some classification and grounding datasets while trailing substantially on reasoning-oriented benchmarks. A single average therefore overstates interchangeability between these systems.

The reported ~30 ms latency is hardware-dependent. RTX PRO 6000 and M4 Max measurements demonstrate technical feasibility but do not establish equivalent latency on commodity CPUs, mobile NPUs, or production servers under concurrency. CPU latency for the 0.8B model is reported at 463 ms.

Prompt sensitivity is another operational concern. If semantically equivalent option wording changes decisions materially, calibration and regression testing become important parts of deployment.

The fine-tuning result from 31.7% to 95.8% is impressive but also weakens the pure zero-shot argument for specialized production tasks: once labeled data and training infrastructure exist, traditional embedding-based or linear classifiers become serious alternatives.

That alternative was repeatedly raised in the Hacker News discussion. Several practitioners reported strong results from frozen embeddings combined with logistic regression or SVM classifiers, sometimes with sub-millisecond inference. These are anecdotal community reports rather than controlled comparisons with Jeff, but they highlight the need for task-specific baselines.

A particularly important unanswered question is therefore not "Jeff versus Jev," but:

**At what level of task novelty does a zero-shot decision model outperform the engineering simplicity, accuracy, and latency of a conventional classifier?**

## 8. Connections

### 1. BERT / ModernBERT + Linear Classifiers

Encoder models followed by logistic regression or SVM classifiers already provide extremely fast semantic classification. The difference is primarily deployment flexibility: conventional classifiers usually require labeled task-specific data, whereas Jeff aims to interpret newly described classes zero-shot.

The HN discussion highlights this trade-off directly, including examples using frozen embeddings and lightweight classifiers.

### 2. LLM Routing and Mixture-of-Experts Systems

Jeff naturally fits hierarchical inference architectures:

**request → lightweight decision model → specialized model/tool → expensive LLM only when needed**

This resembles model routing and mixture-of-experts principles at the application layer. Cheap models handle common decisions while expensive reasoning capacity is invoked selectively.

### 3. Structured Outputs and Constrained Decoding

General-purpose LLMs can already return fixed schemas or restricted labels through constrained decoding. Jeff attacks the same application problem from the opposite direction: instead of constraining a generative model, it removes unnecessary generation from the decision path.

The trade-off becomes **generality versus specialized inference efficiency**, rather than simply structured versus unstructured output.

### 4. System 1 / System 2 AI Architecture

Jeff explicitly behaves like a "System 1" component: fast semantic judgment without deliberate multi-step planning.

A practical architecture can therefore separate:

**System 1:** classification, routing, scoring, filtering  
**System 2:** planning, reasoning, tool orchestration, complex generation

The project's game experiments reinforce this distinction: classification-like judgments can work well, while forecasting and multi-step reasoning remain weak.

### 5. Edge and Local AI

At 0.8B parameters and 1.7 GB of 16-bit weights, Jeff approaches a deployment regime relevant to laptops and potentially quantized edge environments.

This connects decision models with the wider movement toward local AI for lower latency, privacy, offline operation, and reduced dependency on centralized inference APIs.

## 9. Keywords

- Zero-Shot Classification
- Decision Models
- Non-Autoregressive Inference
- Qwen3.5
- Gemma 4
- Model Routing
- System 1 Models
- Local Inference
- Probability Calibration
- Edge AI

## 10. TL;DR

Jeff converts 0.8B–2B language models into local zero-shot decision engines that return option probabilities without generating text.

The 0.8B model reaches ~22–28 ms inference on high-end GPU/Apple hardware, while the 2B model reports competitive classification benchmarks but weaker multi-step reasoning.

The broader lesson is architectural: use code for deterministic logic, lightweight models for semantic decisions, and large generative models only where reasoning or generation actually adds value.
