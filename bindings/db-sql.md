# SQL Database Binding (Draft, Informative)

**Status:** Draft binding for the Dual-Native Pattern.
**Scope:** SQL databases (PostgreSQL/MySQL/etc.) using rows/documents as MR.

## 1. Scope

This binding shows how to apply Dual-Native primitives inside a relational database:

- RID → primary key / composite key
- MR → normalized row/document (optionally JSON column)
- CID → version/hash column
- Safe writes → optimistic concurrency via `WHERE cid = ?`
- Zero-fetch → only re-read rows when CID changed according to a catalog/change feed
- DNC → catalog table or view

## 2. Mapping

- **RID**
  - `rid` column, or natural primary key (`id`, `tenant_id + slug`).

- **MR**
  - The logical row/document (optionally a JSON column `mr`).

- **CID**
  - `cid` column, e.g. `VARCHAR` or `BYTEA`, computed as a hash over canonical MR.

- **Validator (read/write)**
  - Reads return `cid`; writes include expected `cid` in `WHERE` clause.

- **DNC**
  - Catalog table `dual_native_catalog(rid, cid, updated_at, type, status, ...)`.

## 3. Read Semantics

- Client stores `(rid, cid)`.
- To check freshness, query:
  - `SELECT cid FROM dual_native_catalog WHERE rid = ?;`
- If unchanged → reuse cached MR (zero-fetch).
- If changed → fetch full MR (row or JSON).

## 4. Write Semantics (Safe Writes)

Update example:

```sql
UPDATE posts
SET mr = :new_mr, cid = :new_cid, updated_at = NOW()
WHERE rid = :rid AND cid = :expected_cid;
```

- If `row_count = 0` → conflict (precondition failed).
- If `row_count = 1` → success; client uses `new_cid` for next edit.

## 5. Catalog (DNC)

Example schema:

```sql
CREATE TABLE dual_native_catalog (
  rid TEXT PRIMARY KEY,
  cid TEXT NOT NULL,
  type TEXT,
  status TEXT,
  updated_at TIMESTAMPTZ NOT NULL
);
```

Clients can call:

```sql
SELECT * FROM dual_native_catalog
WHERE updated_at > :since
ORDER BY updated_at
LIMIT :limit;
```

## 6. Conformance Notes

A system can claim SQL Dual-Native binding if:

- [ ] There is a stable RID column.
- [ ] CID is deterministic over MR and updated on changes.
- [ ] All writes are guarded by `cid` (OCC).
- [ ] A catalog (or equivalent view/change feed) exposes `rid, cid, updated_at` for incremental sync.
