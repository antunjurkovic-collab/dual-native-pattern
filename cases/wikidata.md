# Wikidata (Knowledge Graph): Dual-Native Case

Domain: Knowledge Graphs / Linked Data

- Human Representation (HR): Wikidata entity pages (e.g., Q42) rendered for human browsing and editing.
- Machine Representation (MR): SPARQL endpoint, RDF/JSON dumps, entity JSON (Special:EntityData).
- Shared Data: Entities, statements, references, sitelinks; identified by stable IRIs/Q-IDs.

## Current State
- Discovery: Public dumps and SPARQL; recent changes feeds exist but broad discovery can be heavy.
- Synchronization/Validators: Revision IDs (lastrevid) and timestamps per entity; strong foundations for CIDs.
- Linking: HR pages link to machine resources (RDF/JSON); MR provides canonical identifiers; pairing is good but not formalized as a single catalog.

## Pattern Gaps
- No unified, live machine index enumerating (rid, hr, mr, cid) across all entities for incremental sync.
- CID semantics are available (revision ID) but not consistently packaged with an index for zero-fetch behavior.

## Recommended Upgrades
1) Shared CID (content identity)
   - Treat `lastrevid` (and/or last-modified) as the entity CID; expose consistently across HR and MR responses.
2) Standardized discovery (Resource Index)
   - Publish a live JSON index (or DCAT feed) enumerating entities and their CIDs, optimized for incremental consumption (since lastrevid).
   - Provide per-namespace or per-topic slices to reduce client load.
3) Bidirectional linking (clarifications)
   - Keep HR <-> MR links; include explicit machine profile metadata where helpful (RDF formats).

## Expected Benefits
- Efficient incremental syncing for AI/analysis without reprocessing large dumps.
- Verifiable equivalence between HR and MR at a point in time via lastrevid.
- Clearer on-ramps for third-party catalogs and mirrors.

## Risks/Considerations
- Index scale and update cadence must be tuned to change volume.
- Avoid indexing sensitive or embargoed content; respect rate limits.

