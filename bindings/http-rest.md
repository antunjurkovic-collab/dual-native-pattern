# HTTP / REST Binding (Informative)

**Status:** Informative binding for the Dual-Native Pattern Core Spec.
**Scope:** HTTP/1.1 and HTTP/2 JSON and HTML APIs using REST-style routes.

This document shows how to map the Dual-Native primitives (RID, MR, CID, safe writes, zero-fetch, DNC) onto HTTP/REST semantics. It does **not** change any normative requirements from `CORE-SPEC.md`; it provides a concrete, copy-pasteable binding for typical web APIs.

---

## 1. Scope and Goals

This binding covers systems that:

- Serve **HR** as HTML (or other human-oriented media) over HTTP
- Serve **MR** as JSON (or similar structured formats) over HTTP
- Use standard HTTP validators and headers:
  - `ETag` as representation validator
  - `If-None-Match` for conditional reads
  - `If-Match` for safe writes
  - `Content-Digest` (RFC 9530) for integrity (RECOMMENDED)

The goal is to show how to reach:

- **Level 1.5 – AI-Native Core (MR-only)** over HTTP
- **Level 2–4 – Full Dual-Native** over HTTP with HR + MR, DNC, and safe writes

---

## 2. Core Mapping: Dual-Native → HTTP

### 2.1 Identities

- **RID (Resource Identity)**
  - Use a stable identifier that survives content changes.
  - In HTTP, common choices:
    - The canonical HR URL, e.g. `https://example.com/article/123`
    - Or a stable URN/ID carried inside MR and DNC (e.g. `"rid": "article:123"`)

- **HR (Human Representation)**
  - **HTTP resource** optimized for humans, typically:
    - `GET /article/123/`
    - `Content-Type: text/html; charset=utf-8`
  - HR MAY have its own `ETag` and caching rules.

- **MR (Machine Representation)**
  - **HTTP resource** optimized for agents, typically:
    - `GET /article/123/llm/`
    - `Content-Type: application/json; profile="dual-native-core-1.0"`
  - MR MUST expose CID via validator and SHOULD expose integrity and profile.

### 2.2 Validators and CID

- **CID (Content Identity)** in HTTP is mapped as:

  - **Per-representation validator:**
    - Use `ETag` as the HTTP validator for a specific representation (HR or MR).
  - **Shared logical content version:**
    - Use `content_id` in the **Dual-Native Catalog (DNC)** as the shared logical CID across HR and MR.

**Guidance:**

- **For MR-only systems (AI-Native Core, Level 1.5):**
  - You MAY set `ETag == CID` on the MR endpoint.
- **For full Dual-Native (HR + MR):**
  - HR and MR MAY have different `ETag` values (different serializations).
  - The DNC's `content_id` field is the shared logical CID for parity checks.

### 2.3 HTTP Header Mapping (Summary)

| Dual-Native primitive | HTTP field / mechanism                          |
|-----------------------|-------------------------------------------------|
| RID                   | HR URL, MR URL, or explicit `"rid"` in JSON    |
| CID (per view)        | `ETag`                                         |
| Logical CID           | `content_id` field in DNC JSON                 |
| Zero-fetch read       | `If-None-Match` → `304 Not Modified`           |
| Safe write            | `If-Match` → `412 Precondition Failed`         |
| Integrity (bytes)     | `Content-Digest: sha-256=…` (RFC 9530)         |
| Conformance level     | `X-Dual-Native-Level`, `X-Dual-Native-Profile` |

---

## 3. Resource Model and URIs

A minimal HTTP dual-native deployment typically uses:

- HR endpoint (human view):
  ```http
  GET /article/123/ HTTP/1.1
  Host: example.com
  ```

- MR endpoint (machine view):
  ```http
  GET /article/123/llm/ HTTP/1.1
  Host: api.example.com
  Accept: application/json
  ```

Both endpoints represent the **same RID** and SHOULD be linked bidirectionally.

Example MR body:

```json
{
  "rid": "https://example.com/article/123",
  "profile": "dual-native-core-1.0",
  "cid": "sha256-abc123...",
  "title": "Article Title",
  "blocks": [
    {
      "type": "heading",
      "level": 1,
      "content": "Article Title"
    },
    {
      "type": "paragraph",
      "content": "Body text..."
    }
  ]
}
```

