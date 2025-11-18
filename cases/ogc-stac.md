# OGC & STAC (Geospatial Data): Dual-Native Case

Domain: Geospatial / Earth Observation

- Human Representation (HR): Map viewers and HTML catalog pages for collections/items (layers, scenes, datasets).
- Machine Representation (MR): OGC services (WMS/WFS/WCS/WPS) and STAC catalogs/items/assets (JSON).
- Shared Data: Collections, features, scenes, rasters, metadata, checksums.

## Current State
- Discovery: STAC catalogs are inherently linked (root/collection/item); OGC APIs provide landing pages and conformance docs. Entry points vary across providers.
- Synchronization/Validators: HTTP validators (ETag/Last-Modified) and STAC extensions (e.g., checksum:multihash, updated fields). Usage is inconsistent across hosts.
- Linking: Many catalogs link MR resources well; HR pages don't always link back to MR assets, and not all MR assets point to a human viewer for the same item.

## Equivalence Scope

**Scope**: Asset metadata, checksums, spatial/temporal extents must match between map viewer and API

### STAC Items/Collections
**Scope**: `{id, bbox, geometry, datetime, properties, assets[].href, assets[].checksum}`
- **Source**: STAC catalog backend (often backed by object storage + metadata DB)
- **CID Strategy**: Asset checksum (`checksum:multihash`, `checksum:sha256`) or `ETag`/`Last-Modified` from HTTP
- **Validation**: Compare metadata displayed in map viewer with STAC JSON item response

### OGC Feature Collections
**Scope**: Feature properties, geometry, temporal validity must align between WFS response and map display
- **Source**: Geospatial database or feature service
- **CID Strategy**: HTTP `ETag` or `Last-Modified` for feature collection; individual feature version if available
- **Validation**: Compare features shown on map with WFS GetFeature response (GeoJSON/GML)

**Tolerance Rules**:
- Coordinate precision: Map may simplify geometry for performance; full precision in GeoJSON (semantically equivalent for display)
- Date/time: Map may show local or abbreviated dates; API returns ISO-8601 with timezone
- Units: Map may show "100 km²"; API returns square meters (`100000000`)
- CRS: Map projection vs. API native CRS (WGS84 / EPSG:4326) - transformation is deterministic

## Pattern Gaps
- Not all providers expose explicit HR <-> MR links for the same resource.
- No single well-known entry point across providers; discovery conventions differ.
- Inconsistent exposure of change tokens (CIDs) for large assets and metadata.

## Recommended Upgrades
1) Bidirectional linking
   - HR: From map/layer pages, expose links to the exact WFS/asset/STAC item for the view.
   - MR: Include `hr_url`/viewer links in STAC item links and OGC landing pages.
2) Shared CID (content identity)
   - Use provider checksums (checksum:multihash) or Last-Modified/ETag for assets and metadata; surface the same in HR contexts.
3) Standardized discovery (Resource Index)
   - Provide a well-known entry (e.g., /.well-known/geodata.json) summarizing collections/items with (rid, hr, mr, cid, updatedAt), or leverage DCAT mapping.

## Expected Benefits
- Easier provenance from map views to raw data; accelerates analysis workflows.
- Efficient update checks for large assets (no full re-downloads when unchanged).
- Clearer interop for AI agents and data brokers crawling multiple providers.

## Risks/Considerations
- Respect licensing/usage constraints; avoid exposing restricted assets in indexes.
- Rate-limit and paginate discovery to protect backends.

