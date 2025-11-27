# Dual-Native Pattern: Case Library

This folder contains concise, real-world platform cases showing how informal dual-interface designs can be strengthened with the formal Dual-Native Pattern.

**Note:** Brand and platform names are used for illustrative purposes only. No endorsement, affiliation, or official support is implied. Case studies reflect independent analysis of publicly available patterns.

Each case follows the same structure:
- HR (human) and MR (machine) surfaces
- Current discovery and synchronization behaviour
- Pattern gaps (linking, CID, index)
- Recommended upgrades
- Expected benefits and considerations

## Technology Bindings

For language/framework-agnostic technical mappings (how to implement Dual-Native over HTTP, SQL, streaming, object storage), see **[/bindings](/bindings)**.

Examples:
- **Generic HTTP/REST API** – A framework-agnostic REST API that exposes MR JSON, uses ETag/CID for zero-fetch and 412 for safe writes, and publishes a DNC endpoint for agents. See [/bindings/http-rest.md](/bindings/http-rest.md) for details.
- **SQL Database** – Using version columns for CID, `UPDATE ... WHERE cid = ?` for safe writes, and catalog tables for DNC. See [/bindings/db-sql.md](/bindings/db-sql.md) for details.

## Cases Included

### 🏆 Reference Implementations

- **[WordPress](wordpress.md)** [Domain: CMS, Level: 4]
  **Reference implementation** demonstrating both internal agentic workflows (DNI plugin with block mutations, `If-Match` concurrency) and public distribution (TCT with zero-fetch). Proven at scale with 5,700+ production URLs.

- **[Model Context Protocol (MCP)](mcp.md)** [Domain: AI Infrastructure, Requires Level: 4]
  **Integration pattern** showing why Dual-Native is the architectural prerequisite for robust MCP servers. Demonstrates safe read/write operations for AI agents with optimistic concurrency and structural mutations.

### Payments
- **[Stripe](stripe.md)** [Domain: Payments, Approx. Level: 4]
  Dashboard + REST API with strong HR/MR surfaces; full CRUD via API with idempotent writes

### Developer Tools
- **[GitHub](github.md)** [Domain: Developer Tools, Approx. Level: 4]
  Web UI + REST/GraphQL APIs; strong informal dual-native pattern with full repository/issue/PR write operations

- **[Shopify](shopify.md)** [Domain: E-commerce, Approx. Level: 2]
  Admin UI + GraphQL API; bidirectional via documentation, needs CID validators for product/inventory state

### Data & AI
- **[Databricks](databricks.md)** [Domain: Data & AI, Approx. Level: 2]
  Workspace UI + REST API for notebooks/jobs; Delta table versions as CID; missing unified resource index

- **[Data.gov](data-gov.md)** [Domain: Open Data, Approx. Level: 3]
  Web portal + CKAN API; strong metadata versioning, bidirectional linking via dataset pages

### Knowledge Graphs
- **[Wikidata](wikidata.md)** [Domain: Knowledge Graph, Approx. Level: 3]
  Web pages + SPARQL endpoint; revision IDs as CID; strong versioning, weak bidirectional linking

### Enterprise
- **[Salesforce](salesforce.md)** [Domain: CRM, Approx. Level: 2]
  Lightning UI + REST/Bulk APIs; API links in UI, needs CID validators for record versioning

- **[OData Services](odata.md)** [Domain: Data APIs, Approx. Level: 2-3]
  Generic OData pattern analysis; ETags present, needs explicit HR linkage and DNC

### IoT & Industrial
- **[Cisco Meraki](meraki.md)** [Domain: Networking/IoT, Approx. Level: 2]
  Dashboard + Dashboard API for network devices; API docs linked, needs device CID (firmware + config hash)

- **[Digital Twin Platforms](digital-twin.md)** [Domain: Industrial/IoT, Approx. Level: 2-3]
  3D visualization + MQTT/REST APIs; device IDs as RID, telemetry timestamps as CID, needs DNC

### Geospatial
- **[OGC & STAC](ogc-stac.md)** [Domain: Geospatial, Approx. Level: 2-3]
  Map viewers + OGC APIs/STAC; strong linking via STAC assets, needs explicit CID strategy

Tip: Reference these from the whitepaper “Real-World Evidence” section, and link deeper from the Implementation Guide for domain-specific tactics.

