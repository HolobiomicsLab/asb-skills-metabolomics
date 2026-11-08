---
name: restful-api-metabolite-count-extraction
description: Use when you need programmatic counts of experimental vs predicted metabolites
  in a metabolomic database, or when integrating CMM 3.0 annotation queries into a
  pipeline rather than using the web UI.
when_to_use_negative:
- When only the CMM web UI is needed interactively and no programmatic/pipeline integration
  is required
- When the target database is not CMM 3.0 — the counts are specific to this tool's
  internal library (including its in-house oxidized lipids)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Target database whose RESTful API is queried for metabolite count extraction;
    provides the experimental (332,665) and predicted (681,198) metabolite totals
  repo: http://ceumass.eps.uspceu.es/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:ad7790092f0f40ba71ec2ae1bc63f69e946aabffa8c96c699ee448456658b676
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/albertogilf/ceuMassMediator
  tools:
  - CEU Mass Mediator (CMM) 3.0
  - CMM 3.0
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# restful-api-metabolite-count-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts database coverage counts (experimental and predicted metabolite totals) from CMM 3.0's RESTful API, which the article created for programmatic access to the metabolite annotation service and integrated into HMDB.

## When to use

Use when you need programmatic counts of experimental vs predicted metabolites in a metabolomic database, or when integrating CMM 3.0 annotation queries into a pipeline rather than using the web UI.

## When NOT to use

- When only the CMM web UI is needed interactively and no programmatic/pipeline integration is required
- When the target database is not CMM 3.0 — the counts are specific to this tool's internal library (including its in-house oxidized lipids)

## Inputs

- RESTful API endpoint at http://ceumass.eps.uspceu.es (CMM 3.0 service)
- HTTP client / Python requests for query
- Target metabolite database identifier (CMM 3.0)

## Outputs

- Count of experimental metabolites (expected: 332,665)
- Count of predicted metabolites (expected: 681,198)
- Structured coverage record (e.g. coverage_counts.csv with metabolite_type,count)

## How to apply

Query the RESTful API at http://ceumass.eps.uspceu.es to retrieve the metabolite library statistics for CMM 3.0, which unified compounds from multiple metabolomic databases plus an in-house library of oxidized lipids not present in other sources. Parse the service response to separate the experimental metabolite count (332,665) from the predicted metabolite count (681,198), as CMM distinguishes empirically observed compounds from in-silico predicted ones. Write the two totals to a structured output file (e.g. coverage_counts.csv with columns metabolite_type,count) for downstream verification or pipeline use. Because the live database can change, flag any discrepancy between the API-returned count and the article-reported values rather than silently overwriting them — the API was created specifically to allow external tools like HMDB to query CMM programmatically.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Target database whose RESTful API is queried for metabolite count extraction; provides the experimental (332,665) and predicted (681,198) metabolite totals) — http://ceumass.eps.uspceu.es/

## Examples

```
import requests; r = requests.get('http://ceumass.eps.uspceu.es/api/rest/coverage'); counts = r.json(); assert counts['experimental'] == 332665 and counts['predicted'] == 681198
```

## Evaluation signals

- API-returned experimental metabolite count matches article value 332,665
- API-returned predicted metabolite count matches article value 681,198
- Experimental vs predicted categories are returned separately, not merged
- Counts are reproducible on repeat API calls within a reasonable time window
- Any discrepancy from published values is flagged rather than overwritten

## Limitations

- Live database may have been updated since publication (2018) — article-reported counts may drift from current API values
- RESTful API depends on the service being hosted at http://ceumass.eps.uspceu.es; availability not guaranteed
- Counts reflect CMM 3.0's internal unified library (including in-house oxidized lipids) and may not correspond to counts from the underlying individual databases

## Evidence
