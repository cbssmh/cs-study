# 1. Title

QM: Multiplayer Agent Harness for Work

---

# 2. Source

- **Author / Organization:** YC Software (Y Combinator)
- **Link:** https://github.com/yc-software/qm
- **Discussion:** https://news.ycombinator.com/item?id=48898850
- **Date:** 2026-08-01

---

# 3. One-line Summary

QM is an open-source enterprise AI agent platform that introduces scoped, persistent, multi-user agent infrastructure rather than treating AI as a personal chatbot.

---

# 4. Key Points

- QM is an open-source **agent harness** designed for organizations rather than individual users.
- Every user, Slack channel, or project owns an isolated **Scope** containing its own memory, files, permissions, secrets, scheduled jobs, and sandbox.
- Supports multiple coding harnesses and models (Claude Code, Codex, OpenCode, Pi, etc.), reducing vendor lock-in.
- Each scope receives a **durable sandbox**, allowing installed tools, repositories, and environments to persist across sessions.
- Agents can perform long-running background tasks through scheduled jobs and watches instead of waiting for user prompts.
- Security follows a user-delegation model with Strict, Auto, and Dangerous execution modes.
- Organizations can build and deploy internal web applications directly through the platform.
- The project separates generic core infrastructure from organization-specific deployment layers.
- Contributions emphasize human-written design proposals instead of code submissions, assuming implementation is increasingly automated by coding agents.

---

# 5. Deep Dive (Structured Understanding)

## Problem

Most AI assistants are designed for a single user interacting with a stateless chatbot.

This model struggles in enterprise environments because organizations require:

- isolated permissions
- persistent working environments
- collaborative context
- long-running automation
- secure delegation of user authority

Traditional chat interfaces become difficult to scale across teams.

---

## Approach

QM introduces a **multi-scope agent operating environment**.

Instead of one assistant, every entity receives its own workspace:

- User Scope
- Project Scope
- Slack Channel Scope

Each scope contains:

- Memory
- Files
- Secrets
- Permissions
- Durable Sandbox
- Scheduled automation

The underlying LLM becomes replaceable while the execution platform remains constant.

---

## Key Insight

The innovation is **not a better language model**.

The innovation is treating AI as **persistent organizational infrastructure**.

Instead of conversations,

QM manages:

- identities
- permissions
- execution environments
- collaboration boundaries
- background automation

similar to how operating systems manage processes and users.

---

## Result / Impact

QM shifts enterprise AI architecture from

> Chat Interface → Tool Calls

toward

> Multi-tenant Agent Platform → Persistent Execution Environment

This enables organizations to build AI-native workflows rather than isolated chatbot interactions.

---

# 6. Why It Matters

- Enterprise AI adoption increasingly depends on orchestration rather than model quality.
- Persistent agents represent the next evolution beyond prompt-based assistants.
- Organizations need infrastructure for managing many AI workers instead of deploying isolated chatbots.
- The project reflects a broader industry transition toward AI operating systems and agent platforms.

---

# 7. Critical Analysis

- The "multiplayer" concept is largely an architectural abstraction rather than a fundamentally new AI capability.
- Many features overlap with existing enterprise automation platforms and agent frameworks.
- The repository emphasizes architecture but provides limited real-world deployment case studies or performance evidence.
- Persistent autonomous agents introduce operational complexity, governance requirements, and higher security risks that remain largely unaddressed.
- The contribution model (human-written proposals instead of code) assumes coding agents are sufficiently reliable, which may not hold for complex architectural changes.

---

# 8. Connections

### 1. Enterprise AI Platforms

QM resembles Microsoft's Copilot ecosystem and Anthropic's enterprise workflows, but focuses on infrastructure rather than proprietary assistants.

### 2. Multi-Tenant Cloud Architecture

The Scope model parallels Kubernetes namespaces, AWS IAM boundaries, and SaaS multi-tenant isolation by separating users, permissions, and resources.

### 3. Agentic AI Frameworks

Shares goals with OpenAI Agents, LangGraph, CrewAI, AutoGen, and OpenHands, but emphasizes long-lived execution environments over conversation orchestration.

### 4. Platform Engineering

Persistent sandboxes and deployment layers resemble Internal Developer Platforms (IDPs) such as Backstage and Golden Paths, extending them with AI-native workflows.

---

# 9. Keywords

- Multiplayer Agent
- Agent Harness
- Enterprise AI
- Multi-tenancy
- Durable Sandbox
- Scoped Memory
- Background Agents
- AI Infrastructure
- Platform Engineering
- Vendor Agnostic

---

# 10. TL;DR

- QM is an enterprise platform for managing persistent AI agents rather than a chatbot.
- Its core innovation is scoped, long-lived execution environments with isolated memory and permissions.
- The project reflects a broader shift from AI assistants toward organizational AI infrastructure.
