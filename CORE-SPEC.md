# Dual-Native Pattern: Core Architecture and Requirements

**Document Version:** 1.0
**Last Updated:** November 2025
**Status:** Specification (Normative)
**Author:** Antun Jurkovikj
**License:** Creative Commons Attribution 4.0 International (CC BY 4.0)

---

## Abstract

This document defines the core architecture and normative requirements for the Dual-Native Pattern—an architectural approach where every resource provides both human-optimized (HR) and machine-optimized (MR) representations linked through bidirectional discovery and semantic equivalence guarantees.

The specification is protocol-agnostic and applies across domains including HTTP/web, databases, streaming systems, IoT, healthcare, and cloud storage.

---

## 1. Introduction

### 1.1 Scope

This specification defines:
- Terminology and concepts (HR, MR, RID, CID, DNC)
- Normative requirements for dual-native implementations
- Conformance levels and criteria
- Security and privacy considerations (summary)

This specification **does not** define:
- Protocol-specific implementations (see domain guides)
- Encoding formats (JSON, Avro, Protobuf—left to implementers)
- Authentication/authorization mechanisms (domain-specific)

### 1.2 Audience

This document is intended for:
- **Standards bodies** evaluating dual-native approaches
- **Deep-architecture reviewers** requiring formal definitions
- **Implementers** needing crisp conformance criteria
- **Compliance teams** assessing pattern requirements

**Note:** Informal dual-native patterns are already observable in production systems (e.g., Stripe, GitHub, Databricks, Wikidata). This specification formalizes these approaches with explicit requirements for bidirectional linking, semantic equivalence guarantees, and standardized discovery.

**For non-normative context**, see [Dual-Native Pattern: Whitepaper](WHITEPAPER.md).

**For implementation guidance**, see [Domain Implementation Guide](IMPLEMENTATION-GUIDE.md).

**For optional enhancements**, see [Optional Enhancements Guide](IMPLEMENTATION-GUIDE.md#8-optional-enhancements-advanced-patterns) covering integrity, immutability, evented sync, and provenance patterns. These are **not required** for conformance.

### 1.3 Relationship to Other Documents

**Normative References**:
- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119): Key words for use in RFCs to Indicate Requirement Levels

