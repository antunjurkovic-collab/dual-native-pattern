# Object Storage Binding (Draft, Informative)

**Status:** Draft binding for the Dual-Native Pattern.
**Scope:** Object / blob storage systems such as Amazon S3, Google Cloud Storage, Azure Blob, MinIO, and similar APIs.

This document shows how to map Dual-Native primitives (RID, MR, CID, zero-fetch, safe writes, DNC) onto **object storage**. It does **not** change any normative requirements from `CORE-SPEC.md`; it provides a concrete, copy-pasteable binding for object-store style systems.

---

## 1. Scope and Goals

This binding targets systems where:

- Content is stored as **objects** (files, blobs) addressed by keys/paths.
- You want agents and services to:
  - sync only the objects that changed,
  - safely update objects without clobbering concurrent edits,
  - treat the bucket as a Dual-Native catalog of resources.

Typical use cases:

- Configuration bundles, prompt libraries, policy packs
- Dataset manifests, training artifacts, checkpoints
- MR snapshots for resources (e.g., posts, documents, models)
- "Headless" content repositories

Dual-Native adds:

- A clear **RID** per logical resource,
- A strong **CID** per MR snapshot,
- Conditional writes for **safe updates**,
- A **DNC manifest** for incremental discovery.

---

## 2. Core Mapping: Dual-Native → Object Storage

### 2.1 Identities

- **RID (Resource Identity)**
  - A stable logical identifier for a resource. For example:
    - `post:123`
    - `model:classifier-v5`
    - `dataset:news-2025-01`
  - RID MAY be encoded in:
    - The object key (e.g., `posts/123/state.json`)
    - A field inside the object content (`"rid": "post:123"`)

- **MR (Machine Representation)**
  - The canonical representation of the resource stored as object content:
    - JSON, CBOR, Avro, YAML, etc.
    - For example: `posts/123/state.json` containing MR JSON.

- **CID (Content Identity)**
  - A deterministic identifier for the MR snapshot. Options:
    - A content hash over canonicalized MR (`sha256(canonical(MR))`)
    - Or a version string stored in object metadata (e.g., `x-amz-meta-cid`)

### 2.2 Object Keys and Layout

A common layout:

- **MR objects**

  ```text
  posts/123/state.json
  models/classifier-v5/state.json
  datasets/news-2025-01/state.json
  ```

- **DNC manifest(s)**

  ```text
  dual-native/catalog.json
  dual-native/catalog-2025-01.json
  ```

You are free to choose your own conventions, but SHOULD document:

- how RID maps to object keys,
- where DNC manifests live.

---

## 3. Validators, CIDs, and Metadata

Object storage APIs typically offer:

- `ETag` for object versions (may be a hash, may be an implementation detail),
- Optional version IDs (for versioned buckets),
- Custom metadata headers.

Dual-Native needs a **semantic CID** that is:

- deterministic from MR,
- stable across transports (e.g., DB, HTTP, object store),
- independent of storage-internal quirks (multipart ETags, compression, etc.).

### 3.1 Recommended Approach

- **CID**: compute as `sha256(canonical(MR))` and store:
  - inside MR (e.g., `"cid": "sha256-..."`), and/or
  - as object metadata (e.g., `x-amz-meta-cid: sha256-...`)
- **ETag**:
  - You MAY treat it as a low-level validator for conditional writes (if the provider supports strong ETags).
  - Do **not** rely on `ETag` alone as the semantic CID if multipart uploads or provider-specific behaviors break the "hash of bytes" assumption.

### 3.2 Minimal Metadata Set

Each MR object SHOULD expose:

- `rid` (in content)
- `cid` (in content and/or metadata)
- `Content-Length`
- `Last-Modified`
- Provider-specific version/ETag (for conditional PUTs/DELETEs)

Example MR object content:

```json
{
  "rid": "post:123",
  "cid": "sha256-abc123...",
  "title": "Article Title",
  "blocks": [
    { "type": "heading", "content": "Article Title" },
    { "type": "paragraph", "content": "Body text..." }
  ]
}
```

