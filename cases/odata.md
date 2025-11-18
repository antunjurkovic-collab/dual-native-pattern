# OData Services (Web Protocol): Dual-Native Case

Domain: Data APIs / BI Integration

- Human Representation (HR): Dashboards or apps that visualize/consume OData (e.g., custom UIs, BI tools).
- Machine Representation (MR): OData feeds (service document, $metadata, entity sets) in JSON or XML.
- Shared Data: Domain-specific entities exposed via the OData model.

## Current State
- Discovery: Service root and $metadata describe entities and relationships; clients enumerate sets via MR. HR pairing is app-specific and often absent.
- Synchronization/Validators: OData supports ETags for concurrency and caching; UIs may not surface or align with these validators.
- Linking: No standard HR <-> MR pairing; apps rarely expose the raw feed link, and services do not point to a recommended viewer.

## Equivalence Scope

**Scope**: Entity properties, navigation properties, timestamps must match between dashboard and OData endpoint

### Entity Records
**Scope**: All entity properties defined in `$metadata` for the entity type (primitives, complex types, collections)
- **Source**: OData service backend database
- **CID Strategy**: OData `ETag` (e.g., `W/"datetime'2025-01-15T10:00:00'"`) as version validator
- **Validation**: Compare entity fields displayed in dashboard with OData entity response (`/EntitySet(key)`)

### Navigation Properties
**Scope**: Referenced entities via navigation properties must resolve to same records
- **Source**: Relationship traversal from entity set
- **CID Strategy**: Parent entity ETag + navigation path signature
- **Validation**: Compare related entities shown in dashboard with OData `$expand` query results

**Tolerance Rules**:
- Date/time: Dashboard may show localized format; OData returns ISO-8601 or Edm.DateTimeOffset (semantically equivalent)
- Null vs. absent: Missing properties in JSON response equivalent to null display in dashboard
- Number precision: Dashboard may round decimals for display; full precision available in OData feed

## Pattern Gaps
- Missing bidirectional links between HR (viewer/app) and MR (OData endpoints).
- ETags exist but are not formalized as shared CIDs across HR and MR.
- No domain-level Resource Index that aggregates key entity sets for discovery beyond $metadata.

## Recommended Upgrades
1) Bidirectional linking
   - HR: Provide a visible “Data source” link to the exact OData endpoint powering the view.
   - MR: Include a `Link` to a recommended client/viewer or documentation for human exploration.
2) Shared CID (content identity)
   - Treat OData ETags as CIDs and surface them in UI contexts to confirm equivalence.
3) Standardized discovery (Resource Index)
   - Complement $metadata with a curated index listing business-relevant entity sets with (rid, hr, mr, cid, updatedAt) for faster onboarding and AI ingestion.

## Expected Benefits
- Transparent provenance from dashboard to data source; easier troubleshooting and trust.
- Efficient caching and zero-fetch logic for clients using CIDs.
- Better interop for AI tools that can enumerate data without manual configuration.

## Risks/Considerations
- Align multi-tenant permissions and masking rules between HR and MR.
- Ensure index does not expose sensitive feeds; provide permission-aware views.

