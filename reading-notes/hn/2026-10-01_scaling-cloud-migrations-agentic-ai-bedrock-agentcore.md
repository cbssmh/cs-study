1. Title

Scaling Cloud Migrations with Agentic AI on Amazon Bedrock AgentCore

2. Source

Author / Organization: Nikhil Jha, Kaushal Agrawal, Tarun Tarun, and Vyasamaharshi Garigipati / Amazon Web Services (AWS Machine Learning Blog)
Link: https://aws.amazon.com/blogs/machine-learning/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore/
Date: 2026-10-01

3. One-line Summary

AWS demonstrates a production-proven multi-agent architecture built on Amazon Bedrock AgentCore and Model Context Protocol (MCP) tools that compressed infrastructure-as-code generation from weeks to minutes across a 300+ application enterprise migration portfolio while enforcing Cedar-based policy guardrails and human approval gates.

4. Key Points

• Hybrid Migration Model: Complements managed migration services (AWS Transform for workload discovery and re-platforming, AWS DMS for database schema transformation) with custom, organization-specific AI agents integrated via the Model Context Protocol (MCP).
• Four Specialized Agent Personas: Divides operational responsibilities across two migration-journey agents (Intake Agent for automated discovery, IaC Agent for infrastructure code synthesis) and two operational agents (Governance Agent for Jira/Confluence alignment, SRE Agent for post-cutover telemetry and optimization).
• Drastic IaC Authoring Compression: Reduced custom infrastructure-as-code (IaC) composition time from 3–4 weeks per application to minutes, eliminating years of manual engineering overhead across a 300+ application portfolio.
• Machine-Checkable Security Baselines: The security office curates versioned policy schemas (e.g., KMS encryption standards, administrative CIDR blocks, mandatory tagging) exposed via an MCP tool (`get_policies`), injecting machine-verifiable assertions directly into the IaC Agent's prompt context.
• Dual-Layer Policy Enforcement: Evaluates Cedar rules at the AgentCore Gateway to authorize whether an agent can invoke a specific tool, while simultaneously validating that the synthesized IaC satisfies organizational security baseline standards.
• Zero-Credential Prompt Context: Prevents secrets, API tokens, and IAM keys from entering model context windows by having AgentCore Identity dynamically resolve short-lived, least-privilege credentials at the gateway boundary.
• Automated Inter-Agent State Persistence: Uses AgentCore Memory to store architecture definitions, dependency maps, and policy sets, eliminating manual handoffs between discovery, code generation, and deployment verification.
• Human-in-the-Loop Governance: Enforces mandatory human approval gates for all mutating infrastructure actions, ticketing updates (Jira/ServiceNow), and SRE remediation playbooks, ensuring agents act as copilots rather than unconstrained autonomous actors.

5. Deep Dive (Structured Understanding)

Problem
Enterprise cloud migration programs spanning hundreds of legacy applications face rigid fiscal-year deadlines and crippling bottlenecks in infrastructure-as-code (IaC) authoring. While hyperscaler migration tools (such as AWS Transform) handle generic lift-and-shift or containerization, they cannot accommodate bespoke internal standards—such as private Terraform/CloudFormation module registries, internal wiki documentation, enterprise ticketing workflows, and strict security baselines. Hand-authoring approved, compliant IaC compositions typically requires 3 to 4 weeks per application. Across a 300+ application portfolio, this manual toil creates years of backlogged engineering effort, inconsistent configurations across migration waves, and severe documentation drift.

Approach
AWS Professional Services developed a four-agent pattern using the open-source Strands Agents SDK running on Amazon Bedrock AgentCore. The Intake Agent queries internal wikis, questionnaires, and dependency mappings via custom MCP tools exposed by AgentCore Gateway to generate target architecture specs. The IaC Agent ingests these specs, queries wave-specific security policies through a `get_policies` MCP endpoint, and composes compliant IaC using approved module libraries. Bedrock Guardrails enforce inference-layer safety, while AgentCore Policy verifies tool execution against Cedar authorization rules. A Governance Agent continuously reconciles progress across Jira and Confluence, and a post-cutover SRE Agent analyzes CloudWatch telemetry to recommend resource right-sizing under human supervision.

