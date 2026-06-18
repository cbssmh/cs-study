# 1. Title



Lore: Epic Games' Open-Source Version Control System for Large-Scale Binary Assets



# 2. Source



* Author / Organization: Epic Games

* Link: https://lore.org

* Date: 2026-06-18 (public launch discussion)



# 3. One-line Summary



Epic Games open-sourced Lore, a centralized version control system designed to replace Perforce for large-scale game development workflows involving massive binary assets.



# 4. Key Points



* Lore is a centralized, content-addressed version control system optimized for large binary assets such as textures, 3D models, audio, and video.

* Repository state is represented using Merkle trees and immutable revision chains for integrity verification.

* Files are stored as reusable chunks, enabling deduplication and partial updates instead of transferring entire files.

* Supports sparse workspaces and on-demand hydration, allowing users to download only required assets.

* Branches are lightweight references that can be created and switched without duplicating underlying data.

* Provides SDKs and APIs for C/C++, C#, Rust, Go, Python, and JavaScript.

* Distributed peer-to-peer workflows are explicitly not a design goal; Lore is centralized by design.

* The system is MIT licensed and positioned as a fully open-source alternative to Perforce.

* Lore originated from Epic's internal Unreal Revision Control system used in UEFN and Fortnite-related workflows.

* Primary target users are game studios and asset-heavy organizations rather than traditional software teams.



# 5. Deep Dive (Structured Understanding)



## Problem



Traditional software version control systems are optimized for source code, not massive binary assets.



Game studios routinely manage:



* Multi-terabyte repositories

* Millions of files

* Large binary assets that cannot be merged effectively

* Asset locking requirements

* Selective access controls

* Large teams of artists, designers, and engineers



Git addresses source-code collaboration well but struggles with:



* Large binary files

* Git LFS complexity

* Storage growth

* Sparse workflows

* Asset locking

* Fine-grained permissions



Perforce became the industry standard largely because it solved these problems despite operational complexity and proprietary licensing.



## Approach



Lore adopts a centralized architecture and treats binary assets as first-class citizens.



Key design choices:



* Content-addressed storage

* Chunk-based file storage

* Immutable revision chains

* Sparse workspace support

* On-demand file hydration

* Service-oriented backend with caching layers

* Lightweight branch references



Instead of extending Git with additional systems (LFS, locking, sparse checkout), Lore integrates these capabilities directly into the core architecture.



## Key Insight



The design assumes that game development has fundamentally different requirements than software development.



Git's strengths come from assumptions such as:



* Text-based files

* Frequent merging

* Distributed workflows



Game development often requires:



* Large binary assets

* File locking instead of merging

* Centralized authority

* Partial asset retrieval



Rather than adapting Git further, Lore redesigns the storage and collaboration model around those realities.



## Result / Impact



If successful, Lore could become:



* An open-source alternative to Perforce

* A common backend for Unreal Engine workflows

* A foundation for asset-heavy industries beyond gaming



Potential adopters include:



* Game studios

* VFX pipelines

* CAD environments

* Digital content production

* Large-scale AI dataset management



# 6. Why It Matters



## Why this is important



For decades, large game studios have relied on proprietary systems because Git was not designed for binary-heavy workloads.



Lore represents one of the first serious attempts by a major industry player to create an open-source replacement tailored to those requirements.



## Broader Trend



This reflects a broader industry shift:



* Infrastructure becoming domain-specific

* Version control expanding beyond source code

* Asset management and source control converging

* Open-source alternatives challenging entrenched enterprise software



# 7. Critical Analysis



## Assumptions



* Lore assumes centralized workflows are preferable for asset-heavy environments.

* It assumes binary asset management is more important than distributed collaboration.



These assumptions hold for many game studios but may not generalize across all industries.



## Weaknesses



* Real-world scalability outside Epic remains unproven.

* Ecosystem support is far behind Git.

* IDE integration, CI/CD tooling, and code review workflows are immature compared to GitHub-centric ecosystems.

* Centralization introduces operational dependencies absent from distributed VCS models.



## Missing Context



* No large independent studio has publicly validated migration at scale.

* Long-term governance remains unclear despite MIT licensing.

* Adoption depends heavily on tooling and workflow integration, not only architecture.



## Potential Overstatement



Some marketing positions Git limitations as architectural failures when many organizations successfully operate with Git + LFS + custom workflows.



The true differentiator will be operational simplicity, not merely technical capability.



# 8. Connections



## 1. Perforce (Helix Core)



* Direct competitive target.

* Similar centralized architecture.

* Focus on binary assets and file locking.



## 2. Git LFS



* Attempts to solve large-file problems within Git.

* Lore integrates chunking and asset management at the storage layer instead of as an extension.



## 3. Google Piper



* Large-scale centralized repository system.

* Demonstrates that centralized models remain viable at massive scale.



## 4. AI Dataset Versioning



* Similar storage challenges appear in ML workflows.

* Comparable to systems such as LakeFS, XetHub, Pachyderm, and Oxen.



## 5. Unreal Engine Ecosystem



* Likely to become a first-class workflow component for Unreal-based development.

* Could reduce dependence on Perforce within Epic's ecosystem.



# 9. Keywords



* Lore

* Epic Games

* Version Control System

* Perforce

* Git LFS

* Merkle Tree

* Content Addressable Storage

* Binary Assets

* Sparse Checkout

* Chunked Storage



# 10. TL;DR



* Lore is Epic Games' open-source, centralized alternative to Perforce.

* It is designed specifically for large binary assets and multi-terabyte repositories.

* Its success depends less on architecture and more on ecosystem adoption and tooling maturity.
