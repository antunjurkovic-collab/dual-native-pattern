# Salesforce (CRM Platform): Dual-Native Case

Domain: CRM / Enterprise SaaS

- Human Representation (HR): Salesforce Lightning Experience (record pages, list views, dashboards).
- Machine Representation (MR): REST, SOAP, Bulk, and Streaming APIs (Change Data Capture, Platform Events).
- Shared Data: Standard and custom objects (Accounts, Contacts, Opportunities), records, metadata, dashboards.

## Current State
- Discovery: Service describe/metadata APIs list objects and fields; record enumeration via query endpoints. No org-level index that pairs HR and MR for specific records.
- Synchronization/Validators: Strong record identifiers and last-modified timestamps; streaming events for changes. Formal shared CIDs across HR and MR are not consistently surfaced.
- Linking: Lightning UI does not consistently expose the precise MR link for the record in view; MR responses seldom include `hr_url` back to the exact Lightning page.

## Equivalence Scope

**Scope**: Record fields, related objects, last modified date must match between Lightning UI and REST API

### Standard and Custom Object Records
**Scope**: `{Id, Name, CreatedDate, LastModifiedDate, OwnerId, SystemModstamp, [custom fields]}`
- **Source**: Salesforce org database (multi-tenant)
- **CID Strategy**: `LastModifiedDate` or `SystemModstamp` as version indicator
- **Validation**: Query record via REST API and compare with Lightning UI display fields

### Related Records and Lookups
**Scope**: Relationship fields (`Account.Contacts`, `Opportunity.Account`) must resolve to same records
- **Source**: Relationship traversal from master record
- **CID Strategy**: Parent record's `LastModifiedDate` + relationship signature
- **Validation**: Compare related records shown in Lightning Related Lists with API query results

**Tolerance Rules**:
- Dates/times: Lightning shows user timezone; API returns UTC (semantically equivalent when timezone-adjusted)
- Picklist labels: API may return value codes; UI shows translated labels (mapping is 1:1)
- Currency formatting: `$1,234.56` (UI) vs. `1234.56` (API) are equivalent

## Pattern Gaps
- Missing HR -> MR deep links for individual records and dashboards.
- No standardized, shared CID ensuring UI and API reflect the same version simultaneously.
- No org-scoped Resource Index enumerating custom objects/records with (rid, hr, mr, cid, updatedAt) for efficient discovery.

## Recommended Upgrades
1) Bidirectional linking
   - HR: Show the exact REST endpoint (and sample request) for the record being viewed.
   - MR: Include `hr_url` linking directly to the record’s Lightning page.
2) Shared CID (content identity)
   - Use record version markers (e.g., last-modified or explicit version IDs) as CIDs and expose them consistently across HR and MR.
   - Enforce stale-write protection by comparing client-provided CID.
3) Standardized discovery (Resource Index)
   - Publish an org-level catalog enumerating core objects and key records with (rid, hr, mr, cid, updatedAt), with incremental updates and filtering.

## Expected Benefits
- Faster integration troubleshooting and safer automations through verifiable equivalence and deep links.
- Efficient sync for analytics and AI agents (skip unchanged records via CID checks).
- Clearer governance and observability across custom objects and apps.

## Risks/Considerations
- Respect object/field-level security; ensure index output is permission-scoped.
- Control catalog enumeration to prevent data leakage; paginate and throttle.

