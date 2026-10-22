1. Title

Delivering and Securing AI Agents in AI Factories: F5 for the NVIDIA OpenShell Reference Architecture

2. Source

Author / Organization: Kunal Anand / F5
Link: https://www.f5.com/company/blog/delivering-and-securing-ai-agents-running-within-ai-factories-with-f5-and-nvidia
Date: 2026-10-21

3. One-line Summary

F5 partnered with NVIDIA to integrate AI guardrails, Model Context Protocol (MCP) traffic management, and BIG-IP Next for Kubernetes into the NVIDIA OpenShell Reference Architecture, shifting agent security out of vulnerable container sidecars onto hardware-isolated BlueField-4 DPUs and Vera CPUs.

4. Key Points

• The Failure of In-Container "Self-Policing": Sidecar guardrails running within the agent’s own container represent a flawed security model; a compromised or jailbroken agent operating in the same trust domain can disable the sidecar, modify local execution logs, or route around filters in under a second.
• Three-Tier Hardware Separation: NVIDIA OpenShell enforces physical isolation across three hardware tiers: 1) NVIDIA Vera CPUs running the OpenShell supervisor outside the sandbox, 2) NVIDIA BlueField-4 DPUs running NVIDIA Sentry for hardware-enforced packet inspection and millisecond containment, and 3) NVIDIA Vera Rubin NVL72 running frontier inference models.
• Out-of-Blast-Radius Inspection: F5 AI Guardrails runs on Arm cores across Vera CPUs and BlueField-4 DPUs entirely outside the agent's container, intercepting supported prompts, model responses, and tool calls to block prompt injection, prevent unauthorized tool execution, and redact sensitive data.
• First-Class MCP Protocol Governance: Establishes fine-grained egress enforcement over Model Context Protocol (MCP) connections, evaluating which external APIs, databases, and enterprise tools an agent is authorized to invoke before traffic leaves the hardware boundary.
• Millisecond Hardware Containment: When F5 AI Guardrails flags high-severity policy violations or prompt injections on the network path, its telemetry directly triggers NVIDIA Sentry running on BlueField-4 DPUs (via NVIDIA DOCA) to isolate or quarantine the offending agent in silicon within milliseconds.
• Tamper-Proof Tripartite Audit Trail: Solves the non-repudiation challenge by correlating three independent logs outside the agent's reach: F5’s application session record (user, prompt context, tool parameters), OpenShell’s runtime policy decisions, and NVIDIA Sentry’s system metrics (kernel calls, file I/O, DPU packet counters).
• Continuous Defense via AI Red Teaming: The F5 AI Security Platform incorporates automated AI Red Teaming running alongside models on the inference cluster, continuously attacking agents and guardrails through identical production ingress paths to turn newly discovered jailbreaks into active security policies.
• Application Delivery Acceleration: Deploying F5 BIG-IP Next for Kubernetes onto BlueField DPUs offloads proxy routing, tenant isolation, and MCP connection pooling from host CPUs, yielding a 40% improvement in token throughput (measured by The Tolly Group) while enforcing external egress boundaries.

5. Deep Dive (Structured Understanding)

Problem
Autonomous AI agents in enterprise "AI factories" diverge from traditional static applications: they ingest semi-structured enterprise documents, invoke external APIs via Model Context Protocol (MCP), communicate with peer agents, and query vector databases. When an agent retrieves a document containing indirect prompt injection, it can be hijacked into exfiltrating proprietary records or invoking destructive tool calls. Historically, developers deployed guardrails as software sidecars or in-process Python interceptors within the agent container. This architecture violates basic zero-trust isolation: because the agent shares the container namespace and execution permissions, a compromised agent can easily terminate the sidecar process, clear audit logs, and establish direct outbound socket connections to exfiltrate data without detection.

Approach
F5 and NVIDIA designed a hardware-enforced security reference architecture (NVIDIA OpenShell) based on the axiom that "the agent cannot police the agent." The architecture isolates the untrusted agent inside an unprivileged sandbox, while placing security supervisors on physical compute tiers outside the agent's blast radius. Outbound agent requests (HTTP, GraphQL, MCP) pass through an OpenShell supervisor running on NVIDIA Vera CPUs. Concurrently, network traffic routes across NVIDIA BlueField-4 Data Processing Units (DPUs) where NVIDIA Sentry and F5 AI Guardrails inspect payloads out-of-band using specialized Arm cores. For enterprise environments, F5 BIG-IP Next for Kubernetes runs on the DPU as a high-throughput data plane, managing tenant rate-limiting, external egress policies, and load balancing across Vera Rubin inference clusters.

