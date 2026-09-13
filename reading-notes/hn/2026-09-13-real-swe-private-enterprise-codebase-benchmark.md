## 1. Title

- Introducing Real-SWE: Benchmarking Frontier AI Models on Private, Real-World Enterprise Codebases

## 2. Source

- Author / Organization: Specific
- Link: https://withspecific.com/benchmarks/real-swe
- Date: September 2026

## 3. One-line Summary

- Real-SWE tests coding agents on licensed private production codebases and finds that even the best model-harness combination resolves only 38.8% of tasks, with requirement discovery, assumption verification, and system integration remaining major weaknesses.

## 4. Key Points

- Real-SWE uses tasks derived from licensed private production codebases rather than synthetic problems or publicly available repositories.
- Tasks include economically consequential work such as billing, taxation, customer migration, API metering, and multi-service changes.
- The benchmark evaluates model-harness combinations because practical agent performance depends on both the model and its development environment.
- Fable 5.1 + Claude Code ranked first at 38.8%, followed by GPT-6 Astra + Codex CLI at 33.8% and Gemini 3.8 Flash at 31.2%.
- GPT-5.6 Sol + Codex CLI scored 16.2%, with unverified assumptions accounting for 43.3% of its failed runs.
- Six of ten analyzed tasks had aggregate resolution rates below 15%; the Analytics Stream Reducer task was not solved in any run.
- Failure modes were categorized as unverified assumptions, missed requirements, integration errors, regressions, and modifications to the wrong file.
- Real-SWE reference solutions modify a median of 11 files, compared with six for FrontierCode and DeepSWE, reflecting broader cross-system changes.
- Shorter execution time did not substantially improve reliability: 71.4% of runs under ten minutes failed versus 73.4% of longer runs.
- Hacker News discussion highlighted a central trade-off: private benchmarks reduce contamination and benchmark gaming but sacrifice reproducibility and independent inspection.

## 5. Deep Dive (Structured Understanding)

### Problem

Public coding benchmarks increasingly risk measuring capabilities that do not transfer cleanly to production software engineering. Public repositories, issues, and solutions may appear in training data, while benchmark tasks often isolate implementation from the business rules, legacy architecture, operational dependencies, and company-specific conventions found in real systems.

The harder question is therefore not whether an AI can generate correct code in isolation, but whether it can enter an unfamiliar production system, discover how it works, infer the required scope without inventing requirements, modify the correct components, and preserve existing behavior.

### Approach

Real-SWE licenses private production codebases from real companies and constructs tasks from work their engineers perform.

Agents receive relatively brief instructions and must discover implementation details through the repository and available services. Environments can expose infrastructure and business tools such as Docker, Kubernetes, databases, AWS emulation, GitHub, Slack, Linear, Google Drive, and email.

Each model uses a native coding harness. Tasks run in isolated sandboxes, while verifiers based on existing test suites are injected during grading.

Each task is attempted eight independent times per model. Resolution rate is calculated as pass@1 averaged across those runs.

### Key Insight

The primary bottleneck is no longer simply code generation.

Agents frequently fail because they misunderstand the surrounding system: they assume behavior without checking it, overlook requirements embedded in existing business logic, or implement a reasonable local change that integrates incorrectly with the broader application.

This shifts the evaluation target from:

`prompt → code`

toward:

`requirement → investigation → system understanding → implementation → integration → verification`

The benchmark therefore measures a broader portion of the software-engineering loop.

### Result / Impact

The strongest combination, Fable 5.1 + Claude Code, resolved 38.8% of tasks. GPT-6 Astra reached 33.8%, while several frontier systems remained below 25%.

Failure patterns also differed significantly between models. GPT-5.6 Sol disproportionately failed through unverified assumptions, while Gemini 3.8 Flash showed a comparatively high proportion of integration errors.

The results suggest that frontier coding agents can perform economically meaningful engineering work, but autonomous reliability remains far below the level implied by successful demonstrations on narrower coding tasks.

