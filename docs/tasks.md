# AI Model for Aggregating AI Trends — Project Plan

## 1. Overview

**Goal:** Build an autonomous agent system that fetches AI-trend data from reliable sources, processes/summarizes it using a local small language model (SLM, e.g. Gemma 4B-class), and renders a polished, template-based report (data + charts + illustrations) — orchestrated via Google ADK with full observability/tracing.

**Core constraints driving the design:**
- SLM has small context window → data pipeline must chunk, pre-aggregate, and route only relevant slices to the model at each step.
- Output must be visually presentable, not raw text — implies a template engine + rendering layer separate from the LLM.
- Agent framework: Google ADK (Agent Development Kit) — orchestration, tool-calling, multi-step planning.
- Observability: every agent step, tool call, and LLM call must be traced (latency, tokens, inputs/outputs, errors).
- Everything below is described in terms of responsibilities and contracts, not specific languages/libraries — pick per team's stack (Python/Go/TS all fit ADK + local inference servers via HTTP/gRPC).

---

## 2. High-Level Architecture

```
[Scheduler/Trigger]
        │
        ▼
[Orchestrator Agent (ADK)] ──tool_call──▶ [Data Fetch Tools] ──▶ [Source Registry]
        │                                        │
        │                                        ▼
        │                              [Raw Data Store (staging)]
        ▼
[Preprocessing/Chunking Service] ──▶ [Context Builder]
        │
        ▼
[SLM Analysis Agent(s)] (map step, per chunk)
        │
        ▼
[Aggregation Agent] (reduce step, cross-chunk synthesis)
        │
        ▼
[Template Selector] ──▶ [Template Registry (fixed N templates)]
        │
        ▼
[Report Renderer] (data binding + charts + illustrations)
        │
        ▼
[Output Store] ──▶ [Delivery: file/webpage/PDF]

   ⎡ Observability/Trace Layer spans every arrow above ⎦
```

---

## 3. Component Breakdown & Task Definitions

### 3.1 Design Phase

**T1. Data source specification**
- Enumerate reliable AI-trend data sources (research labs, engineering blogs, paper feeds, model release notes, community signal sources).
- For each source define: access method (API/scrape/feed), refresh cadence, reliability tier, licensing/ToS constraints, expected schema.
- Deliverable: `sources.yaml`/`sources.json` registry with tier + schema per source.
- The initial registry may retain a small number of free-text metadata fields where useful; when implementing the backend, normalize these into typed fields/enums wherever practical so the registry loads cleanly into Go structs with minimal special-casing.

**T2. Data model definition**
- Define canonical internal schema all sources normalize into: `{topic, subtopic, organization, artifact_type, title, summary, url, published_at, source_id, confidence}` etc.
- Define provenance fields (source, fetch timestamp, raw payload hash) for traceability and later auditing.
- Define the backend-facing schema for the source registry as well, including how free-text fields from T1 such as notes and licensing/ToS notes will later be converted into stricter typed fields, flags, or enums.

**T3. Report template taxonomy**
- Decide fixed set of N report templates (e.g.: "Research Trend Timeline", "Model Release Radar", "AI Tooling Landscape", "Ecosystem Overview", "Comparative Month-over-Month").
- For each template define: required data fields, chart types needed (bar/line/heatmap/map), illustration slots, narrative-block slots, min/max data volume it supports.
- Deliverable: `templates/registry.yaml` describing selection criteria (rules the Template Selector will use — e.g. "if data spans ≥3 regions → Regional Snapshot eligible").

**T4. Context-window budget design (SLM-specific)**
- Define token budget per SLM call (fixed ceiling, leave headroom for system prompt + output).
- Design chunking strategy: split raw data by natural boundaries (per-sector, per-region, per-week) sized to fit budget.
- Design map-reduce prompt contracts:
  - Map prompt: strict schema-in, schema-out (JSON) so partial summaries are machine-mergeable.
  - Reduce prompt: takes N partial summaries (themselves compact) + produces final structured synthesis.
- Define a "compression ladder": raw data → per-chunk structured extraction → per-topic rollup → final synthesis, each step shrinking token footprint before reaching next SLM call.
- Deliverable: `context_policy.md` — chunk size limits, overlap rules, max hierarchy depth, fallback behavior if a chunk still overflows (recursive sub-chunking).

