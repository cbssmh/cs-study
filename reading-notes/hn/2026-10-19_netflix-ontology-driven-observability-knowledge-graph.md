1. Title

Ontology-Driven Observability: Building an End-to-End Knowledge Graph at Netflix Scale

2. Source

Author / Organization: Prasanna Vijayanathan, Renzo Sanchez-Silva / Netflix (Netflix TechBlog)
Link: https://netflixtechblog.com/ontology-driven-observability-building-an-end-to-end-knowledge-graph-at-netflix-scale-421392cf5adf
Date: 2026-10-17

3. One-line Summary

Netflix built an enterprise Observability Ontology and distributed RDF knowledge graph (QuipuDB) that transforms isolated MELT telemetry and unstructured incident Slack discussions into deterministic semantic triples, cutting cross-domain root-cause investigation time from four hours to minutes for its multi-agent AutoRCA system.

4. Key Points

• The Scale vs. Investigation Bottleneck: Netflix ingests over 2 billion requests and 38 million real-time log events per second; during complex production regressions, typical incident investigations required roughly four hours—not due to computational limits, but because engineers spent hours manually reconstructing fragmented entity relationships across siloed systems.
• Semantic Disagreement Across Tooling: Internal telemetry systems (Atlas for metrics, Radar for alerts, incident.io for incidents, PagerDuty for rotations, Spinnaker for deployments) used conflicting entity taxonomies where the same component was labeled an application, service, or cluster depending on tool conventions.
• Standardization via W3C Semantic Stack: Modeled operational entities into an open, portable ontology utilizing W3C standards: RDF for data modeling, OWL for schema definition, SHACL for constraint validation, and SPARQL for graph querying.
• QuipuDB Graph Storage: Persists operational facts as RDF quad triples in QuipuDB, an in-house distributed graph database backed by `redb` (a pure Rust embedded key-value store), while serializing daily snapshots into human-readable, diff-friendly Turtle format for archival.
• Strict Entity Governance over Generic Types: Deliberately eliminated generic "Service" types, resolving all software components strictly into `ops:Application` validated against a 49,000-entry authoritative software catalog, while capping bi-directional dependency edges to prevent high-fanout services from dominating graph traversals.
• Modeling Unstructured Human Reasoning: Ingests 200-message incident Slack channels and parses fleeting operational discussions (hypotheses tested, discarded theories, mitigations) into structured triples, preventing tacit institutional debugging context from evaporating when incident channels close.
• Creating Determinism from Stochastic LLMs: Deploys LLMs and fuzzy string matchers as bounded "entropy reducers" (Enrichers); by forcing stochastic models to emit only typed triples conforming to strict closed enum predicates (e.g., `category`, `sentiment`), the system guarantees deterministic SPARQL query execution over probabilistic source inputs.
• The Correlation Engine and Confidence Scoring: Evaluates 13 operational signal streams (deployments, config changes, metric anomalies, error patterns) within 10-minute sliding windows using causal prior weights, outputting evidence-backed correlation chains scored on temporal proximity and discarding hypotheses below 0.6 confidence.
• Multi-Agent AutoRCA Coordination: AutoRCA coordinates specialized agents (metrics, logs, traces, deployments) using the knowledge graph as a universal shared vocabulary, enabling specialists to exchange contextual findings without requiring direct peer-to-peer coupling or shared internal schemas.

5. Deep Dive (Structured Understanding)

Problem
Modern enterprise observability architectures collect massive volumes of metrics, events, logs, and traces (MELT). However, during critical live-streaming degradation events, each MELT silo answers only a fraction of the problem without shared contextual awareness. At Netflix, root-cause investigations took approximately four hours because engineers had to act as human JOIN operators—manually bridging naming discrepancies between Atlas metrics, Spinnaker deployments, Radar alerts, and Slack debugging threads. Simply deploying generative AI agents onto raw telemetry failed because ungrounded LLMs hallucinated across fragmented naming conventions. The core obstacle was not a lack of telemetry volume, but the absence of a machine-readable data model formalizing how operational entities relate to one another.

Approach
Netflix addressed observability as a foundational data modeling problem by building the Observability Ontology across twelve operational namespaces. Built strictly on W3C standards (RDF, OWL, SHACL, SPARQL), operational facts are represented as subject-predicate-object triples (e.g., `INC-5377 ops:affects api-gateway`). Facts are stored in QuipuDB, a distributed graph store built on the Rust embedded KV engine `redb`. A sequential enrichment pipeline runs specialized agents ("Enrichers") that ingest unstructured telemetry, git commits, and Slack incident channels, converting them into typed triples validated by SHACL schemas. On top of the graph, a Correlation Engine scores causal event chains, and an agentic system (AutoRCA) traverses the graph to deliver pre-assembled, evidence-backed hypotheses to on-call engineers.

