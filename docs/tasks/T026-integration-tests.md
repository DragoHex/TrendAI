# T026. Integration tests

- Full pipeline run against a fixed synthetic dataset (golden dataset) with known expected report-data-object output; catches regressions across the whole map-reduce-render chain.
- Partial-failure simulation: kill one source mid-run, verify graceful degradation and correct annotation in final report.