---

## 4. Read Semantics (Zero-Fetch for Objects)

### 4.1 Initial Fetch

Agent wants MR for `post:123`:

1. Compute or look up the object key for `rid = "post:123"` (e.g., `posts/123/state.json`).

2. Perform a `GET` on that object:
   - S3-style:

     ```http
     GET /bucket/posts/123/state.json HTTP/1.1
     Host: s3.example.com
     ```

   - Or use SDK equivalent.

3. Parse MR and extract `cid`.

### 4.2 Conditional Fetch (Zero-Fetch)

Instead of always downloading the full object:

1. Agent keeps `(rid, cid)` from last sync.
2. Agent wants to check if MR changed:
   - If CID is stored in metadata:
     - Use a metadata-only call (e.g., `HEAD` or `GetObjectAttributes`) and compare stored `x-amz-meta-cid` with cached `cid`.
   - If only `ETag` is used as a validator:
     - Use `If-None-Match` semantics via HTTP (when object store exposes HTTP interface).

If unchanged:

- Skip downloading body (zero-fetch).

If changed:

- Download object, parse MR, store new `cid`.

---

## 5. Write Semantics (Safe Writes)

Object storage usually supports conditional PUT/DELETE based on ETag or version ID. Dual-Native uses this to implement **safe writes**.

### 5.1 Generic Safe Write Pattern

1. Agent reads MR and obtains:
   - `cid_current` (semantic CID from MR),
   - `etag_current` (store validator, if available),
   - optional version ID.

2. Agent computes `new_mr` and `cid_new` from desired change.

3. Agent performs a **conditional PUT**:
   - If the provider supports conditional PUT by ETag:

     ```http
     PUT /bucket/posts/123/state.json HTTP/1.1
     Host: s3.example.com
     If-Match: "ETAG_CURRENT"
     x-amz-meta-cid: sha256-def456...
     Content-Type: application/json

     { ... new MR with "cid": "sha256-def456..." ... }
     ```

   - Or uses SDK equivalents: `IfMatch = etag_current`.

4. Server behavior:
   - If underlying object hasn't changed:
     - PUT succeeds → new version stored → `etag_new` returned.
   - If object changed (concurrent writer):
     - PUT fails (e.g., `412 Precondition Failed`, `PreconditionFailed`, or provider-specific error).
     - Agent MUST re-read MR and retry with updated state.

### 5.2 Safe Writes Using Semantic CID

If your stack doesn't trust ETag, you can implement safe writes **at a higher layer**, combining DB + object storage:

- Persist `cid` for each `rid` in a metadata store (e.g., SQL catalog).
- Before writing object:
  - Check DB row: `WHERE rid = :rid AND cid = :expected_cid`.
  - If row count = 0 → conflict.
  - If row count = 1:
    - Update `cid` in DB to `cid_new`.
    - Upload new MR object with `cid_new`.

This mirrors the **SQL binding**'s safe write semantics, with the object as the actual MR payload.

---

## 6. Catalog (DNC) in Object Storage

You can implement the Dual-Native Catalog as:

- A **manifest object** (`dual-native/catalog.json`), or
- A **sharded manifest** (`dual-native/catalog-001.json`, `dual-native/catalog-002.json`).

### 6.1 Manifest Structure

Example `dual-native/catalog.json`:

```json
{
  "version": 1,
  "profile": "dual-native-catalog-1.0",
  "generatedAt": "2025-01-16T12:00:00Z",
  "items": [
    {
      "rid": "post:123",
      "object": "posts/123/state.json",
      "cid": "sha256-abc123...",
      "type": "post",
      "status": "published",
      "updatedAt": "2025-01-16T10:00:00Z"
    },
    {
      "rid": "dataset:news-2025-01",
      "object": "datasets/news-2025-01/state.json",
      "cid": "sha256-dataset123...",
      "type": "dataset",
      "status": "active",
      "updatedAt": "2025-01-15T08:30:00Z"
    }
  ]
}
```

