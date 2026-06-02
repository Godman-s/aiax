# AIAX — Agent Interface & Agent Experience

> *"What an agent encounters when it shows up wanting to evaluate, navigate, decide, or transact."*

**Status:** v0.1 draft · **License:** CC BY 4.0 (spec) · **Canonical URL:** kognai.ai/aiax

---

AIAX is a design discipline and a protocol for **agent-facing product surfaces**. Where UI/UX optimizes for human eyes and hands, AIAX optimizes the parallel surface for an agent reader, with **minimum tokens to maximum decision-relevant information** as its hard constraint.

A typical SaaS homepage costs an agent 5,000–15,000 tokens to evaluate. The same decision-relevant content can be delivered in 150–300 tokens via an AIAX surface. As agent operation cost becomes a meaningful share of unit economics, the surfaces that respect agent attention are the surfaces that get used.

## The four tiers

| Tier | What | Path | Target size |
|------|------|------|-------------|
| 0 | Manifest | `/.well-known/aiax.json` | ~200 tokens |
| 1 | Capability sections | `/aiax/sections/<section>.json` | 200–1,000 tokens each |
| 2 | Action endpoints | `/aiax/mcp` (MCP) | callable |
| 3 | Reference data | `/aiax/reference/` | deep, on-demand |

The tiering is ordered by **decision relevance**, not by topic. Agents fetch Tier 0 always, Tier 1 when investigating, Tier 2 when transacting, Tier 3 when integrating.

## In this directory

- **`SPEC.md`** — the v0.1 specification (read this first)
- **`BENCHMARK.md`** — the token-cost benchmark methodology
- **`schemas/manifest.schema.json`** — JSON Schema for Tier 0
- **`schemas/section.schema.json`** — JSON Schema for Tier 1
- **`examples/aiax.json`** — example Tier 0 manifest (Invoica)
- **`examples/sections/pricing.json`** — example Tier 1 section
- **`CHANGELOG.md`** — version history

## What AIAX is *not*

- Not a replacement for MCP, OpenAPI, or llms.txt — it composes with all three
- Not a registry or directory protocol — that's LAX
- Not a negotiation protocol — that's PACT
- Not a payment protocol — that's x402 / DRS
- Not a proprietary moat — the spec is permissive by design; the point is the discipline, not the namespace

## Relationship to the Godman Protocols

AIAX sits alongside PACT, LAX, DRS, and SCORE. All share the Godman Protocols' **Retrieval-First Principle** — privilege structured addressable retrieval over full-document delivery at every layer of the stack.

| Protocol | Layer | What it does |
|----------|-------|--------------|
| **AIAX** | Product surface | Agent-readable presentation of a product |
| **LAX** | Discovery | Agent-to-agent capability publication |
| **PACT** | Negotiation | Capability token signing |
| **DRS** | Settlement | On-chain deal receipt (EAS on Base) |
| **SCORE** | Reputation | Sovereign Constitutional Output Rating |

## Reference implementation

Invoica (`invoica.ai`) will be the first published AIAX implementation, with a token-cost benchmark vs the human surface.

## Contributing

This is a v0.1 draft. Comments, issues, and adoption signals welcome via the canonical channel at kognai.ai/aiax (publication URL).
