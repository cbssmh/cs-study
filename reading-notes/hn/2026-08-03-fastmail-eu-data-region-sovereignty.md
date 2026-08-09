## 1. Title

Fastmail Offers EU Data Region

## 2. Source

* Author / Organization: Bron Gondwana / Fastmail
* Link: https://www.fastmail.com/blog/fastmail-offers-eu-data-region/
* Date: 2026-08-03

## 3. One-line Summary

Fastmail now lets users place their primary email data on company-owned infrastructure in Amsterdam, but US-based replicas, backups, logs, and cross-region metadata mean the service does not yet provide full EU-only data residency or sovereignty.

## 4. Key Points

* Fastmail introduced an EU data region, allowing users to choose Amsterdam as the primary location for mail and file data.
* The Amsterdam deployment uses Fastmail-owned hardware in a colocation facility rather than AWS, Azure, GCP, or another hyperscale cloud provider.
* EU-region clients normally connect directly to Amsterdam infrastructure, with US infrastructure available as a fallback during outages.
* Fastmail currently maintains geographically separate replicas of EU-region data in the US because it has only one European location.
* Emergency backups for all users are currently stored in Philadelphia.
* System logs are centralized in the US, while some customer metadata and service-related data are replicated across both European and US sites.
* Selecting the EU region changes the location of the primary data copy, not the overall encryption model or legal obligations of the company.
* Fastmail explicitly states that it cannot guarantee that an EU-region customer's data remains exclusively inside the EU.
* Existing customers can migrate between US and EU regions without interrupting normal account operation.
* The announcement highlights data residency as a product capability distinct from privacy, encryption, and legal sovereignty.

## 5. Deep Dive

### Problem

Customers increasingly care about where their data is physically stored because of privacy expectations, regulatory requirements, latency, compliance, and concerns about foreign legal jurisdiction.

Previously, Fastmail stored all accounts in the US, limiting its ability to satisfy customers who preferred or required European data residency.

The challenge is that simply placing one server in Europe is insufficient for a reliable email service. Production systems require replicas, backups, monitoring, disaster recovery, and operational metadata, all of which can create additional cross-border data flows.

### Approach

Fastmail deployed its own hardware in a secure colocation facility in Amsterdam.

For EU-region accounts:

* Amsterdam becomes the primary location for mail and files.
* Applications normally connect to the Amsterdam infrastructure.
* Incoming mail is preferentially routed to EU infrastructure under supported domain configurations.
* A geographically separate live replica remains in the US.
* US infrastructure provides failover if the EU location becomes unavailable.

Fastmail retains its existing operational model of controlling its own hardware and software rather than outsourcing core infrastructure to a hyperscale cloud provider.

### Key Insight

Data residency is not a binary property determined by the location of the primary database.

A production service consists of multiple data paths:

`Primary Data → Replicas → Backups → Logs → Metadata → Third-party Services`

Moving only the primary copy changes the system's residency characteristics, but it does not automatically provide complete regional isolation or legal sovereignty.

Fastmail's architecture therefore demonstrates the distinction between:

* **Data Residency:** where data is physically stored.
* **Data Sovereignty:** which legal jurisdictions and entities can exercise control over the data.
* **Data Security:** how access to the data is technically restricted.

### Result / Impact

Fastmail can now serve customers who prefer their active mailbox data to reside in Europe while retaining its existing reliability model.

However, the architecture does not yet satisfy strict EU-only requirements because replicas, backups, logs, and some metadata remain in or flow through the US.

The announcement therefore represents an intermediate step toward stronger regional infrastructure rather than a complete sovereign-email architecture.

## 6. Why It Matters

This announcement reflects a broader shift in infrastructure purchasing criteria.

Cloud and SaaS platforms have traditionally competed primarily on:

`Cost + Performance + Reliability + Developer Experience`

Data location and legal jurisdiction are increasingly becoming additional requirements:

`Cost + Performance + Reliability + Data Residency + Jurisdiction + Control`

This is especially relevant in Europe, where data protection, strategic autonomy, and dependence on foreign technology providers are increasingly discussed alongside conventional cloud architecture.

The Fastmail case also illustrates why sovereign infrastructure cannot be achieved merely by selecting an EU region. Organizations must examine the entire data lifecycle, including replication, backups, observability systems, encryption keys, support systems, and third-party services.

