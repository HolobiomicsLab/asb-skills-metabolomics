---
name: published-statistics-reproduction-verification
description: Use when a workflow requires validating that a live metabolomics database's
  coverage statistics (number of experimental and predicted metabolites) match the
  values reported in a publication.
when_to_use_negative:
- The task requires annotating or identifying individual metabolites (e.g. spectral
  comparison, oxidized glycerophosphocholine annotation) rather than verifying aggregate
  database statistics
- The source is a different metabolomics database (e.g. HMDB alone) whose coverage
  counts are not those reported in the CMM 3.0 article
- Only a subset of metabolites (e.g. oxidized lipids from the in-house library) is
  of interest, not the full experimental/predicted totals
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: The web service whose RESTful API is queried to retrieve the experimental
    and predicted metabolite counts
  repo: http://ceumass.eps.uspceu.es/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:57ae8e75bbbab3af139b85a45b28e3a9f6669c06a870ff6530209ed7c3972c87
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
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Published statistics reproduction verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces published database coverage counts (experimental vs. predicted metabolites) for CEU Mass Mediator 3.0 by querying its RESTful API and writing the retrieved counts to a CSV, flagging any discrepancy with the article-reported values rather than silently overwriting them.

## When to use

Use when a workflow requires validating that a live metabolomics database's coverage statistics (number of experimental and predicted metabolites) match the values reported in a publication.

## When NOT to use

- The task requires annotating or identifying individual metabolites (e.g. spectral comparison, oxidized glycerophosphocholine annotation) rather than verifying aggregate database statistics
- The source is a different metabolomics database (e.g. HMDB alone) whose coverage counts are not those reported in the CMM 3.0 article
- Only a subset of metabolites (e.g. oxidized lipids from the in-house library) is of interest, not the full experimental/predicted totals

## Inputs

- CMM 3.0 RESTful API endpoint (http://ceumass.eps.uspceu.es) providing database coverage statistics
- Article-reported target counts: 332,665 experimental metabolites and 681,198 predicted metabolites

## Outputs

- CSV file `coverage_counts.csv` with columns `metabolite_type` and `count`, containing one row for 'experimental' and one row for 'predicted' metabolites
- A discrepancy flag when live database counts diverge from the published figures

## How to apply

Query the CMM 3.0 web service at http://ceumass.eps.uspceu.es via its RESTful API (e.g. using Python's `requests` library) to retrieve the current database coverage statistics. Parse the service response to extract two scalars: the total number of experimental metabolites and the total number of predicted metabolites. Write these to a CSV file (e.g. `coverage_counts.csv`) with one row per metabolite type (experimental, predicted) and columns `metabolite_type,count`. Then compare the retrieved counts to the article-reported CMM 3.0 values — 332,665 experimental and 681,198 predicted metabolites. If any discrepancy appears (e.g. because the live database has been updated since publication), flag it explicitly rather than silently overwriting the reported figure, since the skill's purpose is reproduction verification against the published statistics.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (The web service whose RESTful API is queried to retrieve the experimental and predicted metabolite counts) — http://ceumass.eps.uspceu.es/

## Examples

```
import requests; r = requests.get('http://ceumass.eps.uspceu.es/api/metabolites/stats'); counts = r.json(); exp = counts['experimental']; pred = counts['predicted']
```

## Evaluation signals

- Retrieved experimental metabolite count equals 332,665
- Retrieved predicted metabolite count equals 681,198
- Output CSV `coverage_counts.csv` contains exactly two rows (experimental, predicted) with columns `metabolite_type` and `count`
- Any deviation between live API counts and article-reported figures is explicitly flagged, not silently overwritten
- No additional metabolite types or rows appear beyond the two requested categories

## Limitations

- Live database counts may drift from the published values over time as CMM 3.0 continues to add metabolites, so exact reproduction is not guaranteed indefinitely
- Access to the underlying CEU internal database requires credentials requested via email (<maintainer e-mail, see the repository README>) and a configured data-source on the app server, which may block local reproduction
- The tool's coverage statistics reflect only what CMM 3.0 aggregates; other metabolite sources not integrated there (e.g. the in-house oxidized lipid library) are not included in these counts

## Evidence
