## 0. Filename

`2026-08-20-github-august-17-outage-capacity-failure.md`

## 1. Title

The August 17 Outage, and the Work Ahead

## 2. Source

* **Author / Organization:** Vlad Fedorov / GitHub
* **Link:** https://github.blog/news-insights/company-news/the-august-17-outage-and-the-work-ahead/
* **Date:** 2026-08-20

## 3. One-line Summary

GitHub’s August 17 outage was a capacity failure triggered by record traffic, exposing scaling bottlenecks, retry amplification, and shared dependencies that GitHub is addressing through Azure migration, architectural isolation, and stronger resilience controls.

## 4. Key Points

* GitHub experienced a **7-hour 47-minute outage** on August 17, affecting authentication, GitHub Actions, APIs, pull requests, issues, Copilot, and github.com.
* A critical infrastructure component in the **Central US data center failed to scale** when traffic reached a new peak.
* Capacity pressure propagated through dependent systems and caused authentication failures and broader service disruption.
* Some Copilot clients entered a **retry loop**, generating additional traffic and slowing recovery.
* Neither the August 6 nor August 17 incidents resulted from code or configuration changes; GitHub identifies both primarily as **capacity failures**.
* Monthly commits increased from **1.4 billion in April to 2.9 billion**, substantially increasing infrastructure pressure.
* GitHub added more than **3 million CPU cores, 120 PB of high-speed storage**, and additional network capacity.
* Azure now handles roughly **58% of GitHub platform load and 50% of Git operations**, compared with 12% of platform load in May.
* GitHub is developing an architecture where repository read capacity scales linearly with readers, initially targeting large monorepos.
* Reliability work now includes retry budgets, retry limits, variable timeouts, stronger observability, improved alerts, safer rollouts, and isolation of critical systems.

## 5. Deep Dive (Structured Understanding)

### Problem

GitHub's workload grew faster than some critical infrastructure components could scale.

The immediate trigger was record traffic reaching a component in the Central US data center that lacked sufficient capacity. Because multiple services depended directly or indirectly on affected infrastructure, the local capacity shortage propagated into authentication and other GitHub services.

Recovery introduced another problem: failed Copilot requests caused client-side retries, adding traffic precisely when the system had reduced capacity.

The failure pattern can be simplified as:

`Traffic spike → Capacity exhaustion → Dependency failures → Retries → Additional load → Slower recovery`

### Approach

GitHub is responding at multiple layers rather than treating the incident as an isolated hardware shortage.

**Capacity expansion**

* 3M+ additional CPU cores
* 120 PB additional high-speed storage
* Increased network capacity

**Cloud migration**

* Accelerated migration toward Azure
* Azure platform load increased from 12% in May to approximately 58%
* Around half of Git operations now run on Azure infrastructure

**Architecture**

* Remove shared dependencies between critical systems
* Isolate failure domains
* Develop horizontally scalable read infrastructure for large monorepos

**Resilience**

* Retry limits
* Retry budgets
* Variable timeouts
* Improved testing and safer deployments

**Observability**

* Better alerting
* Review CPU and memory alerts previously considered lower priority
* Detect infrastructure likely to fail under sudden traffic spikes

### Key Insight

**Total infrastructure capacity is different from capacity at a critical bottleneck.**

GitHub could add millions of CPU cores and enormous storage capacity while remaining vulnerable if one component on a critical request path cannot scale at the same rate.

Distributed systems therefore need more than additional hardware:

`Capacity + Horizontal Scaling + Dependency Isolation + Backpressure + Retry Control + Observability`

The incident also demonstrates how retry mechanisms intended to improve reliability can become **load amplifiers** during partial failures.

### Result / Impact

GitHub recovered its core services progressively, while some Copilot services required additional mitigation because retry behavior continued generating load.

Longer term, GitHub is moving toward an infrastructure model with:

* Greater Azure dependence
* Fewer shared critical dependencies
* Horizontally scalable repository reads
* Controlled retry behavior
* Smaller failure blast radius
* Earlier detection of capacity risks

The incident therefore represents both a capacity-planning failure and evidence that GitHub's architecture must evolve for its current growth rate.

## 6. Why It Matters

