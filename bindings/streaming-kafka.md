# Streaming / Kafka Binding (Draft, Informative)

**Status:** Draft binding for the Dual-Native Pattern.
**Scope:** Log-based streaming systems such as Apache Kafka, Pulsar, Kinesis, EventBridge, and similar event buses.

This document shows how to map Dual-Native primitives (RID, MR, CID, safe writes, zero-fetch, DNC) onto **streaming/event systems**. It does **not** change any normative requirements from `CORE-SPEC.md`; it provides one concrete, copy-pasteable binding for event-style transports.

---

## 1. Scope and Use Cases

This binding targets systems where:

- State evolves via **events** written to a log/stream.
- Multiple consumers (services, agents) need to:
  - stay in sync with resource state,
  - rebuild projections,
  - or react to changes in near real-time.

Typical use cases:

- Event-sourced aggregates (CQRS-style systems)
- Change Data Capture (CDC) streams
- Domain event buses for microservices
- ML feature pipelines consuming entity updates

Dual-Native gives these systems:

- A clear **RID** (which entity is this event about?),
- A strong **CID** (where in the state evolution are we?),
- A **DNC**-style way to discover "what changed" since last checkpoint,
- A safe way to coordinate **writes** (when the producer *knows* expected state).

---

## 2. Core Mapping: Dual-Native → Streaming

### 2.1 Identities

- **RID (Resource Identity)**
  - The logical entity whose state the events modify or describe.
  - Common choices:
    - Kafka message key (`key = "order:123"`)
    - A field inside the payload (`payload.rid = "order:123"`)
    - Aggregate ID in event-sourced systems

- **MR (Machine Representation)**
  - The **canonical state** of the resource (e.g., an "order snapshot" or "user profile").
  - In this binding, MR is usually:
    - Reconstructed by consumers from events, or
    - Published periodically as a snapshot message on a "state topic".

- **CID (Content Identity)**
  - A deterministic version of MR:
    - A hash of MR (`sha256(canonical(MR))`), or
    - A monotonically increasing version/sequence number (e.g., `version = 42`).
  - Often encoded as:
    - A field in the event payload (`cid`, `version`)
    - Or implicitly via `(partition, offset)` plus application-level rules

### 2.2 Streams and Topics

- **Event topics**
  - Carry domain events: "OrderCreated", "OrderUpdated", etc.
  - Each event MUST include:
    - `rid`
    - `prev_cid` (optional but recommended)
    - `new_cid` (CID of MR *after* applying event)
- **Snapshot / State topics (optional)**
  - Periodically publish full MR snapshots keyed by `rid`.
  - Each snapshot MUST include `rid` and `cid`.

---

## 3. Read Semantics (Zero-Fetch in Streams)

### 3.1 Consumer Perspective

A consumer (service or agent) wants to stay in sync with a set of resources without re-processing the entire log each time.

There are two complementary patterns:

1. **Offset-based checkpointing (native to streaming)**
   - Consumer stores `(topic, partition, offset)` and resumes from there.
   - This is transport-level; it knows "which events I've seen" but not *semantic* state.

2. **CID-based semantic checkpointing (Dual-Native)**
   - Consumer stores `(rid, cid)` per resource.
   - It only re-fetches or re-processes events for resources whose CID changed.

### 3.2 DNC over Streaming

A **streaming DNC** is a compact topic or stream that publishes "catalog events":

- Topic example: `dual-native.catalog`
- Message payload example:

```json
{
  "rid": "order:123",
  "prev_cid": "sha256-old...",
  "new_cid": "sha256-new...",
  "updatedAt": "2025-01-16T10:00:00Z",
  "type": "order",
  "status": "confirmed"
}
```

Clients use it as:

- A **log of catalog changes**:
  - Subscribe from last known offset
  - For each message, compare `new_cid` to cached `cid`
  - If unchanged → ignore
  - If changed → pull or recompute MR for that `rid`

This is the streaming equivalent of an HTTP DNC endpoint with `?since=`.

### 3.3 Snapshot Pattern

For some systems, you may publish MR snapshots:

- Topic: `dual-native.state.orders`
- Key: `rid = "order:123"`
- Value:

```json
{
  "rid": "order:123",
  "cid": "sha256-new...",
  "mr": {
    "status": "confirmed",
    "items": [...],
    "total": 42.50
  }
}
```

Consumers can:

- Use snapshots for initial sync,
- Use catalog/events for incremental updates.

---

## 4. Write Semantics (Safe Writes in Streams)

Streaming systems are **append-only** by design, but you can still apply Dual-Native's **safe write** semantics at the application level.

### 4.1 Event Envelope

Wrap each event in a Dual-Native envelope:

