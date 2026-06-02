# AIAX v0.1 — Specification

**AIAX** — Agent Interface & Agent Experience
**Version:** 0.1.0
**Status:** Draft for public review
**Date:** 2026-05-21
**Published at:** kognai.ai/aiax

---

## Abstract

AIAX is a design discipline and an associated protocol for **agent-facing product surfaces**. Where UI/UX optimizes a product for human eyes and hands, AIAX optimizes the parallel surface for an agent reader — with **minimum tokens to maximum decision-relevant information** as its hard constraint.

This document specifies v0.1 of the AIAX protocol: a four-tier layered surface that a product publishes for visiting agents, composing with existing standards (MCP, OpenAPI, llms.txt, .well-known) rather than replacing them.

---

## 1. Motivation

Agents are an increasingly large share of web traffic and a fast-growing share of decision-making. When an agent visits a product surface designed for humans, it must:

1. Fetch HTML, CSS, and often JavaScript
2. Render or parse the DOM
3. Extract the few decision-relevant facts (what does this product do? what does it cost? is it trustworthy?)
4. Discard the rest

The cost is real: a typical SaaS homepage costs an agent 5,000–15,000 tokens of LLM context to evaluate. The same decision-relevant information can be delivered in 150–300 tokens through a structured surface.

AIAX names the design discipline of building that parallel surface. It is the agent-economy equivalent of page-load speed: a measurable, optimizable, A/B-testable property that becomes a competitive advantage as agent operation costs grow.

---

## 2. Scope

**In scope (v0.1):**

- A four-tier layered surface architecture
- The Tier 0 manifest schema at `/.well-known/aiax.json`
- The Tier 1 capability section schema at `/aiax/sections/<section>.json`
- The Tier 2 action endpoint convention (MCP server at `/aiax/mcp`)
- The Tier 3 reference data convention (markdown allowed at `/aiax/reference/`)
- A token-cost benchmark methodology (see `BENCHMARK.md`)

**Out of scope (v0.1, candidates for v0.2):**

- Compact binary encoding / Braille-style shared dictionary
- Cross-site AIAX federation / registry
- Authentication and per-agent personalization
- Negotiation primitives (those live in PACT — see §6.2)
- Settlement primitives (those live in x402 / DRS — see §6.3)

---

## 3. Design principles

The principles are the discipline. The schema is only a v0.1 expression of them.

### 3.1 Retrieval-first, not delivery-first

An AIAX surface delivers a small index and lets the agent retrieve only what it needs. It does not bundle "everything an agent might want" into one document. This is the same principle that underlies Kognai's ASMR internal memory architecture.

### 3.2 Tiered by decision-relevance, not by topic

Tiers are ordered by **how likely a visiting agent is to need them**:

- **Tier 0** (~200 tokens): every agent that visits, every time
- **Tier 1** (200–1,000 tokens each): agents pursuing a specific evaluation
- **Tier 2** (callable): agents that have decided to transact
- **Tier 3** (deep): agents committed to integration

If a v0.1 implementation publishes only Tier 0, it is still AIAX-compliant for evaluation purposes.

### 3.3 Compose, don't replace

AIAX composes with existing agent-facing standards:

- **`/.well-known/`** — AIAX uses the registered well-known URI prefix; it does not invent a new discovery convention
- **MCP (Model Context Protocol)** — Tier 2 action endpoints SHOULD use MCP. AIAX does not redefine the call protocol
- **OpenAPI** — Tier 1 sections MAY reference OpenAPI documents for callable APIs
- **llms.txt** — A site MAY publish both AIAX and llms.txt; AIAX is the more structured, tiered counterpart, llms.txt is the narrative counterpart

### 3.4 Token efficiency as the measurable surface property

Every AIAX surface SHOULD publish a token-cost benchmark (see `BENCHMARK.md`). The benchmark is the proof. A surface that claims AIAX compliance without a benchmark is incomplete.

### 3.5 Generosity over territory

This specification is intentionally permissive. Implementations may extend the schema with custom fields (under an `x-` prefix or namespaced object). The point is widespread adoption of the discipline, not control of the namespace.

---

## 4. Surface architecture

### 4.1 Tier 0 — Manifest

**Path:** `/.well-known/aiax.json`
**Target size:** 100–300 tokens (≈ 400–1,200 bytes)
**Content type:** `application/json`
**Caching:** SHOULD include `Cache-Control` and `ETag`; agents SHOULD honor them

The manifest declares what the product is, what it does, and where the entry points are. Schema: `schemas/manifest.schema.json`.

**Required fields:**

- `aiax_version` — protocol version string (e.g. `"0.1"`)
- `product` — `{ name, one_line_purpose, domain }`
- `entry_points` — array of section descriptors `{ id, url, purpose, est_tokens }`
- `pricing_summary` — one-line free-text or `null` if not applicable
- `trust_signals` — minimal summary `{ score?, pact_endpoint?, drs_receipt_count?, partnerships? }`
- `mcp_endpoint` — URL of the Tier 2 MCP server, or `null`
- `updated_at` — ISO-8601 timestamp

**Optional fields:**

- `llms_txt_url` — pointer to the site's `llms.txt` for the narrative counterpart
- `openapi_url` — pointer to the OpenAPI document if one exists
- `extensions` — namespaced object for custom fields

### 4.2 Tier 1 — Capability sections

