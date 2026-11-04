1. Title

Lessons from PD Labs: From Experimentation to a Production-Ready SRE Agent

2. Source

Author / Organization: PagerDuty Engineering / PagerDuty
Link: https://www.pagerduty.com/eng/lessons-from-pd-labs-from-experimentation-to-a-production-ready-sre-agent/
Date: 2026-11-03

3. One-line Summary

PagerDuty details the architectural evolution of its autonomous SRE Agent, demonstrating how hybrid code-and-IaC static dependency analysis (0.93 F1 score), direct versus circumstantial evidence classification, and private EKS fault-injection benchmarking transformed complex multi-agent prototypes into a fast, production-grade incident diagnostic system.

4. Key Points

• The Inadequacy of Naive Agent Prototypes: While connecting an LLM to observability tools is trivial to prototype, simple agents fail during live production incidents due to lack of system isolation, brittle context tracking during multi-hour war rooms, and confident hallucinations blaming co-occurring symptoms rather than underlying causes.
• The Decoupling of Alerting from Causation: In complex microservices, the service that fires an alert is rarely the service that broke (e.g., an API gateway times out because a database two network hops away was migrated); diagnosing root causes requires traversing dependency graphs upstream to find causes and downstream to map blast radius.
• Static Code and IaC Topology Extraction: Rather than relying on expensive, sampled distributed tracing or drifting manual wikis/Backstage catalogs, PagerDuty extracts service topology statically by parsing source code, Helm charts, and Terraform definitions inside security enclaves, achieving an F1 score of 0.93 across 500+ private production repositories.
• Transfer Learning in Rule Generation: When static parsers encounter unknown custom libraries or frameworks, offline agents inspect the unresolvable references, synthesize generalized extraction rules, and extend the engine without touching live runtime graph execution.
• Direct vs. Circumstantial Evidence Grading: Classifies gathered evidence into "direct" (verifiable fault mechanisms, such as a Git diff dropping an environment variable) versus "circumstantial" (metric spikes and coincident alert timestamps), preventing agents from jumping to confident, incorrect conclusions based on mere temporal correlation.
• Collapse of Over-Engineered Multi-Agent Topologies: Early experiments using complex fan-out multi-agent architectures performed poorly—ranking mid-field in diagnostic accuracy while incurring high token latency and cost; consolidating responsibilities into a leaner architecture produced a 20-point diagnosis score improvement.
• Private, Multi-Vector EKS Benchmarking Harness: To prevent evaluation contamination, PagerDuty built a private multi-language microservice benchmark on EKS (Go, Python, TypeScript, Elixir; Kafka, Redis, Postgres) with live fault injections spanning Helm overrides, Chaos Mesh, PR merges, and AWS control-plane alterations.
• Multi-Provider Judge Ensemble: Uses a three-model evaluation panel spanning three independent model providers to score candidate agents across root-cause accuracy, fault localization, mitigation quality, and user misdirection.
• Interface Decoupling via MCP: Exposes ongoing investigations independently of UI surfaces, allowing engineers to interact via the SRE Workbench web app, Slack incident channels, or terminal-based Claude Code skills streaming updates over the Model Context Protocol (MCP).

5. Deep Dive (Structured Understanding)

Problem
During critical production incidents, on-call responders suffer from cognitive overload: sifting through hundreds of firing alerts, deployment logs, metrics dashboards, and Slack threads. While LLM-based autonomous agents offer promise for automated triage, naive implementations fail in high-stakes environments. They lack an accurate, up-to-date mental model of distributed service topologies, frequently misdiagnosing innocent downstream symptoms as root causes because an alert happened to spike at the same time. Furthermore, public AI benchmarks rely on simple open-source demo apps that frontier models have already memorized during pre-training, providing zero signal on whether an agent can reason through novel, un-instrumented failure modes across private enterprise architectures.

Approach
PagerDuty Labs engineered a production-grade SRE Agent ("Paige") integrated into a collaborative SRE Workbench. The system solves the dependency problem up-front: an automated collector runs inside an isolated security enclave, statically parsing source code and infrastructure-as-code (Terraform, Helm, environment variables) across an organization’s repositories to build an auditable, confidence-scored topology graph before any tracing or alert is triggered. When an incident fires, the agent initiates an autonomous, continuous investigation: it generates competing hypotheses, queries observability backends, and strictly tags each finding as direct or circumstantial evidence while logging rejected alternatives. The architecture was systematically optimized against an automated, private EKS benchmark harness that simulates realistic production outages across code, configuration, data integrity, and operational categories.

