1. Title

Cutting Our Inference Bill by 68% by Being Obsessive About Optimization

2. Source

Author / Organization: Ryan Skidmore / Cloudflare (Cloudflare Blog)
Link: https://blog.cloudflare.com/obsessive-inference-optimization/
Date: 2026-10-19

3. One-line Summary

Cloudflare reduced its internal multi-agent AI code review costs by 68% (from $2.43 to $0.77 per review) and cut median review latency by 63% by restructuring prompt-cache prefixes across repositories, coordinating concurrent agent cache reads, and eliminating redundant tool turns.

4. Key Points

• Cost Inflation Drivers: The average cost of an automated code review on Cloudflare’s internal review platform escalated from $1.19 in April to $2.46 in August 2026, driven by higher model intelligence encouraging larger pull requests, migration to frontier models, and token volume tripling to 2.99 million tokens per review despite a high cache hit rate.
• Spend Breakdown Mechanics: Analysis revealed that 43% of AI review spend went to prompt cache writes and 34% to cache reads (totaling 77% on context the model had already seen), while generated output tokens accounted for only 23%.
• Fixing the Ephemeral Cache Invalidation Bug: The first LLM request in a review session hit cache only 14%–26% of the time because the system prompt embedded dynamic environment data (working directory paths and dates) ahead of the cache breakpoint; splitting the block into a static, cross-repo shared prefix and dynamic suffixes raised first-turn cache hits dramatically.
• Inter-Reviewer Cache Coordination: Rather than letting the code quality, security, and performance agents initiate parallel uncoordinated requests, Cloudflare engineered a serialized wait (up to 20 seconds) where the first reviewer writes the shared prefix to the Anthropic prompt cache and the subsequent two reviewers read from it, saving $0.09 per sub-reviewer.
• Eliminating Tool-Call Thrashing: Sub-reviewers previously attempted to call an uncallable final-verdict tool in up to 76% of sessions, wasting an extra turn recovering; stripping inaccessible tool schemas removed 2.9k tokens per turn and eliminated an entire wasted model call.
• Parallel File Hydration: Rather than allowing agents to make single sequential tool calls to read files (which occurred in 64% of turns), reviewers now pre-fetch shared context and all relevant patch files in parallel up to 150 KB on their first response.
• Re-Review Pruning: Since two-thirds of jobs represent follow-up reviews on existing merge requests, the platform checks for diff scope and skips untouched specialist reviews (skipping AGENTS.md reviews 56% of the time, performance 23%, and documentation 13%).
• Zombie Job Reclamation: Discovered that GitLab’s Kubernetes executor fails to propagate cancellation signals to job scripts, leaving 80% of canceled review pipelines burning tokens until completion; added active 15-second status polling to self-terminate canceled containers.
• Quality Calibration via Blind Evaluation: Verifying quality via author action rates (jumping from 34% to 59%) and multi-model blind grading across 657 findings confirmed that the 40%–70% drop in reported critical findings was driven by eliminating false-positive severity inflation rather than missing genuine bugs.

5. Deep Dive (Structured Understanding)

Problem
Multi-agent systems deployed at enterprise scale face compounding cost and latency spirals. When Cloudflare transitioned its automated code review platform (OpenCode) to frontier models, costs doubled. Although Anthropic’s prompt caching charges only 0.1x the base input price for reads, writes cost 1.25x with a strict 5-minute time-to-live (TTL). Because review sessions averaged 16.6 LLM calls sending cumulative conversation histories, token volume exploded to 3M tokens per review. Furthermore, subtle anti-patterns—such as placing dynamic environment strings before cache breakpoints, concurrent reviewers racing each other to write identical cache entries, and zombie CI pods executing orphaned runs—severely eroded prompt caching benefits while pushing median review latency past 4 minutes.

Approach
Cloudflare launched an optimization sprint combining architectural prompt refactoring, agent workflow tuning, and empirical quality validation. They decomposed system prompts to isolate completely static byte prefixes shared across all repositories and merge requests, placing strict cache breakpoints at the boundary. They synchronized concurrent specialist reviewers so that one agent writes the cache entry while sister agents hold connections to read the warm prefix. At the agent runtime layer, they stripped uncallable tool definitions, batched file reads into a single initial turn, and added cancellation hooks. Finally, they systematically benchmarked model tiers (migrating to Sonnet 5.5 and Opus 5.5 at calibrated effort levels), validating output fidelity through blinded LLM judging panels and developer commit-modification tracking.

