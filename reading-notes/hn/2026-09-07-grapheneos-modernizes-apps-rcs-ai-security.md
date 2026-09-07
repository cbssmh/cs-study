## 1. Title

GrapheneOS Overhauls Default Apps, Plans Independent RCS, and Uses AI for Security Review

## 2. Source

- Author / Organization: GrapheneOS / Hacker News discussion
- Link: https://grapheneos.social/@GrapheneOS/117225539756835649
- Date: 2026-09-07

## 3. One-line Summary

- GrapheneOS is expanding beyond Android security hardening into modern first-party apps, independent RCS support, and AI-assisted vulnerability discovery while attempting to reduce reliance on proprietary Google infrastructure.

## 4. Key Points

- GrapheneOS is rewriting its bundled Messaging app with Jetpack Compose, alongside extensive bug fixes, testing, and code-quality improvements.
- The project is increasing investment in bundled applications after historically concentrating development resources on the base OS.
- The outdated AOSP Gallery will be replaced rather than incrementally modernized, while the AOSP Keyboard may receive similar treatment.
- Long-term RCS support is planned directly in GrapheneOS Messaging, including end-to-end encryption through Messaging Layer Security (MLS).
- Initial RCS implementation may still depend on sandboxed Google Play for activation and infrastructure before GrapheneOS can reduce Google dependency.
- GrapheneOS argues that RCS is not meaningfully open in practice because implementations depend heavily on Google and carrier-controlled infrastructure.
- RCS is nevertheless valuable because it provides a default, cross-platform successor to SMS/MMS without requiring contacts to install Signal or another third-party messenger.
- GrapheneOS uses LLMs primarily for code review, vulnerability discovery, boilerplate, and test generation rather than directly accepting large amounts of generated production code.
- The project claims current frontier models produce too much unreliable code for its standards but are highly useful as aggressive reviewers capable of identifying issues missed during human review.
- Hacker News discussion highlighted broader issues around RCS openness, FOSS vs. source-available licensing, GPL compatibility, AOSP sustainability, and Google's influence over the Android ecosystem.

## 5. Deep Dive (Structured Understanding)

### Problem

GrapheneOS inherits parts of its user-facing experience from AOSP, but several bundled AOSP applications have received little development and now lag behind modern Android applications.

At the same time, avoiding proprietary Google applications creates practical compatibility problems. RCS illustrates this tension: GrapheneOS users can currently obtain RCS through Google Messages and sandboxed Google Play, but this introduces dependencies the project would prefer not to require.

Security development faces another scaling problem. Large projects such as the Linux kernel, Chromium, Firefox, and Android contain enough code that traditional human review and static analysis cannot efficiently uncover every vulnerability.

### Approach

GrapheneOS is addressing these problems on three fronts.

**Application modernization**

The Messaging app is being migrated to Jetpack Compose while retaining incremental development, human review, testing, and staged Alpha/Beta/Stable releases. Applications too outdated to justify modernization, particularly AOSP Gallery, will instead be replaced.

**RCS independence**

The project plans to implement the Google Messages-side functionality required for RCS first. Early versions may rely on sandboxed Google Play for activation and related infrastructure.

The longer-term objective is an independent open-source GrapheneOS RCS client capable of communicating through carrier infrastructure without requiring Google Messages or ideally Google infrastructure.

**AI-assisted security engineering**

GrapheneOS treats LLMs primarily as additional analysis tools rather than autonomous programmers.

The workflow is approximately:

`Human implementation → human review → LLM review → validate findings → fixes/tests → incremental merge`

False positives and hallucinations are acceptable if the process exposes real defects that previous review missed.

### Key Insight

The common thread is reducing dependency without sacrificing compatibility.

GrapheneOS does not simply remove Google components. It attempts to preserve the practical Android ecosystem while replacing dependencies where technically feasible.

This explains the apparently contradictory decision to implement RCS despite criticizing its ecosystem: Signal may provide a cleaner security architecture, but RCS provides interoperability with default Android and iOS messaging without requiring social coordination.

The same pragmatic principle applies to AI. GrapheneOS does not need generated code to trust an LLM as a security tool; unreliable suggestions can still be useful when humans verify every reported issue.

### Result / Impact

GrapheneOS is evolving from primarily a hardened Android distribution toward maintaining more of its own application layer.

Successful independent RCS support could reduce reliance on Google Messages while maintaining communication compatibility with mainstream Android and iOS users.

