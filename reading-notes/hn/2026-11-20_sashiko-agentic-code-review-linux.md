1. Title
Sashiko: Lessons from building an agentic code review system for Linux

2. Source
Author / Organization: Roman Gushchin and Eric Brewer / Google Cloud Blog
Link: https://cloud.google.com/blog/products/application-development/lessons-from-building-sashiko-an-agentic-code-review-system
Date: November 19, 2026

3. One-line Summary
Google's open-source, multi-stage agentic code review system Sashiko separates bug discovery from symmetrical call-site verification, cutting Linux kernel 60-day post-merge fix rates by 35% across 241,000 patch reviews.

4. Key Points
- **Code review as the scaling bottleneck**: As coding agents accelerate patch and vulnerability-report generation, human maintainer bandwidth—not code generation—limits velocity in large projects like the Linux kernel (35M+ lines of code, 85,000+ changes per year).
- **LLM failure modes in code review**: Models trained to minimize tokens cut corners and self-dismiss bugs without proof, get distracted when asked to check multiple bug classes at once, and can be biased by commit-log text—though their lack of persistent memory is an asset when reviewing the same patch across 100 slightly different branches.
- **Five-stage verification architecture**: Sashiko gathers surrounding callers, struct definitions, and subsystem guides; runs up to 7 parallel single-focus specialist passes (commit intent, implementation match, control flow, resource lifetimes, locking, security, hardware) averaging ~100 `git` tool calls; triages both reported bugs and *dismissed* concerns; spawns dedicated per-bug deep verification agents requiring concrete call-site proof for either confirmation or dismissal; and formats reports in Linux mailing-list style.
- **Evolving prompt & context design**: Originally built on Chris Mason's open-source Linux review prompts, Sashiko shifted to combining auto-generated per-tree/per-version code notes with subjective maintainer instructions that cannot be inferred from source alone.
- **Seven months of production impact (since March 2026)**: Sashiko completed `241,000` reviews (~`1,200+`/day) across `100+` kernel mailing lists (`75+` opted-in subsystem lists, `1,500+` developer replies) at no cost to contributors via `sashiko.dev`, earning credits in `2,000+` mainline commits and `528` Linux kernel CVEs in 2026.
- **Measurable quality correlation**: Across `254,000` commits in Linux `6.2`–`6.19`, the 60-day follow-up fix rate averaged `3.4%`, dropping to `2.8%` in Linux `7.1` and `2.2%` in Linux `7.2` (30-day rate fell from `2.3%` to `1.4%`); across `21,500` tracked mainline commits, clean passes needed fixes `0.9%` of the time versus `7.6%` for commits merged despite critical warnings (`6.3x` higher risk adjusted for patch size).
- **Historical bug benchmark**: Evaluated against nearly `1,000` real Linux bugs that previously slipped past human maintainers into mainline, Sashiko with Gemini 4 Carbon (high thinking level) detects `75.8%` of them.

5. Deep Dive (Structured Understanding)
**Problem**
In large systems codebases, proving that a patch preserves system-wide invariants is far harder than writing the patch itself. Single-prompt LLM reviewers fail systematically on complex diffs because token-efficiency incentives push them toward premature dismissal, multi-tasking causes one obvious issue to crowd out subtler concurrency or lifetime bugs, and raw commit messages can skew judgment.

**Approach**
Google's AI & Infrastructure team (later donating Sashiko to the Linux Foundation) decoupled candidate bug generation from rigorous verification:
1. **Context & planning**: Hydrate the diff with modified functions, structs, direct callers, and subsystem guides, then assign per-pass reasoning budgets.
2. **Parallel single-focus specialists**: Run up to seven isolated sessions—each restricted to one concern (e.g., locking vs. resource lifetimes)—and require each to log both confirmed concerns and *ruled-out* concerns with supporting code.
3. **Symmetrical triage & deep checks**: A lead verifier audits shaky self-dismissals alongside contested findings, dispatching isolated per-bug verification agents with `git` tools under a strict rule: neither a bug claim nor a dismissal is accepted without concrete proof across all call sites.

