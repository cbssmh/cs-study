# 1. Title



Microsoft Open Sources pg_durable: Durable Execution Inside PostgreSQL



# 2. Source



* Author / Organization: Microsoft

* Link: https://github.com/microsoft/pg_durable

* Date: 2026-06-05



# 3. One-line Summary



Microsoft open-sourced pg_durable, a PostgreSQL extension that enables durable workflow execution with checkpointing, retries, and recovery directly inside the database.



# 4. Key Points



* Microsoft released pg_durable as an open-source PostgreSQL extension for durable workflow execution.

* Workflow state, checkpointing, retries, and recovery are stored directly in PostgreSQL.

* The system targets AI pipelines, ETL workloads, scheduled maintenance, and long-running data workflows.

* Durable execution allows failed jobs to resume from the last successful checkpoint instead of restarting entirely.

* Workflow definitions are expressed through a SQL-native DSL and executed by a PostgreSQL background worker.

* The architecture removes the need for separate orchestration components such as custom workers, job queues, and status-tracking systems.

* pg_durable is built on top of Microsoft's open-source durable execution framework, duroxide.

* The project is integrated with Azure HorizonDB and positioned as part of Microsoft's AI-oriented PostgreSQL strategy.

* Community reactions highlight operational simplicity but raise concerns around observability, maintainability, and business logic placement.



# 5. Deep Dive (Structured Understanding)



## Problem



Many organizations build custom orchestration systems around PostgreSQL:



* Job tables

* Retry counters

* Polling workers

* Cron schedulers

* State-tracking infrastructure



These solutions create operational complexity and often fail to recover gracefully from crashes or interruptions.



Long-running workflows are especially problematic because partial progress can be lost, requiring expensive reprocessing.



## Approach



pg_durable moves workflow orchestration into PostgreSQL itself.



Core components:



* SQL workflow definitions

* Durable checkpoints

* Workflow state persistence

* Background worker execution

* Built-in retry and recovery mechanisms



Instead of external orchestration engines managing workflow state, PostgreSQL becomes both:



* System of record

* Workflow runtime



## Key Insight



The durable component of a workflow is primarily its state.



If workflow state already lives in PostgreSQL, external orchestration layers may be unnecessary for many database-centric workloads.



The architecture follows a "Bring Compute to Data" philosophy rather than moving data to external compute systems.



## Result / Impact



Potential benefits:



* Reduced infrastructure complexity

* Fewer moving parts

* Simpler backup and recovery

* Strong consistency between data and workflow state

* Easier deployment for database-heavy applications



The approach is particularly attractive for:



* AI enrichment pipelines

* Embedding generation

* ETL systems

* Database maintenance workflows

* Data-centric automation



# 6. Why It Matters



This project reflects a broader shift in modern infrastructure:



* Databases are becoming execution platforms rather than passive storage layers.

* AI pipelines increasingly run close to the data they operate on.

* Organizations are attempting to reduce infrastructure sprawl.

* PostgreSQL continues evolving into a general-purpose application platform through extensions.



The release signals Microsoft's growing investment in PostgreSQL as an AI-native platform.



# 7. Critical Analysis



## Assumption: Workflow Logic Belongs in the Database



This works well for data-centric workflows but may become problematic for:



* Complex business logic

* Large application ecosystems

* Multi-service architectures



## Operational Visibility Remains Unclear



Traditional orchestrators such as Temporal and Airflow provide mature tooling for:



* Monitoring

* Debugging

* Workflow visualization

* Lifecycle management



pg_durable currently offers less operational visibility.



## Database Resource Contention



Running orchestration workloads inside PostgreSQL may increase:



* CPU pressure

* Memory pressure

* Storage pressure



This could make database scaling more difficult in high-throughput environments.



## SQL DSL Adoption Risk



The SQL-based workflow syntax may be less approachable than general-purpose programming languages.



Developer adoption may depend heavily on tooling improvements.



# 8. Connections



## Temporal



Both systems provide durable execution and workflow recovery.



Difference:



* Temporal runs externally.

* pg_durable runs inside PostgreSQL.



## DBOS



DBOS promotes database-centric durable execution and workflow orchestration.



Both projects argue that durable state should live where the data already exists.



## AI Data Infrastructure



Related to:



* pgvector

* DiskANN

* AI enrichment pipelines

* Retrieval systems



pg_durable extends PostgreSQL's role from storage to orchestration.



## Serverless Database Trend



Connected to:



* Neon

* Supabase

* Azure HorizonDB



All move toward databases acting as programmable platforms rather than simple storage engines.



# 9. Keywords



* PostgreSQL

* Durable Execution

* Workflow Orchestration

* pg_durable

* AI Pipelines

* ETL

* Database-centric Architecture

* Temporal

* DBOS

* Azure HorizonDB



# 10. TL;DR



* Microsoft released pg_durable to bring durable workflow execution directly into PostgreSQL.

* The project replaces parts of traditional job queues, schedulers, and orchestration layers with database-native workflows.

* It reflects a broader industry trend toward running compute closer to data, especially for AI and data pipelines.
