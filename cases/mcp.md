# Model Context Protocol (MCP)

**Domain:** AI Agent Infrastructure
**Status:** Integration Pattern
**Conformance Level:** Requires Level 4 (Agentic)

---

## 1. Overview

The **Model Context Protocol (MCP)** is the industry standard for connecting AI agents (like Claude) to external systems. MCP defines a protocol for **resources**, **tools**, and **prompts** that enable AI systems to read and modify external data safely.

**The Problem MCP Solves:**
- AI agents need to interact with external systems (databases, CMSs, APIs, filesystems)
- Direct HTML scraping or ad-hoc API access is unsafe and brittle
- Agents need structured, validated interfaces with strong safety guarantees

**Website:** https://modelcontextprotocol.io/

---

## 2. Why Dual-Native is Essential for MCP

**Argument:** The Dual-Native Pattern is the **architectural prerequisite for robust MCP servers**. You cannot build a safe MCP server on top of raw HTML scraping or unstructured data sources.

### 2.1 MCP Requirements Map to Dual-Native Levels

| MCP Capability | Dual-Native Requirement | Level |
|----------------|------------------------|-------|
| `read_resource` | MR with stable RID | Level 2 |
| `list_resources` | Dual-Native Catalog (DNC) | Level 3 |
| Change detection | CID validators | Level 3 |
| `call_tool` (read-only) | MR with zero-fetch | Level 3 |
| `call_tool` (mutations) | Structural Mutations + optimistic concurrency | **Level 4** |

**Conclusion:** To build a **fully capable MCP server**, you need **Level 4 Dual-Native conformance**.

### 2.2 Without Dual-Native: The Failure Modes

**Scenario:** An MCP server built on HTML scraping

1. **Fragility**: Agent parses HTML with CSS selectors → Layout changes break the agent
2. **Corruption Risk**: Agent modifies HTML strings → Introduces invalid markup, breaks rendering
3. **Race Conditions**: Agent reads HTML, computes edit, writes back → Meanwhile, human edited → Lost update
4. **Token Waste**: Agent ingests full HTML (templates, navigation, ads) → 10x token cost
5. **No Validation**: Agent cannot verify it received uncorrupted data → Silent failures

**Result:** Unreliable, expensive, dangerous.

**With Dual-Native:**

1. **Structured**: Agent works on MR (JSON blocks, not HTML strings) → Immune to layout changes
2. **Safe**: Agent uses Structural Mutations (insert_block, not string manipulation) → Cannot corrupt
3. **Concurrent**: Agent uses `If-Match` with CID → Detects conflicts, prevents lost updates
4. **Efficient**: Agent ingests MR (content only, no templates) → 80%+ token savings
5. **Validated**: Agent checks Content-Digest → Detects tampering, ensures integrity

**Result:** Reliable, efficient, safe.

---

## 3. MCP Server Example: WordPress DNI

The WordPress DNI plugin (see [wordpress.md](wordpress.md)) is a **production MCP server** demonstrating Level 4 conformance.

### MCP Resources

```json
{
  "uri": "wordpress://example.com/posts/123",
  "name": "Article Title",
  "mimeType": "application/json",
  "description": "WordPress post #123 in structured block format"
}
```

Implemented as: `GET /wp-json/dual-native/v1/posts/123`

Returns MR with CID:
```json
{
  "rid": 123,
  "cid": "sha256-abc123...",
  "title": "Article Title",
  "blocks": [...]
}
```

### MCP Tools

**1. `list_posts`** (uses DNC)
```json
{
  "name": "list_posts",
  "description": "List all WordPress posts with CIDs for change detection",
  "inputSchema": {
    "type": "object",
    "properties": {
      "since": {"type": "string", "description": "ISO timestamp"},
      "status": {"type": "string", "enum": ["draft", "publish", "any"]}
    }
  }
}
```

Implemented as: `GET /wp-json/dual-native/v1/catalog?since=...&status=...`

**2. `insert_block`** (Structural Mutation)
```json
{
  "name": "insert_block",
  "description": "Safely insert a content block at a specific position",
  "inputSchema": {
    "type": "object",
    "properties": {
      "post_id": {"type": "number"},
      "position": {"type": "string", "enum": ["append", "prepend", "index"]},
      "index": {"type": "number"},
      "block": {"type": "object"},
      "if_match_cid": {"type": "string", "description": "CID for optimistic concurrency"}
    },
    "required": ["post_id", "block"]
  }
}
```

Implemented as:
```http
POST /wp-json/dual-native/v1/posts/123/blocks
If-Match: "sha256-abc123..."

{
  "insert": "index",
  "index": 2,
  "block": {"type": "core/heading", "level": 2, "content": "Key Takeaways"}
}
```

Returns:
- **200 OK** with updated MR and new CID (success)
- **412 Precondition Failed** if CID mismatch (conflict detected)

**3. `read_post`** (Zero-Fetch Optimized)
```json
{
  "name": "read_post",
  "description": "Read a post's MR, with zero-fetch if CID matches",
  "inputSchema": {
    "type": "object",
    "properties": {
      "post_id": {"type": "number"},
      "if_none_match_cid": {"type": "string", "description": "CID for conditional GET"}
    },
    "required": ["post_id"]
  }
}
```