AI-assisted review could also shift security engineering from using LLMs primarily for code generation toward using them as probabilistic vulnerability-analysis systems complementing deterministic tooling and human review.

## 6. Why It Matters

- GrapheneOS demonstrates the growing difficulty of building a usable privacy-oriented mobile OS while remaining compatible with ecosystems increasingly shaped by proprietary services.
- The RCS problem exposes the distinction between an open specification and an operationally open ecosystem: a protocol can be standardized while deployment remains controlled by a small number of infrastructure providers.
- Replacing abandoned AOSP applications shows how downstream open-source projects increasingly inherit maintenance responsibilities when upstream vendors prioritize proprietary products.
- The project illustrates a potentially important LLM security pattern: models may create more value as adversarial reviewers than as primary code authors.
- Rapid AI-assisted vulnerability discovery could temporarily increase disclosed vulnerabilities while ultimately reducing latent vulnerability density in heavily analyzed projects.
- Licensing discussions show that source visibility alone does not determine whether software can be integrated into an open-source operating system.

## 7. Critical Analysis

- GrapheneOS's independent RCS plan remains technically and institutionally uncertain because carriers and Google control important infrastructure outside the client.
- An open-source RCS client would therefore not automatically create an open RCS ecosystem.
- Claims that LLMs are dramatically improving vulnerability discovery are plausible within the project's experience but do not establish how many findings are genuinely novel versus bugs discoverable through expanded conventional analysis.
- Rising CVE counts do not by themselves prove worsening software security; they can reflect improved detection and disclosure of vulnerabilities that already existed.
- Comparing LLM review with traditional static analysis can be misleading because deterministic analyzers and probabilistic language models have different strengths, costs, reproducibility characteristics, and false-positive profiles.
- Hacker News licensing claims are inconsistent in places. Distribution of separately licensed components does not automatically force an entire operating system under one license; derivative-work boundaries and project-specific licensing policy matter.
- RCS's importance is geographically uneven. Its interoperability advantage is particularly relevant where default phone messaging remains dominant, while markets centered on WhatsApp or other third-party messengers have weaker incentives.

## 8. Connections

### AOSP vs. Google Mobile Services

GrapheneOS illustrates the distinction between Android Open Source Project (AOSP) and Google's proprietary service layer. Android can remain open source while mainstream applications increasingly depend on services such as Google Play Services, Firebase Cloud Messaging, or Google-operated infrastructure.

### RCS, MLS, and Signal

RCS attempts to modernize carrier messaging with richer media, group messaging, and E2EE through Messaging Layer Security. Signal represents a different model: a dedicated application and protocol can optimize security more aggressively, while RCS prioritizes interoperability and default availability.

### Jetpack Compose and Legacy Android UI

Migrating Messaging to Jetpack Compose reflects the broader Android transition from legacy View-based UI development toward declarative UI. For GrapheneOS, modernization also lowers the cost of future UI development on applications inherited from aging AOSP codebases.

### LLMs as Security Analysis Tools

GrapheneOS's workflow connects to a broader shift from AI-assisted code generation toward AI-assisted code review, fuzzing support, vulnerability research, and test generation. Human verification remains the trust boundary.

### Open Source vs. Source Available

The FUTO Keyboard discussion illustrates why visible source code does not necessarily satisfy the Open Source Definition. Restrictions on commercial use, modification, or redistribution can make software source-available rather than open source.

### Android Security Architecture vs. Linux Containers

The discussion around Waydroid highlights Android's dependence on application sandboxing and SELinux. Running Android applications is not merely an API-compatibility problem; reproducing Android's isolation and kernel-security assumptions is essential for maintaining its security model.

## 9. Keywords

- GrapheneOS
- AOSP
- RCS
- Messaging Layer Security (MLS)
- End-to-End Encryption (E2EE)
- Jetpack Compose
- Google Play Services
- LLM Code Review
- Vulnerability Discovery
- Open Source Licensing

## 10. TL;DR

- GrapheneOS is modernizing or replacing outdated AOSP apps and expanding from OS hardening into first-party application development.
- Its long-term RCS goal is interoperable E2EE messaging without requiring Google Messages, although Google/carrier infrastructure remains a major obstacle.
- GrapheneOS treats LLMs primarily as human-supervised security reviewers and vulnerability-discovery tools rather than autonomous production-code generators.
