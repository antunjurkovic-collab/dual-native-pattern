# Dual-Native Pattern: Changelog

All notable changes to the Dual-Native Pattern document suite will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.3] - November 2025

### Added

**AI-Native Core Definition and Conformance**
- **CORE-SPEC.md**:
  - Added formal definition of **AI-Native Core (MR-Only Mode)** in § 2.8, including normative MUST/SHOULD requirements for RID, MR, CID, safe writes, zero-fetch reads, DNC, and integrity digests
  - Mapped AI-Native Core and Dual-Native deployments into conformance levels (§ 6.2), introducing **Level 1.5** for MR-only, AI-native systems:
    - Level 0: Legacy (no canonical MR)
    - Level 1: MR Present (informal, no guarantees)
    - **Level 1.5: AI-Native Core (MR-Only Mode)** - MR + Dual-Native primitives
    - Level 2: Full Dual-Native (HR + MR)
    - Level 3: Optimized (zero-fetch at scale)
    - Level 4: Multi-Native / Adaptive (multiple profiles)

**API Styles and MCP Integration**
- **WHITEPAPER.md**:
  - Clarified the **relationship to REST, RPC, and GraphQL** in § 2.5:
    - Positioning Dual-Native as a **state model above API styles**, not a competing transport
    - Added one-liner positioning: "REST/GraphQL/RPC answer 'How do I talk to you?' — Dual-Native answers 'What is the truth, and how do I stay in sync with it?'"
    - Added style-agnostic mappings and expanded GraphQL guidance (RID+CID-keyed caching for POST-based queries)
  - Added § 2.5.1 on how Dual-Native complements MCP and other agent-to-agent protocols:
    - MCP focuses on tooling and invocation
    - Dual-Native focuses on content state (identity, versioning, safe mutations)
    - MCP servers can expose Dual-Native resources as tools (DNC for discovery, MR reads with CID caching, MR writes with validators)

**Enhanced Comparison Table**
- **WHITEPAPER.md**:
  - Expanded comparison table in § 2.6:
    - Softened REST read-efficiency wording from "Often bloated" to "Varies (often includes presentation overhead)"
    - Added "Semantic Equivalence" row to highlight Dual-Native's unique guarantee of HR/MR views bound to same CID snapshot
    - Added "Bidirectional Discovery" row to emphasize explicit HR↔MR links and DNC entries vs. ad-hoc conventions

**Documentation**
- **WHITEPAPER.md**:
  - Added **AI-Native Content Core** to Appendix A (Glossary) for consistent naming and conformance usage

### Changed

**Terminology Refinement**
- **CORE-SPEC.md**:
  - Renamed § 2.8 from "AI-Native Content Core (Single MR Mode)" to "AI-Native Core (MR-Only Mode)" for clarity
  - Updated subsection titles: § 2.8.1 "Normative Requirements (MR-Only Mode)", § 2.8.2 "Relationship to Full Dual-Native"
  - Changed "Integrity over Final Bytes" to "Integrity Digest" for consistency

**Conformance Levels Restructuring**
- **CORE-SPEC.md**:
  - Restructured conformance levels in § 6.2 to include Level 1.5 and clarify progression from MR-only to multi-native
  - Updated Level 0 from "HR-Only Systems" to "Legacy" (no canonical MR)
  - Updated Level 1 from "HR + MR, One-Way Link" to "MR Present (Informal)" (no Dual-Native guarantees)
  - Updated Level 2 from "HR + MR, Bidirectional Links" to "Full Dual-Native" (AI-Native Core + additional representation)
  - Updated Level 3 from "Optimized (Zero-Fetch)" to "Optimized" (zero-fetch effective at scale)
  - Updated Level 4 from "Agentic (Read/Write)" to "Multi-Native / Adaptive" (multiple profiles and views)
  - Renumbered § 6.2.6 to § 6.2.7 "Informal Platform Mapping"

---

## [1.2] - November 2025

### Added

**Documentation Polish and Completeness**
- **WHITEPAPER.md**:
  - Added § 5 "Beyond the Core Principles" - Secondary patterns callout highlighting safe writes, MR format profiles, incremental discovery, legacy encapsulation, and observability metrics with links to detailed spec sections
  - Added § 11 "Limitations and Open Questions" - Comprehensive discussion of scope boundaries, adoption costs, interoperability challenges, performance trade-offs, security/privacy gaps, tooling needs, and agent oversight considerations
  - Renumbered Conclusion to § 12

