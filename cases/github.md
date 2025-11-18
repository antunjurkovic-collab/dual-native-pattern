# GitHub (Developer Platform): Dual-Native Case

Domain: Developer Tools / Collaboration

- Human Representation (HR): GitHub web UI for repositories, issues, pull requests, actions, releases, code browsing.
- Machine Representation (MR): GitHub REST API and GraphQL API.
- Shared Data: Repos, branches, commits, issues, PRs, reviews, checks, workflows, releases, discussions.

## Current State
- Discovery: REST/GraphQL list and search; webhooks for events. No per-repo machine index that pairs HR/MR for major resources.
- Synchronization/Validators: Strong caching and ETag support on REST; GraphQL node IDs provide stable MR identities.
- Linking: MR -> HR exists for many objects via `html_url`. HR -> MR is not consistently advertised from HTML pages.

## Equivalence Scope

### Simple Resources (Repository, Issue)
**Scope**: `{id, name, description, owner, visibility, created_at, updated_at}`
- **Source**: GitHub's authoritative database (globally replicated)
- **CID Strategy**: `updated_at` timestamp or commit SHA
- **Validation**: Field parity test (web page meta vs. API response)

### Composite Resources (Pull Request)
**Scope**: `{pr_number, state, title, head_sha, base_sha, mergeable, required_checks_result}`
- **Composite CID**: `hash(head_sha + checks_summary_hash + updated_at)`
- **Computation**: PR state derives from:
  - Head commit SHA (latest code)
  - Required status checks (CI/CD results)
  - Review approvals
  - Merge conflicts
- **Validation Pattern**: Composite CID verification
  - Both HR (web page) and MR (API) must expose same `head_sha + checks_summary_hash`
  - Recompute checks summary from `/repos/{owner}/{repo}/commits/{sha}/check-runs`

**Tolerance Rules**:
- Timestamp formatting: ISO 8601 (API) vs. relative time (web: "2 hours ago") are semantically different presentations
- Check status: `success` (API) vs. green checkmark (web) are equivalent

## Pattern Gaps
- Missing consistent HR → MR links on key pages (issue, PR, commit, release).
- No lightweight repo/org-level Resource Index listing (rid, hr, mr, cid, updatedAt) for top resources.
- Semantic equivalence rules are implicit (e.g., PR status across UI/API) rather than formally verifiable. **Equivalence scope not documented.**

## Recommended Upgrades
1) Bidirectional linking
   - HR: Add `<link rel="alternate" type="application/json" href="...">` (or visible UI link) for the exact API endpoint; include GraphQL node ID.
   - MR: Keep `html_url` and add explicit `hr_url` for any object lacking it.
2) Shared CID (content identity)
   - Reuse existing validators where available (ETag/Last-Modified) and surface a stable content-id for complex aggregates (e.g., combined status checks).
   - Document equivalence expectations for PRs (title/body/status/commit pointer) and expose a composite CID covering those fields.
3) Standardized discovery (Resource Index)
   - Provide a repo-scoped JSON index of issues, PRs, releases, workflows with (rid, hr, mr, cid, updatedAt).
   - Offer incremental updates or evented diffs for efficient sync.

## Expected Benefits
- Tooling and AI agents can jump directly between UI and API representations.
- Reduced redundant fetches via CID-based zero-fetch behavior for bots and dashboards.
- Clearer guarantees for automation (e.g., safe-to-merge checks referencing the same CID as the UI state).

## Risks/Considerations
- Ensure rate limits account for index access; paginate and cache.
- Avoid leaking private HR URLs in MR for private repos; respect permissions.

