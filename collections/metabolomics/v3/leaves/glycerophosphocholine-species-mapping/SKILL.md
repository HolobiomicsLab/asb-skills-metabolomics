---
name: glycerophosphocholine-species-mapping
description: Use when a lipidomics workflow has a fixed list of oxPC precursor m/z
  values (e.g. oxpc_precursor_mz.csv) and needs each mass annotated with an oxidized
  PC species name, molecular formula, and expert-system annotation label.
when_to_use_negative:
- The target metabolite class is not oxidized glycerophosphocholines — the dedicated
  service covers oxPCs, so other lipid classes should go through the general CMM annotation
  workflow instead
- The compound of interest is already present in public databases (e.g. HMDB) — the
  distinguishing value of this skill is the CMM in-house library of oxidized lipids,
  which CMM 3.0 added because those lipids are absent from other sources
- MS2 spectral data is available and the goal is spectral comparison or spectrum quality
  assessment — those are separate CMM 3.0 services, not the oxPC annotation service
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Provides the dedicated oxidized-glycerophosphocholine annotation service queried
    over its RESTful API; the CMM expert system filters and scores the returned oxidized
    PC species annotations
- name: CMM in-house library of oxidized lipids
  role: Backing compound-reference dataset used as the restricted search space for
    oxPC annotation, since these oxidized lipids are not present in other databases
- name: pandas
  role: Reads the fixed precursor m/z list (oxpc_precursor_mz.csv) and writes the
    annotation output file
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:abc855602163e9d905d87897152e4449b4362da7cb4850494eac7236ab31db42
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
  - CMM in-house library of oxidized lipids
  - pandas
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# glycerophosphocholine-species-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps precursor accurate masses of oxidized glycerophosphocholines (oxPCs) to annotated species using the dedicated oxidized-glycerophosphocholine service in CEU Mass Mediator 3.0, restricted to the CMM in-house library of oxidized lipids that is absent from other databases.

## When to use

Use when a lipidomics workflow has a fixed list of oxPC precursor m/z values (e.g. oxpc_precursor_mz.csv) and needs each mass annotated with an oxidized PC species name, molecular formula, and expert-system annotation label.

## When NOT to use

- The target metabolite class is not oxidized glycerophosphocholines — the dedicated service covers oxPCs, so other lipid classes should go through the general CMM annotation workflow instead
- The compound of interest is already present in public databases (e.g. HMDB) — the distinguishing value of this skill is the CMM in-house library of oxidized lipids, which CMM 3.0 added because those lipids are absent from other sources
- MS2 spectral data is available and the goal is spectral comparison or spectrum quality assessment — those are separate CMM 3.0 services, not the oxPC annotation service

## Inputs

- oxpc_precursor_mz.csv: CSV of oxidized glycerophosphocholine precursor accurate masses
- Access to the CMM 3.0 RESTful API at http://ceumass.eps.uspceu.es/

## Outputs

- oxpc_annotations.csv: one row per queried m/z with oxidized PC species name, molecular formula, and expert-system annotation label; unmatched masses left explicitly empty

## How to apply

Load the fixed list of oxPC precursor accurate masses from oxpc_precursor_mz.csv with pandas, then submit one batch search request per precursor mass to the CMM 3.0 oxidized glycerophosphocholine annotation service over its RESTful API at http://ceumass.eps.uspceu.es/. Restrict each query to the CMM in-house library of oxidized lipids — the oxidized-lipid source CMM 3.0 adds and that is not present in other databases — rather than the full experimental space of 332,665 metabolites or predicted space of 681,198 metabolites. Parse the returned matches and, for each queried m/z, keep the oxidized PC species name, its molecular formula, and the annotation label scored by the CMM expert system that filters and scores query results using knowledge about the experimental setup and compound properties. Write results to oxpc_annotations.csv with one row per queried m/z, leaving masses with no match explicitly empty so unmatched entries remain visible rather than being silently dropped.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Provides the dedicated oxidized-glycerophosphocholine annotation service queried over its RESTful API; the CMM expert system filters and scores the returned oxidized PC species annotations)
- **CMM in-house library of oxidized lipids** (Backing compound-reference dataset used as the restricted search space for oxPC annotation, since these oxidized lipids are not present in other databases)
- **pandas** (Reads the fixed precursor m/z list (oxpc_precursor_mz.csv) and writes the annotation output file)

## Examples

```
python -c "import pandas as pd; mz = pd.read_csv('oxpc_precursor_mz.csv'); ..."  # submit each m/z to http://ceumass.eps.uspceu.es/ oxPC service, save to oxpc_annotations.csv
```

## Evaluation signals

- oxpc_annotations.csv exists and parses as valid CSV
- Every queried m/z row carries populated 'formula' and 'annotation' fields (field_present checks)
- Unmatched masses appear as explicitly flagged empty entries rather than being dropped from the output file
- Row count of oxpc_annotations.csv equals the row count of oxpc_precursor_mz.csv (one annotation row per queried precursor mass)
- Retrieved species labels come from the CMM in-house oxidized-lipid library, not from the general CMM experimental or predicted metabolite space

## Limitations

- The dedicated service covers only oxidized glycerophosphocholines; other metabolite classes need the general CMM annotation workflow
- Deploying the CMM source code yourself requires access to an internal CEU database (request access via email) plus a data-source configured on the app server with provided credentials and the javax.faces-2.2.12.jar library
- Annotation labels depend on the CMM expert system scoring, which uses knowledge of the experimental setup and compound properties — outputs without that context may be less reliable

## Evidence
