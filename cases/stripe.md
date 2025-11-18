# Stripe (Payments): Dual-Native Case

Domain: Payments / Fintech

- Human Representation (HR): Stripe Dashboard pages for customers, subscriptions, invoices, payouts, and reports.
- Machine Representation (MR): Stripe REST API (e.g., /v1/customers, /v1/subscriptions, /v1/invoices).
- Shared Data: Customers, payment methods, charges, subscriptions, invoices, balances, reports.

## Current State
- Discovery: APIs provide listing endpoints; developers traverse object graphs programmatically. No unified account-level machine index that pairs HR/MR.
- Synchronization/Validators: Per-object timestamps; some resources are immutable; aggregated reports computed server-side.
- Linking: API objects include stable IDs but typically do not include a direct Dashboard URL for the exact HR page; Dashboard does not expose the exact MR link.

## Equivalence Scope

### Simple Resources (Customers, Subscriptions)
**Scope**: `{id, email, name, created, status, metadata}`
- **Source**: Stripe's authoritative database (replicated globally)
- **CID Strategy**: `last_updated` timestamp or version counter
- **Validation**: Field parity test (extract scope fields from Dashboard and API, compare)

### Aggregate Resources (MRR Report)
**Scope**: `{mrr_value, month, timezone, computation_definition, included_statuses}`
- **Computation**: `sum(subscription.amount WHERE status IN ('active', 'trialing') AND billing_anchor IN month)`
- **Filter Parameters**:
  - Month: `2025-02`
  - Timezone: `UTC` (all timestamps normalized)
  - Included statuses: `['active', 'trialing']`
  - Currency rounding: 2 decimals
- **Projection CID**: `hash(profile=MRR_v2, month=2025-02, tz=UTC, snapshot=subscriptions@v1234)`
- **Validation**: Recompute MRR from `/v1/subscriptions` with identical filters, compare to Dashboard value

**Tolerance Rules**:
- Currency display: `$1,234.56` (Dashboard) vs. `1234.56 USD` (API) are equivalent
- Timestamp precision: Second-level (no milliseconds required)

## Pattern Gaps
- Missing consistent bidirectional links between Dashboard pages and specific API endpoints.
- No shared content identity (CID) for reports and derived views to guarantee equivalence at a point in time.
- No simple, account-scoped Resource Index enumerating (rid, hr, mr, cid) across key object types.
- **Equivalence scope not formally documented** (especially for MRR and other aggregates).

## Recommended Upgrades
1) Bidirectional linking
   - HR: Add a "Developer info" control exposing the precise MR URL (and sample curl) for the current object.
   - MR: Include `dashboard_url` for core objects pointing to the exact HR page.
2) Shared CID (content identity)
   - For mutable entities: expose a version or last-modified token usable as a CID.
   - For reports/aggregates: compute and expose a content hash or snapshot ID; surface the same CID in the HR report view.
3) Standardized discovery (Resource Index)
   - Publish an account-scoped JSON index that lists key resources (customers, products, plans, subscriptions, invoices, reports) with (rid, hr, mr, cid, updatedAt).
   - Provide incremental discovery (since=timestamp) to avoid full scans.

## Expected Benefits
- Faster debugging and automation through one-click navigation between HR and MR.
- Confidence in finance workflows via verifiable equivalence of reports and underlying API values.
- Lower load for analytics/AI agents by enabling zero-fetch skips when CIDs match.

## Risks/Considerations
- Ensure sensitive HR URLs are only revealed to authorized users/tokens.
- Clearly document CID semantics for aggregates (what inputs constitute the snapshot).

