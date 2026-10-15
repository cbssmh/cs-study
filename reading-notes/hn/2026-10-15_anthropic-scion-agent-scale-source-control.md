1. Title

Redesigning Source Control at Anthropic

2. Source

Author / Organization: Nick G. & the Source Control Team / Anthropic (Engineering at Anthropic)
Link: https://www.anthropic.com/engineering/redesigning-source-control
Date: 2026-10-13 (Updated 2026-10-14)

3. One-line Summary

Anthropic replaced Git across its massive monorepo with Scion, a custom server-authoritative version control system built by seven engineers in five months using Claude, discarding decentralized offline assumptions to scale beyond 30,000 merges per day and 25 million daily CI checkouts.

4. Key Points

• Shift in Developer Bottlenecks: As autonomous Claude agents increased monorepo merges from 2,500/day in May 2026 to over 30,000/day in September 2026, raw code authoring ceased to be the constraint; instead, human-scale developer infrastructure—source control, merge queues, and CI—became the primary operational ceiling.
• Breaking the Limits of Git: With a 100 GB main-branch history and a 25 GB working copy, naive daily checkouts across 25M CI jobs represented over 600 PB/day of Git traffic; scaling via hundreds of internal Git read mirrors collapsed under memory limits and stale repository snapshots.
• Discarding Decentralized Offline Assumptions: Scion explicitly abandons Git’s core design assumptions (offline capability, symmetric client-server parity, full local repository clones), architecting strictly around always-online autonomous agents and containerized CI jobs.
• Changesets and Spanner Storage: Replaces Git commits and tree snapshots with write-ahead log "changesets" that store only modified files relative to the first parent, pointing to compressed blobs in cloud storage with metadata persisted in Google Cloud Spanner.
• Recursive Derived-Data Engine: Computes blame, path history, and skip-list commit ancestry graphs out-of-band via a unified recursive engine, allowing engineers to add custom index structures or defer materialization without client-side modifications.
• Zero-Coordination Tiered Caching: A three-tier caching hierarchy (`scion-daemon` on nodes, `scion-cache` in memory per region, `scion-server` authoritative) achieves >99% cache hits, reducing server infrastructure footprint by 200x compared to Git mirrors and cutting CI checkout times from minutes to seconds.
• Working Copies are Optional and Sparse: CI jobs and agents can query diffs, blame, and ancestry programmatically via RPC or perform sparse/virtualized checkouts, eliminating file downloads for API-driven writes.
• Rethinking Merge Queues and CI: Because hundreds of changes merge concurrently, traditional global merge queue locks were abandoned; Anthropic shifted to post-merge and pre-deploy testing, letting Claude resolve semantic merge conflicts in seconds after they land.
• The Changing Role of Software Engineers: A team of seven engineers shipped Scion and Antique (code review) in five months with Claude; engineers found their primary value shifted from line-by-line coding to high-level system trade-offs, setting invariant boundaries, and steering agents away from myopic over-engineering.

5. Deep Dive (Structured Understanding)

Problem
Git was engineered for decentralized, offline human workflows where every developer clone is a complete repository. At Anthropic, autonomous Claude agents generate software continuously, increasing daily merges by more than 10x in four months (exceeding 30,000 merges/day). A 100 GB monorepo history subjected to 25 million daily CI checkouts generated unsustainable network and disk loads (hundreds of petabytes). Traditional Git mitigations (shallow clones, sparse checkouts, hundreds of read mirrors) degraded under write velocity: read mirrors ran out of RAM, snapshot seeds drifted rapidly from the tip of `main`, and packfile delta compression consumed massive server CPU. Git’s requirement for distributed consensus across refs and branches created an insurmountable serialization bottleneck.

Approach
Anthropic designed Scion from first principles for an always-online, agent-dominated ecosystem, leveraging Google Piper and Meta Sapling design precedents. Built by seven engineers in five months using internal Claude models (including Claude Mythos Preview), Scion enforces client-server asymmetry: the server handles all merges, rebases, blame, and history indexing, while clients run a lightweight daemon managing a local object cache. The underlying storage separates write-ahead changesets (stored in Spanner) from immutable content-addressed blobs. The checkout architecture decouples tree negotiation: clients request specific object hashes recursively over RPC without negotiating what they already possess, pulling through a multi-tiered regional cache (`scion-cache`) that streams pre-formatted disk blobs directly from memory.

