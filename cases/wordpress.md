# WordPress (Content Management System)

**Domain:** Digital Publishing & CMS
**Status:** Reference Implementation
**Conformance Level:** 4 (Agentic / Read-Write)

---

## 1. The Context

WordPress powers ~43% of the web. Historically, it is Human-Native: it stores content as HTML (or HTML-commented blocks via Gutenberg) and renders HTML for the browser.

**HR (Human Representation):** The rendered HTML page and the Visual Gutenberg Editor.

**The Friction:** AI Agents (both crawlers and editors) struggle to parse the HTML layout, leading to high token costs and "hallucinations" when trying to edit content programmatically. Traditional approaches involve scraping the visual editor or manipulating HTML strings, which risks layout corruption and race conditions.

---

## 2. Implementation A: Internal Agentic Workflow (DNI)

**Role:** Production & Editing
**Conformance Level:** 4 (Agentic / Read-Write)

### Machine Representation (MR)

A JSON structure representing the **Block Graph**:

```json
{
  "rid": 123,
  "cid": "sha256-abc123...",
  "title": "Article Title",
  "status": "publish",
  "modified": "2025-11-21T12:00:00Z",
  "author": {
    "id": 1,
    "name": "John Doe",
    "url": "https://example.com/author/john"
  },
  "blocks": [
    {"type": "core/heading", "level": 2, "content": "Introduction"},
    {"type": "core/paragraph", "content": "Article content..."},
    {"type": "core/list", "ordered": false, "items": ["Item 1", "Item 2"]}
  ],
  "links": {
    "human_url": "https://example.com/article/",
    "api_url": "https://example.com/wp-json/dual-native/v1/posts/123",
    "md_url": "https://example.com/wp-json/dual-native/v1/posts/123/md",
    "public_api_url": "https://example.com/wp-json/dual-native/v1/public/posts/123",
    "public_md_url": "https://example.com/wp-json/dual-native/v1/public/posts/123/md"
  }
}
```

### Synchronization Mechanism

**Read Path:**
- Agents fetch the MR to understand the logical block structure without HTML noise
- Zero-fetch optimization: agents check CID in catalog to skip unchanged content
- Conditional GET: `If-None-Match` with CID returns 304 Not Modified if unchanged

**Write Path:**
- Agents use Safe, Index‑Aware Mutations via `/posts/{id}/blocks`:
  - `append`: Add blocks to the end of the post.
  - `prepend`: Add blocks to the start.
  - `insert` (with `index`): Inject blocks at a specific position (e.g., after the intro).
  - Note: Update/Delete by Block ID is planned for a future roadmap.

**Safe Write Protocol:**
1. Agent reads current MR and obtains CID
2. Agent computes desired mutation (e.g., insert heading at index 2)
3. Agent submits: `POST /posts/123/blocks` with `If-Match: "sha256-abc123..."`
4. System verifies CID matches current state
5. If match: applies mutation, serializes to Gutenberg block HTML, recomputes CID.
6. Response: Returns 200 OK with the updated MR and the new `ETag` header. This allows Agents to chain multiple writes immediately without an intermediate GET.
7. If mismatch: returns 412 Precondition Failed with the current `ETag` in headers (signal to re‑read).

**Projection:** The system handles serialization of JSON blocks into WordPress HTML (with Gutenberg block comments), preventing layout corruption. The editor remains usable by humans while agents work on the structured data.

### Use Case

An AI Assistant (e.g., via Model Context Protocol / MCP) reads a draft article, identifies a missing "Key Takeaways" section, and safely inserts a new heading block at index 2:

```http
POST /wp-json/dual-native/v1/posts/123/blocks
Content-Type: application/json
If-Match: "sha256-abc123..."

{
  "insert": "index",
  "index": 2,
  "block": {
    "type": "core/heading",
    "level": 2,
    "content": "Key Takeaways"
  }
}
```

The system:
- Validates the CID
- Inserts the block at the correct position
- Serializes to HTML: `<!-- wp:heading --><h2>Key Takeaways</h2><!-- /wp:heading -->`
- Recomputes CID
- Returns the updated MR with new CID for subsequent operations

---

## 3. Implementation B: Public Crawler Interface (TCT)

**Role:** Distribution & Discovery
**Conformance Level:** 3 (Optimized Read / Zero-Fetch)

### Machine Representation (MR)

A JSON Envelope containing:
- **Envelope**: Canonical metadata (profile, canonical_url, title)
- **Payload**: Content in Markdown or plain text

