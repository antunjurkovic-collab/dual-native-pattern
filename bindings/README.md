# Dual-Native Bindings: Overview

This folder contains binding profiles showing how Dual-Native primitives map onto common transports and storage layers.

Each binding explains:

- How RID, MR, CID, validators, DNC, and digests are represented
- How to implement zero-fetch reads and safe writes
- Example flows and minimal conformance requirements

## Families

### HTTP / Web APIs
- [HTTP / REST](http-rest.md) - JSON/HTML APIs using REST-style routes

### Databases
- SQL (Coming soon)
- NoSQL (Coming soon)

### Streaming
- Kafka (Coming soon)
- Event Sourcing / CQRS (Coming soon)

### Object Storage
- S3-compatible (Coming soon)

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