GitHub is infrastructure for a significant portion of modern software development. Its failure can simultaneously disrupt source control, CI/CD, collaboration, APIs, authentication, and AI-assisted development.

The incident illustrates a broader industry shift from **preventing individual failures** toward designing systems that remain functional when failures inevitably occur.

It also highlights the growing infrastructure consequences of developer-platform expansion and AI workloads. GitHub is no longer scaling only repository hosting; Actions, APIs, Copilot, and increasingly large monorepos create different workload and failure characteristics.

The rapid increase of Azure's share of GitHub workloads also shows GitHub moving from capacity expansion inside existing data centers toward large-scale cloud infrastructure as a central scaling mechanism.

## 7. Critical Analysis

GitHub attributes both August incidents primarily to capacity failures, but the explanation leaves several questions unanswered.

First, rapid workload growth explains increased pressure but does not fully explain why capacity planning failed. Commit volume nearly doubling is significant, yet GitHub does not disclose whether the problematic component had inadequate forecasting, insufficient autoscaling, architectural limits, or delayed capacity provisioning.

Second, aggregate figures such as **3 million additional CPU cores** and **120 PB of storage** sound substantial but reveal little about whether capacity was available where the bottleneck occurred. Distributed-system reliability depends on capacity placement and dependency topology, not only aggregate resources.

Third, Azure migration reduces physical capacity constraints but does not automatically eliminate architectural bottlenecks. A non-scalable critical component can remain a bottleneck regardless of the underlying infrastructure provider.

Fourth, the Copilot retry loop suggests that retry behavior had not been consistently bounded before the incident. Retry budgets and variable timeouts are established distributed-systems techniques, raising the question of why they were not already enforced consistently across critical services.

Finally, GitHub acknowledges that operational practices failed to keep pace with system complexity. This suggests the incident was not purely a capacity problem; **organizational scaling and operational discipline** were also contributing factors.

## 8. Connections

### 1. Retry Storms and Exponential Backoff

Retries improve resilience against transient failures but can amplify overload.

Common protections include:

* Exponential backoff
* Jitter
* Retry limits
* Retry budgets
* Circuit breakers

GitHub's Copilot recovery problem is a practical example of why retry policies must be treated as part of capacity engineering.

### 2. Cascading Failure and Bulkhead Architecture

Shared dependencies can transform a localized failure into a platform-wide outage.

GitHub's plan to isolate critical systems resembles the **bulkhead pattern**, where systems are partitioned so failure in one domain does not consume resources required by others.

This directly reduces **blast radius**.

### 3. Backpressure and Load Shedding

When demand exceeds capacity, accepting every request can make recovery impossible.

Large distributed systems commonly use:

* Backpressure
* Rate limiting
* Load shedding
* Admission control
* Graceful degradation

These mechanisms intentionally reject or delay some work to preserve overall system health.

### 4. Horizontal Scaling and Monorepos

GitHub's planned architecture aims for read capacity that increases approximately linearly with the number of readers.

This reflects the general distributed-systems principle of replacing vertically constrained components with horizontally scalable replicas or partitioned services.

Large monorepos make this particularly important because enormous numbers of developers and automated systems may simultaneously fetch from the same repository.

### 5. Cloud Migration as Capacity Strategy

GitHub's Azure migration demonstrates how hyperscale cloud infrastructure can become a capacity-management strategy when private data centers encounter physical constraints such as available power.

However:

`Cloud capacity ≠ automatically scalable architecture`

Applications must still eliminate bottlenecks and design for distributed failure.

## 9. Keywords

* Capacity Planning
* Cascading Failure
* Retry Storm
* Retry Budget
* Backpressure
* Horizontal Scaling
* Failure Isolation
* Observability
* Azure Migration
* Distributed Systems

## 10. TL;DR

GitHub's August 17 outage occurred when record traffic exceeded the capacity of a critical infrastructure component and failures propagated across dependent services.

Copilot retry loops amplified load during recovery, demonstrating how retries can worsen overloaded distributed systems.

GitHub is responding with Azure expansion, horizontal scaling, dependency isolation, retry controls, and stronger capacity monitoring rather than relying on hardware expansion alone.
