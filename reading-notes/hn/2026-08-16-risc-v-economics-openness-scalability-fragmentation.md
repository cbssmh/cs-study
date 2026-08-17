# 0. Filename

`2026-08-16-risc-v-economics-openness-scalability-fragmentation.md`

# 1. Title

A Third World Embedded Engineer Responds to “RISC-V: They Should Have Known Better”

# 2. Source

* **Author / Organization:** Armstrong Subero / RV Embedded
* **Link:** https://rvembedded.com/blog_post/12/
* **Date:** 2026-08-16

# 3. One-line Summary

RISC-V’s strongest advantage may not be ISA elegance or peak performance, but its combination of low cost, open licensing, customizable implementations, and a transferable ecosystem spanning tiny microcontrollers to Linux-capable systems.

# 4. Key Points

* The author accepts several technical criticisms of RISC-V, including awkward instruction encodings, extension fragmentation, and vendor-specific implementation differences.
* RISC-V critics themselves expect it to succeed strongly in ultra-low-cost embedded systems, where price and die area matter more than peak performance.
* Chips such as the CH32V003 demonstrate that a minimal RISC-V implementation can reach extremely low price points while retaining standard compiler and development tooling.
* The author argues that RISC-V knowledge transfers more easily across product classes because implementations retain a common base ISA, register model, calling convention, and toolchain.
* RISC-V extensions allow vendors to omit expensive features on small chips while adding MMUs, privilege modes, vector support, or other capabilities where required.
* This extensibility also creates fragmentation: different implementations may expose different extensions, interrupt mechanisms, and binary compatibility requirements.
* The article frames ARM’s Cortex-M/Cortex-A division partly as a product and licensing boundary, whereas RISC-V allows implementers to choose their own architectural boundaries.
* Open implementations such as VexRISC-V and Baochip demonstrate how designers can add capabilities such as MMUs without negotiating ISA licenses.
* For developers outside major technology markets, component availability, shipping cost, tool cost, and the ability to experiment with disposable hardware can matter more than theoretical ISA quality.
* Hacker News discussion highlights an unresolved distinction between RISC-V’s strong economic position in embedded systems and its still-unproven competitiveness against ARM and x86 in high-performance computing.

# 5. Deep Dive

## Problem

Most ISA discussions evaluate RISC-V primarily through technical criteria such as instruction encoding, interrupt latency, vector design, performance, and binary compatibility.

The article argues that this perspective misses another constraint: **who can afford to obtain, modify, experiment with, and ship the hardware at all.**

This becomes especially important in embedded systems, where:

* component prices can dominate educational and low-volume projects;
* development tools may cost more than the target hardware;
* shipping and distribution differ greatly by geography;
* proprietary ISA licensing limits what chip designers may modify;
* low-end and high-end products may be separated by vendor-controlled product families.

A second problem is architectural: a single ISA must support systems ranging from tiny MCUs to processors capable of running protected operating systems.

## Approach

The author evaluates RISC-V not only as an instruction set but as an **economic and development ecosystem**.

He compares several systems he has personally used:

* **CH32V003:** extremely inexpensive RV32-class MCU.
* **CH32H417:** higher-end dual-core RISC-V MCU with advanced peripherals.
* **Baochip:** RISC-V SoC with MMU and support for protected operating systems.
* **Orange Pi RV2:** Linux-capable RISC-V SBC.

The argument is that developers can move across these systems while retaining substantial knowledge of:

* registers,
* calling conventions,
* assembly structure,
* compiler tooling,
* debugging concepts,
* base ISA behavior.

RISC-V’s optional extensions then provide specialization without requiring every implementation to carry the full cost of high-end features.

## Key Insight

The central tradeoff is:

**RISC-V scalability and RISC-V fragmentation emerge from the same architectural mechanism.**

Optional extensions let an implementation avoid unnecessary hardware.

A tiny MCU can omit:

* hardware multiplication or division,
* privilege levels,
* MMUs,
* large register sets,
* complex vector capabilities.

A larger processor can add those capabilities while retaining the same architectural family.

However, the same flexibility means that two RISC-V processors may not implement identical extensions or system features.

Therefore:

> modularity enables scalability, but modularity also complicates compatibility.

The article further argues that openness changes who controls these tradeoffs.

With RISC-V, the implementer decides where functionality is added. With proprietary ISA ecosystems, some architectural choices are also constrained by licensing and vendor product segmentation.

## Result / Impact

RISC-V becomes attractive even if it is not technically optimal in every dimension.

Its value proposition becomes:

**good-enough architecture + low-cost silicon + open tooling + implementation freedom + transferable skills**

rather than:

**best possible ISA design**

This could allow RISC-V to become dominant first in inexpensive embedded systems, creating volume, expertise, tooling, and investment that may later support more capable processors.

# 6. Why It Matters

