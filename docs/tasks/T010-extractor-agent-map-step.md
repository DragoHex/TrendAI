# T010. Extractor Agent (map step)

- ADK agent that, per chunk, calls SLM with map prompt, validates structured output against schema, retries on malformed output (bounded retries, then flags chunk as degraded).
- Attaches provenance (source/chunk id) to every extracted fact for later citation in report.
