# Data.gov (Open Data / CKAN): Dual-Native Case

Domain: Open Data Portals

- Human Representation (HR): Dataset landing pages and previews on data.gov.
- Machine Representation (MR): CKAN API (package_show, resource_show) and DCAT/data.json feeds.
- Shared Data: Datasets, resources, distributions, metadata, organizations.

## Current State
- Discovery: DCAT/data.json and CKAN APIs provide catalog discovery. Pairing of specific HR pages with exact MR endpoints is not uniform.
- Synchronization/Validators: Timestamps (metadata_modified) and provider resource checksums vary by dataset.
- Linking: HR pages may link to downloads; MR payloads include metadata but not always the exact `landingPage` or HR deep link.

## Equivalence Scope

**Scope**: Dataset metadata, distribution URLs, modified timestamps must match between portal and CKAN API

### Dataset Metadata Records
**Scope**: `{id, title, description, tags, organization, license, metadata_created, metadata_modified}`
- **Source**: CKAN database and DCAT catalog
- **CID Strategy**: `metadata_modified` timestamp as primary version indicator
- **Validation**: Compare dataset landing page content with CKAN API `package_show` response

### Distribution/Resource Records
**Scope**: `{resource_id, url, format, size, hash/checksum, created, last_modified}`
- **Source**: Dataset resource list
- **CID Strategy**: Resource `last_modified` timestamp or provider-supplied checksum (SHA-256, MD5)
- **Validation**: Compare resource metadata shown on landing page with CKAN `resource_show` response

**Tolerance Rules**:
- Date formatting: Portal may display friendly dates ("January 15, 2025"); API returns ISO-8601 (`2025-01-15T10:00:00Z`)
- File sizes: Portal may show "1.2 MB"; API returns bytes (`1258291`)
- License display: Portal shows full license name; API may use SPDX identifier (semantically equivalent)

## Pattern Gaps
- Missing consistent bidirectional links (HR <-> MR) for each dataset/resource.
- Inconsistent use of change tokens (CID) to enable zero-fetch for metadata and distributions.
- No single, incremental Resource Index emphasizing (rid, hr, mr, cid, updatedAt) and parity checks across the corpus.

## Recommended Upgrades
1) Bidirectional linking
   - HR: Include a machine-readable link to the exact API record (package_show/resource_show) for the page being viewed.
   - MR: Ensure dataset metadata includes the precise `landingPage` (hr_url) and any preview URLs.
2) Shared CID (content identity)
   - Use `metadata_modified` for metadata CIDs; for distributions, prefer provider checksums or last-modified.
3) Standardized discovery (Resource Index)
   - Treat DCAT/data.json as the official index and include explicit CIDs for datasets and distributions.
   - Provide incremental updates (since=timestamp) to support efficient sync by aggregators and AI agents.

## Expected Benefits
- Faster, reliable ingestion by analysis tools with fewer redundant downloads.
- Verifiable metadata/file parity between HR pages and API responses.
- Better AI discoverability and lower operational costs for both providers and consumers.

## Risks/Considerations
- Coordinate across agencies to standardize checksum/last-modified policies.
- Ensure rate limits and pagination protect the catalog service under load.

