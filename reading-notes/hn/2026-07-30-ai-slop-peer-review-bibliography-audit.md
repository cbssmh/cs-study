# 1. Title

Q&A from the Slop Trenches: AI-Generated Papers, Hallucinated Citations, and Peer Review

---

# 2. Source

- **Author / Organization:** Caleb Robinson, Isaac Corley / GeoSpatial ML
- **Link:** https://geospatialml.com/posts/reviewing-ai-slop/
- **Date:** 2026-07-30

---

# 3. One-line Summary

AI-assisted scientific writing is overwhelming peer review with hallucinated references and low-effort submissions, prompting reviewers to advocate automated bibliography auditing and stronger quality gates.

---

# 4. Key Points

- The authors reviewed **22 ML conference submissions** (NeurIPS, WACV, TerraBytes); **15 papers (68%)** contained fabricated citations, hallucinated authors, or obvious LLM-generated content.
- Hallucinated references are increasingly reaching publication because peer review often fails to detect them.
- Existing incentives ("publish or perish") encourage quantity over careful verification, making AI-generated submissions easier to produce.
- Reviewers spend unpaid time verifying citations that could be automatically checked before submission.
- The authors released **bib-audit**, an open-source bibliography verification tool that checks references against Crossref, arXiv, Semantic Scholar, and DataCite.
- AI itself is not considered the problem; **unverified AI output** is.
- Conference policies often restrict reviewers from using external LLMs, limiting AI-assisted validation during peer review.
- The paper argues for automated pre-submission quality gates rather than banning LLM-assisted writing.

---

# 5. Deep Dive (Structured Understanding)

## Problem

Scientific publishing is experiencing a rapid increase in AI-assisted submissions containing fabricated citations, incorrect author lists, and hallucinated technical content. Human reviewers spend significant effort detecting obvious errors instead of evaluating scientific contributions.

## Approach

The authors analyze their own reviewing experience across major ML conferences, compare it with recent studies on hallucinated citations and AI-assisted peer review, and introduce **bib-audit**, a reference verification tool that automates bibliography validation before submission.

## Key Insight

The bottleneck has shifted:

- Writing is now inexpensive because of LLMs.
- **Verification has become the expensive step.**

Future research quality depends less on generating text and more on validating every generated claim, citation, and experimental result.

## Result / Impact

Rather than discouraging AI use, the authors advocate:

- automated citation validation,
- stronger submission quality filters,
- desk rejection of clearly unverified AI-generated papers,
- and preserving reviewers' limited time for evaluating actual scientific merit.

---

# 6. Why It Matters

- AI has dramatically lowered the cost of producing research papers but has **not reduced the cost of verification**.
- Scientific publishing is shifting from a writing bottleneck to a **quality assurance bottleneck**.
- Similar verification challenges are emerging in software development, legal drafting, and enterprise documentation where LLM-generated outputs require systematic validation.

---

# 7. Critical Analysis

- The dataset is limited to **22 reviewed papers**, making the reported 68% figure anecdotal rather than representative of the broader research community.
- The article focuses heavily on citation hallucinations but provides less discussion of whether the underlying scientific contributions were actually invalid.
- Strong recommendations for desk rejection assume fabricated references always indicate bad-faith submissions, although some may result from negligence rather than intentional misconduct.
- Automated bibliography auditing addresses reference accuracy but cannot assess novelty, experimental validity, reproducibility, or research significance.
- The proposed workflow increases submission quality but does not solve the structural incentives behind "publish or perish."

---

# 8. Connections

### 1. AI Code Generation

Similar to GitHub Copilot or Claude-generated code, AI-written research requires human verification. Unit tests and CI pipelines parallel bibliography validation in scientific publishing.

### 2. Software Supply Chain Security

Tools like Dependabot, SBOM scanners, and static analyzers automatically validate dependencies before deployment. **bib-audit** applies the same philosophy to academic references.

### 3. AI Hallucination Research

The article reflects the broader shift from improving generation quality toward building **verification layers** (fact checking, retrieval, grounding, citation validation).

### 4. Publish-or-Perish Incentives

Academic incentive structures reward publication volume, encouraging low-cost AI-generated submissions while placing increasing burden on volunteer reviewers.

### 5. AI Governance

The discussion aligns with growing emphasis on **AI assurance**, where governance focuses on validating outputs rather than prohibiting AI usage.

---

# 9. Keywords

- AI Slop
- Peer Review
- Hallucinated Citations
- Bibliography Verification
- bib-audit
- Scientific Publishing
- NeurIPS
- LLM Hallucination
- Research Integrity
- AI Assurance

---

# 10. TL;DR

- AI dramatically reduces the cost of writing papers but not the cost of verifying them.
- The real bottleneck in scientific publishing is shifting from generation to validation.
- Automated verification tools like **bib-audit** may become standard pre-submission quality gates.
