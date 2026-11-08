---
name: precursor-mass-annotation
description: Use when you hold a list of precursor accurate masses (e.g., oxPC precursor
  m/z values in a CSV) and need compound-level annotations — species name, molecular
  formula, expert-system score — from CMM 3.0.
when_to_use_negative:
- Input is already an annotated feature table — re-annotating peaks that already carry
  formula/annotation fields adds nothing.
- The goal is MS2 spectral comparison or spectral quality assessment; CMM 3.0 provides
  separate dedicated services for those, not the oxPC annotation service.
- The search must span the full CMM experimental (332,665) or predicted (681,198)
  metabolite space rather than the specialized in-house oxidized lipid library.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Provides the oxidized glycerophosphocholine annotation service and the in-house
    oxidized lipid library, queried in batch over the RESTful API; its expert system
    returns the scored annotation label
  repo: https://github.com/albertogilf/ceuMassMediator
- name: pandas
  role: Loads oxpc_precursor_mz.csv and writes oxpc_annotations.csv with one row per
    queried m/z
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:6bf5263e7ef3ed56591fa50b1a9570fb0d2f95c6bfaf208c96849d41041e951b
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
  - CEU Mass Mediator
  - CEU Mass Mediator (CMM) 3.0
  - pandas
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# precursor-mass-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batch-annotation of precursor accurate masses — specifically oxidized glycerophosphocholine (oxPC) m/z values — against the CMM 3.0 dedicated oxidized-lipid annotation service, returning species names, molecular formulas, and expert-system-scored annotation labels. The CMM 3.0 in-house oxidized lipid library covers oxidized lipids not present in other databases.

## When to use

Use when you hold a list of precursor accurate masses (e.g., oxPC precursor m/z values in a CSV) and need compound-level annotations — species name, molecular formula, expert-system score — from CMM 3.0.

## When NOT to use

- Input is already an annotated feature table — re-annotating peaks that already carry formula/annotation fields adds nothing.
- The goal is MS2 spectral comparison or spectral quality assessment; CMM 3.0 provides separate dedicated services for those, not the oxPC annotation service.
- The search must span the full CMM experimental (332,665) or predicted (681,198) metabolite space rather than the specialized in-house oxidized lipid library.

## Inputs

- oxpc_precursor_mz.csv — fixed list of oxidized glycerophosphocholine (oxPC) precursor accurate masses
- CMM 3.0 RESTful API endpoint (http://ceumass.eps.uspceu.es/) with the CMM in-house oxidized lipid library selectable as the search source

## Outputs

- oxpc_annotations.csv — one row per queried m/z with the annotated oxPC species name, molecular formula, and expert-system annotation label; unmatched masses left explicitly empty/flagged

## How to apply

Load the fixed oxPC precursor m/z list from oxpc_precursor_mz.csv with pandas. For each precursor accurate mass, submit a batch search request to the CMM 3.0 oxidized glycerophosphocholine annotation service through its RESTful API at http://ceumass.eps.uspceu.es/. Restrict each query to the CMM in-house oxidized lipid library — added in CMM 3.0 and not present in other sources — rather than the full 332,665 experimental or 681,198 predicted metabolite space, so candidates stay chemically relevant to oxidized PCs. Parse the returned matches and retain, per queried m/z, the annotated oxPC species name, molecular formula, and the annotation label scored by the CMM expert system, which filters and scores query results using knowledge of the experimental setup and compound properties. Write oxpc_annotations.csv with one row per queried m/z, leaving unmatched masses explicitly empty or flagged rather than silently dropped.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Provides the oxidized glycerophosphocholine annotation service and the in-house oxidized lipid library, queried in batch over the RESTful API; its expert system returns the scored annotation label) — https://github.com/albertogilf/ceuMassMediator
- **pandas** (Loads oxpc_precursor_mz.csv and writes oxpc_annotations.csv with one row per queried m/z)

## Examples

```
import pandas as pd, requests; mzs = pd.read_csv('oxpc_precursor_mz.csv'); requests.post('http://ceumass.eps.uspceu.es/', json={'precursor_mz': mzs['mz'].tolist()})
```

## Evaluation signals

- oxpc_annotations.csv exists and parses as valid CSV with exactly one row per queried m/z.
- Every queried m/z row carries populated 'formula' and 'annotation' fields (field_present checks).
- Unmatched masses are visibly flagged as empty rows, never silently dropped from the output file.
- All returned matches originate from the CMM in-house oxidized lipid library, not the full 332,665 experimental / 681,198 predicted metabolite space.
- Species names follow oxidized PC (oxPC) nomenclature and each row retains the annotation label scored by the CMM expert system.

## Limitations

- Annotation coverage is bounded by the CMM in-house oxidized lipid library — it contains oxidized lipids not present in other sources, so other databases cannot back-fill misses.
- Unmatched precursor masses must be explicitly flagged (empty formula/annotation fields) rather than omitted; silent dropping breaks the one-row-per-queried-m/z invariant.
- Self-hosting CMM requires access to CEU's internal database (requested via email) plus an app-server data-source with javax.faces-2.2.12.jar; the public REST endpoint at http://ceumass.eps.uspceu.es/ is the practical access path.

## Evidence
