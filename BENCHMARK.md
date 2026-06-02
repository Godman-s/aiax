# AIAX Token-Cost Benchmark — Methodology v0.1

The AIAX discipline is sharp because it is measurable. This document defines the methodology for measuring an AIAX surface's token efficiency vs the human-facing surface of the same product.

> *Success target (per TICKET-063): ≥80% token reduction for equivalent information retrieval.*

---

## 1. What the benchmark measures

The **AIAX Token Reduction Ratio (ATRR)** for a given task:

```
ATRR = 1 - (tokens_via_aiax / tokens_via_human_surface)
```

Where:

- `tokens_via_human_surface` = total LLM context tokens required for a representative agent to complete the task using only the human-facing surface (HTML/CSS/JS rendering)
- `tokens_via_aiax` = total LLM context tokens required for the same agent to complete the same task using the AIAX surface

An ATRR of `0.80` means the AIAX surface delivers the same task completion using 20% of the tokens.

---

## 2. The representative agent

To enable cross-product comparison, the benchmark is run against a **reference agent configuration**:

- **Model:** `claude-sonnet-4-20250514` (or the contemporary equivalent named in the benchmark publication)
- **Toolset:** HTTP fetch + HTML-to-text conversion + plain LLM reasoning
- **No prior knowledge of the product** — system prompt declares the agent has never visited before
- **Greedy decoding (temperature 0)** for reproducibility
- **Token counter:** Anthropic's published tokenizer for the chosen model

Implementations MAY also publish results for alternative agent configurations (e.g. Haiku, GPT-class models, open-weight models) for additional signal, but the reference configuration is the headline number.

---

## 3. The benchmark task set

A v0.1 benchmark publishes ATRR for at least **three distinct task categories**:

### 3.1 Evaluation
*"Is this product relevant to my principal's goal X?"*

The agent must answer yes/no with a one-sentence justification, citing the product's stated purpose.

### 3.2 Pricing comprehension
*"What does it cost to perform action Y once? Once a month?"*

The agent must return a structured `{ one_time, monthly, currency }` triple.

### 3.3 Capability check
*"Does this product support action Z?"*

The agent must return yes/no plus a pointer to the relevant capability documentation or endpoint.

Each task is run **independently** — no context bleed across tasks. Each is run three times and the median is reported.

---

## 4. Counting tokens

**Tokens counted in the numerator and denominator:**

- All input tokens to the LLM, including the system prompt, the task instruction, and every fetched document concatenated into the context
- All output tokens produced by the LLM during the task

**Tokens NOT counted:**

- Network overhead bytes (HTTP headers, TLS handshake)
- Pre-cached results (each run is cold-start)
- The benchmark harness itself

**Multiple fetches are summed.** If the agent fetches the manifest (180 tokens), then a Tier 1 section (520 tokens), then asks the model to produce the answer (45 tokens output), the total is `180 + 520 + 45 = 745` tokens.

---

## 5. The human-surface baseline

The human-surface baseline is measured by the **same agent**, with the **same toolset**, using only:

- The product's main URL (e.g. `https://invoica.ai/`)
- Whatever pages the agent organically follows via links found in the HTML
- Standard HTML-to-text conversion

The agent is given a hard cap of **30,000 input tokens** for the human-surface baseline. If the task cannot be completed within the cap, it is recorded as `INCOMPLETE` and the ATRR for that task is reported as `≥ 1 - (tokens_via_aiax / 30000)`.

---

## 6. The published benchmark document

A v0.1-conformant AIAX implementation publishes a benchmark document at `/aiax/benchmark.json` with the following structure:

```json
{
  "aiax_version": "0.1",
  "benchmark_methodology_version": "0.1",
  "measured_at": "2026-MM-DDTHH:MM:SSZ",
  "reference_agent": {
    "model": "claude-sonnet-4-20250514",
    "temperature": 0,
    "tokenizer": "anthropic-claude-4"
  },
  "tasks": [
    {
      "category": "evaluation",
      "task": "Is this product relevant to invoicing AI agents?",
      "tokens_via_aiax": 412,
      "tokens_via_human_surface": 8740,
      "atrr": 0.953,
      "atrr_meets_target": true
    }
  ],
  "summary": {
    "median_atrr": 0.91,
    "tasks_meeting_target": 3,
    "tasks_total": 3,
    "headline_claim": "AIAX cuts agent evaluation costs by 91% on this surface."
  },
  "reproducibility": {
    "harness_url": "https://github.com/<org>/<repo>",
    "harness_commit": "<sha>",
    "raw_logs_url": "/aiax/benchmark/logs/<measured_at>/"
  }
}
```

---

## 7. Honesty rules

The benchmark only has value if it is honest. Conformant implementations:

- MUST publish the harness source so any third party can reproduce
- MUST publish raw transcripts (input + output for every task run)
- MUST NOT cherry-pick tasks — the three required categories are minimum, not maximum
- MUST re-run the benchmark when the AIAX surface materially changes (e.g. new capability sections added)
- SHOULD publish results for failed runs alongside passing ones

A published benchmark that does not reproduce within ±10% on a third-party rerun is grounds for de-listing from any AIAX registry.

---

## 8. Open issues for v0.2

- Cross-product comparison framework (today the benchmark is intra-product; an "AIAX leaderboard" would need normalization)
- Latency dimension (tokens are not the only cost — wall-clock matters too)
- Per-task-class weighting (some tasks are more common in real agent traffic than others)
- Adversarial benchmarking (agents that try to game the surface)

These will be addressed in v0.2 informed by results from the first wave of published benchmarks.

---

*This document accompanies AIAX SPEC v0.1.0. Comments welcome via github.com/Godman-s/aiax.*
