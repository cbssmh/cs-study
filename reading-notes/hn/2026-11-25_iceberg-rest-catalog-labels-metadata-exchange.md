1. Title
Sharing catalog metadata across data platforms, engines, and agents

2. Source
Author / Organization: Daniel Weeks, Andrei Tserakhau, Jason Reid, Michelle Leon, and Alex Jiang / Databricks Blog
Link: https://www.databricks.com/blog/sharing-catalog-metadata-across-data-platforms-engines-and-agents
Date: November 24, 2026

3. One-line Summary
An optional `labels` field added to the Apache Iceberg REST Catalog `LoadTable` API standardizes the exchange of table- and column-level governance tags as portable key-value pairs across engines, catalogs, and AI agents.

4. Key Points
- **Closing the metadata portability gap**: While open table formats like Apache Iceberg made table *data* portable across engines, governance tags, ownership, and business context remained trapped in source catalogs, forcing teams to build custom metadata sync pipelines or recreate classifications per tool.
- **Protocol-level extension in Iceberg REST**: Databricks worked with the Apache Iceberg community to add an optional `labels` field to the Iceberg REST Catalog `LoadTable` response, exposing table- and column-level context as string key-value pairs.
- **Separation of exchange format from enforcement**: The specification intentionally standardizes only the transport format without prescribing a fixed tag vocabulary or enforcement semantics—letting downstream platforms map labels (such as sensitivity or certification tags) to their own local masking and access policies.
- **Concrete table and column tagging model**: In the article's `support_cases` example, stewards attach table-level tags (`domain=customer_support`, `system.certification_status=certified`) and column-level tags (`content_type=support_case_summary` on `issue_summary`), which travel directly inside the `LoadTable` payload.
- **Three primary consumption patterns**: External catalogs can surface domain/purpose tags in search results, AI agents can use certification and column labels to select trustworthy datasets and fields, and external governance tools can bind sensitivity labels to local masking rules.
- **Python client and Databricks implementation**: PyIceberg added support for reading catalog labels (slated for its `0.13` release), and Databricks launched `Tag Exchange` in Beta (via the `"Tag Exchange via Iceberg REST Catalog"` preview flag) to expose Unity Catalog tags through the Iceberg REST API.
- **Broad ecosystem alignment**: Google Cloud, Salesforce, ClickHouse, Atlan, Collibra, DataHub, Immuta, Trust3 AI, PhoenixAI, and Datastrato aligned on the approach, with Google Cloud framing catalog labels as a lower layer complementing Open Knowledge Format (OKF) for higher-level business semantics.

5. Deep Dive (Structured Understanding)
**Problem**
Multi-engine lakehouse architectures allow organizations to query the same underlying Parquet/Iceberg files from Spark, ClickHouse, BigQuery, Python, and AI agents. However, semantic and governance context—which tables are certified, who owns a dataset, or which columns contain sensitive fields—remained siloed inside individual catalogs, causing duplicated stewardship effort, brittle custom sync jobs, and context-blind AI agents.

**Approach**
Rather than inventing a standalone metadata synchronization protocol or forcing every vendor onto a single taxonomy, the authors embedded metadata exchange directly into the existing table-loading handshake:
1. Extended the Apache Iceberg REST Catalog `LoadTable` response with an optional `labels` field carrying string key-value pairs at both table and column granularity.
2. Added client-side parsing in PyIceberg (`0.13`) so Python applications and agent workflows can inspect tags alongside schemas upon loading a table.
3. Implemented `Tag Exchange` in Databricks Unity Catalog to project existing Unity Catalog tags into Iceberg REST `labels` without requiring separate metadata export pipelines.

**Key Insight**
Piggybacking live governance and semantic tags onto the `LoadTable` call—while decoupling *schema-neutral key-value transport* from *local vocabulary and policy enforcement*—avoids vendor lock-in and consensus gridlock over universal taxonomies while ensuring context arrives at the exact moment an engine, catalog, or agent touches a table.

**Result / Impact**
Enterprises using Iceberg REST-compatible catalogs can propagate table- and column-level classifications once in a source catalog (such as Unity Catalog) and consume them across search tools, third-party policy engines, and Python/agentic workflows with no custom ETL sync code.

6. Why It Matters
- **Databases & Platform Engineering**: Marks a maturation step for the open lakehouse stack—moving interoperability beyond storage and table formats (Iceberg/Delta) into live catalog metadata and governance context.
- **IT Risk / Governance**: Enables centralized data stewards to define sensitivity and certification tags once while allowing heterogeneous enforcement engines (such as Immuta or Trust3 AI) to enforce local masking and access controls consistently.
- **AI Engineering / Secure AI**: Addresses a practical bottleneck in enterprise data agents—giving agents programmatic access to table- and column-level certification and content semantics directly through standard Iceberg APIs rather than ad-hoc prompt injection or stale documentation.

7. Critical Analysis
- **Unstandardized vocabularies shift complexity downstream**: Because the standard deliberately avoids prescribing key names or value taxonomies, two catalogs can still emit incompatible conventions (for example, `pii=true` vs. `sensitivity=high` vs. `system.certification_status=certified`), requiring organizations to maintain mapping conventions across consumers.
- **Read-only propagation and enforcement gaps**: Passing labels in `LoadTable` gives downstream tools *visibility* into tags, but does not enforce that an arbitrary Iceberg client actually honors a sensitivity label—enforcement still depends on each engine or governance proxy implementing and respecting local policies.
- **Limited technical detail on edge cases**: The announcement omits wire-format JSON examples (presumably shown in stripped graphics), conflict/precedence rules when multiple catalogs federate, caching/staleness behavior between tag updates and `LoadTable` calls, and whether write/update APIs for labels are part of the Iceberg REST spec.

8. Connections
- **Apache Iceberg REST Catalog as the Universal Control Plane**: Follows the broader lakehouse convergence around the Iceberg REST Catalog API as the standard interoperability boundary between storage catalogs (Unity Catalog, Polaris, BigLake) and compute engines.
- **Complementary Semantic Layers (Open Knowledge Format / dbt Metrics)**: As Google Cloud's quote highlights, key-value catalog labels provide the low-level metadata transport layer beneath higher-level semantic standards like Open Knowledge Format (OKF), dbt semantic layers, or MCP-exposed enterprise data dictionaries.
- **Active Metadata & Policy-as-Code**: Aligns with modern governance platforms (DataHub, Atlan, Collibra, Immuta) shifting from periodic batch metadata scraping to event-driven or request-time tag consumption (`LoadTable`), reducing drift between data classification and access enforcement.

9. Keywords
Apache Iceberg, Iceberg REST Catalog, Catalog Labels, Unity Catalog, Tag Exchange, Metadata Interoperability, Data Governance, PyIceberg, Open Lakehouse, Agentic Workflows

10. TL;DR
Apache Iceberg's REST Catalog `LoadTable` API now supports an optional `labels` field to pass table- and column-level string key-value tags alongside table metadata.
Databricks implemented this in Beta as `Tag Exchange` for Unity Catalog (with PyIceberg `0.13` support and backing from Google Cloud, ClickHouse, Salesforce, and major governance vendors), letting external engines, catalogs, and AI agents read classifications without custom sync pipelines.
Because the spec standardizes transport rather than tag vocabularies or enforcement, teams still need consistent naming conventions and local policy engines to turn shared labels into enforceable controls.
