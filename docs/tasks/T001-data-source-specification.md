# T001. Data source specification

- Enumerate reliable AI-trend data sources (research labs, engineering blogs, paper feeds, model release notes, community signal sources).
- For each source define: access method (API/scrape/feed), refresh cadence, reliability tier, licensing/ToS constraints, expected schema.
- Deliverable: `sources.yaml`/`sources.json` registry with tier + schema per source.
- The initial registry may retain a small number of free-text metadata fields where useful; when implementing the backend, normalize these into typed fields/enums wherever practical so the registry loads cleanly into Go structs with minimal special-casing.
