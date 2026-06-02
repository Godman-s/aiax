# Changelog — AIAX

All notable changes to the AIAX specification.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

---

## [0.1.1] — 2026-06-01

### Fixed

- **Tier 1 section path contradiction.** SPEC §4.2 + §3 and the README tier table
  declared the flat form `/aiax/<section>.json`, while the canonical example
  manifest (`examples/aiax.json`) and the first reference implementation
  (Invoica, `invoica.ai`, live) both use the nested form
  `/aiax/sections/<section>.json`. The manifest schema constrains only
  `entry_points[].id`, not the `url`, so it never refereed the conflict and both
  forms validated. Resolved in favor of the **nested** form to match the example,
  the schema-permissive reality, and the deployed reference implementation — the
  v0.1 reference implementation set the de-facto convention. No implementer change
  required for surfaces already following the example.

## [0.1.0] — 2026-05-21

### Added

- Initial public draft (TICKET-063)
- Four-tier surface architecture: manifest → capability sections → MCP endpoints → reference data
- Tier 0 manifest schema (`schemas/manifest.schema.json`)
- Tier 1 capability section schema (`schemas/section.schema.json`)
- Token-cost benchmark methodology (`BENCHMARK.md`)
- Example Tier 0 manifest using Invoica (`examples/aiax.json`)
- Example Tier 1 capability section (`examples/sections/pricing.json`)
- Composition guidance for MCP, OpenAPI, llms.txt, and `.well-known/`
- Relationship mapping to the rest of the Kognai protocol family (PACT, LAX, DRS, SCORE)
- Explicit out-of-scope list for v0.2 (compact binary encoding, federation, per-agent personalization, i18n)

### Notes

- v0.x is the pre-stable series; breaking changes may occur before v1.0
- Reference implementation lands on Invoica in Phase 2 (post PACT-Helixa per TICKET-063)