Key Insight
1. Server-Authoritative Asymmetry for Agent Workloads: Because agents and CI containers run in cloud environments with permanent network connectivity, maintaining full repository history and local Git calculation engines on clients is wasteful. Centralizing computation on scalable server pools eliminates client bloat and reduces cluster infrastructure requirements by 200x.
2. Changeset-Centric Storage over Tree Snapshots: Storing only modified files relative to the main trunk parent decouples commit creation cost from repository size, allowing the system to accept incoming writes at high throughput without packfile locking or ref coordination.
3. Decoupling Base Changesets from Derived Indices: Rather than computing blame, graph skip-lists, and path search eagerly during write transactions, Scion computes them out-of-band using a recursive parent-derivation framework, allowing selective or lazy index materialization.
4. "Superfan" Autonomous Observability: Managing complex distributed systems at machine velocity requires continuous operational scrutiny; deploying specialized, persistent agent loops ("superfans") dedicated to single components detected subtle anomalies that outmatched static metric alert thresholds.

Result / Impact
Reduced source control infrastructure footprint by 200x, cut CI checkouts from minutes to seconds (saving 25% fleet CPU capacity), accommodated over 30,000 daily merges, and established a scalable template for post-Git developer infrastructure in the agentic era.

6. Why It Matters

• Platform Engineering & Developer Tooling: Signals the beginning of a generational departure from Git for frontier technology organizations, proving that AI-driven development velocity breaks legacy distributed VCS paradigms.
• Distributed Systems & Infrastructure: Offers a concrete architectural pattern for scaling version control to hundreds of millions of daily operations via tiered content-addressed object caches, server-side derivation, and Spanner metadata storage.
• Software Engineering Governance & IT Risk: Illustrates how code review and verification must transform when machine code generation outpaces human reading capacity, shifting governance to intent-level review, semantic approval constraints, and post-merge automated remediation.
• AI Engineering & Autonomous Systems: Demonstrates a recursive flywheel where frontier LLMs both create the scale crisis (by authoring code at superhuman volume) and supply the engineering velocity to build, test, and optimize custom distributed infrastructure to resolve it.

7. Critical Analysis

• Extreme Vendor and Token Asymmetry: Anthropic explicitly acknowledges that building Scion makes sense only because their internal access to model tokens is virtually unlimited. For standard enterprises, burning billions of tokens on autonomous agent loops and custom tool building would incur prohibitive cloud API costs.
• Total Ecosystem Incompatibility: Migrating off Git required building an entire proprietary ecosystem from scratch—including Antique (code review), code search, and custom IAM. Adopting this paradigm locks the organization into an in-house walled garden, forfeiting 20 years of open-source tooling, IDE integrations, and commercial CI/CD platforms.
• The "Janky Code" and Quality Debt Controversy: As highlighted in community discussions, eliminating code authoring friction and merge queue locks risks turning the monorepo into a repository for barely-functioning "good enough" code. Deferring integration testing to post-merge relies entirely on the premise that LLMs can fix production breaks faster than they create them, which remains unproven outside highly controlled environments.
• Single Point of Failure (SPOF) in Centralized Cloud Dependency: By abandoning offline capability, any network partition or outage in the centralized Scion service or underlying Spanner database instantly halts all development and CI across the company.

8. Connections

• Google Piper & Meta Sapling: Follows the enterprise monorepo lineage of Google’s internal Piper/Citc and Meta’s Sapling/Eden, validating server-side sparse virtual filesystems for hyper-scale codebases.
• GitHub Agent-Scale Git Replatforming: Provides an alternative design philosophy to GitHub’s approach: while GitHub redesigned backend fileservers to preserve Git wire compatibility, Anthropic discarded Git protocols entirely to optimize purely for agentic throughput.
• Virtual File System for Git (VFS for Git / Microsoft Scalar): Parallels Windows OS kernel virtualization techniques by virtualizing directory trees and hydrating file payloads strictly on demand.

9. Keywords

• Scion
• Version Control Systems
• Anthropic Engineering
• Agent-Scale Development
• Monorepo Architecture
• Client-Server Asymmetry
• Spanner
• Derived Data Engine

10. TL;DR

• Anthropic abandoned Git and deployed Scion, a custom server-authoritative VCS, after AI agents accelerated monorepo merges beyond 30,000 per day.
• Discards decentralized offline priors, organizing storage into write-ahead changesets in Spanner, out-of-band derived indices, and multi-tier memory caches that shrunk infrastructure by 200x.
• Highlights a fundamental shift in software engineering where human engineers guide architecture and reject agent over-engineering while code authoring and merge conflicts are automated.
