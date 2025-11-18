# Dual-Native Pattern: Changelog

All notable changes to the Dual-Native Pattern document suite will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