**Key Insight**
Coding agents are weak reviewers of their own code because generation rewards fast local plausibility while verification requires adversarial, invariant-checking diligence. Separating specialist discovery from symmetrical call-site verification—and treating *false dismissals* as aggressively as *false alarms*—turns noisy LLM pattern-matching into a high-signal review gate.

**Result / Impact**
Beyond preventing pre-merge regressions, Sashiko uncovered hundreds of preexisting vulnerabilities (`528` CVEs in 2026) and demonstrated a `6.3x`–`8x` risk separation between clean passes and critical warnings, prompting developers to integrate local Sashiko instances into coding-agent loops prior to mailing-list submission.

6. Why It Matters
- **AI Engineering / Secure AI**: Provides a concrete architectural blueprint for high-precision agentic verification (single-focus fan-out, logging ruled-out hypotheses, and symmetrical proof requirements) that directly addresses LLM laziness and self-confirmation bias.
- **Software-Industry Changes & Platform Engineering**: Signals a shift from "AI code generation" to an "agentic trust layer" in critical open-source and enterprise workflows, where automated pre-review filters low-quality patches before human maintainers spend scarce attention.
- **Security / DevSecOps & Reliability**: Shows that static-analysis-like depth can be achieved across 35M+ lines of C without language-specific parsers by pairing agentic `git` navigation with historical regression benchmarks.

7. Critical Analysis
- **Causality caveats on the 35% fix-rate drop**: The authors explicitly acknowledge that release-to-release fix rates reflect many concurrent factors, so the drop from `3.4%` to `2.2%` between Linux `6.2`–`6.19` and `7.2` cannot be attributed solely to Sashiko.
- **Compute and token-budget asymmetry**: Sashiko averages ~100 `git` tool calls per review across multiple parallel Gemini sessions—a substantial token budget subsidized by Google at `sashiko.dev`. The article notes contributors would be deterred if paying out-of-pocket, leaving open how smaller organizations or non-sponsored OSS projects (such as GCC, LLVM, or Git) can fund equivalent depth sustainably.
- **Generalizability and false-positive friction**: While `1,500+` kernel developers engaged and pushback improved prompts, the post does not quantify the overall false-positive rate or maintainer triage time spent on non-critical warnings, nor does it evaluate non-C ecosystems where type systems or build graphs behave differently from `git`-navigated C code.

8. Connections
- **Independent Verification in Agentic Workflows**: Mirrors the separation of "generator" and "verifier/critic" roles seen in formal verification, fuzzing-plus-LLM pipelines, and Google's own observation that agents optimized for patch synthesis miss cross-file invariants when asked to self-review.
- **Historical Bug Benchmarks as Regression Suites**: Building a ~1,000-bug evaluation set from real missed mainline defects parallels SWE-bench-style empirical harnesses and production-failure replay benchmarks, grounding prompt/model changes in measured recall rather than intuition.
- **Trust Layer for Open-Source Maintainer Burnout**: Directly responds to the industry-wide surge in low-effort AI-generated PRs and spammy vulnerability reports by inverting the asymmetry—giving maintainers (and contributor pre-commit hooks) an open, auditable filter (`every prompt, tool call, reasoning step, and ruled-out bug is public`).

9. Keywords
Agentic Code Review, Linux Kernel, Sashiko, Multi-Agent Verification, Symmetrical Proof, Gemini, Open-Source Security, CVE Discovery, Software Supply Chain Trust, Empirical Benchmarking

10. TL;DR
Sashiko solves LLM code-review laziness and distraction by splitting analysis into up to 7 single-focus specialist passes, auditing shaky self-dismissals, and requiring symmetrical call-site proof via `git` tools.
Across 241,000 Linux kernel reviews since March 2026, it earned credits in 2,000+ commits and 528 CVEs, while commits merged with clean reviews showed a 0.9% 60-day fix rate versus 7.6% for critical warnings.
Its core lesson is architectural: coding agents are poor self-reviewers, and scaling software quality requires an independent, benchmark-driven verification layer funded and operated transparently.