Implemented as:
```http
GET /wp-json/dual-native/v1/posts/123
If-None-Match: "sha256-abc123..."
```

Returns:
- **200 OK** with MR (content changed)
- **304 Not Modified** (zero-fetch: content unchanged)

---

## 4. Agent Workflow: Safe Multi-Step Edit

**Scenario:** Claude (via MCP) needs to add a "Key Takeaways" section to a draft article.

```
1. Agent: list_posts(status="draft")
   → Returns catalog with CIDs

2. Agent: Identifies post_id=123, cid="sha256-abc"

3. Agent: read_post(post_id=123, if_none_match_cid="sha256-abc")
   → Returns 304 (already cached, zero-fetch)

4. Agent: Analyzes cached MR, decides to insert heading at index 2

5. Agent: insert_block(
     post_id=123,
     position="index",
     index=2,
     block={type: "core/heading", level: 2, content: "Key Takeaways"},
     if_match_cid="sha256-abc"
   )
   → System checks CID matches
   → Inserts block
   → Returns 200 with new cid="sha256-xyz"

6. Agent: Updates local cache with new CID

7. Agent: insert_block(
     post_id=123,
     position="index",
     index=3,
     block={type: "core/paragraph", content: "Summary point 1..."},
     if_match_cid="sha256-xyz"  // Use NEW CID from previous step
   )
   → Success
```

**Safety Properties:**
- **No HTML corruption**: Agent never touches HTML, works on blocks
- **No lost updates**: If human edited between steps 3-5, step 5 returns 412, agent retries
- **No token waste**: Zero-fetch at step 3 saved 80% bandwidth
- **Verifiable**: Content-Digest headers let agent verify integrity

---

## 5. MCP Server Design Patterns

### Pattern A: Level 3 (Read-Only MCP Server)

**Use Case:** Search, analytics, monitoring agents that only need to read data

**Capabilities:**
- `read_resource`: Fetch MR with CID
- `list_resources`: Query DNC
- Conditional GET for zero-fetch

**Examples:**
- Content analysis agents (sentiment, summarization)
- Search indexers
- Monitoring dashboards
- Archival systems

**Implementation:** Requires Level 3 Dual-Native (CID + DNC)

### Pattern B: Level 4 (Read/Write MCP Server)

**Use Case:** Autonomous agents that edit, create, or delete content

**Capabilities:**
- All Level 3 capabilities
- `call_tool` for Structural Mutations
- Optimistic concurrency with `If-Match`
- Write responses include new CID for chaining

**Examples:**
- Content editing agents (like Claude Code)
- Automated workflow agents (approve, publish, schedule)
- Data entry agents
- Configuration management agents

**Implementation:** Requires Level 4 Dual-Native (Structural Mutations + Drift Prevention)

---

## 6. Key Insights

1. **MCP is the "Why"**: The Model Context Protocol is the primary **consumer** of the Dual-Native Pattern in the AI era.

2. **Dual-Native is the "How"**: The pattern provides the architectural foundation MCP servers need:
   - Structured MR (not HTML)
   - CID validators (for zero-fetch and concurrency)
   - Structural Mutations (for safe writes)
   - Drift Prevention (for reliability)

3. **Level 4 is the Target**: Read-only MCP servers (Level 3) are useful, but **agentic MCP servers require Level 4**.

4. **Universal Pattern**: While WordPress demonstrates this for CMSs, the same pattern applies to:
   - **Fintech**: Stripe MCP server (create charges, refunds)
   - **Version Control**: GitHub MCP server (commit code, open PRs)
   - **IoT**: Device control MCP servers (set temperature, lock doors)
   - **Databases**: Query and mutation MCP servers

---

## 7. Adoption Path

### Phase 1: Discovery (Level 2)
- Add bidirectional links (HR ↔ MR)
- Enable basic MCP `read_resource`

### Phase 2: Optimization (Level 3)
- Add CID validators
- Publish DNC
- Enable MCP `list_resources` and zero-fetch

### Phase 3: Agentic (Level 4)
- Implement Structural Mutations
- Add `If-Match` concurrency
- Enable MCP `call_tool` for writes

**Timeline:** Most organizations can achieve Level 4 in 2-6 months, depending on existing API maturity.

---

## 8. Conclusion

**The Model Context Protocol is the killer app for Dual-Native.**

Without Dual-Native:
- MCP servers are fragile (HTML scraping)
- MCP writes are dangerous (corruption risk)
- MCP is expensive (high token costs)

With Dual-Native:
- MCP servers are robust (structured MR)
- MCP writes are safe (validated mutations)
- MCP is efficient (zero-fetch optimization)

**Recommendation:** Organizations building MCP servers should adopt the Dual-Native Pattern as their architectural foundation. Start with Level 3 (read-only) and evolve to Level 4 (agentic) as write use cases mature.

**Reference Implementation:** WordPress DNI (see [wordpress.md](wordpress.md))
