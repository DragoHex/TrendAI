# T017. Run Monitoring UI

- Live/near-live view of an in-progress run: current pipeline stage, sources fetched, chunks processed/remaining, any degraded/failed steps.
- Pulls from observability layer (T6/T15) rather than duplicating state tracking — UI is a consumer of trace/log data, not a second source of truth.