---

## 4. Read Semantics (Zero-Fetch)

### 4.1 Initial MR Fetch

Client fetches MR:

```http
GET /article/123/llm/ HTTP/1.1
Host: api.example.com
Accept: application/json
```

Server response:

```http
HTTP/1.1 200 OK
Content-Type: application/json; profile="dual-native-core-1.0"
ETag: "sha256-abc123..."
Content-Digest: sha-256=:BASE64_DIGEST:
Cache-Control: max-age=60
X-Dual-Native-Level: 3
X-Dual-Native-Profile: dual-native-core-1.0

{
  "rid": "https://example.com/article/123",
  "cid": "sha256-abc123...",
  "title": "Article Title",
  "blocks": [...],
  "hr": "https://example.com/article/123/"
}
```

**Requirements (mapping AI-Native Core & Level 3):**

- MR **MUST** include a validator (`cid` field and/or `ETag`).
- Server **SHOULD** include `Content-Digest` for final-bytes integrity.
- MR **SHOULD** include a resolvable reference to HR (`hr` link).

### 4.2 Conditional Read (Zero-Fetch)

Client revalidates MR:

```http
GET /article/123/llm/ HTTP/1.1
Host: api.example.com
If-None-Match: "sha256-abc123..."
```

If unchanged:

```http
HTTP/1.1 304 Not Modified
ETag: "sha256-abc123..."
```

- Body MUST be omitted.
- Client reuses cached MR.

If changed:

```http
HTTP/1.1 200 OK
ETag: "sha256-def456..."
Content-Digest: sha-256=:BASE64_NEW:
...
{
  "cid": "sha256-def456...",
  ...
}
```

---

## 5. Write Semantics (Safe Writes)

### 5.1 General Contract

- All **mutations** MUST accept a validator precondition via `If-Match`.
- On validator mismatch:
  - Server MUST reject the mutation with `412 Precondition Failed`.
- On success:
  - Server MUST return the **new** validator (new `ETag` and/or `cid`) so clients can chain edits.

### 5.2 Example: Block-Level Mutation

Client wants to append a block to MR:

```http
POST /article/123/llm/blocks HTTP/1.1
Host: api.example.com
Content-Type: application/json
If-Match: "sha256-abc123..."
```

Body:

```json
{
  "insert": "append",
  "block": {
    "type": "core/paragraph",
    "content": "New content to add"
  }
}
```

#### 5.2.1 Success

```http
HTTP/1.1 200 OK
Content-Type: application/json
ETag: "sha256-def456..."
Content-Digest: sha-256=:BASE64_NEW:

{
  "rid": "https://example.com/article/123",
  "cid": "sha256-def456...",
  "mutation": {
    "inserted_at": 3,
    "blocks_before": 3,
    "blocks_after": 4
  },
  "mr": {
    "blocks": [...]
  }
}
```

- Server applied the mutation atomically.
- New `ETag` / `cid` represents the new MR snapshot.
- Optional telemetry helps agents reason about the change.

#### 5.2.2 Conflict (`412 Precondition Failed`)

If MR changed between read and write:

```http
HTTP/1.1 412 Precondition Failed
Content-Type: application/json

{
  "error": {
    "type": "precondition_failed",
    "message": "Validator mismatch",
    "current_cid": "sha256-zzz999..."
  }
}
```

Client algorithm:

1. Re-fetch MR to get current `cid`.
2. Apply its edit to the new MR.
3. Retry write with updated `If-Match`.

---

## 6. Catalog Semantics (DNC over HTTP)

The **Dual-Native Catalog (DNC)** gives agents a way to discover resources and track which ones changed.

### 6.1 Catalog Endpoint

Example:

```http
GET /dual-native/catalog.json?since=2025-01-16T00:00:00Z HTTP/1.1
Host: api.example.com
Accept: application/json
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json; profile="dual-native-catalog-1.0"

{
  "version": 1,
  "profile": "dual-native-catalog-1.0",
  "items": [
    {
      "rid": "https://example.com/article/123",
      "hr": "https://example.com/article/123/",
      "mr": "https://api.example.com/article/123/llm/",
      "content_id": "sha256-def456...",
      "updatedAt": "2025-01-16T10:00:00Z",
      "type": "article",
      "status": "published"
    },
    {
      "rid": "https://example.com/article/456",
      "hr": "https://example.com/article/456/",
      "mr": "https://api.example.com/article/456/llm/",
      "content_id": "sha256-ghi789...",
      "updatedAt": "2025-01-16T11:30:00Z",
      "type": "article",
      "status": "draft"
    }
  ]
}
```

