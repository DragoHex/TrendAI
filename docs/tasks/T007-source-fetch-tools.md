# T007. Source Fetch Tools

- Implement one fetch tool per source tier, each normalizing to canonical schema (T2) before returning.
- Implement source-level caching (avoid re-fetching unchanged data) and a raw-payload staging store with hashes for reproducibility.
- Unit-testable in isolation with mocked source responses.
