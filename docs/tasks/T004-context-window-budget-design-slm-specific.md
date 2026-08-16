# T004. Context-window budget design (SLM-specific)

- Define token budget per SLM call (fixed ceiling, leave headroom for system prompt + output).
- Design chunking strategy: split raw data by natural boundaries (per-sector, per-region, per-week) sized to fit budget.
- Design map-reduce prompt contracts:
  - Map prompt: strict schema-in, schema-out (JSON) so partial summaries are machine-mergeable.
  - Reduce prompt: takes N partial summaries (themselves compact) + produces final structured synthesis.
- Define a "compression ladder": raw data → per-chunk structured extraction → per-topic rollup → final synthesis, each step shrinking token footprint before reaching next SLM call.
- Deliverable: `context_policy.md` — chunk size limits, overlap rules, max hierarchy depth, fallback behavior if a chunk still overflows (recursive sub-chunking).
