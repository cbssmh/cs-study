# Claude Code Quota Explosion: Context, Cache, and System Design Lessons

## Source
- GitHub Issue: Claude Code quota exhausted in 1.5 hours
- Link: https://github.com/anthropics/claude-code/issues/45756

---

## Overview

This note analyzes a real-world issue where Claude Code exhausted its quota in a very short time despite moderate usage.  

The problem is not simply a bug. It reveals a deeper structural issue in how Large Language Model (LLM) systems handle **context, caching, and session management**.

This case is important for backend engineers because it shows how system design decisions directly affect performance and cost.

---

## Problem Summary

- Plan: Pro Max 5x (Opus, 1M context)
- Expected: Several hours of moderate usage
- Actual: Quota exhausted in **1.5 hours**

Even after a quota reset, the system consumed tokens much faster than expected.

---

## Key Observation

The user measured:

- Moderate usage → low effective token usage (~8.7M/hour)
- But actual raw token usage → **~70M/hour**

This mismatch indicates that the system is consuming more tokens than expected.

---

## Root Cause Analysis

### 1. Full Context Re-sending

In LLM systems, each request includes the entire conversation history:

Request = previous context + new input

As the session grows:

- Context size increases
- Every request becomes more expensive

If context reaches ~1M tokens:

- Each request may cost ~1M tokens

---

### 2. Context Inflation

Context grows over time:

32k → 100k → 500k → 960k

Near the limit:

- Every API call sends almost the entire context
- Cost scales with context size

**Insight:**  
Cost depends more on **context size** than number of requests.

---

### 3. Cache Behavior and Misses

LLM systems use caching:

- `cache_read` → cheap
- `cache_create` → expensive

However:

- Cache TTL may be short (e.g., 5 minutes)
- If session pauses → cache expires
- Next request triggers full cache creation

This leads to sudden cost spikes.

---

### 4. Auto-Compact Overhead

When context becomes too large:

→ auto-compact triggers  
→ full context processed again  

This creates a **large hidden cost**:

- ~900k+ tokens in one operation
- Happens automatically (no user action)

---

### 5. Background Sessions

Multiple sessions were running:

- token-analysis
- career-ops

Even when idle:

- They still consume tokens
- They share the same quota

Result:

- Background sessions used ~78% of quota

---

### 6. Large Context Window Trade-off

A 1M context window seems beneficial, but:

Larger context → more tokens per request → faster quota depletion

This creates a trade-off:

- More context = better reasoning
- But also = higher cost

---

## Key Insights

### 1. Cost is Context-Driven

LLM cost is not linear:

- Not based on request count
- Based on **context size × request frequency**

---

### 2. Long Sessions Are Dangerous

Long-running sessions:

- Accumulate context
- Increase cost per request

Short, focused sessions are more efficient.

---

### 3. Cache is Critical

Caching is not just optimization:

- It is essential for cost control

Cache misses can multiply cost dramatically.

---

### 4. Sessions Are Active Processes

An open session is not idle:

- It may run background tasks
- It continues consuming resources

---

### 5. System Behavior is Non-Transparent

Important costs come from:

- Auto-compact
- Background processes
- Cache regeneration

These are often invisible to the user.

---

## Engineering Implications

This issue is not specific to Claude Code.

It applies to all LLM-based systems:

### Traditional Backend

Request → fixed cost

### LLM System

Request → cost depends on entire history

This makes LLM systems:

- Stateful
- Cost-sensitive
- Hard to predict

---

## Practical Guidelines

### 1. Keep Sessions Short

- One task per session
- Avoid long conversations

---

### 2. Use Context Management

- Reduce unnecessary history
- Use summarization or compaction

---

### 3. Monitor Cache Behavior

- Avoid frequent pauses (cache expiration)
- Reuse context efficiently

---

### 4. Close Unused Sessions

- Background sessions consume quota
- Always terminate unused ones

---

### 5. Be Careful with Large Context

- Bigger context is not always better
- Balance between accuracy and cost

---

## Conclusion

This issue highlights a key principle in LLM system design:

> In LLM systems, cost is driven by context, not just usage.

Efficient usage requires:

- Careful session management
- Context control
- Awareness of hidden system behaviors

For backend engineers, this is a reminder that:

**System design choices (state, caching, context size) directly impact scalability and cost.**

---

## One-Line Takeaway

In LLM-based systems, managing context is more important than managing requests.
