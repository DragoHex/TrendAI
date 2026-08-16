# T007. Source Fetch Tools

- Implement one fetch tool per source tier, each normalizing to canonical schema (T2) before returning.
- Implement source-level caching (avoid re-fetching unchanged data) and a raw-payload staging store with hashes for reproducibility.
- Unit-testable in isolation with mocked source responses.
- Backend implementation should consume the normalized source-registry schema from T001/T002 and avoid ad hoc branching on free-text registry fields wherever possible.
