# HERMES.md in Git Commit Messages Caused Claude Code Billing Misrouting

## 2. Source

- Author / Organization: sasha-id / Anthropic Claude Code GitHub Issue
- Link: anthropics/claude-code issue #53262
- Date: 2026-04-25

## 3. One-line Summary

A Claude Code anti-abuse system incorrectly routed Max plan requests to extra-usage billing when recent Git commit messages contained the exact string `HERMES.md`.

## 4. Key Points

- The bug triggered from Git commit history, not from files on disk.
- The exact case-sensitive string `HERMES.md` caused failures.
- Lowercase `hermes.md`, `HERMES`, `HERMES.txt`, `AGENTS.md`, and `README.md` did not trigger the issue.
- Claude Code appears to include recent commit messages in its system prompt.
- Server-side routing treated affected requests as extra usage instead of plan quota.
- The reporter lost about `$200.98` in extra usage while weekly Max quota remained mostly unused.
- Error messages only said “out of extra usage,” hiding the content-based cause.
- Anthropic confirmed it was an “overactive anti-abuse system” and marked the issue fixed.
- Public pressure later led to refunds and credits for affected users.

## 5. Deep Dive

### Problem

Claude Code billing behavior depended on hidden prompt content derived from Git history. A harmless commit message string caused requests to bypass included subscription quota and consume paid extra usage.

### Approach

The user isolated the issue through binary testing: affected repositories, orphan branches, commit history changes, and minimal repositories with controlled commit messages.

### Key Insight

The failure was not caused by project files, model choice, or local configuration. It was caused by the literal string `HERMES.md` entering Claude Code’s context through recent Git commits.

### Result / Impact

A false-positive anti-abuse rule created real billing damage, blocked usage after extra credits ran out, and exposed poor observability in Anthropic’s billing and support flow.

## 6. Why It Matters

This shows how AI tooling can turn invisible context injection into operational and financial risk. Developer agents increasingly read repository metadata, but users often cannot see how that metadata affects routing, safety filters, or billing.

It connects to a broader shift: AI coding tools are no longer just editors; they are metered cloud services with policy layers that can silently change cost behavior.

## 7. Critical Analysis

- Billing should never depend on opaque content classification without clear user-visible explanation.
- The initial refusal to refund weakened trust more than the technical bug itself.
- “Anti-abuse system” is too vague; users need to know whether similar strings, files, or histories could trigger billing changes.
- The dashboard showing unused plan quota while extra usage was depleted indicates poor product-state consistency.
- The issue suggests Claude Code’s system prompt includes repository metadata in ways users may not fully understand or audit.
- Public escalation through Hacker News appearing faster than formal support is a bad signal for enterprise reliability.

## 8. Connections

- **Prompt injection surface**: Git history becomes part of model context and can influence downstream behavior.
- **AI billing observability**: Similar to cloud cost bugs where routing or classification errors create unexpected charges.
- **False-positive abuse detection**: Safety systems can misclassify benign input and cause denial of service or financial harm.
- **Developer agent context design**: Tools like Claude Code, Copilot, Cursor, and Codex must expose what project metadata is sent.
- **Support escalation dynamics**: Public visibility often pressures AI vendors into remediation faster than standard support paths.

## 9. Keywords

- Claude Code
- Anthropic
- HERMES.md
- billing bug
- anti-abuse system
- extra usage
- Max plan
- prompt context
- Git commit history
- AI coding agents

## 10. TL;DR

Claude Code misrouted requests to extra billing when Git history contained `HERMES.md`.

The root cause was an overactive anti-abuse system reading repository context.

The incident highlights hidden context, opaque billing, and weak support accountability in AI developer tools.
