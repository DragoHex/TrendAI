# T005. Agent role & tool contract design (ADK)

- Define distinct ADK agents/roles: Orchestrator, Fetcher(s), Extractor (map), Synthesizer (reduce), Template Selector, Renderer-invoker.
- Define each tool's input/output contract (typed, versioned schemas) so tools are swappable without touching agent logic.
- Define retry/backoff and failure-escalation contract per tool (what orchestrator does on partial source failure — degrade gracefully, mark data gap in report rather than block).
