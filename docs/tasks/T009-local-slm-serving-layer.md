# T009. Local SLM serving layer

- Stand up local inference endpoint(s) for the SLM (e.g. Gemma-class model) behind a stable HTTP/gRPC interface, decoupled from the agent code so model can be swapped/upgraded independently.
- Implement structured-output enforcement (JSON schema constrained decoding or strict parse+retry loop) since map/reduce steps depend on machine-parseable output.
- Implement concurrency/batching strategy appropriate to local hardware (queue + worker pool) to keep throughput acceptable under low-context multi-call design.
