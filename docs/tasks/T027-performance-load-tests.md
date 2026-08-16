# T027. Performance/load tests

- Measure SLM throughput under realistic chunk volume (national-scale data) on target hardware; validate context-budget policy holds under worst-case chunk sizes.
- Measure end-to-end run latency; identify bottleneck stage (likely SLM inference) and validate batching/concurrency settings (T9).
