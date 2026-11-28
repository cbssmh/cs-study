1. Title
False computer-checked proofs

2. Source
Author / Organization: D. J. Bernstein
Link: https://cr.yp.to/proofs/falseccp.html
Date: November 25, 2026
HN discussion: https://news.ycombinator.com/item?id=50667989

3. One-line Summary
A catalog of 197 reported soundness bugs across seven major proof assistants shows that computer-checked proofs still depend on unverified or complex trusted computing bases—requiring tiny, audited kernels, multi-checker diversity, and simpler verified checker binaries.

4. Key Points
- **Catalog of soundness bugs across seven provers**: Compiling reports across default configurations (excluding UI-level misleading-output bugs), Bernstein documents `3` soundness bugs in HOL Light (not Candle), `7` in Mizar (not `mizar-rs`), `9` in Metamath (not `mmverify.py`), `20` in Isabelle, `23` in Lean 3/4 (not `nanoda`, excluding `native_decide`), `24` in HOL98/HOL4 (not experimental kernel), and `111` in Coq/Rocq (not `coqchk`/`rocqchk`).
- **2026 surge in discovered kernel flaws**: While HOL Light (`1996`–`2003`) and Mizar (`2001`–`2023`) show few or older reports, Isabelle (`7` in 2026), Lean (`10` in 2026), HOL98/HOL4 (`14` in 2026), and Coq/Rocq (`42` in 2026) saw sharp increases in reported soundness bugs during 2026.
- **Small LOC alone is insufficient**: Although HOL Light's 400-line kernel is attractive for minimizing the trusted computing base (TCB), line count neither measures code-review rigor nor eliminates bugs in the underlying programming language or runtime.
- **Multi-checker validation has limits**: Running the same proof through multiple isolated proof checkers catches single-implementation bugs (especially when sharing an input format), but Bernstein cautions that independent checkers can still share underlying specification or logic bugs.
- **Self-verified and cross-verified checkers**: Progress includes a 2006 HOL Light proof of its own inference rules (excluding definition rules, under large-cardinal `Vω+ω` or without the axiom of infinity), plus 2022 machine-checked proofs in HOL4 for `Candle` (a HOL Light clone, conditional on faithful CPU instruction execution) and a standalone OpenTheory HOL checker.
- **Trusting large binaries remains a risk**: Bernstein argues that justifying trust in one large proof-checking binary because another large binary verified it is still unsatisfactory from a security perspective, calling for simpler proof-checking binaries.

5. Deep Dive (Structured Understanding)
**Problem**
Formal verification is increasingly positioned as the gold standard for high-assurance cryptography, OS kernels, and AI-generated mathematics/code. Yet "machine-checked" only means *accepted by a specific proof-checking program*—and soundness bugs inside those checkers can allow `False` to be proven and accepted.

**Approach**
Bernstein compiles a longitudinal inventory of reported soundness bugs across seven major proof assistants (with archival links), contrasts three defense-in-depth strategies—TCB minimization, multi-checker diversity, and formally verified proof checkers—and evaluates where residual trust assumptions remain (underlying languages, CPUs, UI layers, and binary size).

**Key Insight**
Formal verification does not eliminate the trusted computing base; it *concentrates* trust into the proof checker, its host language/compiler, and the hardware executing it. As proof volume scales—especially with automated and AI-driven proof search—rare type-theory corner cases and kernel bugs shift from theoretical curiosities to actively probed attack surfaces.

**Result / Impact**
The catalog provides a concrete empirical baseline showing that soundness bugs are a recurring, non-zero reality in mainstream proof assistants, strengthening the case for stripped-down standalone checkers (`coqchk`, `nanoda`, `mmverify.py`, `Candle`) and simplified TCBs.

6. Why It Matters
- **Security / DevSecOps & IT Risk / Governance**: Prevents "verification theater" in high-assurance engineering (cryptographic libraries, secure enclaves, microkernels) by clarifying that a green prover check is only as trustworthy as the checker kernel, compiler, and UI presenting the theorem statement.
- **AI Engineering / Secure AI**: Directly relevant to LLM-assisted formal verification and automated theorem proving, where high-volume search and optimization loops can inadvertently—or adversarially—exploit soundness loopholes unless paired with independent hardened kernels.
- **Software-Industry Changes**: Reinforces the classic systems-security principle of minimizing TCB complexity at a moment when industry teams are rapidly adopting Lean, Isabelle, and Rocq to certify AI-written or critical infrastructure code.

7. Critical Analysis
- **Raw bug counts without normalization or status**: The list aggregates every historical soundness report equally without distinguishing fixed vs. open bugs, severity/exploitability, or differences in codebase age, feature complexity (dependent type theory in Coq/Lean vs. simpler foundations in HOL Light/Metamath), and user-base/fuzzing intensity.
- **Excludes standalone hardened checkers by design**: By tracking default usages (`Coq/Rocq` rather than `coqchk`, `Lean` rather than `nanoda`, `HOL Light` rather than `Candle`), the headline numbers reflect interactive prover front-ends/default kernels rather than the stricter multi-checker pipelines actually used in high-assurance settings.
- **No comparative baseline**: As critics noted, counting soundness bugs in proof assistants without comparing against bug rates in unverified compilers/static analyzers or counting real defects prevented by verification leaves the net risk-reduction trade-off unquantified.
- **HN community perspective**: Practitioners (`twotwotwo`, `dwheeler`, `gr_norm`, `u1hcw9nx`, `nickpsecurity`) emphasized that Metamath's main databases require passing 5 independent verifiers in 5 languages (C, Java, Rust, C++, Python), Lean's `lake check --paranoid` and verified kernels (`con-leche`) target exactly this threat, and the 2026 bug spike largely reflects LLMs probing obscure type-theory corners—though they agreed that minimizing and cross-verifying kernels remains essential.

8. Connections
- **Ken Thompson's "Reflections on Trusting Trust" & Diverse Double-Compiling**: Bernstein's concern about verifying one large binary with another large binary—and relying on the underlying language/CPU—echoes Thompson's compiler-trust problem and David A. Wheeler's Diverse Double-Compiling (DDC) countermeasure.
- **Defense-in-Depth for AI-Generated Proofs**: Parallels the growing consensus in AI-assisted formal methods (such as Lean's external kernel arena and `Candle`/CakeML on verified ARMv8 semantics) that automated proof generation must be decoupled from multiple small, independent, non-LLM-authored verifiers.
- **Minimal-TCB Systems Engineering**: Aligns with Bernstein's broader security philosophy (`qmail`, `djbdns`, Curve25519) and microkernel/hypervisor isolation (seL4, Hyperlight): security guarantees degrade rapidly as TCB line count and feature surface expand.

9. Keywords
Formal Verification, Soundness Bugs, Proof Assistants, Trusted Computing Base (TCB), Lean 4, Coq / Rocq, Isabelle, HOL Light, Candle, Multi-Verifier Diversity

10. TL;DR
D. J. Bernstein catalogs 197 reported soundness bugs across seven major proof assistants—ranging from 3 in HOL Light to 111 in Coq/Rocq—with a sharp 2026 uptick in Isabelle, Lean, HOL4, and Coq/Rocq.
Minimizing kernel size (like HOL Light's 400 lines) helps, but bugs in underlying languages, shared flaws across checkers, UI spoofing, and large verified binaries still leave meaningful trust gaps.
For high-assurance and AI-generated proofs, teams should avoid trusting a single default prover binary and instead combine minimal standalone checkers, independent multi-verifier pipelines, and human review of theorem statements.