### 6.2 Incremental Discovery

Options:

- **Timestamp-based manifests**
  - Generate `dual-native/catalog-YYYY-MM-DD.json` each day.
  - Agents:
    1. Keep track of last fetched manifest(s).
    2. Fetch new manifests only for days after their last `updatedAt`.
    3. Compare `cid` per `rid` to decide which MR objects to pull.

- **Single manifest with `since` semantics (external service)**
  - Expose a small HTTP/API layer in front of the bucket that:
    - Reads object metadata (e.g., `Last-Modified`, `x-amz-meta-cid`),
    - Serves a DNC JSON response with `?since=` filters.

The exact generation mechanism is operational, not part of the core pattern; what matters is that agents can get **RID → CID + object key** with a `since`-like filter.

---

## 7. Example: Full Object Lifecycle

1. **Create resource** `post:123`:
   - Compute MR.
   - Compute `cid = sha256(canonical(MR))`.
   - `PUT posts/123/state.json` with `"cid"` in body + metadata.
   - Add entry to `dual-native/catalog.json` (or next manifest).

2. **Agent syncs**:
   - Downloads `dual-native/catalog.json`.
   - For each `rid`, stores `(rid, cid, object_key)`.

3. **Agent wants MR**:
   - Downloads `posts/123/state.json` if local `cid` missing or different.

4. **Agent edits MR**:
   - Reads current MR + `cid_current`.
   - Computes `new_mr` + `cid_new`.
   - Conditional PUT using ETag or DB-level safe write.
   - On success, updates catalog (or triggers catalog generator).

5. **Another agent syncs later**:
   - Fetches updated catalog.
   - Sees `cid` has changed for `post:123`.
   - Downloads new MR; zero-fetchs others.

---

## 8. Conformance Checklist (Object Storage Binding)

A system using object storage can claim a **Dual-Native object binding** if:

- [ ] There is a documented mapping from **RID → object key**.
- [ ] MR is stored as object content in a deterministic format (e.g., JSON).
- [ ] A **semantic CID** is computed over canonical MR:
  - [ ] CID stored in object content and/or metadata.
  - [ ] CID recomputation on MR changes is well-defined.
- [ ] Read path supports zero-fetch:
  - [ ] Agents can check CID (via metadata, manifest, or DB) before downloading.
- [ ] Write path supports safe writes:
  - [ ] Either conditional PUT/DELETE using ETag/version ID, or
  - [ ] Application-level OCC via metadata store (`WHERE rid AND cid`).
- [ ] A **DNC mechanism** exists:
  - [ ] Manifest(s) or an API exposing `(rid, object_key, cid, updatedAt, type/status)` with some form of `since`/cursor semantics.
- [ ] (Recommended) MR content includes:
  - [ ] `rid`
  - [ ] `cid`
  - [ ] Optional HR link (if an HTTP view exists).

---

## 9. Relationship to HTTP / DB / Streaming Bindings

This binding typically complements:

- **Database binding (SQL/NoSQL)**
  - DB stores MR and/or metadata tables (catalog, versions).
  - Object store holds large MR payloads or derived artifacts (e.g., snapshots).

- **Streaming binding (Kafka/etc.)**
  - Streams emit events when MR objects change (`rid, prev_cid, new_cid, object_key`).
  - Agents listen to DNC events and then fetch MR objects only when `cid` changed.

- **HTTP binding (REST/TCT)**
  - HTTP APIs expose MR as JSON (possibly reading from object store).
  - Agents can either:
    - Call HTTP endpoints, or
    - Talk directly to object storage for bulk operations.

All bindings share the same essentials:

- **RID, MR, CID**
- **Safe writes**
- **Zero-fetch**
- **DNC-based discovery**

Object storage becomes the **AI-native "disk"** for MR snapshots, while DB, streams, and HTTP provide indexing, change notification, and interactive access.
