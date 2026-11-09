---
name: pubchem-compound-lookup
description: Use when GNPS (or similar) MS/MS library spectra lack InChIKey annotations
  but their metadata contains compound names, molecular formulas, and/or parent masses
  usable for compound lookup.
when_to_use_negative:
- Spectra already carry InChIKey annotations — re-querying PubChem would only risk
  introducing mismatches
- The task is downstream spectral similarity scoring (library matching / molecular
  networking) on an already-annotated dataset — use Spec2Vec or cosine-based scores
  instead
- Metadata lacks compound name, molecular formula, and parent mass alike, leaving
  no fields to query PubChem with
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3335
- http://edamontology.org/topic_3314
tools:
- name: matchms
  role: Cleaning and correcting spectrum metadata (ionmode, charge, compound name,
    molecular formula, parent mass) before PubChem lookup
  repo: https://github.com/matchms/matchms
- name: pubchempy
  role: Python interface used to run the automated lookup search against PubChem by
    compound name, molecular formula, and parent mass
- name: PubChem
  role: External compound database queried to resolve InChIKeys for unannotated spectra
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:3a8ae6977614db0a45c09e35dd4ac94dccb809ec1f83cf99fc3b49c60dc4ec95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - matchms
  - pubchempy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3335
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# pubchem-compound-lookup

## Summary

An automated lookup strategy that resolves missing structural annotations for MS/MS library spectra by querying PubChem via pubchempy with compound name, molecular formula, and parent mass, then attaching full and planar InChIKeys to spectrum metadata.

## When to use

Use when GNPS (or similar) MS/MS library spectra lack InChIKey annotations but their metadata contains compound names, molecular formulas, and/or parent masses usable for compound lookup.

## When NOT to use

- Spectra already carry InChIKey annotations — re-querying PubChem would only risk introducing mismatches
- The task is downstream spectral similarity scoring (library matching / molecular networking) on an already-annotated dataset — use Spec2Vec or cosine-based scores instead
- Metadata lacks compound name, molecular formula, and parent mass alike, leaving no fields to query PubChem with

## Inputs

- GNPS ALL_GNPS.json library (154,820 spectra, https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.json)
- Per-spectrum metadata: compound name, molecular formula, parent mass, ionmode, charge

## Outputs

- Annotated library JSON (gnps_allannotated_cleaned.json) with full and planar InChIKey attached to spectrum metadata
- Coverage statistics: 128,042/154,820 spectra annotated, 14,978 unique planar InChIKeys

## How to apply

First clean and correct the provided metadata with matchms, harmonizing fields such as ionmode, charge, compound name, molecular formula, and parent mass so lookups operate on consistent fields. For spectra still lacking InChIKey annotations, run an automated lookup against PubChem via pubchempy, querying by compound name, molecular formula, and parent mass. Derive both full and planar (first 14-character block) InChIKeys for matched compounds and attach them to the spectrum metadata. Save the annotated library (in the Spec2Vec workflow as gnps_allannotated_cleaned.json) and validate against the published outcome: 128,042 of 154,820 ALL_GNPS spectra linked to an InChIKey (14,978 unique planar InChIKeys), matching Zenodo 10.5281/zenodo.3979010.

## Related tools

- **matchms** (Cleaning and correcting spectrum metadata (ionmode, charge, compound name, molecular formula, parent mass) before PubChem lookup) — https://github.com/matchms/matchms
- **pubchempy** (Python interface used to run the automated lookup search against PubChem by compound name, molecular formula, and parent mass)
- **PubChem** (External compound database queried to resolve InChIKeys for unannotated spectra)

## Examples

```
import pubchempy as pcp; hits = pcp.get_compounds(compound_name, 'name')
```

## Evaluation signals

- Fraction of spectra linked to an InChIKey reaches the published 128,042/154,820 (~82.9%) for the ALL_GNPS library
- Number of unique planar InChIKeys approximates 14,978 as reported in the preprocessing outcome
- Every assigned InChIKey is structurally valid (full 27-character key; planar key = first 14 characters)
- Output agrees with the published preprocessed dataset on Zenodo (10.5281/zenodo.3979010)

## Limitations

- Coverage is not complete: ~26,778 of 154,820 spectra remain without an InChIKey after the PubChem lookup
- Lookup success depends on the quality of cleaned metadata; incorrect or ambiguous compound names/formulas can produce wrong compound matches
- InChIKeys were obtained from a 2020-05-11 snapshot of ALL_GNPS.json; later library versions will shift the counts

## Evidence

- [discussion] PubChem lookup as annotation route: "an extensive automated lookup search was run against PubChem [35] (via pubchempy [36]). As a result, 128,042 out of 154,820 spectra could be linked to an InChIKey"
- [results] matchms cleaning precedes lookup: [evidence span withheld — anchor status: not_found]
- [readme] Source data URL for the library: [evidence span withheld — anchor status: external_doc]
