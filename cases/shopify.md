# Shopify (E-commerce Platform): Dual-Native Case

Domain: E-commerce / Retail

- Human Representation (HR): Shopify Admin dashboard pages for products, collections, orders, customers, inventory, and analytics.
- Machine Representation (MR): Shopify GraphQL Admin API and REST Admin API.
- Shared Data: Products, variants, inventory levels, orders, customers, fulfillments, discounts, collections.

## Current State
- Discovery: Product listings (e.g., products.json) and Admin APIs allow enumeration; GraphQL global IDs (gid://shopify/...) offer stable identities.
- Synchronization/Validators: Updated timestamps are common; some endpoints support caching validators; formal, shared CIDs between HR and MR are not universally exposed.
- Linking: Admin UI does not consistently expose the exact API reference for the item in view; API responses rarely include direct `human_url` back to the Admin page.

## Pattern Gaps
- No consistent bidirectional links between Admin pages and exact API endpoints for specific resources.
- No formal, shared CID across HR and MR to guarantee point-in-time equivalence (e.g., inventory state, product fields).
- No standardized, shop-scoped Resource Index enumerating (rid, hr, mr, cid, updatedAt) across key object types.

## Recommended Upgrades
1) Bidirectional linking
   - HR: Add a "Show API details" control that reveals the GraphQL GID and a direct API link for the current resource.
   - MR: Include `human_url` fields pointing to the exact Admin page.
2) Shared CID (content identity)
   - Use a version hash or strong validator for mutable entities (products, inventory, orders) and surface it in both HR and MR.
   - Ensure aggregates (e.g., analytics snapshots) expose snapshot/version identifiers as CIDs.
3) Standardized discovery (Resource Index)
   - Publish a shop-scoped JSON index (e.g., /llm-sitemap.json) enumerating products, orders, customers with (rid, hr, mr, cid, updatedAt).
   - Support incremental discovery (since=timestamp) to avoid full scans.

## Expected Benefits
- Faster debugging and app development with one-click navigation between Admin and APIs.
- Reliable synchronization for inventory and catalog management via CID checks.
- Lower costs for AI/automation by enabling zero-fetch when content is unchanged.

## Risks/Considerations
- Protect PII and sensitive Admin URLs; respect scopes and app permissions.
- Document CID semantics for aggregates (inputs and time windows) to avoid misinterpretation.

