# T002. Data model definition

- Define canonical internal schema all sources normalize into: `{topic, subtopic, organization, artifact_type, title, summary, url, published_at, source_id, confidence}` etc.
- Define provenance fields (source, fetch timestamp, raw payload hash) for traceability and later auditing.
- Define the backend-facing schema for the source registry as well, including how free-text fields from T001 such as notes and licensing/ToS notes will later be converted into stricter typed fields, flags, or enums.
