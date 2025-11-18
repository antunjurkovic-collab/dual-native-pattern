# Databricks (Data & AI): Dual-Native Case

Domain: Data Engineering / Analytics / ML

- Human Representation (HR): Web-based workspace (notebooks, jobs UI, clusters, model registry, data explorer).
- Machine Representation (MR): Databricks REST APIs (jobs, clusters, repos, MLflow, Unity Catalog), SQL endpoints.
- Shared Data: Notebooks, jobs, runs, logs, tables, schemas, models, permissions.

## Current State
- Discovery: Unity Catalog and APIs enumerate data assets; control-plane objects are discoverable via API listings. No single cross-object index pairing HR and MR uniformly.
- Synchronization/Validators: Delta/UC provide table snapshot/version IDs; control-plane objects expose timestamps/ids but not standardized CIDs.
- Linking: Some MR objects include `run_page_url` (MR -> HR). HR -> MR links vary by surface.

## Equivalence Scope

### Data Plane Resources (Delta Tables)
**Scope**: `{table_name, schema, partition_spec, table_version, row_count}`
- **Source**: Delta Lake transaction log (authoritative, versioned)
- **CID Strategy**: Delta table version number (monotonic integer)
- **Projection CID**: `delta_version:42` (snapshot at version 42)
- **Validation**: Query table metadata via UI and REST API, compare version + schema

### Control Plane Resources (Jobs, Clusters)
**Scope**: `{job_id, name, schedule, cluster_spec, last_run_status, last_run_timestamp}`
- **Source**: Databricks control plane database
- **CID Strategy**: `last_modified_timestamp` or job run ID
- **Validation**: Field parity (UI job details vs. `/api/2.1/jobs/get`)

### Aggregate Resources (Job Run Metrics)
**Scope**: `{run_duration, resource_usage, task_count, success_rate}`
- **Computation**: Aggregates computed from individual task metrics
- **Filter Parameters**: Time window, cluster type, job tags
- **Projection CID**: `hash(job_id, run_id, aggregation_version)`
- **Validation**: Recompute metrics from task logs, compare to UI dashboard

**Tolerance Rules**:
- Duration formatting: Seconds (API) vs. human-readable "2h 15m" (UI)
- Resource units: DBU (Databricks Units) displayed consistently

## Pattern Gaps
- Inconsistent bidirectional links across object types (jobs, clusters, repos, models, tables).
- No uniform CID model for control-plane objects; tables have strong CIDs (snapshotId), others do not.
- No consolidated Resource Index spanning data + control planes with (rid, hr, mr, cid, updatedAt).
- **Equivalence scope not formally documented** (especially for aggregate dashboards and metrics).

## Recommended Upgrades
1) Bidirectional linking
   - HR: Expose exact MR links on object pages (jobs, runs, clusters, repos, models, tables).
   - MR: Include `hr_url` consistently across API responses for deep-linking back to the UI.
2) Shared CID (content identity)
   - Data plane: use table snapshot/version as CID; surface in both HR and MR.
   - Control plane: define object CIDs (e.g., spec hash or version counters) and expose consistently.
3) Standardized discovery (Resource Index)
   - Generate a workspace-level index from Unity Catalog + control-plane APIs listing (rid, hr, mr, cid, updatedAt) for key asset types.
   - Provide incremental updates or event streams for change notices.

## Expected Benefits
- Faster navigation and automation across data and control planes.
- Verified equivalence for datasets and configurations used by pipelines and notebooks.
- Lower egress and compute via skip-download behavior for unchanged assets.

## Risks/Considerations
- Respect workspace permissions in index output; avoid cross-tenant exposure.
- Document CID semantics for control-plane objects to prevent confusion with data-plane versions.