```json
{
  "profile": "tct-1",
  "canonical_url": "https://example.com/article/",
  "title": "Article Title",
  "content_media_type": "text/markdown; charset=utf-8",
  "content": "## Introduction\n\nArticle content...\n\n- Item 1\n- Item 2"
}
```

### Synchronization Mechanism

**Discovery:**
- HTTP `Link` header on root: `Link: </llm-sitemap.json>; rel="index"; type="application/json"`
- HTML `<link rel="index" type="application/json" href="/llm-sitemap.json">` in `<head>`
- **Public Endpoints:** Published posts are accessible via `/wp-json/dual-native/v1/public/posts/{id}` (JSON) and `/wp-json/dual-native/v1/public/posts/{id}/md` (Markdown).
- Sitemap lists all articles with C-URL (human) → M-URL (machine) mappings

**Validation:**
- Strong CIDs (Content IDs) based on SHA-256 hash of canonical JSON
- M-Sitemap includes CID hints for each article
- ETag headers on M-URL responses match CID format: `ETag: "sha256-..."`

**Zero-Fetch:**
1. Crawler fetches M-Sitemap: `/wp-json/llm-sitemap.json`
2. Sitemap returns:
   ```json
   {
     "version": 2,
     "items": [
       {
         "cUrl": "https://example.com/article/",
         "mUrl": "https://example.com/article/llm.json",
         "etag": "sha256-abc123..."
       }
     ]
   }
   ```
3. Crawler checks: if cached ETag matches sitemap `etag`, skip fetch (zero-fetch)
4. If mismatch or no cache: fetch M-URL with `If-None-Match`
5. Server returns 304 Not Modified if CID matches, or 200 with new content

### Use Case

An AI Search Crawler indexing 10,000 WordPress articles:
- Fetches the M-Sitemap once
- Compares sitemap CIDs with cached versions
- Skips ~90% of fetches (zero-fetch rate)
- Bandwidth savings: 83% (observed in production at 5,700+ URLs)
- Processing savings: no HTML parsing, direct Markdown ingestion

---

## 4. Key Insights

WordPress demonstrates that the Dual-Native Pattern can have **different views for different actors**:

| Aspect | Internal Agents (DNI) | Public Crawlers (TCT) |
|--------|----------------------|----------------------|
| **Purpose** | Production & Editing | Distribution & Discovery |
| **MR Format** | Granular block JSON | Markdown/text payload in JSON envelope |
| **Read** | Block-level structure | Document-level content |
| **Write** | Full CRUD with `If-Match` | Read-only |
| **Zero-Fetch** | Catalog with CID | M-Sitemap with ETag hints |
| **Discovery** | REST API documentation | `Link` headers + HTML `<link>` tags |

Both rely on the same underlying truth:
- **The Machine Representation is the Source of Truth for Logic**
- **The HTML is the View for Humans**
- **Synchronization is Bidirectional and Deterministic**

---

## 5. Architecture Pattern

WordPress serves as a reference implementation of the **Dual-Native Operating System** concept:

```
┌─────────────────────────────────────────────────────────┐
│                    WordPress Core                        │
│                 (Content OS / Kernel)                    │
└─────────────────────────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────┐
          │                                 │
    ┌─────▼─────┐                    ┌─────▼─────┐
    │    DNI    │                    │    TCT    │
    │ (Internal)│                    │ (Public)  │
    └─────┬─────┘                    └─────┬─────┘
          │                                 │
    ┌─────▼──────┐                   ┌─────▼──────┐
    │  Editors   │                   │  Crawlers  │
    │ (MCP/AI)   │                   │  (Search)  │
    └────────────┘                   └────────────┘
```

**DNI (Dual-Native Internal):** The "kernel API" for agentic workflows—full read/write access with CID-based concurrency control.

**TCT (Trusted Collaboration Tunnel):** The "distribution protocol" for content delivery—optimized read-only access with zero-fetch caching.

Together, they make WordPress look like an **Operating System for Content**, where:
- DNI/TCT are the kernel APIs
- Gutenberg is the "shell" (structured editor)
- AI agents are first-class "processes"

---

## 6. Implementation Details

### Block-Aware Extraction

WordPress Gutenberg stores content as HTML with structured comments:

```html
<!-- wp:heading {"level":2} -->
<h2>Introduction</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Article content...</p>
<!-- /wp:paragraph -->
```

