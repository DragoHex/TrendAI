# T008. Preprocessing/Chunking Service

- Implement chunker per T4 policy: deterministic, reproducible chunk boundaries given same input + config.
- Implement token-estimation utility calibrated to the target SLM's tokenizer (must be accurate, not approximate, since budget is tight).
- Implement recursive sub-chunk fallback when a natural chunk still exceeds budget.
