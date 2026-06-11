# 1. Title



πfs: The Data-Free Filesystem and the Limits of Compression



# 2. Source



* Author / Organization: Philip Langdale

* Link: https://github.com/philipl/pifs

* Date: 2026-06-11 (HN discussion resurfaced)



# 3. One-line Summary



πfs is a humorous filesystem that "stores files in the digits of π," illustrating fundamental limits of compression, metadata, and information theory.



# 4. Key Points



* πfs is a FUSE filesystem that stores file locations in π rather than storing file contents directly.

* The project relies on the conjecture that π is a normal number and therefore contains every finite digit sequence.

* Any file could theoretically be represented by its position and length within π.

* The implementation stores metadata describing where bytes occur inside π.

* Metadata quickly becomes larger than the original file, eliminating any storage advantage.

* The project serves as a practical demonstration of information-theoretic limits.

* Hacker News discussions highlighted that file addresses require roughly as much information as the original data.

* The concept resembles Borges' "Library of Babel," where all possible books already exist.

* Modern LLM discussions revived interest because models can be interpreted as lossy compression systems.

* The project remains valuable as a thought experiment rather than a usable storage technology.



# 5. Deep Dive (Structured Understanding)



## Problem



Traditional storage systems consume physical space by storing data directly.



This raises a thought experiment:



> If all possible files already exist somewhere inside an infinite mathematical object such as π, do we really need to store data?



## Approach



πfs treats π as a universal storage medium.



Instead of storing:



* file contents



it stores:



* index within π

* length

* metadata



A file can theoretically be reconstructed by extracting the corresponding digit sequence from π.



To improve lookup performance, the implementation searches and stores individual bytes separately.



## Key Insight



The filesystem appears to achieve "100% compression," but the compression is illusory.



The location information required to recover data is itself information.



Information cannot disappear merely by changing its representation.



The filesystem exposes a core principle:



> Metadata is data.



## Result / Impact



The project demonstrates why perfect general-purpose compression is impossible.



Although mathematically intriguing, storage requirements simply move from file contents to file addresses.



The result is a memorable illustration of:



* Shannon information theory

* Kolmogorov complexity

* Compression limits

* Addressability costs



# 6. Why It Matters



* Demonstrates why information cannot be compressed arbitrarily.

* Provides an intuitive example of Shannon's limits.

* Helps explain why metadata management matters in storage systems.

* Connects surprisingly well to current discussions around AI and model compression.

* Shows the difference between lossless representation and useful abstraction.



Broader trend:



The project has resurfaced because AI systems increasingly frame intelligence as compression, prediction, and world modeling.



# 7. Critical Analysis



## Unproven Assumption



The concept assumes π is normal.



This has never been proven.



If π is not normal, the core premise weakens significantly.



## Compression Misconception



The project intentionally ignores address costs.



For arbitrary data, locating a sequence inside π generally requires as much information as storing the sequence itself.



## Computational Feasibility



Even if the mathematical premise were true, searching for long sequences inside π is computationally impractical.



The lookup cost dominates any theoretical storage benefit.



## Storage Reality



The implementation ultimately stores large amounts of metadata.



Storage consumption is shifted rather than eliminated.



# 8. Connections



## 1. Library of Babel



Jorge Luis Borges' fictional library contains every possible book.



πfs applies the same idea to digital files.



## 2. Shannon Information Theory



The project illustrates why arbitrary data cannot be compressed below its information content.



## 3. Kolmogorov Complexity



The shortest description of an object often approaches the object's own complexity.



File locations in π become another description of the file.



## 4. Data Deduplication



Real systems such as ZFS and enterprise storage arrays reduce redundancy by sharing identical blocks.



Unlike πfs, these approaches provide measurable storage savings.



## 5. LLMs as Compression



Recent AI discussions frame language models as lossy compressors that store patterns instead of exact data.



πfs highlights why lossless universal compression remains impossible.



# 9. Keywords



* PiFS

* Data-Free Filesystem

* Information Theory

* Shannon Entropy

* Kolmogorov Complexity

* Normal Number

* Compression

* Metadata

* Library of Babel

* LLM Compression



# 10. TL;DR



* πfs stores file locations in π instead of storing file contents.

* The project demonstrates that metadata ultimately carries the same information burden as the original data.

* It remains one of the most memorable practical thought experiments about compression, information theory, and modern AI-as-compression discussions.
