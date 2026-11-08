---
name: oxidized-lipid-library-search
description: Use when precursor accurate masses from oxidized phospholipid screening
  need annotation against oxidized lipids not present in public databases such as
  LIPID MAPS.
when_to_use_negative:
- When the metabolites of interest are general (non-oxidized) lipids or metabolites
  already covered by databases CMM queries — use the full CMM batch search instead
  of the oxidized-lipid library restriction.
- When the task requires MS2 spectral comparison or spectral quality assessment —
  these are separate CMM 3.0 services, not the oxidized-lipid library search.
- When no CMM access to the internal CEU database is available; deployment requires
  requesting database access by email and configuring an app-server data-source.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Provides the dedicated oxidized glycerophosphocholine annotation service and
    the in-house library of oxidized lipids not present in other sources; queried
    via its RESTful API
  repo: https://github.com/albertogilf/ceuMassMediator
- name: CMM RESTful API
  role: Programmatic endpoint used to submit batch precursor-mass searches restricted
    to the oxidized lipid library
  repo: https://github.com/albertogilf/ceuMassMediator
- name: LIPID MAPS
  role: Public oxidized phospholipid reference set used to quantify overlap against
    the CMM in-house library
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:438a60e01cfac6758fda380faf991a9109c246f20363d3ed7fd6b552ef455eab
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
  - CMM RESTful API
  - LIPID MAPS
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# oxidized-lipid-library-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate oxidized glycerophosphocholine (oxPC) precursor masses against CMM 3.0's in-house library of oxidized lipids, which the article states is absent from other sources, via the CMM RESTful API.

## When to use

Use when precursor accurate masses from oxidized phospholipid screening need annotation against oxidized lipids not present in public databases such as LIPID MAPS.

## When NOT to use

- When the metabolites of interest are general (non-oxidized) lipids or metabolites already covered by databases CMM queries — use the full CMM batch search instead of the oxidized-lipid library restriction.
- When the task requires MS2 spectral comparison or spectral quality assessment — these are separate CMM 3.0 services, not the oxidized-lipid library search.
- When no CMM access to the internal CEU database is available; deployment requires requesting database access by email and configuring an app-server data-source.

## Inputs

- CSV of oxidized glycerophosphocholine precursor accurate masses (oxpc_precursor_mz.csv)
- Optional LIPID MAPS oxidized phospholipid download for overlap comparison
- Network access to the CMM 3.0 RESTful API at http://ceumass.eps.uspceu.es/

## Outputs

- oxpc_annotations.csv with one row per queried m/z, populated 'formula' and 'annotation' fields, unmatched masses visibly flagged
- Tally of CMM library entries overlapping vs non-overlapping with public sources (e.g. LIPID MAPS)

## How to apply

Load precursor m/z values (e.g. oxpc_precursor_mz.csv) and submit batch search requests through the CMM 3.0 RESTful API at http://ceumass.eps.uspceu.es/, restricted to the CMM in-house oxidized lipid library rather than the full experimental/predicted metabolite space. Restricting the query exploits the finding that CMM 3.0 added an in-house library of oxidized lipids 'not present in other sources', so overlaps with LIPID MAPS quantify genuinely novel annotations. For each queried m/z, parse the returned matches and retain the oxidized PC species name, molecular formula, and the expert-system annotation label, which is the same filter-and-score mechanism CMM applies to experimental setup and compound properties. Write results to oxpc_annotations.csv with one row per queried m/z, explicitly flagging unmatched masses rather than dropping them. For overlap analysis, normalize identifiers (PC class plus oxidation-level nomenclature) before matching entries against the LIPID MAPS oxidized phospholipid subset and tally overlapping vs non-overlapping counts.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Provides the dedicated oxidized glycerophosphocholine annotation service and the in-house library of oxidized lipids not present in other sources; queried via its RESTful API) — https://github.com/albertogilf/ceuMassMediator
- **CMM RESTful API** (Programmatic endpoint used to submit batch precursor-mass searches restricted to the oxidized lipid library) — https://github.com/albertogilf/ceuMassMediator
- **LIPID MAPS** (Public oxidized phospholipid reference set used to quantify overlap against the CMM in-house library)

## Examples

```
import pandas as pd; mzs = pd.read_csv('oxpc_precursor_mz.csv')['mz']; # POST each m/z to http://ceumass.eps.uspceu.es/ restricting queries to the CMM oxidized lipid library
```

## Evaluation signals

- oxpc_annotations.csv exists and parses as valid CSV
- Every queried m/z row carries populated 'formula' and 'annotation' fields
- Unmatched masses are visibly flagged rather than silently dropped
- Overlap analysis produces a per-entry table with present/absent calls against LIPID MAPS and a tally of overlapping vs non-overlapping entries
- Retained fields match the article's annotation outputs: oxidized PC species name, molecular formula, and expert-system annotation label

## Limitations

- The service is scoped to oxidized glycerophosphocholines from CMM's in-house library; other lipid classes must be queried through the general CMM annotation workflow.
- Access to the internal CEU database is required; deployment needs credentials provided after email request and a data-source configured on the app server (javax.faces-2.2.12.jar dependency).
- The article reports no concrete m/z or ppm tolerance thresholds for this service, so match criteria must be established by the practitioner.

## Evidence
