---
name: web-converter-integration-testing
description: Use when adding or modifying an MSMetaEnhancer WebConverter that queries
  a throttled public API, and you must verify that asynchronous requests never exceed
  the service's per-second access limit.
when_to_use_negative:
- Computed converters that never touch a webservice (e.g. RDKit computing molecular
  weight from SMILES locally) need no throttling
- Unit-testing pure response parsing or input sanitization logic (e.g. parse_response,
  @escape_single_quotes) does not require the throttler or network access
- Services without published access-rate restrictions, where added pacing only reduces
  annotation throughput
tools:
- name: MSMetaEnhancer
  role: Host package whose asynchronous WebConverters (query_the_service / parse_response)
    are paced by the Throttler and tested with pytest
  repo: github.com/RECETOX/MSMetaEnhancer
- name: pytest
  role: Framework used to run the throttler burst test alongside the existing MSMetaEnhancer
    test suite
- name: PubChem
  role: Rate-limited target webservice (~5 requests/second) used as the reference
    case for throttler validation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:b6eb2b9ddbb28704761d22db8cb0abefb35f527e66f73ae9c7cfd489f411d1d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSMetaEnhancer
  - PubChem
  - pytest
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# web-converter-integration-testing

## Summary

Pattern for integrating and testing MSMetaEnhancer web converters against rate-limited services (e.g. PubChem's ~5 requests/second API) by combining asynchronous request pacing (Throttler), request-timestamp logging, and pytest-based burst tests.

## When to use

Use when adding or modifying an MSMetaEnhancer WebConverter that queries a throttled public API, and you must verify that asynchronous requests never exceed the service's per-second access limit.

## When NOT to use

- Computed converters that never touch a webservice (e.g. RDKit computing molecular weight from SMILES locally) need no throttling
- Unit-testing pure response parsing or input sanitization logic (e.g. parse_response, @escape_single_quotes) does not require the throttler or network access
- Services without published access-rate restrictions, where added pacing only reduces annotation throughput

## Inputs

- MSMetaEnhancer web converter module (e.g. PubChem, CTS, CIR, IDSM, BridgeDb) issuing asynchronous HTTP queries
- Rate-limit specification for the target webservice (e.g. PubChem ~5 requests/second, i.e. 0.2 s minimum inter-request interval)
- pytest test suite of the MSMetaEnhancer repository

## Outputs

- Throttler module at MSMetaEnhancer/libs/utils/throttler.py pacing converter requests
- Log of timestamps of all issued requests for QC purposes
- pytest unit test asserting inter-request interval >= 0.2 s for a 5 req/s limit
- Annotated spectra metadata obtained without violating the service rate limit

## How to apply

Create the Throttler class in MSMetaEnhancer/libs/utils/throttler.py following the error-handling conventions of the package's utils modules, and pace HTTP requests issued by WebConverters so a service like PubChem (about 5 requests per second) is never exceeded while throughput is maximized — for 5 req/s this means no interval between consecutive requests shorter than 0.2 s. Integrate the throttler with the package's asynchronous converter code, where converter methods await `self.query_the_service('MyService', args)` before calling `self.parse_response(response)`, and record timestamps of all issued requests for logging and QC. For validation, write a pytest unit test that instantiates the Throttler with a 5-requests-per-second limit, fires a burst of asynchronous requests, and asserts that no interval between consecutive recorded timestamps is below 0.2 s. Run the new test alongside the existing pytest suite, since all MSMetaEnhancer functionality is tested with the pytest framework.

## Related tools

- **MSMetaEnhancer** (Host package whose asynchronous WebConverters (query_the_service / parse_response) are paced by the Throttler and tested with pytest) — github.com/RECETOX/MSMetaEnhancer
- **pytest** (Framework used to run the throttler burst test alongside the existing MSMetaEnhancer test suite)
- **PubChem** (Rate-limited target webservice (~5 requests/second) used as the reference case for throttler validation)

## Examples

```
asyncio.run(app.annotate_spectra(services, jobs))
```

## Evaluation signals

- Recorded request timestamps show inter-request interval >= 0.2 s when the Throttler is configured for 5 requests/second under an async burst
- The new module and test pass pytest alongside the existing MSMetaEnhancer test suite
- Converter calls still flow through `await self.query_the_service(...)` and responses are parsed via parse_response without errors
- Request timestamps are actually recorded in the optional detailed log file for QC

## Limitations

- The pacing constant is service-specific: 0.2 s spacing is correct for PubChem's ~5 req/s limit but must be re-derived for other services
- PubChem metadata handling has known gaps (generic SMILES instead of ISOMERIC_SMILES/CANONICAL_SMILES, issue #67) that affect converter output regardless of throttling
- CIR behavior for multiple hits (take only first result, issue #69) is not fully specified and is orthogonal to rate limiting

## Evidence

- [abstract] throttling under async annotation: [evidence span withheld — anchor status: not_found]
- [methods] async converter query pattern: "response = await self.query_the_service('MyService', args)"
- [readme] pytest testing convention: [evidence span withheld — anchor status: external_doc]
- [readme] asynchronous annotation entry point: "# run asynchronous annotations of spectra data
asyncio.run(app.annotate_spectra(services, jobs))"
