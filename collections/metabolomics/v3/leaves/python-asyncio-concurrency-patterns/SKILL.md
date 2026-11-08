---
name: python-asyncio-concurrency-patterns
description: Use when batch-annotating many .msp spectra records against external
  HTTP services that enforce access-rate limits, so request throughput must be maximized
  without exceeding a fixed requests-per-second cap.
when_to_use_negative:
- The metadata can be computed locally without a web service (e.g. molecular weight
  from SMILES via RDKit) — no throttling or network concurrency is needed
- Only a handful of records must be annotated, so sequential requests cannot exceed
  the service's rate limit
- The target service does not enforce a documented requests-per-second limit, making
  throttling unnecessary overhead
edam_operation: http://edamontology.org/operation_2409
tools:
- name: MSMetaEnhancer
  role: Package providing the asynchronous Application loop, WebConverters (CTS, CIR,
    IDSM, PubChem, BridgeDb), and the Throttler utility for rate-limited services
  repo: https://github.com/RECETOX/MSMetaEnhancer
- name: Python
  role: Language whose asyncio module underpins the concurrent, await-based query
    pattern (asyncio.run, await self.query_the_service)
- name: matchms
  role: Used for data IO of .msp files and for validating the form of obtained metadata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.04494
    title: 'MSMetaEnhancer: A Python package for mass spectra metadata annotation'
  source_content_hash: sha256:05c495e3e69aa209a739e7d1610fc576b1ecd7446c35856400786d4e1a988333
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RECETOX/MSMetaEnhancer
  tools:
  - MSMetaEnhancer
  - Python
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# python-asyncio-concurrency-patterns

## Summary

Asynchronous, throttled annotation of mass spectral metadata via Python asyncio: concurrent queries to rate-limited chemical web services (CIR, CTS, IDSM, PubChem, BridgeDb) while respecting per-service request limits such as PubChem's ~5 requests/second.

## When to use

Use when batch-annotating many .msp spectra records against external HTTP services that enforce access-rate limits, so request throughput must be maximized without exceeding a fixed requests-per-second cap.

## When NOT to use

- The metadata can be computed locally without a web service (e.g. molecular weight from SMILES via RDKit) — no throttling or network concurrency is needed
- Only a handful of records must be annotated, so sequential requests cannot exceed the service's rate limit
- The target service does not enforce a documented requests-per-second limit, making throttling unnecessary overhead

## Inputs

- Mass spectral library file in .msp format (e.g. 'input_spectra_file.msp')
- List of jobs as (source attribute, target attribute, service) tuples, e.g. ('inchikey', 'inchi', 'CIR')
- Per-service rate limits (e.g. ~5 requests/second for PubChem's public API)

## Outputs

- Annotated .msp file (e.g. 'annotated_spectra_file.msp') with added SMILES, InChI, InChIKey, and CAS number metadata
- Cached metadata per compound identifier
- Timestamped log of issued HTTP requests for QC, plus an optional detailed log of annotation issues

## How to apply

Structure the annotation as asyncio tasks: the Application runs 'asyncio.run(app.annotate_spectra(services, jobs))' over jobs like ('inchi', 'canonical_smiles', 'IDSM'), and each converter's methods issue calls via 'await self.query_the_service(service, args)'. For rate-limited services such as PubChem, instantiate a Throttler (MSMetaEnhancer/libs/utils/throttler.py) configured to the service's limit (≈5 requests/second for PubChem) and record the timestamp of every issued request so pacing can be verified. The Throttler spaces consecutive requests so no inter-request interval falls below the limit's threshold (≥0.2 s at 5 req/s), maximizing throughput subject to that cap. Cache results containing all metadata related to a compound so repeated identifiers are not re-fetched, and iterate annotation steps per entry until no new metadata is found. Validate fetched metadata (SMILES, InChI, CAS) with matchms validators and test the Throttler with pytest by firing a burst of requests and asserting no consecutive-timestamp interval is shorter than 0.2 s.

## Related tools

- **MSMetaEnhancer** (Package providing the asynchronous Application loop, WebConverters (CTS, CIR, IDSM, PubChem, BridgeDb), and the Throttler utility for rate-limited services) — https://github.com/RECETOX/MSMetaEnhancer
- **Python** (Language whose asyncio module underpins the concurrent, await-based query pattern (asyncio.run, await self.query_the_service))
- **matchms** (Used for data IO of .msp files and for validating the form of obtained metadata)

## Examples

```
asyncio.run(app.annotate_spectra(services, jobs))
```

## Evaluation signals

- Recorded request timestamps show no interval between consecutive requests shorter than 0.2 s when the Throttler is set to 5 requests/second (PubChem limit respected)
- Throttler unit test (burst of async requests, timestamp assertion) passes under pytest alongside the existing suite
- Annotated output .msp contains the requested target attributes (e.g. SMILES, InChI, CAS) and values pass matchms validators
- Repeated queries for the same compound are served from cache rather than re-hitting the service
- Iterative annotation terminates when no new metadata is found for any entry

## Limitations

- PubChem SMILES handling was incomplete: generic 'SMILES' instead of distinct 'ISOMERIC_SMILES' and 'CANONICAL_SMILES' (issue #67)
- CIR behavior for multiple hits was not fully specified — 'take only first result when there are multiple hits in CIR conversions' (issue #69)
- Throttling is bound to the specific documented limits of each service; unknown or undocumented limits may still cause blocking
- Attribute validation tracking in the log was not yet implemented (issue #68)

## Evidence

- [abstract] Asynchronous processing with caching and throttling for rate-limited services: "this task is suitable for the asynchronous approach, making the tool computationally efficient. Additionally, results containing all metadata related to a compound are cached"
- [intro] Asynchronous implementation rationale: "The app uses asynchronous implementation of annotation process allowing for optimal fetching speed."
- [methods] Await-based service query pattern in WebConverters: "response = await self.query_the_service('MyService', args)"
- [other] Throttling mechanism for rate-limited services like PubChem: [evidence span withheld — anchor status: not_found]
- [readme] Asynchronous annotation entry point in README usage: "# run asynchronous annotations of spectra data
asyncio.run(app.annotate_spectra(services, jobs))"