Key Insight
1. Prompt Cache Topology Dictates Multi-Agent Architecture: In multi-agent pipelines, concurrent execution is often an economic anti-pattern. Delaying downstream agents by a few seconds to let a primary agent write a prompt cache prefix yields massive net savings ($0.09 per reviewer) with negligible impact on overall job latency.
2. The Illusion of Cache Hit Rates on Tripled Token Counts: A 97% cache hit rate can obscure a budget crisis if the underlying token footprint expands faster than the cache discount. Controlling total conversational turn depth and pruning uncalled tool schemas is just as critical as maximizing hit percentages.
3. Separating Noise Reduction from Quality Degradation: A steep drop in flagged issues often triggers alarm that an optimized model is "lazier." By measuring the downstream "addressed rate" (whether human developers actually modified the flagged code) and evaluating blinded diff samples, Cloudflare proved that Sonnet 5.5 at high effort maintained real bug discovery (0.30 important findings/review) while reducing severe false-positive ratings by 70%.

Result / Impact
Slashed per-review cost by 68% (from $2.43 to $0.77), reduced median review latency from 4m 09s to 1m 32s (a 63% reduction), and dropped long-tail reviews exceeding 5 minutes from 37.9% to just 0.7%, while increasing developer trust in review findings.

6. Why It Matters

• AI Engineering & FinOps: Establishes a concrete, production-tested playbook for optimizing agentic LLM workloads, demonstrating how to align prompt engineering with provider-specific KV cache mechanics (such as Anthropic’s 5-minute prefix cache).
• Platform Engineering & Developer Productivity: Proves that AI developer tooling can achieve sub-2-minute turnaround times, removing CI bottlenecks and preventing pull request review latency from blocking engineering velocity.
• Reliability & Distributed Systems: Highlights the operational failure modes of running LLM agent pipelines on container orchestrators (e.g., Kubernetes executor signal dropping) and provides self-healing patterns for zombie task reclamation.
• Software Engineering Governance: Demonstrates how to design objective quality benchmarks for generative AI tools by combining automated blind multi-model adjudication with empirical developer action telemetry.

7. Critical Analysis

• Provider Lock-In on Cache Mechanics: The architectural optimizations (prefix splitting, ordered streaming waits) are heavily coupled to the specific prefix-matching, breakpoint-based caching model of Anthropic’s API. Porting this platform to alternative providers with different caching semantics (e.g., OpenAI’s automatic prefix caching or local vLLM PagedAttention) would require re-engineering prompt assembly and coordination locks.
• Subjective Degradation in Secondary Reviewers: While the primary code-quality reviewer retained parity at high effort, performance and security reviewers at medium effort lost significant finding coverage (performance findings dropped by 64%). Relying on medium effort to hit cost targets risks missing subtle distributed system regressions unless specialized reviewers are continuously benchmarked.
• Fragility of Static Prefix Invariants: Maintaining a byte-identical prompt prefix across thousands of repositories requires disabling dynamic per-repository skills and custom tool descriptions. This rigid constraint limits the ability of individual engineering teams to customize reviewer behavior for domain-specific monorepo conventions.

8. Connections

• Prompt Caching & KV Cache Reuse: Directly leverages transformer attention caching mechanisms (e.g., vLLM Chunked Prefill, PagedAttention) to avoid re-evaluating static prefix embeddings across sequential requests.
• Amdahl’s Law in Agent Workflows: Demonstrates that optimizing the non-model plumbing (tool schemas, file ingestion batching, cancellation handling) often yields greater performance and cost dividends than model switching alone.
• Human-in-the-Loop Telemetry (Acceptance Tracking): Parallels modern IDE completion evaluation (e.g., GitHub Copilot acceptance rates) by using developer commit diffs as an implicit ground-truth feedback loop.

9. Keywords

• Prompt Caching
• AI Cost Optimization
• Multi-Agent Systems
• FinOps
• LLM Code Review
• Cloudflare OpenCode
• Claude 5.5
• Prefix Caching

10. TL;DR

• Cloudflare reduced internal AI code review costs by 68% (to $0.77 per review) while cutting median review turnaround from 4m 09s to 1m 32s.
• Restructured prompts around Anthropic’s prefix cache, synchronized concurrent sub-reviewers to share warm caches, and eliminated wasteful tool turns and zombie CI containers.
• Blinded LLM evaluations and developer commit tracking confirmed that finding reductions were driven by a 70% drop in false-positive severity inflation, preserving true bug detection rates.
