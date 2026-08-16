# T024. Unit tests

- Fetch tools: mocked source responses → correct normalization, correct handling of malformed/missing source data.
- Chunker: deterministic chunk boundaries, token-budget compliance, correct recursive sub-chunk fallback triggering.
- Template Selector: given synthetic report-data-object shapes, correct template chosen per rule table; explainability log correctness.
- Renderer: given fixed (template, data) pairs, byte-stable/structurally-stable output (snapshot testing).