Key Insight
1. Statically Derived System Truth Outperforms Runtime Tracing: Distributed tracing captures only exercised, sampled execution paths and requires intrusive, manual library instrumentation that frequently misses message queues and shared databases. Deriving topology statically from versioned code and IaC definitions provides a complete, deterministic, line-referenced architecture graph that updates automatically with every Git commit.
2. The Circumstantial Correlation Trap: In distributed systems, temporal co-occurrence is not causality. A database failover may cause ten microservices to throw connection errors simultaneously. Forcing an AI agent to explicitly separate circumstantial evidence (error spikes) from direct mechanistic proof (a dropped database configuration parameter) eliminates premature, confident misdirection.
3. Multi-Agent Complexity Inversion: Enterprise agent developers frequently assume that decomposing tasks across dozens of specialized sub-agents improves reasoning. Empirical benchmarking proved the opposite: uncoordinated multi-agent fan-out introduced communication overhead, context fragmentation, and excessive latency. Consolidating into a focused agent with access to clean, deterministic topology and evidence structures was both 20 points more accurate and significantly cheaper.

Result / Impact
Delivered a production-ready SRE Agent capable of proactively localizing root causes and calculating business blast radius before human responders are paged, supported by a validated 0.93 F1 topology engine and an open, vendor-neutral benchmarking framework for autonomous incident response.

6. Why It Matters

• Reliability Engineering & SRE: Replaces reactive, high-stress human war room triage with automated, proactive hypothesis generation, dramatically shortening Mean Time to Resolution (MTTR).
• AI Engineering & System Architecture: Demonstrates that production-grade agentic performance requires rigorous systems engineering—static analysis graphs, evidence verification taxonomies, and private empirical benchmarking—rather than merely chaining raw LLM calls.
• Platform Engineering & Service Cataloging: Solves the chronic failure of enterprise service catalogs (Backstage, wikis), proving that software architecture models must be automatically derived from Git and IaC rather than maintained manually.
• IT Risk / Governance: Enforces transparency and safety by requiring autonomous agents to present clear evidence chains and confidence scores for human steering, ensuring automated remediation proposals can be audited before execution.

7. Critical Analysis

• Static Analysis Inherent Blind Spots: While achieving a 0.93 F1 score on service dependencies, static analysis cannot easily resolve runtime-dynamic RPC destinations (e.g., services that resolve target URLs dynamically from database tables or external service registries). In highly dynamic, plugin-based environments, static extraction must still fall back to runtime tracing or LLM-assisted rule synthesis.
• The Human Input Performance Gap: The authors note that providing live human responder input to the agent during an incident adds a massive 5 percentage points to the agent's diagnosis score. This highlights that autonomous agents remain bounded by the tacit operational context stored in human minds that never appears in code, dashboards, or tickets.
• Disruption Budget Governance Constraints: While the agent excels at diagnosing root causes and suggesting mitigations, enabling fully autonomous remediation (e.g., executing rollbacks, draining nodes, or scaling clusters) remains constrained by operational risk; without airtight guardrails, an autonomous agent executing remediation based on flawed hypotheses could exacerbate an outage.

8. Connections

• Netflix Ontology-Driven Observability & Pensive AI: Mirrors Netflix’s shift from raw MELT telemetry toward structured knowledge graphs and deterministic diagnostic wrappers, confirming an industry-wide convergence around semantic topology for SRE.
• Distributed Tracing & OpenTelemetry (OTel): Complements runtime distributed tracing by providing the structural, un-sampled "skeleton" of microservice connections, filling tracing gaps across asynchronous queues and third-party SaaS APIs.
• Chaos Engineering & Chaos Mesh: Leverages cloud-native chaos injection tools inside Kubernetes to generate authentic, non-synthetic incident data for continuous AI regression testing.

9. Keywords

• PagerDuty SRE Agent
• PD Labs
• Incident Management
• Service Topology Extraction
• Blast Radius Analysis
• Direct vs Circumstantial Evidence
• Model Context Protocol (MCP)
• Automated Root Cause Analysis

10. TL;DR

• PagerDuty Engineering detailed the architecture behind its production-grade SRE Agent, moving from early experimental prototypes to an automated incident response system.
• Built a hybrid static-analysis engine parsing source code and IaC inside security enclaves, achieving a 0.93 F1 score in automatically extracting distributed service topology without tracing overhead.
• Abandoned complex multi-agent fan-out in favor of a leaner architecture that separates direct from circumstantial evidence, scoring 20 points higher on an un-contaminated EKS chaos benchmark.