**Informative References**:
- [Dual-Native Pattern: Whitepaper](WHITEPAPER.md): Non-normative context and motivation
- [Domain Implementation Guide](IMPLEMENTATION-GUIDE.md): Non-normative implementation cookbook
- [Optional Enhancements Guide](IMPLEMENTATION-GUIDE.md#8-optional-enhancements-advanced-patterns): Non-normative guidance on integrity, immutability, evented sync, and provenance
- [TCT Internet-Draft](https://datatracker.ietf.org/doc/draft-jurkovikj-collab-tunnel/): HTTP-specific profile

### 1.4 Conventions and Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119].

**Key Terms**:
- **Resource**: An information object served to both humans and machines
- **Human Representation (HR)**: Human-optimized interface/representation
- **Machine Representation (MR)**: Machine-optimized interface/representation
- **Resource Identity (RID)**: Stable identifier shared by HR and MR
- **Content Identity (CID)**: Version-specific validator for change detection
- **Dual-Native Catalog (DNC)**: Registry enumerating all dual-native resources
- **Client**: System consuming HR or MR
- **Publisher**: System providing dual-native resources
- **Catalog Service**: System maintaining the DNC
- **Validator**: System verifying CID or semantic equivalence

**Terminology Note on "CID"**:
- **CID in this document** means "Content Identity" or "content validator"—a version-specific token for change detection
- **Distinct from HTTP ETag**: HTTP ETag is a validator for a specific HTTP representation; CID is a cross-domain concept mapped to native validators (HTTP ETag, FHIR meta.versionId, Kafka offset, etc.)
- **Distinct from IPFS CID**: IPFS uses "CID" for Content-Addressable Identifiers; this specification uses "CID" for content validators/change tokens
- **Not a new global standard**: CID is a defined term in this specification, mapped to existing domain-native validators (see Section 4.2 for mappings)

**For HTTP-Specific Implementations**: This specification uses "validator (ETag in HTTP)" in normative requirements; the catalog field is named `content_id` to avoid confusion with HTTP ETag scope.

---

## 2. Terminology and Concepts

### 2.1 Resource

A **resource** is an information object that:
- Has a stable identity (RID)
- Can be represented in multiple forms (HR, MR)
- Maintains semantic consistency across representations

**Examples**:
- HTTP: A blog article (HTML page + JSON endpoint), payment transaction (Stripe Dashboard + REST API)
- Database: A table (dashboard view + API query), data pipeline (Databricks Workspace UI + REST API)
- Streaming: A topic (monitoring UI + consumer API), knowledge graph entity (Wikidata page + SPARQL endpoint)
- Healthcare: A patient record (portal + FHIR endpoint)
- IoT: Network device (Cisco Meraki dashboard + Dashboard API), industrial asset (Digital Twin 3D view + MQTT API)

### 2.2 Human Representation (HR)

The **Human Representation (HR)** is an interface optimized for human consumption.

**Characteristics**:
- Includes presentation elements (formatting, navigation, context)
- Optimized for visual rendering and human cognition
- Examples: HTML pages, dashboards (Stripe Dashboard, Databricks Workspace), PDF documents, map viewers, GitHub web UI

**Normative**: See Section 4.3.1.

### 2.3 Machine Representation (MR)

The **Machine Representation (MR)** is an interface optimized for programmatic access.

**Characteristics**:
- Structured, machine-readable format (JSON, Avro, Protobuf, etc.)
- Consistent schema and minimal presentation overhead
- Optimized for AI, automation, and archival
- Examples: JSON APIs (Stripe REST API, GitHub API), Avro streams, FHIR resources, GeoJSON, SPARQL endpoints (Wikidata)

**Normative**: See Section 4.3.2.

### 2.4 Resource Identity (RID)

The **Resource Identity (RID)** is a stable identifier shared by both HR and MR.

**Properties**:
- **Stable**: Does not change when content updates
- **Shared**: Identical for HR and MR of the same resource
- **Domain-scoped**: Unique within a given domain or system
- **Separate from CID**: RID identifies the resource; CID validates content version

**Examples**:
- HTTP: `https://example.com/article/123` (URL as RID)
- Database: `database.schema.table_name`
- Streaming: `topic://cluster/event-stream`
- Healthcare: `Patient/12345` (FHIR resource ID)

**Normative**: See Section 4.1.

### 2.5 Content Identity (CID)

The **Content Identity (CID)** is a version-specific validator that changes when content updates.

**Properties**:
- **Versioned**: Changes with every content update
- **Lightweight**: Cheap to compute and compare
- **Precise**: Detects content changes reliably

**Purpose**: Enables zero-fetch optimization (skip downloads when CID matches cached version).

**Normative**: See Section 4.2.

#### 2.5.1 Validator Mapping Table (Informative)

CID is a cross-domain concept mapped to native validators in each protocol/domain:

| Domain | Native Validator | Example | Notes |
|--------|------------------|---------|-------|
| **HTTP** | ETag | `"sha256-abc123..."` | Strong ETag preferred; catalog uses `content_id` field |
| **OData** | ETag | `W/"datetime'2025-01-15T10:00:00'"` | Weak ETags acceptable |
| **FHIR** | meta.versionId | `"42"`, `meta.lastUpdated` | Version ID or timestamp |
| **Delta Lake** | snapshotId | `42` | Monotonic version number |
| **Apache Iceberg** | snapshot-id | `1234567890` | Snapshot identifier |
| **Kafka** | Offset vector | `{partition0: 12345, partition1: 12346}` | Per-partition offsets |
| **PostgreSQL** | LSN | `0/1A2B3C4D` | Log Sequence Number |
| **Wikidata** | lastrevid | `987654321` | Revision ID |
| **S3** | VersionId | `"3/L4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY"` | ETag has caveats for multipart uploads |
| **Git** | Commit SHA | `abc123def456...` | 40-character SHA-1 hash |
| **Avro** | Schema fingerprint | `rabin:abc123` | Canonical form fingerprint |
| **Protobuf** | Descriptor hash | `sha256:def456` | Hash of serialized descriptor |

**Important**: This specification does NOT define a new global validator standard. Implementations MUST use domain-native validators as listed above.

#### 2.5.2 CID Categories (Informative Guidance)

The following categories describe common CID usage patterns. These are **informative taxonomies**, not normative requirements:

**Resource CID** (one representation → one validator):
- Use case: Single resource with one validator
- Example: HTTP ETag for a specific representation
- When: Most simple resources (articles, customer records, files)

**Composite CID** (combine multiple inputs deterministically):
- Use case: Composite state from multiple components
- Example: GitHub PR state = `hash(head_sha + checks_summary_hash + updated_at)`
- When: Resources with dependent fragments, multi-component state
- Method: Merkle root, vector of CIDs, or deterministic hash of inputs

**Projection CID** (bind profile + parameters + source snapshot):
- Use case: Aggregates, parameterized queries, filtered views
- Example: Stripe MRR = `hash(profile=MRR_v2, month=2025-02, tz=UTC, snapshot=txn@v1234)`
- When: Same data with different filters/aggregations
- See Section 4.5.3 for details

**Stream CID** (position-based, not hash-based):
- Use case: Streaming data, event logs
- Example: Kafka offset vector, database replication position
- When: Continuous data streams, real-time systems
- Note: Position (offset/watermark), not payload hash

**Schema CID** (schema version fingerprint):
- Use case: Schema evolution tracking
- Example: Avro canonical fingerprint, Protobuf descriptor hash
- When: Data with evolving schemas
- Method: Canonical form + hash (Avro rabin, Protobuf SHA-256)

**Guidance**: Choose the category that fits your resource type. Composite and Projection CIDs are **descriptive taxonomies** built on existing validators, not new standards.

### 2.6 Dual-Native Catalog (DNC)

The **Dual-Native Catalog (DNC)** is a registry that enumerates all dual-native resources in a system.

**Contents**:
- List of all resources with RID, HR, MR references
- Current CID for each resource
- Metadata (profile, schema, ownership, sensitivity)

**Purpose**:
- **Discovery**: AI systems enumerate available resources
- **Freshness**: Check if cached content is stale
- **Governance**: Track dual-native coverage and compliance

**Normative**: See Section 4.6 and Section 5.

### 2.7 Actors

**Client**: A system consuming HR or MR
- Examples: Web browser, AI agent, crawler, mobile app

**Publisher**: A system providing dual-native resources
- Responsibilities: Maintain semantic equivalence, expose CIDs, publish DNC

**Catalog Service**: A system maintaining the DNC
- Responsibilities: Keep catalog up-to-date, provide discovery API

**Validator**: A system verifying CID or semantic equivalence
- Responsibilities: Test HR ↔ MR equivalence, alert on drift

---

## 3. Architectural Overview

### 3.1 Dual-Native Resource Model

```
┌─────────────────────────────────────────────────────────────┐
│                     Resource (RID)                          │
│                  Stable Identity                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────┐       ┌──────────────────────┐   │
│  │  Human Representation│◄──────►│Machine Representation│   │
│  │        (HR)          │  Link  │        (MR)          │   │
│  ├──────────────────────┤       ├──────────────────────┤   │
│  │ Format: HTML, UI     │       │ Format: JSON, Avro   │   │
│  │ CID: version X       │       │ CID: version X       │   │
│  └──────────────────────┘       └──────────────────────┘   │
│           ▲                               ▲                 │
│           │                               │                 │
│           └───────────────────────────────┘                 │
│              Semantic Equivalence                           │
└─────────────────────────────────────────────────────────────┘
                       │
                       ▼
          ┌────────────────────────┐
          │  Dual-Native Catalog   │
          │        (DNC)           │
          ├────────────────────────┤
          │ RID → HR, MR, CID      │
          │ Profile, Schema        │
          │ Metadata, Ownership    │
          └────────────────────────┘
```

### 3.2 Relationship Between HR, MR, RID, and CID

**Invariants**:
1. **One RID, Two Interfaces**: Every resource has exactly one RID and at least one HR and one MR
2. **Shared RID**: HR and MR for the same resource share the same RID
3. **Synchronized CID**: At any point in time, HR and MR have the same CID (semantic equivalence)
4. **CID Changes, RID Doesn't**: Content updates change CID but not RID

**Lifecycle Example**:
```
T0: Resource created
  RID: article/123
  CID: v1
  HR: HTML at /article/123
  MR: JSON at /article/123/llm/

T1: Content updated
  RID: article/123 (unchanged)
  CID: v2 (changed)
  HR: Updated HTML
  MR: Updated JSON (semantically equivalent to HR)

T2: Resource deleted/archived
  RID: article/123 (unchanged, but resource marked inactive)
  DNC: Entry removed or marked as retired
```

### 3.3 Catalog and Discovery Model

**Discovery Flow**:
```
1. Client queries DNC (e.g., GET /catalog?since=2025-01-01)
2. DNC returns list of resources with RID, HR, MR, CID
3. Client checks local cache: does cached CID match catalog CID?
   - If YES: Skip fetch (zero-fetch optimization)
   - If NO: Fetch MR from catalog's mr reference
4. Client navigates to HR or MR via bidirectional links
```

**Incremental Discovery**:
- Clients query `?since=timestamp` to get only updated resources
- Event-driven notifications (webhooks, WebSockets, FHIR Subscriptions)
- Change Data Capture (CDC) streams in databases

### 3.4 Validation and Synchronization Model

**CID-Based Validation**:
```
Client has cached MR with CID: v1

Step 1: Check catalog CID
  Catalog says: CID = v1 → Content unchanged, use cache (zero-fetch)
  Catalog says: CID = v2 → Content changed, fetch new MR

Step 2 (optional): Verify semantic equivalence
  Fetch both HR and MR
  Extract semantic fields (title, price, status)
  Assert: HR.title == MR.title
```

**Synchronization Guarantees**:
- **Strong Consistency** (RECOMMENDED for critical systems): HR and MR updated atomically
- **Eventual Consistency** (ACCEPTABLE for non-critical systems): HR and MR updated separately, converge within SLA
- **Validator Parity** (REQUIRED): Catalog CID MUST match live MR CID

---

## 4. Cross-Domain Core Requirements (Normative)

This section defines the **normative, protocol-agnostic requirements** for implementing the Dual-Native Pattern across any domain.

### 4.1 Resource Identity

**Requirement 4.1.1**: Every dual-native resource MUST define a stable Resource Identifier (RID) that:

1. **MUST** be shared identically by both HR and MR representations
2. **MUST** remain stable across content updates (content changes do not change RID)
3. **SHOULD** be domain-scoped and globally unique within that domain
4. **SHOULD** keep content identity (CID) separate from resource identity

**Example RIDs by Domain**:
- HTTP: `https://example.com/article` (URL as RID)
- Database: `database.schema.table_name`
- Streaming: `topic://cluster/event-stream`
- Healthcare: `Patient/12345` (FHIR resource ID)
- Lakehouse: `s3://bucket/dataset/table`

**Rationale**: Stable RIDs enable bidirectional linking and discovery even when content versions change.

#### 4.1.1 RID Properties

**Stability**: RID MUST NOT change when:
- Content is updated (text, images, data)
- CID changes (versioning)
- HR or MR format changes (HTML → Markdown, JSON → Avro)

**RID MAY change when**:
- Resource is moved/renamed (subject to HTTP 301/308 redirects or equivalent)
- System undergoes major migration (with documented transition plan)

#### 4.1.2 RID Stability and Uniqueness Requirements

**Uniqueness**: Within a domain, each RID MUST uniquely identify a single resource.

**Collision Prevention**: Publishers MUST ensure:
- No two resources share the same RID
- Deleted resources' RIDs are not immediately reused (use tombstones or retirement periods)

### 4.2 Content Identity

**Requirement 4.2.1**: Every dual-native resource MUST expose a Content Identity (CID) validator that:

1. **MUST** change when content updates
2. **SHOULD** be cheap to compute and compare (avoid expensive hashing if native validators exist)
3. **MUST** be consistent across HR and MR for the same resource at the same point in time

**Domain-Specific Validators**:
- **HTTP/Web**: Validator (ETag in HTTP), preferably strong ETags with SHA-256 content hash
- **Databases**: MVCC version, LSN (Log Sequence Number), SCN (System Change Number)
- **Streaming**: Offset vector, high-water mark
- **Lakehouse**: Snapshot ID, table version
- **Healthcare**: `meta.versionId`, `meta.lastUpdated`
- **Geospatial**: `updated` timestamp + `checksum:multihash`
- **ERP**: Concurrency tokens, entity validators

#### 4.2.1 CID Properties

**Precision**: CID MUST detect content changes reliably. False negatives (missed changes) are unacceptable.

**Efficiency**: CID computation SHOULD be cheap:
- **Prefer**: Native metadata (LSN, offset, version)
- **Avoid**: Hashing entire payloads (unless domain has no native validator)

#### 4.2.2 CID Derivation and Versioning

**Derivation Strategies**:

1. **Metadata-Based** (RECOMMENDED):
   - Use database LSN, Kafka offset, FHIR versionId
   - Cheap, precise, domain-native

2. **Hash-Based**:
   - Compute hash of canonical content (RFC 8785 for JSON)
   - Use for domains without native validators (e.g., file systems)

3. **Timestamp-Based** (ACCEPTABLE with caveats):
   - Use `lastModified` timestamp
   - Risk: Clock skew, sub-second updates may be missed

**CID Format**: No standard format required. Examples:
- `"sha256-abc123..."` (validator in HTTP using ETag format)
- `"lsn:0/1A2B3C4D"` (PostgreSQL LSN)
- `"offset:12345"` (Kafka)
- `"v42"` (simple version counter)

**Important: HTTP CID Semantics**:
- **Do NOT reuse ETag across HR and MR endpoints** for conditional GET purposes (HR and MR are distinct resources with different representations)
- **Instead**: Expose a shared `content_id` field in the DNC/catalog for parity checking between HR and MR
- ETags on individual endpoints (HR ETag, MR ETag) can differ as they reflect different serializations
- The catalog's `content_id` field represents the logical content version shared by both representations

#### 4.2.3 Validators, Conditional Reads, and Safe Writes (Cross-Domain)

**CID as the Canonical Validator**:
- CID MUST serve as the canonical validator for MR representations
- Implementations MUST support conditional read and safe write semantics using CID

**Conditional Read Behavior**:

**Requirement 4.2.3.1**: When a client requests MR with a validator precondition:
1. If the client's validator equals the current CID → Return "Not Modified" result (transport-specific) with no body
2. If the client's validator differs from the current CID → Return full MR representation with current CID

**Transport-Specific Mappings**:
- HTTP: 304 Not Modified response with ETag header
- gRPC: Empty response with metadata carrying current version
- MQTT: Omit message body, publish version-only message
- GraphQL: Null data with extensions carrying freshness metadata

**Safe Write Behavior**:

**Requirement 4.2.3.2**: Mutations that modify dual-native resources MUST accept a validator precondition:
1. Client submits mutation with expected CID (validator)
2. If current CID matches expected CID → Apply mutation, return new CID (and optionally new MR or delta)
3. If current CID differs from expected CID → Return "Precondition Failed" result; no mutation applied

**Transport-Specific Mappings**:
- HTTP: `If-Match` header; 412 Precondition Failed on mismatch
- gRPC: version field in request metadata; FAILED_PRECONDITION status on mismatch
- Database: `WHERE rowversion = :cid` clause; zero rows affected on mismatch
- Streaming: expected offset/position in produce request; rejection on mismatch

**Rationale**: Conditional reads enable zero-fetch optimization (skip unchanged content). Safe writes prevent race conditions where agents overwrite human edits based on stale state.

**Example Flow (HTTP)**:
```
1. Agent reads MR → receives CID "abc123"
2. Human edits HR → CID changes to "xyz789"
3. Agent attempts write with If-Match: "abc123"
4. Server returns 412 Precondition Failed
5. Agent re-reads MR with new CID, retries write
```

**Result**: Zero data loss. Agent's update is based on current content, not stale state.

### 4.3 Dual Interfaces

**Requirement 4.3.1**: For each RID, implementations MUST provide:

1. **HR (Human Representation)**: An interface optimized for human consumption
2. **MR (Machine Representation)**: An interface optimized for programmatic access

**Requirement 4.3.2**: HR and MR for the same RID MUST represent semantically equivalent content at any given point in time.

#### 4.3.1 HR Requirements

**HR MUST**:
- Be accessible and usable by human users
- Present the same core information as MR (semantic equivalence)

**HR SHOULD**:
- Optimize for human cognition (visual formatting, navigation, context)
- Include presentation elements (images, styling, interactive widgets)

**Examples**: HTML pages, dashboards, PDF documents, map viewers

#### 4.3.2 MR Requirements

**MR MUST**:
- Be machine-readable and parseable without human intervention
- Present the same core information as HR (semantic equivalence)
- Expose CID for validation

**MR SHOULD**:
- Optimize for AI/automation (structured data, consistent schema, minimal overhead)
- Document its schema/profile and intended AI/automation use
- Use domain-standard formats (JSON, Avro, Protobuf, FHIR, GeoJSON)

**Examples**: JSON APIs, Avro streams, FHIR resources, GeoJSON

#### 4.3.3 MR Format Profiles

**Requirement 4.3.3**: A resource MAY expose multiple MR profiles, each optimized for different consumption patterns.

**Common MR Profile Categories**:

1. **Structured MR**:
   - Machine-readable structured data with schema
   - Formats: JSON, Avro, Protobuf, FHIR, GeoJSON, Parquet
   - Use case: API consumption, data pipelines, schema evolution
   - Example: Blog article as JSON with title, content, author, metadata

2. **Text MR**:
   - Text-based representation optimized for language models and text processing
   - Formats: Markdown, plain text, reStructuredText
   - Use case: LLM consumption, RAG (Retrieval-Augmented Generation), content analysis
   - Example: Blog article as Markdown preserving headings, lists, emphasis

3. **Binary MR**:
   - Compact binary serialization for high-throughput systems
   - Formats: CBOR, MessagePack, Protocol Buffers binary, Avro binary
   - Use case: Streaming, IoT, inter-service communication
   - Example: Sensor reading as CBOR for efficient transmission

**Semantic Equivalence Across Profiles**:
- All MR profiles for a given RID at a given CID MUST represent semantically equivalent content
- The equivalence scope (Section 4.5) MUST be consistent across all profiles
- DNC entries SHOULD advertise available MR profiles for each RID (see Section 5)

**Rationale**: Different agents have different optimization needs (structure vs tokens vs bandwidth). Multiple MR profiles allow publishers to serve diverse consumption patterns while maintaining semantic equivalence.

### 4.4 Bidirectional Linking

**Requirement 4.4.1**: Implementations MUST expose resolvable HR ↔ MR links such that:

1. **MUST** enable navigation from HR to MR and from MR to HR
2. **MUST** persist links in native metadata or registry (not only in rendered UI)
3. **SHOULD** ensure links survive renames/moves via RID indirection
4. **SHOULD** use domain-native linking mechanisms

**Domain-Specific Linking Mechanisms**:
- **HTTP**: Link headers with `rel="alternate"` for HR ↔ MR bidirectional linking (reserve `rel="canonical"` for preferred URI semantics, not dual-native linking)
- **Databases**: Schema metadata (extended properties, tags, comments)
- **Streaming**: Topic metadata, schema registry annotations
- **Lakehouse**: Table properties, catalog metadata
- **Healthcare**: FHIR Bundle.link, resource metadata
- **Geospatial**: OGC links, STAC assets

**Rationale**: Bidirectional linking prevents HR and MR from diverging and enables automatic discovery.

#### 4.4.1 Linking from HR to MR

**HR MUST** include reference to corresponding MR via:
- Native metadata (HTTP Link header, database schema comment)
- Embedded in rendered content (HTML `<link>` tag, dashboard config)
- Registry lookup (DNC query by RID)

**Example (HTTP)**:
```html
<head>
  <link rel="alternate" type="application/json"
        href="https://example.com/article/123/llm/" />
</head>
```

#### 4.4.2 Linking from MR to HR

**MR MUST** include reference to corresponding HR via:
- Response header (HTTP `Link` header)
- Embedded field in payload (e.g., `canonical_url` in JSON)
- Registry lookup (DNC query by RID)

**Example (JSON)**:
```json
{
  "canonical_url": "https://example.com/article/123",
  "title": "Article Title",
  "content": "..."
}
```

### 4.5 Semantic Equivalence

**Requirement 4.5.1**: HR and MR for the same RID MUST represent semantically equivalent content at any given point in time.

**Definition**: Two representations are **semantically equivalent** if they agree on a **declared equivalence scope** (the set of claims that must match) when derived from the same authoritative source at the same CID/snapshot.

**Equivalence Scope**: The declared set of claims (fields, computations, or aggregates) that MUST match between HR and MR at a given point in time. This scope is resource-specific and MUST be documented.

**Examples of Equivalence Scopes**:
- **Article**: `{title, content, author, published_date, modified_date}`
- **GitHub PR**: `{state, head_sha, base_sha, required_checks_result}`
- **Stripe MRR Report**: `{mrr_value, month, timezone, included_statuses, computation_definition}`
- **IoT Device**: `{device_id, status, firmware_version, last_seen_timestamp}`

**Semantic Equivalence Does NOT Require**:
- Identical formatting (HTML vs. JSON)
- Identical field names (HR may use `lastModified`, MR may use `modified_at`)
- Identical field sets (HR and MR may show different subsets or aggregation levels)
- Byte-for-byte identity (representations may differ as long as claims match)

#### 4.5.1 Single Source of Truth

**Requirement**: HR and MR MUST derive from a **shared authoritative domain source** at the same logical state (CID/snapshot).

**Clarifications**:
- **NOT** one physical database: Read replicas, materialized views, event-sourced read models, and caches are acceptable if they derive from canonical data and track CIDs
- **IS** shared authoritative state: Both HR and MR reflect the same version of the underlying data at a point in time
- **Examples**:
  - PostgreSQL primary with read replicas (both at same LSN)
  - Kafka topic with schema registry (both at same offset)
  - Event-sourced system with projections (both at same event sequence number)

**Anti-Pattern**: Separate databases updated independently without synchronization tracking.

#### 4.5.2 Subsets, Aggregation, and Derived Resources

**Requirement**: HR and MR MAY present **different subsets, aggregation levels, and interaction modes** as long as they:
1. Derive from the same authoritative source
2. Preserve equivalence within the declared equivalence scope
3. Track the same CID/snapshot

**First-Class Treatment of Aggregates**:
- Aggregates (reports, summaries, dashboards) are **derived resources** with their own RID and CID
- CID for aggregates binds: profile/schema version + filter parameters + source data snapshot
- Examples:
  - Stripe MRR Report: RID = `report/mrr/2025-02`, CID = `hash(profile=MRR_v2, month=2025-02, tz=UTC, input_snapshot=txn_table@v1234)`
  - GitHub PR Summary: RID = `pr/42`, CID = `hash(head_sha + checks_summary_hash)`

**Projection Rules MUST Be Documented**:
- Filter parameters (time zones, status filters, rounding rules)
- Computation definitions (how aggregates are derived)
- Allowed transformations (field mappings, normalization)
- Tolerance rules (e.g., currency rounding to 2 decimals)

**Ambiguity in projection rules is the primary cause of drift.**

#### 4.5.3 Projection CID for Parametric Views (Informative)

**Note**: "Projection CID" is an **informative category** (non-normative guidance), not a new standard. It describes a pattern for binding profile + parameters + source snapshot using existing domain validators.

**Definition**: A **Projection CID** identifies a specific view/query over authoritative data, binding:
- Profile or schema version
- Query parameters (filters, time window, pagination)
- Source data snapshot (LSN, offset, version)

**Format**: `hash(profile + parameters + source_snapshot)` or use domain-native position (offset, LSN)

**Examples**:
```
# Stripe MRR for February 2025
projection_cid: "sha256(profile=MRR_v2, month=2025-02, tz=UTC, snapshot=txn@v1234)"

# GitHub PR checks at specific commit
projection_cid: "sha256(pr=42, head_sha=abc123, checks_version=v3)"

# Kafka consumer group offset
projection_cid: "offset:12345"
```

**Use Cases**:
- Queries and aggregates (different slices of same data)
- Streaming positions (offset/watermark, not payload hash)
- Parametric reports (same data, different filters)

**Validation**: Clients can verify HR and MR show the same projection by comparing projection CIDs.

#### 4.5.4 Validation Patterns for Semantic Equivalence

**Field Parity** (for simple resources):
- Shared core fields MUST match across HR and MR
- Document which fields are in-scope vs. out-of-scope (UI-only, metadata-only)
- Test: Extract declared scope fields from HR and MR, compare values

**Aggregate Parity** (for derived resources):
- Documented function over MR records MUST reproduce HR aggregate at same snapshot
- Specify: time zone, status filters, rounding rules, computation definition
- Test: Recompute aggregate from MR with identical parameters, compare to HR value and CID

**Composite CID** (for multi-component state):
- Combine identifiers of inputs that define state
- Example: `hash(head_sha + checks_summary_hash)` for GitHub PR
- Test: Verify both HR and MR expose the same composite CID

**Tolerance Rules** (for allowed differences):
- Declare permitted variances (rounding, formatting, localization)
- Example: Currency values may differ in display (€1,234.56 vs. 1234.56 EUR) but must be numerically equivalent
- Test: Normalize values per tolerance rules before comparison

**Requirement**: Implementations MUST define at least one validation pattern for each resource type and execute it periodically (at least on major updates).

### 4.6 Discovery via DNC

**Requirement 4.6.1**: Implementations MUST publish a **Dual-Native Catalog (DNC)** that:

1. **MUST** enumerate all dual-native resources
2. **MUST** include required fields (see Section 4.6.2)
3. **SHOULD** support incremental discovery (e.g., `?since=timestamp`)
4. **SHOULD** be accessible to authorized clients

**Rationale**: Centralized discovery enables AI systems to find dual-native resources without prior knowledge.

#### 4.6.1 DNC Responsibilities

**Catalog Service MUST**:
- Maintain current list of all dual-native resources
- Update CIDs when resources change
- Provide query API for discovery

**Catalog Service SHOULD**:
- Support pagination for large catalogs
- Provide filtering by domain, owner, sensitivity
- Emit change notifications (webhooks, events)

#### 4.6.2 DNC Record Requirements

**Minimal Required Fields**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `rid` | String | REQUIRED | Resource identifier (stable) |
| `hr` | String/Object | REQUIRED | Human representation reference |
| `mr` | String/Object | REQUIRED | Machine representation reference |
| `content_id` | String | REQUIRED | Content identity (current validator) |
| `updatedAt` | Timestamp | REQUIRED | Last modification timestamp |
| `profile` | String | RECOMMENDED | Implementation profile/version |

**Optional Governance Fields**:

| Field | Type | Optional | Description |
|-------|------|----------|-------------|
| `schema` | String | OPTIONAL | MR schema reference (OpenAPI, JSON Schema, Avro) |
| `auth` | String/Object | OPTIONAL | Authentication requirements |
| `owner` | String | OPTIONAL | Resource owner/team |
| `sensitivity` | String | OPTIONAL | Data classification (public, internal, confidential) |
| `retired` | Boolean | OPTIONAL | Marks resource as deleted/archived |

### 4.7 Synchronization and Validation

**Requirement 4.7.1**: Implementations MUST expose validators that enable zero-fetch optimization:

1. **MUST** provide a Content Identity (CID) validator for each resource
2. **SHOULD** use cheap, precise validators appropriate to the domain
3. **MUST** ensure catalog CID matches live MR validator (validator parity)
4. **SHOULD** enable clients to skip fetches when CID matches cached version

#### 4.7.1 CID-Based Validation

**Zero-Fetch Flow**:
```
Client has cached MR with CID_old

Step 1: Client queries DNC for resource
Step 2: DNC returns CID_current
Step 3: Client compares CID_old == CID_current
  - If TRUE: Use cached MR (zero-fetch)
  - If FALSE: Fetch new MR from catalog's mr reference
```

**Validator Parity** (REQUIRED):
```
Catalog CID MUST equal live MR CID at all times

Bad:
  Catalog CID: v1
  Live MR CID: v2
  → Zero-fetch broken (clients use stale cache)

Good:
  Catalog CID: v2
  Live MR CID: v2
  → Zero-fetch works
```

#### 4.7.2 Consistency Guarantees (Eventual vs Strong)

**Strong Consistency** (RECOMMENDED for critical systems):
- HR and MR updated in same transaction
- CID updated atomically with content
- Catalog updated atomically with content

**Eventual Consistency** (ACCEPTABLE for non-critical systems):
- HR and MR may be briefly out of sync
- CID updated within SLA (e.g., <1 minute)
- Catalog updated within SLA (e.g., <5 minutes)

**Requirement**: Implementations MUST document their consistency model.

#### 4.7.3 Recommended Validator Behaviors

**Publishers SHOULD**:
- Pre-compute CIDs at write time (store in database, not compute on-demand)
- Invalidate caches atomically when content changes
- Monitor validator parity (alert if catalog CID ≠ live MR CID)

**Clients SHOULD**:
- Respect validator-based caching (in HTTP: honor 304 Not Modified responses)
- Implement backoff when CID changes rapidly (avoid thundering herd)
- Log zero-fetch rates for observability

### 4.8 Structural Mutations (The Write Path)

**Requirement 4.8.1**: In dual-native systems that support write operations, agents MUST NOT modify the Human Representation (HR) directly (e.g., scraping and POSTing HTML), as this risks corruption and race conditions.

Instead, agents MUST act on the **Machine Representation (MR)** using **Structural Mutations**—atomic operations that modify system state in a way that guarantees integrity in both representations.

#### 4.8.1 Mutation Types

Structural Mutations are domain-specific atomic operations. Common patterns include:

1. **Append**: Add a discrete unit of content to the end of a resource
   - **CMS**: Appending a block to a document
   - **Fintech**: Creating a new transaction
   - **IoT**: Logging a device event

2. **Insert**: Inject a unit at a specific logical position
   - **CMS**: Inserting a heading at a specific index in the block structure
   - **Database**: Inserting a row at a specific sequence
   - **Workflow**: Adding a step at a specific stage

3. **Update**: Modify the data of a specific unit identified by ID
   - **CMS**: Updating block content by block ID
   - **Fintech**: Updating charge metadata
   - **IoT**: Setting device state

4. **Delete**: Remove a specific unit
   - **CMS**: Removing a block by ID
   - **Database**: Deleting a record
   - **Fintech**: Voiding a charge

#### 4.8.1.1 Addressable Content Units and Atomic Operations

**Requirement 4.8.1.1**: For structured resources, implementations SHOULD expose an addressing scheme for sub-units and support atomic operations at that address.

**Addressable Content Units**:
- Content MAY be composed of discrete, addressable units (blocks, records, events, steps)
- Each unit SHOULD be uniquely identifiable within the resource (by index, path, or ID)

**Addressing Schemes by Domain**:
- **CMS/Documents**: Block index (0-based), block ID (UUID), JSON path ($.blocks[2])
- **Databases**: Primary key, row identifier, partition key + sort key
- **Streaming**: Event offset, timestamp range, message ID
- **Healthcare**: FHIR resource path (e.g., `Patient.name[0]`), section identifier
- **IoT**: Component path (e.g., `device.sensors.temperature`), telemetry stream ID

**Atomic Operations at Address**:
Implementations SHOULD support:
- `append`: Add unit at end (equivalent to insert at index = length)
- `prepend`: Add unit at beginning (equivalent to insert at index = 0)
- `insert(address)`: Add unit at specific position (index, path, or ID)
- `replace(address)`: Replace existing unit at address
- `delete(address)`: Remove unit at address
- `patch(address)`: Partial update of unit at address (RFC 6902 JSON Patch or domain equivalent)

**Atomicity Requirements**:
- Each operation MUST be all-or-nothing for the addressed units
- Partial application is NOT acceptable (e.g., inserting 3 blocks must either insert all 3 or none)
- Operations MUST preserve structural integrity (e.g., valid block order, referential integrity)

**Rationale**: Addressable units enable safe, granular edits without full resource replacement. Atomic operations prevent partial corruption during concurrent updates.

**Example (CMS)**:
```
# Insert heading at index 2
POST /article/123/blocks
{
  "operation": "insert",
  "index": 2,
  "block": {"type": "heading", "level": 2, "content": "Key Findings"}
}
```

**Example (Database)**:
```sql
-- Insert row at specific sequence
INSERT INTO tasks (id, sequence, title, status)
VALUES (gen_uuid(), 5, 'Review PR', 'pending')
WHERE (SELECT COUNT(*) FROM tasks WHERE sequence >= 5) = 0; -- Atomic check
```

#### 4.8.2 Projection Responsibility

**Requirement 4.8.2**: The system is responsible for projecting MR mutations back into the HR.

- **MUST**: Changes to the MR MUST be immediately reflected in the HR (or within documented consistency bounds)
- **MUST**: The projection MUST preserve semantic equivalence
- **MUST**: The system MUST recompute the CID after mutations

**Examples**:
- **CMS**: Serializing JSON blocks into HTML with Gutenberg comments
- **Fintech**: Updating dashboard UI when a charge is created via API
- **IoT**: Moving a physical dial when an API changes temperature setpoint

#### 4.8.3 Optimistic Concurrency

**Requirement 4.8.3**: Write operations SHOULD support optimistic concurrency control to prevent lost updates.

**Recommended Pattern**:
1. Client reads MR and obtains current CID
2. Client computes desired mutation
3. Client submits mutation with `If-Match: <CID>` (or domain equivalent)
4. System verifies CID matches current state
5. If match: apply mutation, recompute CID, return new CID
6. If mismatch: reject with 412 Precondition Failed (or domain equivalent)

**Domain Examples**:
- **HTTP**: `If-Match` header with ETag
- **Database**: WHERE clauses on version columns
- **Fintech**: Idempotency keys (Stripe)
- **Healthcare**: FHIR conditional updates

#### 4.8.4 Drift Prevention

**Requirement 4.8.4**: Systems MUST prevent **representation drift**—when HR and MR fall out of semantic equivalence.

**Drift occurs when**:
- A human edits the HR in a way the MR cannot represent
- Asynchronous updates cause temporary inconsistency
- System bugs or race conditions cause desynchronization

**Prevention Strategies**:
1. **Structured Editors**: Use editors that work on the MR (e.g., Gutenberg block editor), preventing invalid HR edits
2. **Event Sourcing**: Make MR the source of truth; HR is always derived
3. **Validation**: Reject HR edits that cannot be represented in MR
4. **Monitoring**: Alert when CID(HR) ≠ CID(MR)

**Consistency Guarantee**: Dual-native systems MUST ensure that any change to the HR triggers a re-computation of the MR and its CID. The two representations must remain eventually consistent (or strictly consistent, depending on domain requirements).

---

## 5. DNC Data Model

### 5.1 Minimal DNC Record Fields

A conforming DNC record MUST include:

```json
{
  "rid": "https://example.com/article/123",
  "hr": "https://example.com/article/123",
  "mr": "https://example.com/article/123/llm/",
  "content_id": "sha256-abc123...",
  "updatedAt": "2025-01-16T10:00:00Z",
  "profile": "tct-1"
}
```

**Field Descriptions**:

- **`rid`** (REQUIRED): Resource identifier. MUST be stable across content updates.
- **`hr`** (REQUIRED): Human representation reference. MAY be URL, file path, or domain-specific identifier.
- **`mr`** (REQUIRED): Machine representation reference. MAY be URL, API endpoint, or domain-specific identifier.
- **`content_id`** (REQUIRED): Content identity validator. MUST change when content updates.
- **`updatedAt`** (REQUIRED): Last modification timestamp (ISO 8601).
- **`profile`** (RECOMMENDED): Implementation profile/version (e.g., `"tct-1"`, `"dual-native-core-1.0"`).

### 5.2 DNC Operations (Register, Update, Retire)

**Register** (add new resource):
```
POST /catalog
{
  "rid": "article/456",
  "hr": "/article/456",
  "mr": "/article/456/llm/",
  "content_id": "v1",
  "updatedAt": "2025-01-16T10:00:00Z"
}
```

**Update** (content changed):
```
PUT /catalog/article/456
{
  "content_id": "v2",
  "updatedAt": "2025-01-16T11:00:00Z"
}
```

**Retire** (delete/archive):
```
DELETE /catalog/article/456

OR

PUT /catalog/article/456
{
  "retired": true,
  "retiredAt": "2025-01-16T12:00:00Z"
}
```

### 5.3 Reference JSON Encoding

**Example Full Record**:
```json
{
  "rid": "database.public.customers",
  "hr": "https://dashboard.example.com/tables/customers",
  "mr": "https://api.example.com/v1/customers",
  "content_id": "lsn:0/1A2B3C4D",
  "updatedAt": "2025-01-16T10:00:00Z",
  "profile": "dual-native-core-1.0",
  "schema": "https://api.example.com/schemas/customers.json",
  "auth": "oauth2",
  "owner": "data-platform-team",
  "sensitivity": "internal"
}
```

### 5.4 Alternative Encodings (Non-Normative Guidance)

While JSON is the reference encoding, implementations MAY use:

- **SQL Table** (Databases): See Section 4.6 of Implementation Guide
- **Avro Schema** (Streaming): Embed DNC in schema registry
- **Protocol Buffers**: For high-performance systems
- **YAML**: For human-editable configuration

**Requirement**: All encodings MUST preserve required fields from Section 5.1.

---

## 6. Conformance

### 6.1 Conformance Classes

An implementation MAY conform as one or more of:

1. **Resource Publisher**: Provides dual-native resources (HR + MR)
2. **Catalog Service**: Maintains and serves DNC
3. **Client**: Consumes HR or MR with zero-fetch optimization
4. **Validator**: Tests HR ↔ MR semantic equivalence

#### 6.1.1 Resource Publishers

A **conforming resource publisher** MUST:
- Provide both HR and MR for each resource (Section 4.3)
- Ensure semantic equivalence (Section 4.5)
- Expose CID validators (Section 4.2)
- Maintain bidirectional links (Section 4.4)

A conforming publisher SHOULD:
- Register resources in a DNC (Section 4.6)
- Use native validators (LSN, offset) instead of expensive hashing
- Monitor validator parity

#### 6.1.2 Catalog Services

A **conforming catalog service** MUST:
- Enumerate all dual-native resources (Section 4.6)
- Include required fields (rid, hr, mr, cid, updatedAt) (Section 5.1)
- Maintain validator parity (catalog CID == live MR CID) (Section 4.7.1)

A conforming catalog service SHOULD:
- Support incremental discovery (`?since=timestamp`)
- Provide change notifications (webhooks, events)
- Expose governance metadata (schema, auth, owner, sensitivity)

#### 6.1.3 Clients

A **conforming client** MUST:
- Respect CID-based caching (zero-fetch optimization) (Section 4.7.1)
- Follow bidirectional links when provided (Section 4.4)

A conforming client SHOULD:
- Query DNC for discovery
- Log zero-fetch rates for observability
- Implement backoff when CID changes rapidly

#### 6.1.4 Validators

A **conforming validator** MUST:
- Test HR ↔ MR semantic equivalence (Section 4.5)
- Alert on validator parity failures (catalog CID ≠ live MR CID)

A conforming validator SHOULD:
- Run automated tests on every content update
- Compare core semantic fields (title, price, status, timestamps)
- Provide compliance reports for governance teams

### 6.2 Conformance Levels (0–4)

Implementations typically evolve through **maturity levels**:

#### 6.2.1 Level 0 – HR-Only Systems (Non-Conformant)

**Characteristics**:
- Only HR exists (HTML pages, dashboards)
- No MR, no dual-native design
- AI systems must parse HR

**Conformance**: Not conformant.

#### 6.2.2 Level 1 – HR + MR, One-Way Link

**Characteristics**:
- Both HR and MR exist
- One-way link (HR → MR **OR** MR → HR, but not both)
- No DNC, discovery requires external knowledge

**Conformance**: **Partially conformant** (violates bidirectional linking requirement).

**Recommendation**: Add reverse link to achieve Level 2.

#### 6.2.3 Level 2 – HR + MR, Bidirectional Links

**Characteristics**:
- Both HR and MR exist
- Bidirectional linking (HR ↔ MR)
- Discovery enabled via links
- No CID, no zero-fetch optimization

**Conformance**: **Conformant** (minimal requirements met).

**Recommendation**: Add CID to achieve Level 3.

#### 6.2.4 Level 3 – Optimized (Zero-Fetch)

**Characteristics**:
- CID validators exposed
- Zero-fetch optimization enabled
- Semantic equivalence enforced via automated tests
- Monitoring for drift
- Dual-Native Catalog (DNC) published for efficient discovery

**Conformance**: **Conformant** (recommended level for read-only systems).

**Recommendation**: Add write capabilities to achieve Level 4.

#### 6.2.5 Level 4 – Agentic (Read/Write)

**Characteristics**:
- All Level 3 capabilities (bidirectional linking, CID, zero-fetch, DNC)
- **Structural Mutations**: Agents can modify system state via MR without corrupting HR
- **Atomic Operations**: Insert, append, update, delete operations on MR
- **Optimistic Concurrency**: Write operations support `If-Match` or equivalent to prevent lost updates
- **Drift Prevention**: System ensures HR ↔ MR consistency after writes
- **Actuator Access**: Agents can safely read AND write to the system

**Conformance**: **Fully conformant** (agentic/best-in-class).

**Use Cases**:
- **CMS**: AI agents editing content blocks (e.g., WordPress with DNI plugin)
- **Fintech**: AI agents creating/updating transactions (e.g., Stripe API)
- **IoT**: AI agents controlling device state (e.g., thermostat APIs)
- **Version Control**: AI agents committing code changes (e.g., GitHub API)

**Recommendation**: This is the target state for systems enabling autonomous AI agents.

#### 6.2.6 Informal Platform Mapping (Non-Normative)

Many production platforms exhibit characteristics of dual-native design, though without the formal guarantees defined in this specification:

| Platform | Approximate Level | Characteristics |
|----------|-------------------|-----------------|
| **Stripe** | ~Level 4 | Dashboard (HR) + REST API (MR), full CRUD via API, idempotent writes, read/write symmetry |
| **GitHub** | ~Level 4 | Web UI (HR) + REST/GraphQL APIs (MR), full repository/issue/PR write operations via API |
| **WordPress (DNI)** | ~Level 4 | Editor (HR) + JSON MR, block insertion API with `If-Match`, CID validation, structural mutations |
| **Wikidata** | ~Level 3 | Web pages (HR) + SPARQL endpoint (MR), revision IDs act as CID, read-optimized |
| **Databricks** | ~Level 2-4 | Workspace UI (HR) + REST API (MR), bidirectional linking, write APIs for jobs/clusters |
| **Cisco Meraki** | ~Level 4 | Dashboard (HR) + Dashboard API (MR), full device configuration via API |
| **Data.gov** | ~Level 3 | Web portal (HR) + CKAN API (MR), metadata versioning, read-focused |

**Note:** These platforms typically lack explicit semantic equivalence guarantees or formal conformance declarations. Level 4 platforms demonstrate write capabilities but may not implement all drift prevention mechanisms. This specification provides a framework to make such implementations fully conformant.

#### 6.2.7 Legacy Encapsulation as Normative Migration Pattern

**Requirement 6.2.7.1**: Legacy or unstructured resources SHOULD be represented as a single opaque unit in MR to enable incremental migration without requiring full content restructuring.

**Pattern**: Legacy Encapsulation
- Wrap legacy/unstructured content (HTML, plain text, binary) as a single "freeform" or "legacy" unit in MR
- Allow additive structured units around the legacy payload
- Enable incremental migration: new content uses structured MR, old content remains encapsulated

**Examples by Domain**:

**CMS (WordPress Classic Editor)**:
```json
{
  "blocks": [
    {
      "type": "freeform",
      "content": "<p>Legacy HTML content...</p><!-- Classic editor output -->"
    },
    {
      "type": "heading",
      "level": 2,
      "content": "New structured heading"
    }
  ]
}
```
- Old posts: Single `freeform` block containing HTML
- New content: Structured blocks can be appended without touching legacy content

**Databases (Unstructured Text Columns)**:
```json
{
  "id": "12345",
  "legacy_notes": "Unstructured text from old system...",
  "structured_metadata": {
    "status": "active",
    "tags": ["migration", "reviewed"]
  }
}
```

**File Systems (Binary Files)**:
```json
{
  "file_path": "/docs/legacy-report.pdf",
  "content_type": "application/pdf",
  "legacy_binary": true,
  "extracted_text": "OCR or text extraction for MR...",
  "structured_annotations": [
    {"page": 3, "type": "signature", "verified": true}
  ]
}
```

**IoT (Proprietary Binary Protocols)**:
```json
{
  "device_id": "sensor-42",
  "raw_payload": "base64-encoded-proprietary-format...",
  "decoded_fields": {
    "temperature": 22.5,
    "humidity": 45
  }
}
```

**Benefits**:
- **Zero Content Loss**: Legacy content preserved exactly as-is
- **Incremental Migration**: New features use structured MR without rewriting all old content
- **Safe Coexistence**: Structured and legacy units can exist in same resource
- **Agent-Safe**: Agents can append/prepend structured units without touching legacy payload

**Anti-Pattern**: Requiring full content restructuring before enabling dual-native (blocks migration burden)

**Rationale**: Legacy encapsulation enables pragmatic adoption of dual-native patterns in systems with large existing content bases (databases, CMS, file systems, IoT).

### 6.3 Declaring Conformance

Implementations SHOULD declare conformance in machine-readable metadata:

**HTTP Example**:
```http
GET /article/123/llm/ HTTP/1.1

HTTP/1.1 200 OK
Content-Type: application/json; profile="dual-native-core-1.0"
X-Dual-Native-Level: 4
X-Dual-Native-Profile: tct-1
```

**JSON Example**:
```json
{
  "profile": "dual-native-core-1.0",
  "conformance_level": 4,
  "canonical_url": "https://example.com/article/123",
  ...
}
```

**DNC Example**:
```json
{
  "version": 1,
  "profile": "dual-native-core-1.0",
  "conformance_level": 4,
  "items": [...]
}
```

#### 6.3.1 Conformance Declaration Examples (Informative)

**Public Website Declaration** (HTML `<meta>` tag):
```html
<!DOCTYPE html>
<html>
<head>
  <meta name="dual-native-conformance" content="level=4; profile=dual-native-core-1.0">
  <link rel="alternate" type="application/json" href="/article/123/llm/">
  <link rel="sitemap" type="application/json" href="/m-sitemap.json">
</head>
<body>...</body>
</html>
```

**Repository Badge** (README.md):
```markdown
![Dual-Native Level 4](https://img.shields.io/badge/Dual--Native-Level%204-brightgreen)

This platform conforms to the Dual-Native Pattern (Level 4):
- Specification: https://github.com/[org]/dual-native-pattern/spec
- Catalog: https://api.example.com/dual-native-catalog.json
- Conformance Level: 4 (Full DNC with governance)
```

**Programmatic Conformance Query** (API endpoint):
```http
GET /.well-known/dual-native HTTP/1.1
Host: api.example.com

HTTP/1.1 200 OK
Content-Type: application/json

{
  "conformance": {
    "profile": "dual-native-core-1.0",
    "level": 4,
    "classes": ["resource_publisher", "catalog_service"],
    "catalog_url": "https://api.example.com/dual-native-catalog.json",
    "last_validated": "2025-01-15T10:00:00Z",
    "validator_reports": "https://api.example.com/conformance/reports"
  },
  "contact": {
    "email": "support@example.com",
    "documentation": "https://docs.example.com/dual-native"
  }
}
```

**Database Schema Declaration** (SQL table comment):
```sql
CREATE TABLE products (
  product_id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  price DECIMAL(10,2),
  last_modified TIMESTAMP DEFAULT NOW()
);

COMMENT ON TABLE products IS
  'Dual-Native conformance: level=3; hr=admin/products; mr=api/v1/products; cid=last_modified';
```

**Container Image Label** (Dockerfile):
```dockerfile
FROM node:18-alpine
LABEL org.dual-native.conformance.level="4"
LABEL org.dual-native.conformance.profile="dual-native-core-1.0"
LABEL org.dual-native.catalog.url="https://api.example.com/m-sitemap.json"
```

**Kubernetes Service Annotation**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: products-api
  annotations:
    dual-native.conformance.level: "4"
    dual-native.conformance.profile: "dual-native-core-1.0"
    dual-native.catalog.url: "https://api.example.com/catalog.json"
spec:
  selector:
    app: products
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**Note**: These declaration formats are illustrative. Implementations SHOULD choose formats appropriate to their domain and discovery mechanisms.

### 6.4 Optional Capabilities (Informative)

Implementations MAY adopt optional enhancements (integrity, immutability, evented sync, provenance) described in the [Optional Enhancements Guide](IMPLEMENTATION-GUIDE.md#8-optional-enhancements-advanced-patterns). These are **not required** for conformance but may be declared in metadata for discoverability:

```json
{
  "profile": "dual-native-core-1.0",
  "conformance_level": 4,
  "optional_capabilities": ["integrity", "immutability", "evented_sync"],
  "digest": {"algo": "sha256", "value": "abc123..."},
  "immutable": true,
  "snapshot": "v42",
  "event_endpoint": "https://api.example.com/webhooks"
}
```

**Note**: Declaring capabilities does not affect conformance level. Conformance is based solely on the core requirements (§ 4) and level definitions (§ 6.2).

### 6.5 Implementation Observability and Conformance Testing

**Requirement 6.5.1**: Implementations SHOULD collect hard metrics to verify conformance and operational health.

**Recommended Observability Metrics**:

1. **Performance Metrics**:
   - MR endpoint latency (p50, p95, p99)
   - HR endpoint latency (p50, p95, p99)
   - DNC freshness lag (time between content update and catalog update)
   - Payload size (HR vs MR bytes transferred)

2. **Efficiency Metrics**:
   - Zero-fetch hit rate (% of requests returning 304 Not Modified)
   - Bandwidth savings (HR bytes - MR bytes, as percentage)
   - Token savings (for LLM-optimized MR profiles)
   - Database query reduction (HR queries vs MR queries)

3. **Consistency Metrics**:
   - Validator parity failures (catalog CID ≠ live MR CID)
   - HR ↔ MR drift incidents (semantic equivalence violations)
   - Synchronization lag (time for HR update to appear in MR)

4. **Safety Metrics**:
   - Optimistic concurrency failures (412 Precondition Failed count)
   - Write conflict rate (concurrent edit attempts)
   - Access control violations (unauthorized MR access attempts)

**Conformance Test Suite**:

Implementations SHOULD define or adopt a conformance test suite that validates:

1. **CID Determinism**:
   - Same resource state → same CID (repeated reads)
   - Different resource state → different CID (after mutation)
   - CID format matches declared validator type

2. **HR ↔ MR Semantic Equivalence**:
   - Test declared equivalence scope on sample resources
   - Automated comparison of HR and MR claims
   - Alert on drift exceeding tolerance threshold

3. **Conditional Read / Safe Write Behavior**:
   - Conditional GET with matching CID → 304 Not Modified (or equivalent)
   - Conditional GET with mismatched CID → 200 OK with full MR
   - Write with matching CID → Success + new CID
   - Write with mismatched CID → 412 Precondition Failed (or equivalent)

4. **DNC Freshness vs Live MR**:
   - Catalog CID matches live MR ETag/validator
   - Catalog RID resolves to valid HR and MR
   - Bidirectional links are resolvable and correct

**Test Automation Examples**:
- HTTP: `curl` scripts validating ETag parity, 304 responses, If-Match behavior
- Database: SQL queries checking LSN consistency, validator freshness
- Streaming: Consumer offset validation, schema fingerprint checks

**Conformance Report Format** (Informative):
```json
{
  "conformance_level": 4,
  "profile": "dual-native-core-1.0",
  "test_date": "2025-01-15T10:00:00Z",
  "tests_passed": 47,
  "tests_failed": 0,
  "metrics": {
    "zero_fetch_rate": 0.87,
    "payload_reduction": 0.56,
    "validator_parity": 1.0
  },
  "validator_url": "https://github.com/org/dual-native-validator"
}
```

**Rationale**: Metrics enable verification of conformance claims and operational monitoring. Test suites prevent regression and validate spec adherence.

---

## 7. Security Considerations (Summary)

This section provides a **summary** of security considerations. For detailed threat models and mitigation strategies, see [Security & Governance Guide] (planned companion document).

**Baseline Requirement**: Implementations SHOULD perform domain-specific security and privacy analysis using the planned Security & Governance Guide or an equivalent internal security review process before deploying dual-native systems in production.

### 7.1 Attack Surface Introduced by Dual-Native Interfaces

**Risks**:
- **HR/MR Divergence**: Attackers exploit inconsistencies between HR and MR to bypass validation
- **MR Data Poisoning**: Malicious data injected into MR exploits AI systems' trust in structured data
- **Synchronization Failures**: Partial updates leave resources in inconsistent states

**Mitigations** (see Implementation Guide, Section on Pattern-Specific Safety Considerations):
- Single source of truth architecture
- Atomic updates (ACID transactions)
- Automated equivalence testing
- Monitoring and alerting for drift

### 7.2 DNC-Specific Concerns

**Risks**:
- **Catalog Poisoning**: Attacker modifies DNC to redirect AI systems to malicious MR endpoints
- **Enumeration Attacks**: DNC exposes all resources, enabling targeted attacks
- **Privacy Leakage**: Catalog metadata reveals system structure, ownership, sensitivity

**Mitigations**:
- Authenticated access to DNC (OAuth2, API keys)
- Digital signatures on DNC entries
- Rate limiting and abuse detection
- Filtering sensitive resources from public catalogs

### 7.3 Validation and Integrity Protections

**Risks**:
- **CID Forgery**: Attacker creates fake ETag/validator that matches cached version
- **MITM Attacks**: Attacker intercepts and modifies MR responses

**Mitigations**:
- Cryptographic validators (HMAC, digital signatures)
- TLS/HTTPS for all MR endpoints
- Subresource Integrity (SRI) for critical content

### 7.4 Reference to Security & Governance Guide

For comprehensive threat models, domain-specific attack vectors, and detailed mitigations, see:

**[Security & Governance Guide]** (planned companion document) covering:
- Authentication and authorization patterns
- Data integrity and validation strategies
- Confidentiality and privacy controls
- DNC governance and lifecycle management
- Rate limiting and abuse resistance
- Secure deployment patterns
- Incident response playbooks

---

## 8. Privacy Considerations (Summary)

### 8.1 Identity and Linkability

**Concern**: Bidirectional linking (HR ↔ MR) creates linkability between human and AI access patterns.

**Example**: A user views HR (HTML page) at 10:00 AM. AI system fetches corresponding MR at 10:01 AM. Correlation reveals user's identity or behavior.

**Mitigations**:
- Separate authentication for HR and MR (don't share session cookies)
- Audit logs should redact personally identifiable information (PII)
- DNC should not expose user-specific access patterns

### 8.2 Catalog-Level Metadata Exposure

**Concern**: DNC metadata (owner, sensitivity, schema) reveals organizational structure and data classification.

**Mitigations**:
- Filter sensitive metadata from public catalogs
- Require authentication for full catalog access
- Provide "public catalog" vs. "internal catalog" variants

**Recommendation**: Consult privacy/compliance teams when publishing DNC.

### 7.5 Access Tiers and Redaction Baseline

**Normative Baseline for Authorization and Redaction**:

**Requirement 7.5.1**: MR reads and mutations MUST be gated by existing authorization mechanisms (roles, capabilities, scopes, ACLs).

**Requirement 7.5.2**: MR access control MUST be at least as restrictive as HR access control for the same resource.

**Bad Example**: HR requires authentication, MR is publicly accessible → Privacy leak

**Good Example**: Both HR and MR require the same authentication and authorization

**Public MR Exposure**:

**Requirement 7.5.3**: If MR is exposed publicly (without authentication), the default SHOULD be to expose only published/approved resources.

**Implementation Guidance**:
- Filter draft/unpublished content from public MR endpoints
- Require authentication for draft content MR access
- Document public vs. authenticated access tiers in DNC metadata

**Sensitive Field Redaction**:

**Requirement 7.5.4**: If sensitive fields (PII, credentials, internal IDs) are redacted from MR, implementations MUST choose one of:

1. **CID over redacted view**: Compute CID over the redacted MR representation
   - Pro: CID matches what clients receive
   - Con: CID changes if redaction rules change, even if source content doesn't

2. **CID excludes sensitive fields**: Define an exclude list; CID computation ignores those fields
   - Pro: CID stable across redaction rule changes
   - Con: Requires documented exclude list

**Example Redaction Patterns**:
- **Healthcare**: Redact patient SSN, insurance numbers from public FHIR MR
- **Fintech**: Redact full credit card numbers, show only last 4 digits in MR
- **HR Systems**: Redact salary, performance ratings from employee MR

**Rationale**: Prevents accidental PII/credential exposure to AI agents. Access control must match HR to avoid privilege escalation via MR.

**For detailed access tier patterns by domain**, see Implementation Guide Section 6 (planned).

---

## 9. IANA Considerations

This document has no IANA actions.

**Note**: Domain-specific profiles (e.g., TCT for HTTP) may register:
- Media types (e.g., `application/dual-native+json`)
- Link relation types (e.g., `rel="dual-native-mr"`)
- URI schemes (if applicable)

Such registrations are out of scope for this core specification.

---

## 10. References

### 10.1 Normative References

- **[RFC 2119]**: Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997. https://www.rfc-editor.org/rfc/rfc2119

### 10.2 Informative References

- **[Dual-Native Whitepaper]**: "The Dual-Native Pattern: Serving Humans and AI from a Single System", November 2025. [WHITEPAPER.md](WHITEPAPER.md)

- **[Domain Implementation Guide]**: "Dual-Native Pattern: Domain Implementation Guide", November 2025. [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md)

- **[TCT Internet-Draft]**: Jurkovikj, A., "Collaboration Tunnel Protocol (TCT)", draft-jurkovikj-collab-tunnel, IETF. https://datatracker.ietf.org/doc/draft-jurkovikj-collab-tunnel/

- **[RFC 8288]**: Nottingham, M., "Web Linking", RFC 8288, October 2017. https://www.rfc-editor.org/rfc/rfc8288

- **[RFC 8785]**: Rundgren, A., et al., "JSON Canonicalization Scheme (JCS)", RFC 8785, June 2020. https://www.rfc-editor.org/rfc/rfc8785

---

## Appendix A. Example DNC Records (Informative)

**HTTP/Web (TCT)**:
```json
{
  "rid": "https://wellbeing-support.com/beyond-the-blues-cultivating-everyday-mental-resilience/",
  "hr": "https://wellbeing-support.com/beyond-the-blues-cultivating-everyday-mental-resilience/",
  "mr": "https://wellbeing-support.com/beyond-the-blues-cultivating-everyday-mental-resilience/llm/",
  "content_id": "sha256-k7B2hJ9mK4pL3qR5sT6vW8xY0zA1bC2dE3fG4hI5jK6",
  "updatedAt": "2025-01-20T14:30:00Z",
  "profile": "tct-1"
}
```

**Database**:
```json
{
  "rid": "postgres://prod-db/public/customers",
  "hr": "https://dashboard.example.com/tables/customers",
  "mr": "https://api.example.com/v1/customers",
  "content_id": "lsn:0/1A2B3C4D",
  "updatedAt": "2025-01-16T10:00:00Z",
  "profile": "dual-native-db-1.0",
  "schema": "https://api.example.com/schemas/customers.json",
  "owner": "data-platform-team",
  "sensitivity": "internal"
}
```

**Streaming (Kafka)**:
```json
{
  "rid": "kafka://prod-cluster/sensor-readings",
  "hr": "https://monitoring.example.com/topics/sensor-readings",
  "mr": "kafka://prod-cluster:9092/sensor-readings",
  "content_id": "offset:9876543",
  "updatedAt": "2025-01-16T10:00:00Z",
  "profile": "dual-native-kafka-1.0",
  "schema_registry_id": 123
}
```

**Healthcare (FHIR)**:
```json
{
  "rid": "Patient/12345",
  "hr": "https://portal.hospital.org/patients/12345",
  "mr": "https://fhir.hospital.org/Patient/12345",
  "content_id": "versionId:42",
  "updatedAt": "2025-01-16T10:00:00Z",
  "profile": "dual-native-fhir-1.0",
  "sensitivity": "phi"
}
```

---

## Appendix B. Example RID/CID Schemas (Non-Normative)

**JSON Schema for DNC Record** (Reference Encoding):

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Dual-Native Catalog Record",
  "type": "object",
  "required": ["rid", "hr", "mr", "content_id", "updatedAt"],
  "properties": {
    "rid": {
      "type": "string",
      "description": "Resource Identifier (stable)"
    },
    "hr": {
      "type": ["string", "object"],
      "description": "Human Representation reference"
    },
    "mr": {
      "type": ["string", "object"],
      "description": "Machine Representation reference"
    },
    "content_id": {
      "type": "string",
      "description": "Content Identity validator"
    },
    "updatedAt": {
      "type": "string",
      "format": "date-time",
      "description": "Last modification timestamp (ISO 8601)"
    },
    "profile": {
      "type": "string",
      "description": "Implementation profile/version"
    },
    "schema": {
      "type": "string",
      "format": "uri",
      "description": "MR schema reference"
    },
    "auth": {
      "type": ["string", "object"],
      "description": "Authentication requirements"
    },
    "owner": {
      "type": "string",
      "description": "Resource owner/team"
    },
    "sensitivity": {
      "type": "string",
      "enum": ["public", "internal", "confidential", "phi"],
      "description": "Data classification"
    },
    "retired": {
      "type": "boolean",
      "description": "Marks resource as deleted/archived"
    },
    "retiredAt": {
      "type": "string",
      "format": "date-time",
      "description": "Retirement timestamp"
    }
  }
}
```

---

**Document Version:** 1.0
**Last Updated:** November 2025
**Status:** Normative Specification

**For companion documents**:
- [Whitepaper](WHITEPAPER.md): Non-normative context and motivation
- [Implementation Guide](IMPLEMENTATION-GUIDE.md): Domain-specific cookbook
- [TCT Internet-Draft](https://datatracker.ietf.org/doc/draft-jurkovikj-collab-tunnel/): HTTP profile

**Feedback welcome:** Issues and pull requests at [repository URL to be determined]
