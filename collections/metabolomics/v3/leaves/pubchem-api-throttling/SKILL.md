---
name: pubchem-api-throttling
description: Use when writing or calling a WebConverter that queries a webservice
  enforcing a programmatic access-rate limit (e.g. PubChem's ~5 requests/second),
  inside MSMetaEnhancer's asynchronous annotation loop.
when_to_use_negative:
- The target webservice publishes no programmatic rate limit (e.g. local computed
  converters such as RDKit, which compute identifiers like molecular weight from SMILES
  without network calls)
- Requests are already deduplicated by the package's result caching, and only a single
  unique request per compound would be issued
- You need to retry or back off after an HTTP error rather than proactively space
  requests — that is a different error-handling concern
tools:
- name: MSMetaEnhancer
  role: Host package providing the asynchronous annotation loop, WebConverter template,
    query_the_service call site, and utils module conventions where the Throttler
    lives
  repo: github.com/RECETOX/MSMetaEnhancer
- name: PubChem
  role: Rate-limited remote service (~5 requests/second) that the throttling mechanism
    keeps the converter within
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:55b5f83e84bf1d12a69c95beee0a319009bf7c29069ffaca2d17a5620cf3c0e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSMetaEnhancer
  - PubChem
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pubchem-api-throttling

## Summary

A client-side rate-limiting technique for asynchronous web converters in MSMetaEnhancer, pacing HTTP requests to rate-limited services such as PubChem (~5 requests per second) while maximizing annotation throughput.

## When to use

Use when writing or calling a WebConverter that queries a webservice enforcing a programmatic access-rate limit (e.g. PubChem's ~5 requests/second), inside MSMetaEnhancer's asynchronous annotation loop.

## When NOT to use

- The target webservice publishes no programmatic rate limit (e.g. local computed converters such as RDKit, which compute identifiers like molecular weight from SMILES without network calls)
- Requests are already deduplicated by the package's result caching, and only a single unique request per compound would be issued
- You need to retry or back off after an HTTP error rather than proactively space requests — that is a different error-handling concern

## Inputs

- Asynchronous web-converter request stream (query_the_service calls from converters like CTS, CIR, PubChem, IDSM, BridgeDb)
- Per-service rate limit configuration (e.g. 5 requests per second for PubChem's public API)

## Outputs

- Paced sequence of HTTP requests whose inter-request intervals are >= 0.2 s for a 5 req/s limit
- Recorded timestamps of all issued requests for logging and QC purposes

## How to apply

Create a Throttler class in MSMetaEnhancer/libs/utils/throttler.py following the error-handling and utility conventions of the package's utils modules. Implement pacing logic that spaces HTTP requests issued by WebConverters — which make calls via `await self.query_the_service(...)` inside the asynchronous annotation process — so the per-second limit of the service (e.g. 5 requests/s for PubChem) is never exceeded, while keeping throughput as high as the limit permits. Record the timestamp of every issued request so pacing can be logged and QC-checked. Validate with a unit test that instantiates the Throttler with a 5-requests-per-second limit, fires a burst of asynchronous requests, and asserts no interval between consecutive recorded timestamps is shorter than 0.2 s; the module must pass pytest alongside the existing test suite.

## Related tools

- **MSMetaEnhancer** (Host package providing the asynchronous annotation loop, WebConverter template, query_the_service call site, and utils module conventions where the Throttler lives) — github.com/RECETOX/MSMetaEnhancer
- **PubChem** (Rate-limited remote service (~5 requests/second) that the throttling mechanism keeps the converter within)

## Examples

```
pytest MSMetaEnhancer/tests/test_throttler.py  # burst of async requests at limit=5, assert consecutive timestamps spaced >= 0.2 s
```

## Evaluation signals

- Unit test fires a burst of asynchronous requests with a Throttler configured to 5 requests/second and asserts no interval between consecutive recorded timestamps is shorter than 0.2 s
- Emitted request timestamps confirm the service rate limit is respected (inter-request interval >= 0.2 s for 5 req/s)
- The new module passes pytest alongside the existing MSMetaEnhancer test suite
- Annotation throughput over a burst is maximized subject to the limit (requests are spaced at the limit, not slower than necessary)

## Limitations

- The rate limit (~5 requests/s) is a property of PubChem's public API; other services listed in MSMetaEnhancer (CIR, CTS, IDSM, BridgeDb) impose different or unspecified limits, so the Throttler must be configured per service
- Throttling applies to web converters only; computed metadata (e.g. molecular weight from SMILES via RDKit) does not go through the service and needs no pacing
- PubChem metadata handling has known incompleteness in the package (e.g. generic SMILES rather than ISOMERIC_SMILES/CANONICAL_SMILES, issue #67), which throttling does not address

## Evidence

- [methods] For rate-limited services like PubChem, MSMetaEnhancer implements a throttling mechanism designed to maximize performance while staying within the restrictions imposed by the requested webservice.: [evidence span withheld — anchor status: not_found]
- [methods] response = await self.query_the_service('MyService', args): "response = await self.query_the_service('MyService', args)"
- [readme] The app uses asynchronous implementation of annotation process allowing for optimal fetching speed.: "The app uses asynchronous implementation of annotation process allowing for optimal fetching speed."
- [readme] All functionality is tested with the pytest framework.: [evidence span withheld — anchor status: not_found]