Key Insight
1. Hardware Separation of Policy from Execution: True containment requires physical domain separation. By executing runtime supervisors (OpenShell) and content inspection (F5 Guardrails) on separate CPU/DPU hardware rings using isolated memory buses, no degree of software compromise or container privilege escalation within the guest agent can subvert network inspection or alter audit telemetry.
2. The DPU as an Autonomous Security Enforcement Point: Running security controls on host CPUs still leaves an attack window open to host kernel compromise. Offloading traffic inspection and quarantine logic to BlueField-4 DPUs running NVIDIA DOCA ensures that network isolation is enforced at the network interface card (NIC) level in silicon, executing agent containment in milliseconds.
3. Decoupling Semantic Inspection from Transport Delivery: The architecture cleanly splits responsibilities between content-aware semantic inspection (F5 AI Guardrails detecting prompt injection and PII leakage) and high-speed application delivery (BIG-IP Next for Kubernetes managing MCP connection scaling and token balancing), achieving deep security without degrading token throughput.

Result / Impact
Provides enterprise AI factories with a validated blueprint to deploy autonomous tool-using agents safely at scale, eliminating sidecar evasion vectors, achieving 40% higher token throughput via DPU offloading, and maintaining strict compliance auditing across multi-tenant inference clusters.

6. Why It Matters

• Platform Engineering & AI Infrastructure: Establishes a standard reference architecture for "AI factories," demonstrating how next-generation data centers must integrate DPUs, specialized CPUs, and GPU clusters into unified security fabrics.
• Security & DevSecOps / Secure AI: Addresses the vulnerability of in-container guardrail implementations, proving that production AI safety requires physical hardware isolation, egress network filtering, and out-of-band inspection.
• Cloud Networking & Kubernetes: Demonstrates the practical realization of cloud-native DPU networking (BIG-IP Next on BlueField-4), offloading heavy L7 proxy processing from expensive GPU host nodes to dedicated smart network cards.
• IT Risk / Governance: Establishes a tamper-proof, non-repudiable audit standard for autonomous agent actions, enabling regulated enterprises (financial services, healthcare, defense) to prove exact execution lineages for compliance audits.

7. Critical Analysis

• High Hardware Barrier to Entry: The architecture depends on cutting-edge, proprietary NVIDIA enterprise silicon (Vera CPUs, BlueField-4 DPUs, Vera Rubin NVL72). Organizations operating on generic commodity x86 servers or alternative cloud hypervisors (AWS Graviton/Nitro, Google Cloud TPU/Titan) cannot directly deploy this reference architecture without extensive bespoke re-engineering.
• Latency Accumulation on Complex Tool Chains: While DPU offloading optimizes transport throughput, routing multi-agent workflows through multiple sequential inspection hops (OpenShell supervisor -> BlueField-4 Sentry -> F5 AI Guardrails -> BIG-IP Next -> LLM inference) risks accumulating round-trip latency overhead on high-frequency, multi-turn tool calling pipelines.
• MCP Protocol Maturity Risk: The Model Context Protocol (MCP) is an evolving, rapid-moving open specification. Encoding rigid hardware-level parsing and policy evaluation for MCP calls onto DPU firmware (DOCA) risks protocol drift and maintenance friction as the MCP schema evolves.
• Vendor Co-Dependency: While F5 and NVIDIA promote "open" architectures, the tight coupling between F5 software and NVIDIA hardware creates a mutual vendor lock-in that limits an enterprise’s ability to swap individual security components without disrupting the broader factory control plane.

8. Connections

• SmartNICs & DPU Offloading (NVIDIA BlueField / AWS Nitro): Extends the concept of offloading hypervisor networking and storage virtualization to DPUs into offloading AI application-layer security and prompt inspection.
• Model Context Protocol (MCP): Integrates emerging standards for AI agent tool execution and context retrieval directly into enterprise network security firewalls.
• Zero Trust Architecture (NIST SP 800-207): Rigorously applies zero-trust principles to autonomous software actors, treating internal agent traffic as untrusted and enforcing continuous, out-of-band verification at every boundary.

9. Keywords

• NVIDIA OpenShell
• AI Factory Security
• BlueField-4 DPU
• F5 AI Guardrails
• Model Context Protocol (MCP)
• Zero Trust Architecture
• BIG-IP Next for Kubernetes
• Hardware Isolation

10. TL;DR

• F5 and NVIDIA introduced a security reference architecture for AI factories, replacing easily bypassed container sidecars with hardware-isolated inspection on Vera CPUs and BlueField-4 DPUs.
• F5 AI Guardrails and BIG-IP Next run on the DPU network path to block prompt injection, enforce MCP tool policies, and deliver 40% higher token throughput.
• Combines runtime supervision, silicon-level agent quarantine, and tamper-proof multi-plane auditing to ensure autonomous AI agents cannot disable security controls or manipulate audit logs.

