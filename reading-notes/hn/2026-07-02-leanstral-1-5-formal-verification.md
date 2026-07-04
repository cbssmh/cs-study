# 1. Title



Leanstral 1.5: Proof Abundance for All



# 2. Source



- **Author / Organization:** Leanstral Team, Mistral AI

- **Link:** https://mistral.ai/news/leanstral-1-5

- **Date:** 2026-07-02



# 3. One-line Summary



Leanstral 1.5 is an open-source Lean 4 theorem-proving model that achieves state-of-the-art formal verification performance while demonstrating practical code verification and automated bug discovery.



# 4. Key Points



- Released under the Apache-2.0 license with 119B total parameters (6B active MoE).

- Achieves 100% on miniF2F and solves 587/672 PutnamBench problems.

- Sets new state-of-the-art on FATE-H (87%) and FATE-X (34%).

- Trained using three stages: mid-training, supervised fine-tuning, and reinforcement learning (CISPO).

- Supports long-horizon proof engineering through iterative interaction with Lean compiler feedback.

- Operates as an autonomous proof agent capable of editing files, running shell commands, and using the Lean Language Server.

- Improves FLTEval pass@8 to 43.2, outperforming larger open models and reportedly exceeding Opus 4.6 at substantially lower inference cost.

- Automated verification pipeline analyzed 57 Rust repositories, identifying 11 real bugs, including 5 reportedly undisclosed issues.

- Demonstrated formal proof of AVL tree O(log n) complexity on a real implementation.

- Available through Hugging Face, a free API endpoint, and integrated into Mistral Vibe.



# 5. Deep Dive (Structured Understanding)



## Problem



Formal verification remains difficult because writing machine-checkable proofs requires specialized expertise, significant manual effort, and extensive interaction with theorem provers such as Lean. Existing coding LLMs generate code but cannot reliably produce mathematically verified correctness proofs.



## Approach



Leanstral 1.5 specializes exclusively in Lean 4 proof engineering.



Rather than only predicting proof tokens, it is trained in interactive reinforcement learning environments where it repeatedly:



- proposes proofs,

- receives Lean compiler feedback,

- revises failed proofs,

- edits repositories,

- invokes development tools,

- and iterates until verification succeeds.



The model also functions as an autonomous code agent capable of navigating real repositories during proof construction.



## Key Insight



Proof generation benefits from extremely long reasoning trajectories.



Instead of degrading with longer contexts, Leanstral continues refining proofs over millions of tokens, repeatedly using compiler feedback as supervision. Formal verification therefore becomes an iterative search process rather than a single-pass text generation task.



## Result / Impact



The model establishes new state-of-the-art performance across multiple Lean benchmarks while demonstrating that AI can perform practical proof engineering on real software projects. Beyond mathematical proofs, it shows potential for automated software correctness verification and bug discovery.



# 6. Why It Matters



- Signals a shift from AI-assisted coding toward AI-assisted formal verification.

- Demonstrates that reinforcement learning with compiler feedback scales effectively for theorem proving.

- Suggests future software development pipelines may integrate proof generation alongside code generation.

- Strengthens the ecosystem around Lean as a practical verification language.

- Highlights increasing convergence between autonomous software agents and formal methods.



# 7. Critical Analysis



- Benchmark comparisons rely largely on earlier-generation theorem-proving models rather than the latest frontier reasoning models, making relative performance difficult to judge.

- The highlighted bug discovery example (integer overflow at `u64::MAX`) is a classic boundary-condition error that conventional testing or fuzzing can often detect, weakening its value as a showcase.

- Hacker News users noted that the featured Rust bug already appeared in a public GitHub issue before publication, raising questions about whether Leanstral independently discovered it first.

- The reported low inference cost is compelling but depends heavily on evaluation methodology and is not directly comparable across all competing systems.

- Practical adoption still depends on organizations investing in formal specification, which remains the largest bottleneck rather than theorem proving itself.



# 8. Connections



### 1. Formal Verification



- Lean 4

- Coq

- Isabelle/HOL

- Dafny

- Proof assistants



These tools aim to mathematically prove software correctness rather than relying solely on testing.



### 2. AI for Software Engineering



- OpenAI Codex

- Claude Code

- DeepMind AlphaProof

- OpenHands



The industry is moving from code generation toward autonomous software engineering agents capable of reasoning over entire repositories.



### 3. AI + Reinforcement Learning



Leanstral follows a growing trend of reinforcement learning using executable feedback (compiler, verifier, environment) instead of only human preference optimization, similar to approaches used in AlphaProof and recent reasoning models.



### 4. Automated Software Assurance



The work aligns with broader efforts to combine AI, static analysis, symbolic reasoning, and theorem proving for security-critical software verification.



# 9. Keywords



- Lean 4

- Formal Verification

- Theorem Proving

- Proof Engineering

- Reinforcement Learning

- Leanstral

- AI Agents

- PutnamBench

- FATE

- Software Verification



# 10. TL;DR



- Leanstral 1.5 is a specialized open-source AI model for Lean 4 formal verification.

- It achieves state-of-the-art theorem-proving performance while demonstrating repository-scale proof engineering.

- Although benchmark results are impressive, real-world impact depends on broader adoption of formal verification workflows rather than theorem-proving capability alone.
