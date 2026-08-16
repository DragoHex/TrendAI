# T014. Orchestrator Agent (ADK top-level)

- Coordinates T7→T13 as an ADK workflow: fetch → chunk → extract (parallel map) → synthesize (reduce) → select template → render.
- Owns run-level state, error aggregation, partial-failure policy (e.g., proceed with available sources, annotate report with "data unavailable for X").
- Exposes a single entrypoint (e.g. `generate_report(config)`) for scheduler/UI/API to call.
