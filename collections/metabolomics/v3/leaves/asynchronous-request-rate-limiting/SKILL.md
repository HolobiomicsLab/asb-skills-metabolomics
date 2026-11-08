---
name: asynchronous-request-rate-limiting
description: Use when an asynchronous Python client queries a rate-limited web service
  (e.g. PubChem at ~5 req/s) during batch metadata enrichment and requests must be
  spaced without serializing the whole pipeline.
when_to_use_negative:
- The target web service imposes no access-rate limit — plain asynchronous annotation
  with caching suffices and throttling only adds latency.
- Workloads are served from cached results rather than live service queries; the throttler
  is only needed on the request path.
- The codebase is not asynchronous; a simple synchronous sleep-based limiter is the
  wrong fit for `await self.query_the_service` call sites.
tools:
- name: MSMetaEnhancer
  role: 'Host package: its asynchronous annotation loop and web converters (CTS, CIR,
    IDSM, PubChem, BridgeDb) are where the throttler is installed and applied'
  repo: https://github.com/RECETOX/MSMetaEnhancer
- name: Python
  role: Implementation language for the Throttler utility and its pytest unit test
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:9a92ebd3ca4b8def7a21e0a1f7cd45ab577ad7391a929eb1b6db582ce3bb562d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RECETOX/MSMetaEnhancer
  tools:
  - MSMetaEnhancer
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# asynchronous-request-rate-limiting

## Summary

A throttling pattern for asynchronous HTTP clients that paces requests to external chemical web services (e.g. PubChem's ~5 requests/second public API) so the per-second limit is never exceeded while throughput is maximized.

## When to use

Use when an asynchronous Python client queries a rate-limited web service (e.g. PubChem at ~5 req/s) during batch metadata enrichment and requests must be spaced without serializing the whole pipeline.

## When NOT to use

- The target web service imposes no access-rate limit — plain asynchronous annotation with caching suffices and throttling only adds latency.
- Workloads are served from cached results rather than live service queries; the throttler is only needed on the request path.
- The codebase is not asynchronous; a simple synchronous sleep-based limiter is the wrong fit for `await self.query_the_service` call sites.

## Inputs

- Asynchronous converter call path issuing HTTP requests to external services (e.g. PubChem, CIR, CTS, IDSM, BridgeDb)
- Service rate limit specification (e.g. 5 requests per second, i.e. minimum 0.2 s between consecutive requests)

## Outputs

- Throttler module pacing all outgoing service requests within the configured rate limit
- Timestamps of all issued requests for logging and QC
- Pytest unit test asserting inter-request intervals >= 0.2 s for a 5 req/s limit

## How to apply

Implement a Throttler utility (e.g. in MSMetaEnhancer/libs/utils/throttler.py) that spaces HTTP requests issued by web converters so a service's per-second access limit, such as PubChem's roughly 5 requests per second, is never exceeded. Wire the throttler into the converter call path that awaits `self.query_the_service(...)` inside the package's asynchronous annotation loop, pacing bursts but keeping per-request waiting minimal so throughput is maximized subject to the limit. Record the timestamp of every issued request for logging and QC. Validate with a unit test that fires a burst of asynchronous requests under a 5 req/s limit and asserts no inter-request interval is shorter than 0.2 s.

## Related tools

- **MSMetaEnhancer** (Host package: its asynchronous annotation loop and web converters (CTS, CIR, IDSM, PubChem, BridgeDb) are where the throttler is installed and applied) — https://github.com/RECETOX/MSMetaEnhancer
- **Python** (Implementation language for the Throttler utility and its pytest unit test)

## Examples

```
response = await self.query_the_service('MyService', args)
```

## Evaluation signals

- Recorded request timestamps show every interval between consecutive requests to the limited service is >= 0.2 s (i.e. <= 5 requests per second for PubChem).
- The burst test still completes with high throughput — pacing is minimal and requests are spaced, not serialized beyond the limit.
- The new throttler module passes pytest alongside the existing MSMetaEnhancer test suite.
- All web converters route their HTTP calls through the throttled `self.query_the_service` path, so no converter bypasses the limiter.

## Limitations

- The limit value is service-specific and must be supplied per service (PubChem's public API is roughly 5 requests per second); other services need their own thresholds.
- Throttling was introduced specifically for rate-limited services like PubChem; for unrestricted services the article reports caching and the asynchronous approach as the performance mechanisms.
- PubChem-related metadata handling was still incomplete in the discussion (e.g. issue #67 on supporting ISOMERIC_SMILES and CANONICAL_SMILES instead of generic SMILES), so returned SMILES fields may need further refinement.

## Evidence

- [abstract] The package uses asynchronous processing, caching of results, and throttling for rate-limited services to improve performance: [evidence span withheld — anchor status: not_found]
- [intro] Asynchronous implementation: "The app uses asynchronous implementation of annotation process allowing for optimal fetching speed."
- [methods] Converter call path: "response = await self.query_the_service('MyService', args)"
- [readme] Testing framework: [evidence span withheld — anchor status: external_doc]
