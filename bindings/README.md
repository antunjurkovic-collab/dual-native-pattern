# Dual-Native Bindings (Informative)

This folder contains **binding profiles** that map the Dual-Native primitives:

- RID (Resource Identity)
- MR (Machine Representation)
- CID (Content Identity)
- Validators & safe writes
- Zero-fetch reads
- DNC (Dual-Native Catalog)
- Integrity digests

onto specific transports and storage layers.

The **core pattern** is defined in `CORE-SPEC.md` and `WHITEPAPER.md`.
Bindings are **non-normative** and show concrete, copy-pasteable mappings for real systems.

## Available Bindings

- **HTTP / REST**
  - [`http-rest.md`](./http-rest.md) – REST-ish JSON APIs over HTTP/1.1 and HTTP/2, with HR (HTML) and MR (JSON) views.

- **Databases (SQL)**
  - [`db-sql.md`](./db-sql.md) – RID as primary key, CID as version/hash column, safe writes via `UPDATE ... WHERE id AND cid`, DNC via catalog table. (Draft skeleton)

## Planned Bindings (TODO)

- **NoSQL Databases**
  - `db-nosql.md` – Document stores, key-value stores with version fields.

- **Streaming / Event Systems (Kafka, etc.)**
  - `streaming-kafka.md` – RID as key/aggregate, CID as sequence, DNC as compact index topic.

- **Object Storage (S3-like)**
  - `object-s3.md` – RID as key, CID as ETag, DNC as manifest object.

Bindings can evolve faster than the core spec and can be extended by other implementers without changing the main documents.

## How to Use These Bindings

These binding profiles are **informative** (non-normative). They show concrete implementations of the Dual-Native Pattern as defined in the Core Specification, but are not required for conformance.

**When to use a binding:**
- You're implementing Dual-Native over a specific transport (HTTP, database, streaming, etc.)
- You want to see how RID/MR/CID/validators/DNC map to your technology stack
- You need examples of zero-fetch and safe write patterns in your domain

**Relationship to Core Spec:**
- The [Core Specification](../CORE-SPEC.md) defines the pattern at the content/state level
- These bindings show how to project that pattern onto specific technologies
- All bindings implement the same core primitives, just using different mechanisms

## Adding New Bindings

Want to document a binding for your technology stack? Follow the template structure:

1. **Scope** - What this binding covers
2. **Mapping** - RID, MR, CID, validators, DNC, digests
3. **Read Semantics** - Zero-fetch implementation
4. **Write Semantics** - Safe writes implementation
5. **Catalog Semantics** - DNC exposure
6. **Example Flows** - Concrete examples
7. **Conformance Notes** - Requirements for this binding

See [http-rest.md](http-rest.md) for a complete example.