**Path:** `/aiax/sections/<section_id>.json` (referenced from the manifest's `entry_points`)
**Target size:** 200–1,000 tokens each
**Content type:** `application/json`

A capability section describes one product capability in enough detail for an agent to decide whether and how to use it. Schema: `schemas/section.schema.json`.

**Required fields:**

- `aiax_version`
- `section_id` — must match the manifest's `entry_points[].id`
- `purpose` — one-paragraph description
- `inputs` — JSON Schema or prose description
- `outputs` — JSON Schema or prose description
- `pricing` — `{ model, amount, currency, rail }` or `null`
- `examples` — array of `{ description, request, response }` (1–3 examples max)
- `pact_mandate_template` — URL to a PACT mandate template, or `null`

**Optional fields:**

- `mcp_tool_name` — name of the MCP tool that performs this capability
- `openapi_operation_id` — for compatibility with existing OpenAPI specs
- `latency_p50_ms` / `latency_p99_ms` — measured response latencies
- `extensions` — namespaced object

### 4.3 Tier 2 — Action endpoints

**Path:** `/aiax/mcp` (or a sub-URL declared in the manifest)
**Protocol:** Model Context Protocol (MCP)

Tier 2 is where decided-to-transact agents make calls. AIAX does not redefine call semantics; it points to an MCP server. Each capability section MAY reference an `mcp_tool_name` to bind the section to a specific MCP tool.

REST-only implementations are acceptable for v0.1 (declare `mcp_endpoint: null` and reference OpenAPI operations in sections), but MCP is the recommended path.

### 4.4 Tier 3 — Reference data

**Path:** `/aiax/reference/` (prefix; structure left to the implementer)
**Content type:** Markdown, JSON, OpenAPI, schema documents — implementer's choice

Tier 3 is for full documentation, schemas, change logs, and similar deep-retrieval artifacts. The constraint relaxes here: agents arriving at Tier 3 have committed to integration. Token-efficiency discipline still applies but the budget is bigger.

---

## 5. Conformance

An implementation is **AIAX v0.1 conformant** if it publishes:

1. A Tier 0 manifest at `/.well-known/aiax.json` that validates against `schemas/manifest.schema.json`
2. At least one Tier 1 capability section per entry in the manifest's `entry_points`
3. A token-cost benchmark following the methodology in `BENCHMARK.md`

Tier 2 (MCP endpoints) and Tier 3 (reference data) are RECOMMENDED but not required for v0.1 conformance.

An implementation is **AIAX v0.1 partial** if it publishes only the Tier 0 manifest. Partial implementations SHOULD publish a roadmap section explaining what is missing and when it is expected.

---

## 6. Relationship to existing Kognai protocols

### 6.1 LAX (Linked Agent eXchange)

LAX advertises an agent's *capabilities* for discovery by other agents. AIAX presents a *product surface* to a visiting agent. Both are retrieval-first; they operate at different layers:

- **LAX:** agent-to-agent capability discovery and negotiation routing
- **AIAX:** product-to-agent surface for evaluation, navigation, and decision

A single deployment may publish both. An agent looking for "an agent that can issue invoices" reads LAX manifests. An agent visiting `invoica.ai` to evaluate it reads AIAX.

### 6.2 PACT (Programmatic Agent Capability Token)

When a visiting agent decides to transact based on an AIAX Tier 1 section, it initiates a PACT negotiation. Tier 1 sections SHOULD include a `pact_mandate_template` URL that pre-fills the negotiation parameters.

### 6.3 DRS (Deal Receipt Standard)

Settlement of a transaction initiated via AIAX → PACT → x402 produces a DRS receipt. Trust signals in the Tier 0 manifest MAY include a `drs_receipt_count` summarizing settled volume.

### 6.4 SCORE

A product's SCORE (Sovereign Constitutional Output Rating) MAY be surfaced in the Tier 0 manifest's `trust_signals.score` field.

---

## 7. Versioning

AIAX uses semantic versioning. v0.x is the pre-stable series; breaking changes MAY occur between v0.x releases. v1.0 will commit to a stable schema.

Implementations declare conformance against a specific version via the `aiax_version` field in every published document.

---

## 8. Security and privacy

### 8.1 No exfiltration

AIAX surfaces are public by design. An implementation MUST NOT include personal data, secrets, or internal identifiers in any AIAX document. Treat published documents as if they were on the homepage.

### 8.2 Rate limiting

Implementations SHOULD apply reasonable rate limits to AIAX endpoints to prevent agent traffic from degrading human experience. Manifest fetches SHOULD be free or near-free. Tier 2 calls SHOULD follow the pricing declared in the relevant section.

### 8.3 Honest pricing

The pricing fields are part of the trust contract. An implementation that publishes one price in AIAX and charges another at Tier 2 is non-conformant and should be flagged in any AIAX registry.

---

## 9. Acknowledgements

AIAX builds on years of work by others. We name the discipline; we do not claim the underlying ideas.

- **Model Context Protocol (Anthropic, 2025)** — Tier 2 builds directly on MCP
- **llms.txt (Jeremy Howard, 2024)** — the narrative-counterpart precedent
- **OpenAPI Initiative** — schema vocabulary for callable interfaces
- **IETF .well-known URI registry (RFC 8615)** — discovery convention
- **Schema.org** — the long history of structured data for non-human readers

AIAX is a contribution to this lineage, not a replacement.

---

## 10. Open issues for v0.1 → v0.2

- Compact binary encoding (Braille-style shared concept dictionary)
- Cross-site federation and an AIAX registry
- Per-agent personalization (auth + state-aware sections)
- Negotiation primitives folded into AIAX vs left in PACT
- Internationalization (single-locale only in v0.1)

Comments welcome via the canonical channel at kognai.ai/aiax.

---

## Change Log

| Version | Date | Notes |
|---------|------|-------|
| 0.1.0   | 2026-05-21 | Initial public draft. |