Key Insight
1. Separation of Policy Authorization and Code Governance: The architecture cleanly splits governance into two decoupled layers: Cedar-based AgentCore Policy determines whether an agent has authorization to invoke a specific tool, while versioned JSON policy schemas evaluate whether the synthesized Terraform/CloudFormation code complies with security standards.
2. In-Band Policy Injection over Dynamic Retrieval: Rather than allowing the LLM to decide when or whether to look up compliance rules, the agent entrypoint deterministically queries `get_policies` for the target wave up-front and injects current rules and active exception waivers directly into the system prompt.
3. Secret-Isolated Tooling via MCP: Resolving credentials at the API Gateway level (via AgentCore Identity) ensures that zero secrets or administrative tokens enter the model context, eliminating prompt injection risks associated with credential exfiltration.
4. Continuous Lifecycle Continuity: By extending agentic workflows past cutover into an SRE Agent, the architecture avoids treating migration as a one-off project, establishing an automated pipeline for ongoing observability, tuning, and cost optimization.

Result / Impact
Demonstrated that agentic AI can eliminate repetitive platform engineering bottlenecks at enterprise scale, shrinking per-application IaC generation from weeks to minutes, enforcing 100% compliance auditability, and maintaining pattern consistency across 300+ production workloads.

6. Why It Matters

• AI Engineering & Secure AI: Serves as an architectural reference implementation for building production-grade enterprise agents, combining the Model Context Protocol (MCP), runtime memory persistence, session isolation, and deterministic guardrails.
• Platform Engineering & Cloud / DevOps: Illustrates the transition of platform teams from manually writing infrastructure templates to curating machine-readable module libraries, policy assertion engines, and MCP tool endpoints that AI agents consume.
• Security / DevSecOps & Governance: Provides a robust blueprint for policy-as-code governance in agentic workflows, proving that LLMs can safely operate on infrastructure when constrained by least-privilege IAM roles, Cedar authorization rules, and immutable audit logs.
• IT Risk / Governance: Addresses compliance liability by attaching exact policy set version hashes and waiver expiration timestamps to generated infrastructure artifacts, creating end-to-end provenance for audit committees.

7. Critical Analysis

• Heavy Reliance on Upstream Data Quality: The Intake Agent's accuracy is strictly bounded by the completeness of existing architecture documents, questionnaires, and wiki pages. In legacy enterprise portfolios with outdated or contradictory documentation, human validation of the initial discovery phase remains a non-negotiable bottleneck.
• Vendor and Ecosystem Lock-in: Although the agents utilize the open-source Strands SDK and the open Model Context Protocol, the implementation is deeply coupled to proprietary AWS services (Bedrock AgentCore, Bedrock Guardrails, AWS Transform, CloudWatch). Porting this pattern to multi-cloud or hybrid on-premises setups requires replacing multiple core proprietary primitives.
• High Inference and Tooling Run Costs: While saving human engineering weeks, passing full architecture schemas, module definitions, and compliance rules through foundation models incurs significant token consumption. In long-running multi-agent pipelines with frequent iteration loops, inference costs can scale unpredictably unless aggressive prompt caching and local model tiering are employed.
• Residual Risk of Synthesized Configuration Errors: Although policy assertions catch known rule violations (e.g., unencrypted S3 buckets), LLMs can still generate subtly invalid resource relationships, race conditions in state locking, or sub-optimal networking routes that pass syntactic validation but fail during deployment execution.

8. Connections

• Model Context Protocol (MCP): The emerging open standard for connecting AI systems to enterprise data sources, development tools, and internal APIs in a standardized, secure manner.
• Cedar Policy Language (AWS Verified Permissions): The declarative, fine-grained authorization engine used to enforce policy-based access control over agent tool calls prior to execution.
• Platform as a Product & Internal Developer Platforms (IDP): Aligns with platform engineering trends where central platform teams publish curated APIs and modules, treating AI agents as primary internal consumers alongside human developers.

9. Keywords

• Amazon Bedrock AgentCore
• Model Context Protocol (MCP)
• Cloud Migration Architecture
• Infrastructure as Code (IaC)
• Strands Agents SDK
• Cedar Policy
• Agentic AI
• DevSecOps

10. TL;DR

• AWS deployed a four-agent pattern on Bedrock AgentCore to compress enterprise cloud migration IaC authoring from weeks to minutes across 300+ applications.
• Integrates MCP tools, Cedar-based authorization, and versioned compliance policies to generate secure, approved infrastructure without exposing credentials to model context.
• Combines automated discovery, code synthesis, and post-cutover SRE tuning with mandatory human approval gates to bridge the gap between managed migration services and custom enterprise standards.
