# T013. Report Renderer service

- Given (template + report data object) → binds data into template placeholders, generates required charts (as data-driven chart-spec, rendered via any charting library appropriate to the output medium), and slots illustrations (static asset library or generated icons per sector/theme).
- Outputs final artifact in target format(s): HTML page (primary "presentable page") + optional PDF export.
- Rendering is a pure function of (template, data) — no LLM call needed here, keeping this step fast/deterministic/testable.
