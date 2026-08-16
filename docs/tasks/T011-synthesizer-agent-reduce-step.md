# T011. Synthesizer Agent (reduce step)

- ADK agent that merges validated per-chunk extractions into topic-level rollups, then a final cross-topic synthesis, respecting the same context budget at each merge level (hierarchical reduce, not one giant concat).
- Produces final structured "report data object" — the single contract consumed by rendering, decoupled from any LLM-specific formatting.