- **CORE-SPEC.md**:
  - Added § 4.2 "Wire-Integrity vs. Semantic Identity" - Clarification distinguishing wire-integrity digests (HTTP Content-Digest, RFC 9530) from CID (semantic versioning), explaining their different purposes and coexistence

- **README.md**:
  - Updated Overview (line 15) to mention wp-dual-native plugin implementation with block-level atomic operations

### Changed

**Version Increments**
- Updated all document headers from v1.0 → v1.2:
  - README.md
  - WHITEPAPER.md
  - CORE-SPEC.md
  - IMPLEMENTATION-GUIDE.md
- Updated README version table (all documents now show v1.2)
- Updated README Document Suite Version from 1.0 → 1.2

**Documentation Improvements**
- Enhanced intellectual honesty by acknowledging pattern limitations and open questions
- Improved navigability by calling out secondary patterns with direct spec links
- Clarified distinction between transmission integrity (bytes) and content versioning (semantics)

---

## [1.1] - November 2025

### Added

**Universal Pattern Enhancements**
- **CORE-SPEC.md**:
  - Added § 4.3.3 "MR Format Profiles" - Support for multiple MR profiles (structured JSON/Avro, text/Markdown, binary/CBOR) with formal requirements
  - Added § 4.2.3 "Safe Writes & Validators" - Normative requirements for conditional mutations using validators (If-Match, version columns, offsets) with transport-specific mappings (HTTP, gRPC, databases, streaming)
  - Added § 4.8.1.1 "Addressable Content Units" - Support for sub-resource addressing and atomic operations (append, prepend, insert, replace, delete, patch) with domain-specific addressing schemes
  - Added § 7.5 "Access Tiers and Redaction" - Normative baseline for MR access control, redaction strategies (CID over redacted view vs CID excludes sensitive fields), and authorization requirements
  - Added § 6.2.7 "Legacy Encapsulation" - Normative pattern for wrapping legacy/unstructured content as opaque units in MR, enabling incremental migration and safe coexistence
  - Added § 6.5 "Observability & Conformance Testing" - Metrics recommendations (performance, efficiency, consistency, safety) and conformance test suite (CID determinism, semantic equivalence, conditional behavior, DNC freshness)

- **IMPLEMENTATION-GUIDE.md**:
  - Added CORE-SPEC cross-references to all 5 domain sections (File Systems, Databases, IoT, Streaming, Cloud Storage) mapping implementation patterns to normative requirements
  - Added safe write examples for Databases (§ 2.2):
    - PostgreSQL RowVersion pattern with conditional UPDATE and ROW_COUNT validation
    - SQL Server ROWVERSION pattern with timestamp-based concurrency control
    - API integration with If-Match/412 responses
  - Added safe write examples for Streaming (§ 2.4):
    - Kafka idempotent producer with exactly-once semantics and transactional writes
    - Pulsar message deduplication with sequence IDs
    - Expected offset validation patterns
  - Added § 3.4 "Access Tiers and Redaction" with implementation guidance:
    - Two redaction strategies with Python code examples
    - Domain-specific patterns (Database RLS, Healthcare HIPAA, IoT admin vs public)
    - Test matrix for validating redaction correctness

**Narrative and Cross-Linking Improvements**
- **README.md**:
  - Updated "Pattern at a Glance" (line 23) to introduce MR format profiles concept
  - Added conformance level cross-reference to Case Library section with link to CORE-SPEC § 6.2

- **WHITEPAPER.md**:
  - Updated § 5 "Pattern at a Glance" to introduce MR format profiles in Key Principles
  - Updated § 5.1 MR definition to describe profiles (structured, text, binary) with shared RID/CID
  - Added conformance level cross-reference to Case Library (§ 4.6) with link to CORE-SPEC § 6.2

### Changed
- Made dual-native pattern truly universal and protocol-agnostic:
  - Emphasized cross-domain applicability (databases, IoT, streaming, file systems, cloud storage) beyond HTTP/web
  - Added domain-native validator mappings (PostgreSQL LSN, SQL Server ROWVERSION, Kafka offset, HTTP ETag)
  - Connected write-path patterns to optimistic concurrency control across all domains

