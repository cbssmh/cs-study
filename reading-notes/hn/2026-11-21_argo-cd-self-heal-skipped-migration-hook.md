1. Title
When a sync window closes: How Argo CD self-heal skipped the migration hook

2. Source
Author / Organization: Afzal Ansari (CNCF Ambassador) / Cloud Native Computing Foundation (CNCF) Blog
Link: https://www.cncf.io/blog/2026/11/20/when-a-sync-window-closes-how-argo-cd-self-heal-skipped-the-migration-hook/
Date: November 20, 2026

3. One-line Summary
A race condition between Argo CD's level-1 object-change refresh and level-2 periodic refresh when a deny sync window closes causes self-heal to treat a new release as drift repair, deploying application code while silently skipping database migration hooks.

4. Key Points
- **Silent partial rollout in production**: Across five regional Argo CD Applications behind an off-hours deny sync window with automated sync, prune, and self-heal enabled, three regions ran the `Sync` hook database migration Job while two deployed new app code without migrating—causing 36–44 minutes of schema mismatch while the UI reported `Synced` and `Healthy`.
- **Reproducible race condition**: In a minimal lab (one `ConfigMap`, one `Deployment`, one `Sync` hook `Job`, and a 4-minute deny window), pushing release `v11` during the deny window and triggering a level-1 refresh just after the window closed reproduced the skipped migration 5 out of 6 times.
- **Root cause in Argo CD refresh paths**: Level-2 timer refreshes (every ~2–3 minutes) compare against Git and write the new commit to `status.sync.revision` even while a deny window blocks syncing, whereas level-1 refreshes (triggered by managed-object changes) read `status.sync.revision` as if it were the last *synced* revision—so `evaluateRevisionChanges` sees no change and `alreadyAttemptedSync` routes to self-heal drift repair.
- **Why self-heal skips hooks**: Self-heal performs a partial sync of only out-of-sync resources, which skips all resource hooks (`Sync`, `PreSync`) and writes no sync history; `RunHooksOnPartialSync` is also ignored for automated syncs, and shortening the sync window does not eliminate the race at window close.
- **Desired-state fix via sync waves**: Removing hook annotations and declaring the migration `Job` as an ordinary managed resource with a per-release unique name (e.g., `db-migration-v11` via image tag in Helm) and `argocd.argoproj.io/sync-wave: "-1"` ensures both full syncs and partial self-heal syncs create the missing Job and wait for health before applying app manifests.
- **Operational caveats for Job-based waves**: Keep `prune: true` so prior release Jobs are cleaned up, avoid `ttlSecondsAfterFinished` (which causes Kubernetes deletion to trigger self-heal re-runs), keep migrations idempotent, and test failing migrations in staging.
- **Blunter fallback**: Disabling `selfHeal: false` on apps with migrations avoids the race—the app stays on the old version until the next level-2 timer refresh (~2 minutes later) triggers a full sync with hooks—at the cost of losing automatic drift correction for manual cluster edits.

5. Deep Dive (Structured Understanding)
**Problem**
Teams using Argo CD sync windows alongside automated sync and `selfHeal: true` rely on `Sync` or `PreSync` hooks for pre-rollout database migrations. When a commit lands during a deny window and the window later closes, Applications can roll out new application manifests while leaving the database schema on the previous version, with no error surfaced in the Argo CD UI.

**Approach**
The author isolated the incident in a minimal reproduction cluster using a timed 4-minute deny window and debug controller logs, traced the state machine in Argo CD's controller (`evaluateRevisionChanges` and `alreadyAttemptedSync`), and evaluated two architectural mitigations in the lab: converting the hook into a wave-ordered, release-named managed `Job`, and disabling `selfHeal` so level-2 refreshes govern rollouts.

**Key Insight**
As the author summarizes, when multiple controller paths can apply manifests, "the hook is a hope, not a guarantee." On Argo CD's level-1 path, the last *compared* revision is mistaken for the last *synced* revision—so encoding migrations as side-effect hooks makes release correctness depend on event ordering, whereas encoding them as versioned desired-state resources guarantees execution regardless of which sync path wins.

**Result / Impact**
In lab verification, the `sync-wave: "-1"` managed-Job pattern went through the exact broken self-heal path and still completed the migration one second before applying the new `ConfigMap` and `Deployment`, while pruning the previous release's Job. Disabling `selfHeal` also prevented the mismatch by deferring rollout ~2 minutes to the next level-2 refresh.

6. Why It Matters
- **Cloud / DevOps & Platform Engineering**: Exposes a subtle failure mode in a widely adopted GitOps pattern (Argo CD + sync windows + `selfHeal` + DB migration hooks) where UI status (`Synced` / `Healthy`) actively misleads operators during schema-skew incidents.
- **Reliability / Observability**: Illustrates why controller logs (checking operation resource counts and self-heal attempt counters) matter more than high-level dashboard states when diagnosing GitOps anomalies, and why idempotent, state-driven migrations outperform imperative lifecycle hooks.
- **Distributed Systems**: Demonstrates a classic state-aliasing bug in reconciliation loops—confusing *last observed/compared target state* with *last converged/applied state* across asynchronous event triggers.

7. Critical Analysis
- **Untested failure and hook-flag cases**: The author transparently notes two gaps: they did not test how a deliberately failing `sync-wave: "-1"` migration Job behaves in staging (e.g., whether wave blocking cleanly halts subsequent waves or interacts badly with retries), and they inferred from source inspection—without lab execution—that `PreSync` hooks and `RunHooksOnPartialSync` fail the same way.
- **Chart-management and Helm coupling**: Requiring unique per-release `Job` names (such as embedding an image tag) forces changes across Helm charts or Kustomize overlays and creates friction when multiple commits share an image tag or when non-idempotent migrations exist.
- **Scope**: The post documents a specific Argo CD controller behavior without specifying the exact Argo CD version tested or whether an upstream issue/PR was filed to fix the level-1 `status.sync.revision` comparison bug directly.

8. Connections
- **Kubernetes Controller Reconciliation Patterns**: Echoes classic Kubernetes operator pitfalls where level-triggered (periodic resync) and edge-triggered (watch/event) paths diverge in their assumptions about cached `.status` fields.
- **Expand-and-Contract Database Migrations**: Reinforces why backward-compatible schema migrations (expand-and-contract) remain essential even in GitOps pipelines—because timing skew or partial syncs between app pods and schema jobs can still leave code and schema temporarily out of sync.
- **Declarative Desired State vs. Imperative Hooks**: Mirrors broader infrastructure-as-code lessons (such as Terraform `provisioner` blocks vs. first-class resources): imperative hooks outside the resource graph are fragile whenever partial-apply or drift-remediation paths exist.

9. Keywords
Argo CD, GitOps, Sync Windows, Self-Heal, Database Migrations, Kubernetes Jobs, Sync Waves, Resource Hooks, Reconciliation Race Condition, Drift Detection

10. TL;DR
When an Argo CD deny sync window closes, a race between level-1 object-change refreshes and level-2 timer refreshes can mistake the last compared commit for the last synced commit, triggering a partial self-heal sync that deploys new app code while skipping migration hooks.
Switching hook types (`PreSync`), enabling `RunHooksOnPartialSync`, or shortening windows does not prevent the race because partial self-heal syncs skip all hooks.
Make migrations part of the desired state—using a uniquely named `Job` per release on `argocd.argoproj.io/sync-wave: "-1"` without `ttlSecondsAfterFinished`—or disable `selfHeal` on apps with migrations.