**Requirements (HTTP binding of CORE-SPEC DNC):**

- Each item MUST include:
  - `rid`, `hr`, `mr`, `content_id`, `updatedAt`
- `content_id` is the **logical CID** shared across HR and MR.
- Catalog SHOULD support:
  - `since` filters (timestamp or cursor)
  - Pagination (`limit`, `cursor`)

### 6.2 Client Strategy

- Poll `GET /dual-native/catalog.json?since=…`
- For each entry:
  - Compare `content_id` with cached value
  - If unchanged, skip fetch (zero-fetch at catalog level)
  - If changed, fetch MR (and HR if needed), using conditional GET

---

## 7. Bidirectional Linking (HR ↔ MR)

### 7.1 HR → MR

HTML page SHOULD link to MR and catalog:

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="alternate"
        type="application/json"
        href="https://api.example.com/article/123/llm/"
        title="Machine Representation">
  <link rel="sitemap"
        type="application/json"
        href="https://api.example.com/dual-native/catalog.json">
  <meta name="dual-native-conformance"
        content="level=3; profile=dual-native-core-1.0">
</head>
<body>…</body>
</html>
```

### 7.2 MR → HR

MR SHOULD echo a pointer to HR:

```json
{
  "rid": "https://example.com/article/123",
  "hr": "https://example.com/article/123/",
  "cid": "sha256-def456...",
  ...
}
```

This satisfies the **bidirectional discovery** requirement without inventing new link mechanisms.

---

## 8. Conformance Checklist (HTTP / REST Binding)

### 8.1 AI-Native Core (Level 1.5) over HTTP

MR-only system can claim **AI-Native Core (MR-Only Mode)** plus this binding if:

- [ ] Stable RID per resource (in URL or `rid`)
- [ ] MR endpoint returns:
  - [ ] `ETag` validator (MAY equal `cid`)
  - [ ] `cid` field in body (deterministic from MR)
  - [ ] Optional `Content-Digest` for integrity
- [ ] Supports conditional read:
  - [ ] `If-None-Match` → `304 Not Modified`
- [ ] Supports safe writes:
  - [ ] Mutation route(s) that honor `If-Match`
  - [ ] On mismatch: `412 Precondition Failed`
  - [ ] On success: returns new `ETag`/`cid`
- [ ] DNC endpoint (RECOMMENDED) exposing at least `rid`, `mr`, `content_id`, `updatedAt`

### 8.2 Full Dual-Native (Level 2–4) over HTTP

To claim **Level 2+** with this binding:

- [ ] HR endpoint exists and is human-usable (HTML or equivalent)
- [ ] HR ↔ MR bidirectional links:
  - [ ] `<link rel="alternate">` in HR
  - [ ] `hr` field in MR
- [ ] DNC enumerates HR + MR with shared `content_id`
- [ ] CID-based zero-fetch and safe writes as above
- [ ] Integrity digest (`Content-Digest`) for MR responses (SHOULD)
- [ ] Optional:
  - [ ] `X-Dual-Native-Level`, `X-Dual-Native-Profile` for observability

---

## 9. Relationship to TCT

This binding describes a **generic HTTP/REST mapping** for Dual-Native systems.

- Use this binding when:
  - You control your own JSON/HTML endpoints.
  - You want Dual-Native semantics for your APIs and UIs.

- Use **TCT (Collaboration Tunnel Protocol)** when:
  - You want a dedicated HTTP profile specifically for **crawler/agent-friendly textual envelopes** (Markdown/plain text + metadata).
  - You need sitemap-based discovery and strict canonicalization for large-scale web crawling.

Both share the same underlying pattern:

- RID, MR, CID
- Zero-fetch reads
- Safe writes (where applicable)
- DNC for discovery

TCT is effectively a **specialized HTTP MR profile** optimized for web-scale crawlers; this binding is a **general HTTP/REST profile** suitable for app APIs, backends, and dashboards.