**T5. Agent role & tool contract design (ADK)**
- Define distinct ADK agents/roles: Orchestrator, Fetcher(s), Extractor (map), Synthesizer (reduce), Template Selector, Renderer-invoker.
- Define each tool's input/output contract (typed, versioned schemas) so tools are swappable without touching agent logic.
- Define retry/backoff and failure-escalation contract per tool (what orchestrator does on partial source failure — de[118;1:3ugrade gracefully, mark data gap in report rather than block).

**T6. Observability design**
- Decide trace schema: one root trace per report-generation run; child spans per agent step/tool call/LLM call.
- Define required span attributes: source id, chunk id, token counts (in/out), latency, model name/version, cache hit/miss, error class.
- Decide export target (any OTEL-compatible backend) and dashboards needed (run success rate, per-source latency, SLM token efficiency, template selection distribution).

**Design phase exit criteria:** all schemas/registries drafted and reviewed; a dry-run walkthrough of one full pipeline path on paper/whiteboard.

---

### 3.2 Backend Phase

**T7. Source Fetch Tools**
- Implement one fetch tool per source tier, each normalizing to canonical schema (T2) before returning.
- Implement source-level caching (avoid re-fetching unchanged data) and a raw-payload staging store with hashes for reproducibility.
- Unit-testable in isolation with mocked source responses.
- Backend implementation should consume the normalized source-registry schema from T1/T2 and avoid ad hoc branching on free-text registry fields wherever possible.

**T8. Preprocessing/Chunking Service**
- Implement chunker per T4 policy: deterministic, reproducible chunk boundaries given same input + config.
- Implement token-estimation utility calibrated to the target SLM's tokenizer (must be accurate, not approximate, since budget is tight).
- Implement recursive sub-chunk fallback when a natural chunk still exceeds budget.

**T9. Local SLM serving layer**
- Stand up local inference endpoint(s) for the SLM (e.g. Gemma-class model) behind a stable HTTP/gRPC interface, decoupled from the agent code so model can be swapped/upgraded independently.
- Implement structured-output enforcement (JSON schema constrained decoding or strict parse+retry loop) since map/reduce steps depend on machine-parseable output.
- Implement concurrency/batching strategy appropriate to local hardware (queue + worker pool) to keep throughput acceptable under low-context multi-call design.

**T10. Extractor Agent (map step)**
- ADK agent that, per chunk, calls SLM with map prompt, validates structured output against schema, retries on malformed output (bounded retries, then flags chunk as degraded).
- Attaches provenance (source/chunk id) to every extracted fact for later citation in report.

**T11. Synthesizer Agent (reduce step)**
- ADK agent that merges validated per-chunk extractions into topic-level rollups, then a final cross-topic synthesis, respecting the same context budget at each merge level (hierarchical reduce, not one giant concat).
- Produces final structured "report data object" — the single contract consumed by rendering, decoupled from any LLM-specific formatting.

**T12. Template Selector**
- Deterministic (rule-based) or lightweight-model-assisted logic that scores available templates (T3 registry) against the report data object's shape (# regions, # sectors, time range, data completeness) and picks best-fit template(s).
- Must be explainable — log which rule fired, for debugging and trust.

**T13. Report Renderer service**
- Given (template + report data object) → binds data into template placeholders, generates required charts (as data-driven chart-spec, rendered via any charting library appropriate to the output medium), and slots illustrations (static asset library or generated icons per sector/theme).
- Outputs final artifact in target format(s): HTML page (primary "presentable page") + optional PDF export.
- Rendering is a pure function of (template, data) — no LLM call needed here, keeping this step fast/deterministic/testable.

**T14. Orchestrator Agent (ADK top-level)**
- Coordinates T7→T13 as an ADK workflow: fetch → chunk → extract (parallel map) → synthesize (reduce) → select template → render.
- Owns run-level state, error aggregation, partial-failure policy (e.g., proceed with available sources, annotate report with "data unavailable for X").
- Exposes a single entrypoint (e.g. `generate_report(config)`) for scheduler/UI/API to call.

**T15. Observability instrumentation**
- Wrap every tool call, agent step, and SLM call with trace spans per T6 schema.
- Add structured logging correlated with trace ids.
- Implement a lightweight run-summary emitted at end of each pipeline run (sources used, chunks processed, template chosen, total tokens, wall-clock time, degraded steps).

**Backend phase exit criteria:** end-to-end pipeline run on real/sample data produces a rendered report artifact; traces visible in observability backend for the full run.

---

### 3.3 UI Phase

**T16. Trigger/Config UI**
- Simple interface (web form or CLI, framework-agnostic) to configure a report run: date range, region/sector filters, template preference (optional override), source selection.
- Calls orchestrator entrypoint (T14) and surfaces run status.

**T17. Run Monitoring UI**
- Live/near-live view of an in-progress run: current pipeline stage, sources fetched, chunks processed/remaining, any degraded/failed steps.
- Pulls from observability layer (T6/T15) rather than duplicating state tracking — UI is a consumer of trace/log data, not a second source of truth.

**T18. Report Viewer UI**
- Renders/embeds the final HTML report artifact (T13 output); supports browsing past reports, filtering by template type/date/region.
- Download/export action (PDF, shareable link).

**T19. Template & Source Management UI (admin)**
- CRUD-style view over the template registry (T3) and source registry (T1) so non-engineers can enable/disable sources or tweak template selection thresholds without code changes.

**UI phase exit criteria:** a user can configure a run, watch it progress, and view the resulting report end-to-end through the interface alone.

---

### 3.4 Integration Phase

**T20. Scheduler integration**
- Wire recurring triggers (e.g., weekly AI trend digest) into the orchestrator entrypoint; ensure idempotency (re-running for same period doesn't duplicate/corrupt stored reports).

**T21. Storage/versioning integration**
- Persist report data objects (T11 output) and rendered artifacts (T13 output) with versioning, so historical comparisons (e.g. month-over-month templates) can pull prior runs' data objects directly rather than re-deriving.

**T22. Observability backend integration**
- Finalize export pipeline from ADK trace/span emission into chosen OTEL-compatible backend; build the dashboards specified in T6.

**T23. Access/delivery integration**
- Hook report viewer to distribution channels if needed (email digest, shared drive, internal portal link) — kept as a pluggable delivery tool, not hardcoded into the renderer.

**Integration phase exit criteria:** scheduled run executes unattended, produces versioned report, sends/notifies delivery channel, and full trace is queryable in the observability dashboard.

---

### 3.5 Testing Phase

**T24. Unit tests**
- Fetch tools: mocked source responses → correct normalization, correct handling of malformed/missing source data.
- Chunker: deterministic chunk boundaries, token-budget compliance, correct recursive sub-chunk fallback triggering.
- Template Selector: given synthetic report-data-object shapes, correct template chosen per rule table; explainability log correctness.
- Renderer: given fixed (template, data) pairs, byte-stable/structurally-stable output (snapshot testing).

**T25. Contract tests**
- SLM map/reduce prompts: validate structured-output schema compliance across a battery of representative and adversarial (truncated/ambiguous) inputs; measure retry rate.
- Tool I/O schemas: contract tests between orchestrator and each tool to catch breaking schema changes early.

**T26. Integration tests**
- Full pipeline run against a fixed synthetic dataset (golden dataset) with known expected report-data-object output; catches regressions across the whole map-reduce-render chain.
- Partial-failure simulation: kill one source mid-run, verify graceful degradation and correct annotation in final report.

**T27. Performance/load tests**
- Measure SLM throughput under realistic chunk volume (ecosystem-scale AI data) on target hardware; validate context-budget policy holds under worst-case chunk sizes.
- Measure end-to-end run latency; identify bottleneck stage (likely SLM inference) and validate batching/concurrency settings (T9).

**T28. Observability validation**
- Verify every pipeline run produces a complete, correctly-correlated trace (no orphan spans, no missing attributes) against the T6 schema.
- Verify dashboards reflect real run data accurately (spot-check against raw logs).

**T29. UX/report quality review**
- Human review pass: sample rendered reports for factual accuracy vs. source data (spot-check against raw sources), chart correctness, template appropriateness, illustration relevance.
- Establish a lightweight rubric (accuracy, completeness, visual clarity, narrative coherence) for repeatable quality scoring across template types.

**T30. Regression/CI gate**
- Wire T24–T26 into CI so any change to chunking policy, prompt contracts, or templates is validated automatically before merge.

**Testing phase exit criteria:** golden-dataset run passes integration tests, performance within target SLA, observability fully validated, and a human quality review signs off on at least one report per template type.

---

## 4. Milestones Summary

| Milestone | Depends on | Exit signal |
|---|---|---|
| M1 — Design sign-off | T1–T6 | All registries/schemas drafted & reviewed |
| M2 — Backend walking skeleton | T7–T15 | One end-to-end run produces a report + full trace |
| M3 — UI usable | T16–T19 | User can run/monitor/view via UI alone |
| M4 — Integrated & scheduled | T20–T23 | Unattended scheduled run, delivered, traced |
| M5 — Tested & hardened | T24–T30 | Golden-dataset + perf + quality review pass |

## 5. Key Risks & Mitigations

- **SLM context limits causing lossy synthesis** → hierarchical map-reduce (T4/T11) with strict structured intermediate formats, not free-text summaries, to minimize compounding information loss.
- **Unreliable/rate-limited sources** → tiered source registry (T1) + graceful degradation policy (T5/T14) so one bad source doesn't block the whole report.
- **Template mismatch with unusual data shapes** → explainable rule-based selector (T12) with a documented fallback template (generic overview) for edge cases.
- **Silent quality drift from a local small model** → contract tests (T25) + recurring human quality review (T29) as an ongoing gate, not a one-time check.
