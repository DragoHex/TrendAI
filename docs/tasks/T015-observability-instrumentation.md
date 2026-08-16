# T015. Observability instrumentation

- Wrap every tool call, agent step, and SLM call with trace spans per T6 schema.
- Add structured logging correlated with trace ids.
- Implement a lightweight run-summary emitted at end of each pipeline run (sources used, chunks processed, template chosen, total tokens, wall-clock time, degraded steps).