For infrastructure and security engineering, this expands architecture review from technical availability and security toward questions of geographic and legal control.

## 7. Critical Analysis

Fastmail's phrase "EU data region" could easily be interpreted more strongly than the implementation warrants.

The primary copy resides in Amsterdam, but several important data categories still exist in the US. Organizations with strict requirements that prohibit data from leaving the EU would therefore need to examine the architecture carefully rather than relying on the region label.

The company deserves neither special credit nor criticism for this limitation without considering availability requirements. A resilient service normally requires geographically separated replicas and backups. With only one European location, keeping disaster-recovery copies elsewhere is technically understandable.

However, this reveals an architectural constraint: meaningful EU-only operation would likely require at least a second independent European site for replication and disaster recovery.

Another limitation is that physical data location does not resolve jurisdictional questions. Fastmail is an Australian company and operates internationally. Legal access therefore depends on more than the location of a disk.

Hacker News discussion heavily focused on the CLOUD Act, Five Eyes, and Australian surveillance legislation. These comments expose an important concern but should not be treated as authoritative legal analysis; several participants made conflicting claims about jurisdiction and international data-access mechanisms.

The debate also risks overstating the security benefit of geographic residency. Email itself involves communicating with external mail servers, and recipients may store copies under completely different jurisdictions. Data residency can therefore improve compliance and control without making email inherently private.

The strongest aspect of Fastmail's announcement is not absolute sovereignty but architectural transparency: it explicitly identifies where primary data, replicas, backups, logs, and metadata reside instead of treating "EU region" as a complete description of the system.

## 8. Connections

### 1. Sovereign Cloud

Fastmail's architecture connects directly to the growing sovereign-cloud market.

A sovereign architecture must consider more than datacenter location. Relevant questions include:

* Who owns the infrastructure?
* Who operates it?
* Where are encryption keys stored?
* Where are replicas and backups located?
* Which legal entity controls the service?
* Can foreign authorities compel the operator?

This distinction is increasingly important when evaluating AWS, Azure, GCP, European cloud providers, and private-cloud platforms.

### 2. Disaster Recovery and Data Residency

Data sovereignty can conflict with conventional disaster-recovery design.

A robust system commonly uses geographically separated replicas:

`Region A → Region B`

If Region B is outside the permitted jurisdiction, resilience creates a compliance problem.

Strict regional architectures therefore often require multiple independent facilities inside the same legal boundary:

`EU Region A ↔ EU Region B`

Fastmail currently lacks this second European location.

### 3. Observability as a Data Boundary

Logs and monitoring systems are frequently overlooked when evaluating data residency.

Even when application databases remain inside a required region, observability pipelines may export:

* IP addresses
* identifiers
* request metadata
* authentication events
* error payloads

to another jurisdiction.

Fastmail's US-centralized logging demonstrates why infrastructure reviews must include telemetry rather than examining only application storage.

### 4. Colocation vs Public Cloud

Fastmail operates its own hardware in colocated datacenters rather than building its core service on hyperscale public clouds.

This model trades some cloud abstraction and elasticity for greater control over:

* hardware
* storage
* network architecture
* operational procedures
* infrastructure ownership

The case shows that colocation and private infrastructure remain strategically relevant for services where control and jurisdiction are important requirements.

### 5. Infrastructure Security and Compliance Convergence

Modern infrastructure engineers increasingly need to understand how security, availability, and compliance interact.

A design decision such as cross-region replication can simultaneously affect:

* disaster recovery
* latency
* privacy
* regulatory compliance
* legal exposure

Infrastructure architecture is therefore becoming less separable from security and governance.

## 9. Keywords

* Data Residency
* Data Sovereignty
* Sovereign Cloud
* Data Localization
* Disaster Recovery
* Cross-region Replication
* Colocation
* Data Governance
* Cloud Security
* Jurisdiction

## 10. TL;DR

Fastmail now allows primary mailbox data to reside on its own servers in Amsterdam.

US replicas, backups, logs, and shared metadata mean this is not yet an EU-only or fully sovereign architecture.

The larger lesson is that data sovereignty depends on the entire data flow and control structure, not merely the region containing the primary server.
