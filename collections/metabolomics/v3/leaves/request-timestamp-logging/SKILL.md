---
name: request-timestamp-logging
description: Use when annotating spectra via web converters against rate-limited public
  APIs (e.g. PubChem ~5 req/s) and you need to record, log, or test the spacing of
  issued requests.
when_to_use_negative:
- The target web service has no documented access-rate limit, so unrestricted asynchronous
  fetching is already safe and throttling would only add latency
- Data comes from local computed converters (e.g. RDKit computing molecular weight
  from SMILES) that issue no HTTP requests, so there is nothing to timestamp
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MSMetaEnhancer
  role: 'Host package: provides the Throttler utility and the asynchronous `query_the_service`
    converter path whose requests are timestamped'
  repo: https://github.com/RECETOX/MSMetaEnhancer
- name: Python
  role: Implementation language for the Throttler class, asyncio-based request pacing,
    and pytest test suite
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:28368aa16adcad9de98d82d12f103797610b3c0ea1901d24aeeaf824c45926bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RECETOX/MSMetaEnhancer
  tools:
  - MSMetaEnhancer
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# request-timestamp-logging

## Summary

Records timestamps of every HTTP request issued by MSMetaEnhancer's asynchronous WebConverters so that request pacing against rate-limited services (e.g. PubChem at ~5 requests/second) can be logged, QC'd, and verified.

## When to use

Use when annotating spectra via web converters against rate-limited public APIs (e.g. PubChem ~5 req/s) and you need to record, log, or test the spacing of issued requests.

## When NOT to use

- The target web service has no documented access-rate limit, so unrestricted asynchronous fetching is already safe and throttling would only add latency
- Data comes from local computed converters (e.g. RDKit computing molecular weight from SMILES) that issue no HTTP requests, so there is nothing to timestamp

## Inputs

- Asynchronous HTTP requests issued from MSMetaEnhancer WebConverters (e.g. PubChem, CIR, CTS, IDSM, BridgeDb) via `await self.query_the_service(...)`
- Service-specific rate limit (e.g. 5 requests per second for PubChem)

## Outputs

- Timestamped record of every issued request for logging/QC
- Paced request stream that respects the service's per-second limit
- pytest unit test asserting inter-request intervals >= 0.2 s at 5 req/s

## How to apply

Implement (or extend) the Throttler in MSMetaEnhancer/libs/utils/throttler.py, instantiated with a per-second limit matching the target service (5 req/s for PubChem). Hook the throttling logic into the asynchronous converter path so every awaited `self.query_the_service` call is spaced so consecutive requests never violate the service's per-second access limit, while throughput is otherwise maximized. On each issued request, append its timestamp to a log for QC purposes. Validate with a pytest test that fires a burst of asynchronous requests through a Throttler configured at 5 req/s and asserts no interval between consecutive recorded timestamps is shorter than 0.2 s.

## Related tools

- **MSMetaEnhancer** (Host package: provides the Throttler utility and the asynchronous `query_the_service` converter path whose requests are timestamped) — https://github.com/RECETOX/MSMetaEnhancer
- **Python** (Implementation language for the Throttler class, asyncio-based request pacing, and pytest test suite)

## Examples

```
response = await self.query_the_service('MyService', args)  # request issued through the Throttler, which records its timestamp
```

## Evaluation signals

- For a Throttler configured at 5 requests/second, no interval between consecutive recorded request timestamps is shorter than 0.2 s
- All issued HTTP requests appear in the timestamp log (no requests bypass the throttler)
- The new module passes pytest alongside the existing MSMetaEnhancer test suite
- Annotation throughput is still maximal within the limit — no unnecessary idle gaps beyond the 0.2 s minimum spacing

## Limitations

- The 0.2 s spacing threshold is specific to services like PubChem (~5 req/s); other services (CIR, CTS, IDSM, BridgeDb) may impose different limits requiring a different Throttler parameter
- Timestamp logging is a QC/logging aid — it records pacing but does not itself handle server-side rejections or retries
- Requires the asynchronous (asyncio) annotation path; it does not apply to purely computed metadata such as molecular weight from SMILES via RDKit

## Evidence

- [abstract] The package uses asynchronous processing, caching of results, and throttling for rate-limited services to improve performance: [evidence span withheld — anchor status: not_found]
- [methods] asynchronous converter query path: "response = await self.query_the_service('MyService', args)"
- [readme] README async fetching rationale: "The app uses asynchronous implementation of annotation process allowing for optimal fetching speed."