The article reframes CPU architecture competition from a purely technical problem into an **ecosystem economics problem**.

RISC-V connects several broader trends:

* **Open hardware:** processor architecture is becoming less tightly controlled by a single vendor.
* **Custom silicon:** companies increasingly want workload-specific processors and accelerators.
* **Technology sovereignty:** countries and companies want to reduce dependence on proprietary foreign CPU IP.
* **Vertical integration:** organizations increasingly design their own chips rather than purchasing standardized processor products.
* **Low-cost embedded computing:** inexpensive MCUs are enabling computation in increasingly disposable and ubiquitous devices.

The important question may therefore shift from:

> “Is RISC-V better than ARM?”

to:

> “Is RISC-V sufficiently good that ownership, customization, and economics outweigh its technical disadvantages?”

That is a much more favorable competitive question for RISC-V.

# 7. Critical Analysis

The article makes a strong economic argument but sometimes overextends it into architectural conclusions.

First, the comparison between cheap RISC-V parts and ARM ecosystems is not always equivalent. Very inexpensive ARM microcontrollers and debugging tools also exist, especially through Chinese suppliers. Some Hacker News participants therefore dispute the implication that low component cost is uniquely associated with RISC-V.

Second, shipping cost is mainly a logistics and distribution issue rather than an ISA property. RISC-V currently benefits from inexpensive Chinese components and direct distribution, but that does not guarantee the advantage will remain exclusive to RISC-V.

Third, the article’s examples do not fully answer the high-performance criticism. A CH32 MCU, Baochip, and a Linux SBC demonstrate architectural range, but they do not demonstrate competitiveness with high-end ARM application processors, Apple Silicon, or x86 server CPUs.

Fourth, a free ISA does not imply a free high-performance processor. Competitive CPUs still require:

* advanced microarchitecture,
* memory controllers,
* caches,
* interconnects,
* high-speed I/O,
* compiler optimization,
* physical design expertise,
* leading semiconductor process nodes.

These investments can cost hundreds of millions of dollars regardless of ISA licensing.

Fifth, RISC-V customization creates a coordination problem. Proprietary extensions may improve individual implementations while weakening binary portability and increasing compiler and library maintenance.

The strongest version of the article’s argument is therefore not that RISC-V is technically superior, but that **its economic and governance model may compensate for technical imperfections**.

# 8. Connections

## 1. Linux and Open-Source Infrastructure

RISC-V resembles Linux in an important economic sense.

Linux did not initially defeat proprietary Unix systems because every technical component was superior. Its openness allowed:

* widespread experimentation,
* vendor participation,
* low-cost deployment,
* shared tooling,
* incremental improvement.

RISC-V may follow a similar path if increasing adoption attracts enough engineering investment to close performance and tooling gaps.

## 2. x86 and Path Dependence

x86 demonstrates that ISA elegance is not sufficient to predict market success.

Despite substantial architectural complexity, x86 became dominant because of:

* software compatibility,
* manufacturing scale,
* engineering investment,
* compiler optimization,
* ecosystem lock-in.

This suggests that sufficiently large ecosystems can compensate for architectural weaknesses.

RISC-V could benefit from the same feedback loop if low-cost adoption creates enough volume.

## 3. ARM Licensing vs. Open ISA Governance

ARM and RISC-V represent different governance models.

ARM provides highly mature processor IP and ecosystems but retains substantial control over architecture licensing and product families.

RISC-V standardizes the ISA while allowing independent implementations.

The tradeoff resembles:

* proprietary platform vs. open standard,
* vertically controlled ecosystem vs. decentralized ecosystem,
* consistency vs. customization.

## 4. CUDA vs. Open Compute Ecosystems

The same tension appears in GPU computing.

CUDA succeeds partly because NVIDIA controls hardware and software together, producing a consistent and optimized platform.

Open alternatives provide greater independence but face fragmentation and coordination costs.

RISC-V faces a similar challenge: openness expands participation but makes ecosystem standardization harder.

## 5. Custom Silicon and AI Accelerators

AI has increased demand for workload-specific processors.

Companies such as hyperscalers increasingly design custom accelerators and SoCs.

An open ISA can provide a reusable general-purpose control processor around specialized accelerators without requiring a new ISA or proprietary architecture license for every design.

This may become one of RISC-V’s strongest industrial roles.

# 9. Keywords

* RISC-V
* Instruction Set Architecture
* ISA Fragmentation
* Open Hardware
* Embedded Systems
* Microcontroller
* ARM
* Custom Silicon
* Technology Sovereignty
* CPU Ecosystem

# 10. TL;DR

RISC-V does not need to be the technically best ISA to become commercially important.

Its open licensing, low-cost implementations, and customization freedom create advantages that proprietary architectures cannot replicate as easily.

Its greatest strength—modular extensibility—is also its greatest risk, because the same flexibility that enables scalability creates fragmentation.