- Enhanced safety and security guidance:
  - Connected abstract security requirements in CORE-SPEC to concrete implementation patterns in IMPLEMENTATION-GUIDE
  - Added access control baseline requiring MR to be at least as restrictive as HR
  - Provided practical redaction strategies with trade-offs and code examples

### Improved
- Case Library documentation now explicitly references conformance levels and links to formal definitions
- Implementation Guide now provides bidirectional traceability between domain patterns and normative requirements
- All documents now consistently introduce MR format profiles as first-class concept

---

## [1.0] - November 2025

### Added

**Document Suite Structure**
- Created focused document suite with flat structure at repository root:
  - **WHITEPAPER.md**: "The Dual-Native Pattern: An Architecture for Humans and AI" (non-normative) - Context, motivation, case studies, adoption guidance
  - **CORE-SPEC.md**: Formal requirements, conformance levels, architectural invariants (normative)
  - **IMPLEMENTATION-GUIDE.md**: Domain-specific discovery mechanisms, safety considerations, testing strategies, migration playbooks, optional enhancements (non-normative)
- Merged optional capabilities guide into IMPLEMENTATION-GUIDE.md (Section 8)
- Repository structure: All core documents at root, case studies in `cases/` subdirectory

**Case Library**
- Added 11 platform case studies demonstrating informal dual-native patterns:
  - Payments: Stripe
  - Developer Tools: GitHub, Shopify
  - Data & AI: Databricks, Data.gov
  - Knowledge Graphs: Wikidata
  - Enterprise: Salesforce, OData Services
  - IoT & Industrial: Cisco Meraki, Digital Twin Platforms
  - Geospatial: OGC & STAC
- Each case includes:
  - Current state analysis
  - Equivalence scope definitions with CID strategies and validation methods
  - Pattern gaps and recommended upgrades
  - Expected benefits and risk considerations
- Added machine-readable `cases/index.json` with metadata (level, RID/CID examples, equivalence scope)

**Pattern Enhancements**
- Added "Pattern at a Glance" quick-reference sections to README and whitepaper
- Added conformance level (0-4) mapping across all documents
- Added domain-specific implementation checklists (File Systems, Databases, IoT, Streaming, Cloud Storage)
- Added structured safety considerations (HR/MR Divergence, MR Data Poisoning, Sync Failures) with Risk/Example/Detection/Impact/Mitigation format
- Added optional capabilities framework with JSON examples (integrity digests, immutability, evented sync, provenance)

**Documentation Improvements**
- Added planned documents section (Security & Governance Guide, Adoption & Maturity Playbook)
- Added role-based reading paths (Executive, CTO, Engineering Lead, Security/Compliance, Platform Engineer)
- Added conformance glossary note on optional capability reporting
- Added informative labels to non-normative spec sections
- Added security baseline requirement (§ 7) to core specification
- Added FAQ sections to optional capabilities guide

**Metadata & Structure**
- Added document version headers to all files (Version 1.0, November 2025)
- Added CHANGELOG.md (this file)
- Added equivalence_scope field to all case study index entries
- Added level (conformance maturity) field to all case study index entries
- Added rid_example and cid_example fields to all case study index entries

### Changed
- Softened marketing tone in whitepaper ("key questions are 'how soon' and 'where to start'")
- Made "Pattern at a Glance" more skimmable with 6-concept summary table
- Restructured Implementation Guide safety section with consistent Risk/Detection/Mitigation format
- Updated optional capabilities guide from "Draft, exploratory" to "Non-Normative Guidance" status
- Connected migration playbooks to spec conformance levels with explicit links

### Fixed
- Fixed truncated strings in cases/index.json (Shopify, Salesforce recommended_upgrades)
- Removed RFC-2119 keywords (MUST/SHOULD) from non-normative whitepaper
- Standardized case library headings and structure

---

## Format Notes

- **Version 1.0** represents the first public release of the document suite
- All documents are versioned together; updates to any document increment the suite version
- Document status: Draft for Review (pending community feedback)
- Repository structure: Flat layout with all core documents at root

---

**Repository:** https://github.com/antunjurkovic-collab/dual-native-pattern
**License:** Creative Commons Attribution 4.0 International (CC BY 4.0)
**Author:** Antun Jurkovikj