```json
{
  "rid": "order:123",
  "prev_cid": "sha256-old...",
  "new_cid": "sha256-new...",
  "event_type": "order.updated",
  "timestamp": "2025-01-16T10:00:00Z",
  "payload": {
    "status": "confirmed"
  }
}
```

### 4.2 Producer Contract (Safe Writes)

When producing an event:

1. Producer reads current MR and its `cid_current`.
2. Producer computes `cid_new` for the MR *after* applying its intended change.
3. Producer sends event with:
   - `prev_cid = cid_current`
   - `new_cid = cid_new`

On the **consumer/validator** side (could be a service or a stream processor):

- If the latest known CID for `rid` equals `prev_cid`:
  - Accept the event, apply it, persist MR, update CID to `new_cid`.
- If it does **not** match:
  - Treat as **conflict** (application-level `precondition_failed`):
    - Park event for manual review, or
    - Trigger a "retry with fresh MR" workflow.

This reproduces the **If-Match → 412** semantics of HTTP in a streaming context.

### 4.3 Observability

You can record:

- Conflict counts (how often `prev_cid` != current CID)
- Latency between `prev_cid` and `new_cid` becoming visible
- Per-aggregate history of `(rid, prev_cid, new_cid, offset)`

These become your streaming-level SLOs for "agent-safe writes".

---

## 5. Integrity and Digests

Streaming systems usually give you low-level integrity (checksums, CRCs). Dual-Native adds **semantic integrity**:

- Each event MAY include:
  - `digest` field (e.g., `sha-256` over payload or MR snapshot).
- For snapshot topics, the `cid` may double as a digest if computed over canonical MR.

Example:

```json
{
  "rid": "order:123",
  "prev_cid": "sha256-old...",
  "new_cid": "sha256-new...",
  "digest": "sha256-KJH98...",
  "payload": { ... }
}
```

Consumers can verify:

- That `digest(payload)` matches `digest` field.
- That the recomputed `cid(MR_after)` matches `new_cid`.

---

## 6. Catalog (DNC) on Streams

### 6.1 Catalog Topic

A simple pattern:

- Topic: `dual-native.catalog`
- Key: `rid`
- Value: catalog entry:

```json
{
  "rid": "order:123",
  "cid": "sha256-new...",
  "type": "order",
  "status": "confirmed",
  "updatedAt": "2025-01-16T10:00:00Z"
}
```

### 6.2 Client Workflow

1. **Initial sync:**
   - Read catalog topic from start or from a known checkpoint.
   - Build a map `rid → cid`.
   - For each `rid`, optionally pull MR from:
     - Snapshot topic, or
     - Database, or
     - HTTP MR endpoint.

2. **Incremental sync:**
   - Resume catalog subscription from last offset.
   - For each new message:
     - If `cid` changed vs cached:
       - Refresh MR for that `rid`.

This is the streaming variant of "poll `GET /catalog?since=` + conditional MR fetch."

---

## 7. Conformance Checklist (Streaming Binding)

A system using streaming can claim a **Dual-Native streaming binding** if:

- [ ] **RID** is clearly defined per resource / aggregate (e.g., message key or payload field).
- [ ] Each event affecting a resource includes:
  - [ ] `rid`
  - [ ] `new_cid` (CID after the event)
  - [ ] (Recommended) `prev_cid` (CID before the event)
- [ ] There is a well-defined **MR** (snapshot) that `cid` is computed from:
  - [ ] Either as a snapshot topic, or
  - [ ] As a reconstruction rule over the event log.
- [ ] Producers adhere to **safe write** semantics:
  - [ ] Events carry `prev_cid` and `new_cid`.
  - [ ] A validator component checks `prev_cid` against current CID before applying.
- [ ] A **DNC mechanism** exists:
  - [ ] Catalog topic or equivalent stream with `(rid, cid, updatedAt, type/status)`.
  - [ ] Consumers can use it to do incremental sync (from an offset or watermark).
- [ ] (Recommended) Events/snapshots include `digest` for payload integrity.

---

## 8. Relationship to HTTP / DB Bindings

This streaming binding often sits **between**:

- A **database binding** (SQL/NoSQL) where MR is persisted with CID and catalog tables, and
- An **HTTP binding** where MR is exposed to agents via REST/GraphQL/TCT.

Typical flow:

1. Database MR changes → new CID → event published on stream.
2. Streaming DNC broadcasts `(rid, prev_cid, new_cid)` to interested services/agents.
3. HTTP MR endpoints expose the same MR/CID for interactive agents.

All three bindings (DB, streaming, HTTP) share the same:

- RID, MR, CID model,
- Safe write semantics,
- DNC-based discovery and sync.

This gives you an end-to-end Dual-Native path from storage → streams → APIs → agents.
