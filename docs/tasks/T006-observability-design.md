# T006. Observability design

- Decide trace schema: one root trace per report-generation run; child spans per agent step/tool call/LLM call.
- Define required span attributes: source id, chunk id, token counts (in/out), latency, model name/version, cache hit/miss, error class.
- Decide export target (any OTEL-compatible backend) and dashboards needed (run success rate, per-source latency, SLM token efficiency, template selection distribution).
