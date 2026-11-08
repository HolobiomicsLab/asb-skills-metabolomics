---
name: metabolite-database-coverage-querying
description: Use when characterizing the search space of the CMM 3.0 metabolite annotation
  database — e.g. verifying that the refreshed coverage (332,665 experimental;
when_to_use_negative:
- When the goal is annotating actual MS/MS spectra (use CMM's annotation, MS2 spectral
  comparison, or spectral quality-assessment services instead of querying coverage
  counts).
- When the database of interest is a different resource (HMDB, METLIN, etc.) — the
  counts and API here are specific to CMM 3.0 and its in-house oxidized lipids library.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Web service queried via its RESTful API to retrieve experimental and predicted
    metabolite coverage counts
  repo: https://github.com/albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:184cb3d020327d00082c263077362d1b04e776d0e6591a89f4ac830b66e472ab
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/albertogilf/ceuMassMediator
  repo_url: https://github.com/albertogilf/ceuMassMediator
  tools:
  - CEU Mass Mediator (CMM) 3.0
  - CMM 3.0
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolite-database-coverage-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves the metabolite coverage statistics (experimental vs. predicted metabolite counts) of the CEU Mass Mediator (CMM) 3.0 database via its RESTful API at http://ceumass.eps.uspceu.es, so that database version, coverage, and any live-update drift can be documented for annotation workflows.

## When to use

Use when characterizing the search space of the CMM 3.0 metabolite annotation database — e.g. verifying that the refreshed coverage (332,665 experimental; 681,198 predicted metabolites, including the in-house oxidized lipids library) still matches the published values before annotating MS/MS spectra.

## When NOT to use

- When the goal is annotating actual MS/MS spectra (use CMM's annotation, MS2 spectral comparison, or spectral quality-assessment services instead of querying coverage counts).
- When the database of interest is a different resource (HMDB, METLIN, etc.) — the counts and API here are specific to CMM 3.0 and its in-house oxidized lipids library.

## Inputs

- CMM 3.0 RESTful API endpoint (http://ceumass.eps.uspceu.es) database coverage response
- Reference coverage values from the article: 332,665 experimental and 681,198 predicted metabolites

## Outputs

- coverage_counts.csv with columns metabolite_type,count and one row per metabolite type (experimental, predicted)
- Discrepancy flag when retrieved counts differ from the article-reported CMM 3.0 values

## How to apply

Query the CMM 3.0 web service at http://ceumass.eps.uspceu.es through its RESTful API (created for programmatic access and already integrated into HMDB), e.g. with Python requests. Parse the service response to extract the total number of experimental metabolites and the total number of predicted metabolites; note that CMM 3.0's predicted space is larger than its experimental space because compounds are expanded from the unified database entries, and the in-house library of oxidized lipids (not present in other sources) contributes additional experimental entries. Write the counts to coverage_counts.csv with columns metabolite_type,count (one row per type: experimental, predicted). Compare retrieved counts against the article-reported values of 332,665 experimental and 681,198 predicted metabolites; flag any discrepancy (e.g. from live database updates) rather than silently overwriting the reference values.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Web service queried via its RESTful API to retrieve experimental and predicted metabolite coverage counts) — https://github.com/albertogilf/ceuMassMediator

## Examples

```
import requests; r = requests.post('http://ceumass.eps.uspceu.es/mediator/api/v2/getdbmetabolites', data={'database': 'all'}); print(r.json())
```

## Evaluation signals

- coverage_counts.csv contains exactly two rows (experimental, predicted) with columns metabolite_type,count.
- Retrieved experimental count equals 332,665 and predicted count equals 681,198 (article-reported CMM 3.0 values); any deviation must be explicitly flagged, not silently overwritten.
- Counts are retrieved from the live RESTful API at http://ceumass.eps.uspceu.es, not hard-coded from the article.
- Discrepancies between live and reference counts are annotated as potential live database updates with a documented timestamp.

## Limitations

- The live CMM database may be updated after publication, so retrieved counts can drift from the article-reported 332,665 experimental / 681,198 predicted values.
- Programmatic self-hosting of CMM requires access to an internal CEU database (requested by email) and a configured app-server data-source with javax.faces-2.2.12.jar deployed.
- Counts reflect the unified compound space including the in-house library of oxidized lipids; they do not describe per-ontology or per-taxonomy subsets despite CMM 3.0's new taxonomy and ontology metadata.

## Evidence
