# The Dual-Native Pattern: Document Suite

**Version:** 1.0
**Last Updated:** November 2025
**Status:** Draft for Review

**Feedback:** Report issues or suggest improvements at [GitHub Issues](https://github.com/antunjurkovic-collab/dual-native-pattern/issues)

---

## Overview

This repository contains the complete **Dual-Native Pattern** document suite—an architectural approach for serving both human and AI users efficiently from unified systems.

The pattern is proven at scale through the **Collaboration Tunnel Protocol (TCT)**, demonstrating 83% bandwidth savings and 90%+ zero-fetch rates across 5,700+ production URLs.

---

## Pattern at a Glance

The Dual-Native Pattern provides a formal architecture for systems serving both human and AI users:

- **Every resource has two representations**: HR (Human-optimized: HTML, dashboards, PDFs) and MR (Machine-optimized: JSON APIs, FHIR, Avro)
- **Bidirectional linking**: Navigate seamlessly between HR ↔ MR with explicit links (not just documentation)
- **Semantic equivalence**: HR and MR represent the same content at the same point in time, verified through Content Identity validators (CID)
- **Zero-fetch optimization**: AI agents skip redundant downloads when CID matches cached version (83% bandwidth savings observed)

**Real-world evidence**: 11+ platforms already implement informal dual-native patterns (Stripe, GitHub, Databricks, Wikidata). See [Case Library](cases/README.md) for detailed analysis.

---

## Document Structure

This suite consists of **three core documents** at the repository root:

```
dual-native-pattern/
├── README.md                ← You are here (start here!)
├── WHITEPAPER.md            ← [1] Non-normative whitepaper
├── CORE-SPEC.md             ← [2] Normative specification
├── IMPLEMENTATION-GUIDE.md  ← [3] Domain implementation cookbook (includes optional enhancements)
├── CHANGELOG.md             ← Version history
└── cases/                   ← Platform case studies
    ├── README.md
    ├── index.json           ← Machine-readable catalog
    ├── github.md
    ├── stripe.md
    └── ...
```

### Planned Documents (Not Yet Available)

The following companion documents are planned for future release:

- **Security & Governance Guide** (planned) — Comprehensive threat models, domain-specific attack vectors, detailed mitigations, incident response playbooks, and governance frameworks
- **Adoption & Maturity Playbook** (planned) — Step-by-step organizational change management, stakeholder engagement, pilot selection criteria, and capability assessment tools

**Note:** Current documents reference these as "planned companion documents." The Core Specification provides a [security considerations summary](CORE-SPEC.md#7-security-considerations-summary), and the Implementation Guide includes [pattern-specific safety considerations](IMPLEMENTATION-GUIDE.md#3-pattern-specific-safety-considerations) as interim guidance.

---

## Which Document Should I Read?

### 🎯 I'm Evaluating if This Pattern Applies to My Organization

**Read:** [Whitepaper](WHITEPAPER.md)

**Time:** 30-45 minutes

**You'll Learn:**
- What the Dual-Native Pattern is (and isn't)
- Why it matters now (AI as content consumer)
- Real-world evidence (TCT case study with 83% bandwidth savings)
- When to use (and when NOT to use)
- Business value and ROI

**Best For:** CTOs, architects, product leads, decision-makers

---

### 📋 I Need Formal Requirements and Conformance Criteria

**Read:** [Core Specification](CORE-SPEC.md)

**Time:** 1-2 hours

**You'll Learn:**
- Normative requirements (RFC-2119 language: MUST, SHOULD, MAY)
- Formal definitions (RID, CID, HR, MR, DNC)
- Conformance levels (0-4)
- Architectural invariants
- Security and privacy considerations (summary)

**Best For:** Standards bodies, compliance teams, deep-architecture reviewers

---

### 🛠️ I'm Ready to Implement This in My Domain

**Read:** [Implementation Guide](IMPLEMENTATION-GUIDE.md)

**Time:** 2-3 hours (skim to your domain)

**You'll Learn:**
- Domain-specific discovery mechanisms (file systems, databases, MQTT, CoAP, Kafka, S3)
- Pattern-specific safety considerations (HR/MR divergence, poisoning, sync failures)
- Performance optimization strategies
- Testing strategies (semantic equivalence, validator parity, zero-fetch)
- Migration playbooks (Level 0 → Level 4)

**Best For:** Engineers, platform teams, technical leads

---

### ⚡ I Want Advanced Features (Integrity, Immutability, Events, Provenance)

**Read:** [Optional Enhancements Guide](IMPLEMENTATION-GUIDE.md#8-optional-enhancements-advanced-patterns)

**Time:** 45 minutes

**You'll Learn:**
- Integrity digests and signatures for tamper evidence
- Immutability patterns for snapshots and versioning
- Evented synchronization (webhooks, CDC, subscriptions)
- Provenance and lineage tracking
- When to use each enhancement (and when to skip)
- Cost/benefit analysis for each option

**Best For:** Mature deployments seeking advanced assurance, performance, or governance capabilities

**Important:** These are **optional** enhancements, not required for conformance. Implement the core pattern first.

---

## Quick Start: 15-Minute Overview

**Goal:** Understand the core idea without reading full documents.

1. **Read:** [Whitepaper § Executive Summary](WHITEPAPER.md#executive-summary) (2 min)
2. **Read:** [Whitepaper § The Problem](WHITEPAPER.md#2-the-problem-humans-ai-and-fragmented-systems) (3 min)
3. **Read:** [Whitepaper § Pattern at a Glance](WHITEPAPER.md#5-the-dual-native-pattern-at-a-glance) (5 min)
4. **Read:** [Whitepaper § TCT Case Study](WHITEPAPER.md#6-case-study-dual-native-http-with-tct) (5 min)

**Key Takeaway:** Every resource has HR (human) and MR (machine) representations, linked bidirectionally with semantic equivalence guarantees. Proven to save 83% bandwidth with 90%+ zero-fetch rates.

---

## Reading Paths by Role

### Executive / Business Leader (20 minutes)

1. [Whitepaper § Executive Summary](WHITEPAPER.md#executive-summary)
2. [Whitepaper § Why Now](WHITEPAPER.md#3-why-now-the-shift-to-dual-native-systems)
3. [Whitepaper § Impact and Opportunities](WHITEPAPER.md#9-impact-and-opportunities)
4. [Whitepaper § Getting Started](WHITEPAPER.md#10-getting-started)

**Decision Point:** "Should we adopt this pattern?" → See [When NOT to Use](WHITEPAPER.md#84-when-dual-native-is-a-bad-fit-anti-patterns)

---

### CTO / Chief Architect (45 minutes)

1. [Whitepaper](WHITEPAPER.md) (Sections 1-7)
2. [Core Spec § Architectural Overview](CORE-SPEC.md#3-architectural-overview)
3. [Core Spec § Conformance Levels](CORE-SPEC.md#62-conformance-levels-04)

**Decision Point:** "How mature are we?" → Assess against [Conformance Levels 0-4](CORE-SPEC.md#62-conformance-levels-04)

---

### Engineering Lead (90 minutes)

1. [Whitepaper § Pattern at a Glance](WHITEPAPER.md#5-the-dual-native-pattern-at-a-glance)
2. [Core Spec § Cross-Domain Core Requirements](CORE-SPEC.md#4-cross-domain-core-requirements-normative)
3. [Implementation Guide § Your Domain](IMPLEMENTATION-GUIDE.md#2-domain-specific-discovery-mechanisms)
4. [Implementation Guide § Safety Considerations](IMPLEMENTATION-GUIDE.md#3-pattern-specific-safety-considerations)
5. [Implementation Guide § Testing Strategies](IMPLEMENTATION-GUIDE.md#5-testing-strategies)

**Next Step:** Choose pilot domain, implement Level 1 (basic MR)

---

### Security / Compliance Team (60 minutes)

1. [Whitepaper § Introduction](WHITEPAPER.md#1-introduction)
2. [Core Spec § Security Considerations](CORE-SPEC.md#7-security-considerations-summary)
3. [Core Spec § Privacy Considerations](CORE-SPEC.md#8-privacy-considerations-summary)
4. [Implementation Guide § Pattern-Specific Safety](IMPLEMENTATION-GUIDE.md#3-pattern-specific-safety-considerations)

**Action Items:** Review threat models, assess compliance implications (HIPAA, GDPR, etc.)

---

### Platform Engineer (2-3 hours)

1. [Implementation Guide](IMPLEMENTATION-GUIDE.md) (full read)
2. [Core Spec § DNC Data Model](CORE-SPEC.md#5-dnc-data-model)
3. [Implementation Guide § Migration Playbooks](IMPLEMENTATION-GUIDE.md#6-migration-playbooks)

**Next Step:** Choose domain, implement discovery mechanism, add bidirectional links

---

## Key Concepts (Glossary)

| Term | Definition | Example |
|------|------------|---------|
| **HR** | Human Representation (human-optimized interface) | HTML page, dashboard, PDF |
| **MR** | Machine Representation (machine-optimized interface) | JSON API, Avro stream, FHIR resource |
| **RID** | Resource Identity (stable identifier shared by HR and MR) | `https://example.com/article/123` |
| **CID** | Content Identity (version validator, enables zero-fetch) | `sha256-abc123...`, `lsn:0/1A2B3C4D` |
| **DNC** | Dual-Native Catalog (registry of all dual-native resources) | JSON sitemap, SQL table, Kafka topic |
| **Zero-Fetch** | Skip download when CID matches cached version | 304 Not Modified (0 bytes transferred) |
| **Semantic Equivalence** | HR and MR represent the same content | Dashboard shows $100, API returns `{"price": 100}` |

**For full glossary:** See [Whitepaper Appendix A](WHITEPAPER.md#appendix-a-glossary)

---

## Real-World Evidence

The pattern is not theoretical—it's deployed in production:

### TCT (HTTP Implementation)

**Live Sites** (5,700+ URLs):
- bestdemotivationalposters.com
- wellbeing-support.com
- omacedonii.com
- llmpages.org

**Measured Results:**
- **83% bandwidth savings** (18.3 KB HTML → 3.1 KB JSON)
- **90%+ zero-fetch rate** (304 Not Modified responses)
- **Reduced infrastructure costs** (CDN egress, server load)

**Example:** [Whitepaper § TCT Case Study](WHITEPAPER.md#6-case-study-dual-native-http-with-tct)

### Informal Dual-Native Patterns in Production

Many industry platforms already provide dual interfaces (HR + MR), though without formal guarantees:

**[Case Library](cases/README.md)** — 11+ platform case studies:
- **Payments**: Stripe Dashboard + REST API
- **Developer Tools**: GitHub Web UI + GraphQL/REST APIs
- **Knowledge Graphs**: Wikidata pages + SPARQL endpoint
- **Data Platforms**: Databricks Workspace + REST API
- **E-commerce**: Shopify Admin + GraphQL API
- **CRM**: Salesforce Lightning + REST/Bulk APIs
- **IoT**: Cisco Meraki Dashboard + Dashboard API
- **Open Data**: Data.gov portal + CKAN API
- **Geospatial**: OGC/STAC map viewers + APIs
- And more...

Each case shows current state, pattern gaps, recommended upgrades, and expected benefits.

---

## Adoption Path (Stage-Based)

Organizations adopt through **capability stages** with clear entry/exit criteria, not fixed timelines:

| Stage | Name | Key Exit Criteria | Example KPIs |
|-------|------|-------------------|--------------|
| **1** | Linked Interfaces | HR ↔ MR links resolvable, RID defined, index entry exists | Coverage: 100% of pilot resources |
| **2** | Basic Sync | CID exposed, index includes `cid` + `updatedAt`, first skip-fetch observed | CID Coverage: 100%, Skip Rate: >0% |
| **3** | Optimized Delivery | Skip-rate ≥70% (stable content), drift checks pass, incremental discovery works | Skip Rate: ≥70%, Bandwidth Saved: ≥50% |
| **4** | Adaptive MR | Profiles documented, cost/latency budgets enforced, governance dashboards operational | Profile Adoption: >50%, Cache Hit: >80% |

**Goal:** Stage 3 is recommended for most systems. Stage 4 is best-in-class.

**Details:** [Whitepaper § Adoption Path: Stage-Based Progression](WHITEPAPER.md#8-adoption-path-stage-based-progression)

---

## Related Specifications

**HTTP Profile:**
- [TCT Internet-Draft](https://datatracker.ietf.org/doc/draft-jurkovikj-collab-tunnel/) (draft-jurkovikj-collab-tunnel)
- Formal HTTP specification for C-URL/M-URL, M-Sitemap, ETags

**Other Domains:**
- Databases: See [Implementation Guide § Databases](IMPLEMENTATION-GUIDE.md#22-database-systems)
- Streaming: See [Implementation Guide § Kafka/Pulsar](IMPLEMENTATION-GUIDE.md#24-streaming-systems-kafka-pulsar)
- IoT: See [Implementation Guide § MQTT/CoAP](IMPLEMENTATION-GUIDE.md#23-iot-protocols-mqtt-coap)
- Cloud Storage: See [Implementation Guide § S3/Azure/GCS](IMPLEMENTATION-GUIDE.md#25-cloud-storage-s3-azure-blob-gcs)

---

## FAQ

### Q: Why can't AI systems just parse HTML?

**A:** They can (using LLMs or custom parsers), but it's expensive and brittle:
- Parsing 100KB HTML with an LLM costs ~$0.10-$0.50 per request
- HTML structure changes break parsers
- 60-90% bandwidth wasted on navigation, ads, styling

Dual-native design provides clean, structured data that's smaller and doesn't require parsing.

### Q: Isn't this just "having APIs"?

**A:** No. Most APIs:
- Are separate from human content (no bidirectional discovery)
- Lack semantic equivalence guarantees (drift is common)
- Aren't optimized for AI (missing ETags, catalogs, zero-fetch)

The pattern adds **explicit linking, semantic equivalence, and standardized discovery**.

### Q: Do I need to implement TCT to use this pattern?

**A:** No. TCT is the **HTTP-specific profile**. The pattern applies to databases, streaming, IoT, and other domains using their native mechanisms (not HTTP headers).

### Q: What if my content changes constantly (real-time data)?

**A:** Zero-fetch optimization doesn't help for real-time streams (CID always changes). But dual-native design still benefits:
- MR is more efficient than parsing HR (smaller, structured)
- Bidirectional linking enables discovery
- Semantic equivalence prevents drift

---

## Contributing

This is a community-driven effort. Contributions welcome:

- **Feedback:** Open issues for clarifications, corrections, suggestions
- **Implementations:** Share domain-specific guides (e.g., "Dual-Native for PostgreSQL")
- **Case Studies:** Document your deployment experiences
- **Code Examples:** Contribute reference implementations

**Repository:** https://github.com/antunjurkovic-collab/dual-native-pattern

---

## License

All documents in this suite are licensed under:

**Creative Commons Attribution 4.0 International (CC BY 4.0)**

You are free to:
- **Share:** Copy and redistribute in any medium or format
- **Adapt:** Remix, transform, and build upon the material

Under the following terms:
- **Attribution:** Give appropriate credit, provide a link to the license

**Full License:** https://creativecommons.org/licenses/by/4.0/

---

## Authors and Acknowledgements

**Author:** Antun Jurkovikj

**Contributors:**
- IETF HTTP Working Group (TCT feedback)
- W3C Community (Linked Data, Semantic Web)
- Early adopters and production deployers

**Special Thanks:**
- Open-source contributors to TCT reference implementations
- AI and web standards communities for ongoing dialogue

---

## Feedback and Contributions

**We Welcome Feedback!** This is a community-driven effort in active development. Your input helps refine the pattern and improve the documentation.

**How to Provide Feedback:**
- **GitHub Issues**: https://github.com/antunjurkovic-collab/dual-native-pattern — Bug reports, clarifications, suggestions
- **Discussions**: https://github.com/antunjurkovic-collab/dual-native-pattern — Questions, use cases, implementation experiences
- **Email**: [contact email TBD] — Direct contact for private inquiries

**What We're Looking For:**
- Real-world implementation experiences (successes and challenges)
- Domain-specific guidance contributions (e.g., "Dual-Native for PostgreSQL")
- Case study additions documenting your platform
- Clarifications on confusing sections
- Corrections or updates to examples

---

## Contact and Support

**Questions?**
- Read the [FAQ](WHITEPAPER.md#appendix-b-frequently-asked-questions)
- Check the [Whitepaper](WHITEPAPER.md) for context
- Review the [Implementation Guide](IMPLEMENTATION-GUIDE.md) for technical details

**Technical Support:**
- See "Feedback and Contributions" above for reporting issues
- Check [CHANGELOG.md](CHANGELOG.md) for recent updates and known issues

---

## Document Versions

| Document | Version | Last Updated | Status |
|----------|---------|--------------|--------|
| README.md | 1.0 | November 2025 | Draft |
| WHITEPAPER.md | 1.0 | November 2025 | Draft for Review |
| CORE-SPEC.md | 1.0 | November 2025 | Draft for Review |
| IMPLEMENTATION-GUIDE.md | 1.0 | November 2025 | Draft for Review |

**Versioning:** All documents will be versioned together. Updates to any document will increment the suite version.

---

**Last Updated:** November 2025
**Document Suite Version:** 1.0
**Status:** Draft for Review

**Start Reading:** [Whitepaper →](WHITEPAPER.md)