## 6. Why It Matters

- Coding-agent evaluation is moving from isolated code generation toward end-to-end software engineering inside existing systems.
- Private codebases provide a potential defense against benchmark contamination because task implementations and company-specific conventions are not publicly searchable.
- The results emphasize context discovery, requirements analysis, testing, and architecture comprehension as persistent bottlenecks even when code-generation capability is strong.
- Model selection alone is insufficient: harness design, repository structure, validation tooling, instructions, and human supervision can materially affect practical performance.
- The benchmark supports a human-AI workflow in which engineers increasingly supervise scope, assumptions, architecture, and verification rather than merely write every implementation manually.
- Enterprise adoption therefore depends not only on stronger models but also on better agent scaffolding, validation systems, observability, and controlled execution environments.

## 7. Critical Analysis

- Private codebases reduce contamination risk but make independent reproduction difficult; outsiders cannot fully inspect whether the selected repositories and verifiers represent enterprise software broadly.
- The benchmark contains a limited set of analyzed tasks, so the exact ranking should not be generalized into a universal ordering of coding models.
- Results represent model-harness combinations rather than isolated model capability. Differences between Claude Code, Codex CLI, Gemini tooling, and other harnesses can materially affect rankings.
- A binary resolution rate does not capture development speed, interaction quality, maintainability, partial progress, or how easily a human engineer can repair an almost-correct solution.
- Real companies vary enormously in architecture, code quality, documentation, languages, testing practices, and domain complexity; "real-world enterprise codebase" is therefore not a uniform category.
- Private data is not automatically uncontaminated. The benchmark reduces exposure probability but cannot prove that no related code, patterns, documentation, or interactions influenced model training.
- Hacker News anecdotes show substantial disagreement with the ranking, reinforcing that agent performance can be highly workflow- and project-dependent.
- A benchmark without a skilled human operator may underestimate the productivity of human-agent systems while still being appropriate for measuring autonomous execution.
- Cost rankings also require context: a cheaper failed run may be economically worse than a more expensive run that consistently completes the task.

## 8. Connections

- **SWE-bench / FrontierCode / DeepSWE:** Real-SWE extends repository-level coding evaluation toward private production systems and more company-specific requirements, addressing contamination and realism concerns found in public benchmarks.
- **Terminal-Bench:** Both move beyond isolated code generation toward agents operating through tools and environments; Terminal-Bench emphasizes terminal/system tasks, while Real-SWE focuses on production software changes.
- **Benchmark Contamination:** Keeping repositories private is an attempt to reduce training-data leakage and benchmark optimization, but it creates a transparency-versus-contamination trade-off.
- **Agent Harnesses:** Claude Code, Codex CLI, and comparable systems demonstrate that model capability cannot be cleanly separated from context management, tool execution, repository navigation, and validation loops.
- **Software Verification:** The high rate of assumption and integration failures increases the importance of automated tests, executable specifications, static analysis, CI, and validation harnesses for agentic development.
- **Human-AI Centaur Workflows:** Hacker News discussion suggests that planning, scope review, explicit guardrails, and human validation can substantially change outcomes even when the underlying model is unchanged.
- **Enterprise AI Adoption:** Production deployment requires agents to understand organization-specific business rules and operational constraints, not merely common programming patterns learned from public code.

## 9. Keywords

- Real-SWE
- Coding Agents
- Software Engineering Benchmark
- Enterprise Codebase
- Agent Harness
- Pass@1
- Benchmark Contamination
- Requirement Discovery
- Integration Testing
- AI-Assisted Software Engineering

## 10. TL;DR

- Real-SWE evaluates coding agents on private production codebases and finds the best model-harness combination resolves only 38.8% of tasks.
- The dominant failures come from incorrect assumptions, missed requirements, and integration mistakes rather than basic inability to generate code.
- The results suggest that reliable agentic engineering depends increasingly on system understanding, verification, harness design, and human supervision rather than model capability alone.