Key Insight
1. Constraining Stochastic AI with Closed Type Systems: Generative models are effective at parsing messy human language and fuzzy server names, but their non-deterministic nature makes them dangerous for production diagnostics. By constraining model outputs strictly to typed RDF triples validated against authoritative software catalogs, a stochastic pipeline yields a 100% deterministic knowledge graph that returns identical SPARQL query results every time.
2. Modeling Human Chat as First-Class Telemetry: Traditional observability ignores incident communication channels, losing the institutional rationale behind why a rollback was initiated or which hypothesis failed. Converting Slack incident threads into structured triples captures organizational wisdom, transforming the graph from a static infrastructure map into an operational institutional memory.
3. Decoupling Agentic Specialists via a Universal Semantic Bus: AutoRCA avoids brittle peer-to-peer agent prompt routing by having every specialist agent (log specialist, trace specialist, deployment specialist) communicate exclusively by reading and writing triples against the shared ontology, ensuring modular extensibility across independent platform teams.

Result / Impact
Replaced hours of manual cross-system correlation with instant, graph-backed hypothesis ranking. Enabled on-call engineers and autonomous agents to query complex cross-domain dependencies in single SPARQL lookups, laying the architectural foundation for automated remediation and self-healing infrastructure.

6. Why It Matters

• Reliability & Observability: Represents an architectural paradigm shift in Site Reliability Engineering, proving that scaling beyond MELT requires structured knowledge graphs and semantic modeling rather than merely adding more dashboards.
• AI Engineering & Autonomous Systems: Provides a blueprint for grounding autonomous incident response agents (AutoRCA) in deterministic, verifiable facts, completely eliminating LLM hallucinations in mission-critical SRE workflows.
• Platform Engineering & Data Modeling: Demonstrates how enterprise platform teams can adapt formal W3C semantic web standards (RDF/OWL/SPARQL) to solve real-world microservice sprawl and configuration drift.
• Software Engineering Governance: Preserves engineering context across incident lifecycles, enabling organizations to analyze cross-incident failure patterns (the "Knowledge Flywheel") and eliminate recurring failure modes systematically.

7. Critical Analysis

• Custom Graph Database Overhead: Rather than adopting established open-source or commercial graph engines (Neo4j, Amazon Neptune, GraphDB), Netflix engineered QuipuDB in-house on top of `redb`. While optimizing for internal quad-store throughput and Turtle diffability, maintaining proprietary database storage introduces significant ongoing systems maintenance debt.
• Sequential Enrichment Latency Bottleneck: The authors explicitly note that the enrichment pipeline runs sequentially to avoid write race conditions across the graph. While eliminating concurrency bugs, this trade-off sacrifices real-time streaming velocity, introducing a lag between real-time telemetry spikes and graph ingestion.
• Cold-Start Dependency on Authoritative Software Catalogs: The system's strict gating against a 49,000-entry software catalog ensures clean data, but it also creates failure modes in fast-moving environments where newly spun-up ephemeral microservices, canary clusters, or dynamic cloud lambdas are dropped from the graph if catalog registration lags.

8. Connections

• W3C Semantic Web Standards (RDF, OWL, SHACL, SPARQL): Directly operationalizes formal knowledge representation technologies, demonstrating their utility for cloud-native infrastructure topology and observability.
• Knowledge Graphs in AIOps (e.g., Dynatrace Davis, Uber Service Topology): Advances automated root-cause analysis from basic statistical anomaly detection to structural, graph-traversal causal inference.
• Rust Embedded Key-Value Storage (`redb`): Leverages modern embedded storage engines written in memory-safe systems languages to build high-performance distributed persistence engines.

9. Keywords

• Observability Ontology
• Knowledge Graph
• Netflix SRE
• QuipuDB
• Automated Root Cause Analysis (AutoRCA)
• W3C Semantic Web (RDF/SPARQL)
• MELT Telemetry
• Entity Resolution

10. TL;DR

• Netflix built an enterprise Observability Ontology and distributed RDF knowledge graph (QuipuDB) to solve the cross-domain correlation bottleneck across 38M events/sec.
• Enforces deterministic SPARQL querying over stochastic LLM enrichers, parsing raw telemetry and incident Slack chats into structured, validated triples.
• Powers AutoRCA, an agentic incident investigation system that replaces hours of manual cross-dashboard correlation with ranked, graph-backed causal hypotheses.