The MR parser:
1. Uses `parse_blocks()` to extract the block tree
2. Maps each block type to MR structure:
   - `core/heading` → `{"type": "core/heading", "level": 2, "content": "..."}`
   - `core/paragraph` → `{"type": "core/paragraph", "content": "..."}`
   - `core/list` → `{"type": "core/list", "ordered": bool, "items": [...]}`
3. Strips HTML tags to get clean text
4. Preserves semantic structure (heading levels, list ordering, etc.)

### Determinism & Stability
To ensure CIDs are stable (only changing when content changes):
- **Taxonomy Sorting:** Categories and Tags are sorted by Term ID before hashing. This prevents database query order from artificially changing the hash.
- **Field Exclusion:** Volatile fields (like `links` which depend on environment URLs) are excluded from the CID computation by default.

### Drift Prevention

**Challenge:** If a human manually edits the HTML in "Code Editor" mode, they could break the block structure, causing drift.

**Solution:** Gutenberg itself is a **Structured Editor**—it works on the MR (blocks), not raw HTML. Invalid edits are either:
- Prevented by the UI (can't create malformed blocks)
- Recovered as "unsupported" blocks (preserves content, alerts human)

**CID Invalidation:** Any `save_post` action triggers:
```php
delete_post_meta($post_id, '_dni_cid'); // Force recompute
```

Next MR request recomputes CID from current block structure, ensuring synchronization.

### Content-Digest (RFC 9530)

DNI implements RFC 9530 `Content-Digest` headers:

```http
Content-Digest: sha-256=:ABC123base64...:
```

Serve‑Time Computation: The digest is computed over the final wire bytes (after JSON encoding or Markdown generation) to guarantee that the signature matches the payload exactly, bypassing PHP output buffering quirks.

This provides:
- **Integrity**: Clients can verify the response wasn't tampered with
- **Symmetry**: Both read and write responses include digests
- **Debugging**: Detect CDN/proxy corruption

---

## 7. Model Context Protocol (MCP) Integration

The DNI implementation is designed to serve as a robust **MCP Server** for WordPress.

**Why Dual‑Native is Essential for MCP:**
1. **MCP agents cannot safely work with raw HTML** (risk of corruption).
2. **MCP requires structured, validated operations** (not DOM manipulation).
3. **MCP needs optimistic concurrency** (prevent lost updates from simultaneous edits).

**DNI as MCP Server (Tool Mapping):**
- `read_mr`: Fetch Machine Representation (JSON) with CID.
- `read_md`: Fetch Raw Markdown content.
- `list_posts`: Query the Catalog (supports status/since filters).
- `append_block`: Add content safely to the end.
- `insert_at_index`: Inject content at a specific position.
- `set_title`: Update post title safely (supports Optimistic Locking).
- `set_slug` / `set_status`: Granular metadata control.
- All write tools accept `if_match` to enforce optimistic locking.

**Result:** Claude (or other MCP clients) can edit WordPress content with the same safety guarantees as native API clients, without risk of HTML corruption or race conditions.

---

## 8. Production Evidence

**Deployment Scale:** 5,700+ WordPress URLs serving TCT M-Sitemaps

**Observed Metrics:**
- **Zero-Fetch Rate:** 90%+ (crawlers skip unchanged content)
- **Efficiency Gains:**
  - **Public Crawlers (TCT):** 83% bandwidth savings vs Full HTML (removes theme, nav, footer noise).
  - **Internal Agents (DNI):** ~68% token savings vs Standard REST API (removes JSON escaping, `_links`, and HTML comments).
- **Write Safety:** 100% (no reported HTML corruption from DNI mutations)
- **Latency:** Sub-50ms CID verification, <200ms for conditional GET

**Operational Notes:**
- CIDs cached in post meta for O(1) lookup
- Taxonomies sorted by ID for deterministic CID computation
- Content-Digest computed at serve time (no pre-computation needed)
- Gutenberg block parser is fast (~10ms for typical articles)

---

## 9. Conclusion

WordPress demonstrates that:

1. **Dual-Native is not just for APIs**: CMS platforms benefit equally
2. **One system, multiple views**: DNI (internal) + TCT (public) serve different actors from the same source of truth
3. **Agentic editing is safe**: Structural mutations + CID concurrency prevent corruption
4. **The pattern scales**: 5,700+ production URLs validate the architecture

The WordPress implementation proves the Dual-Native Pattern is the **architectural prerequisite for robust MCP servers** and **agentic content systems**. You cannot build safe AI editing on top of HTML scraping—you need Dual-Native.
