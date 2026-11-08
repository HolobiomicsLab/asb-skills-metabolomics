---
name: per-library-median-spectra-computation
description: Use when a newly integrated or existing mass spectral library collection
  must be characterized before integration — e.g., reporting how many spectra each
  compound contributes and which chemical classes dominate across GNPS, MassBank,
  MoNA, and MS-Dial-VS17.
when_to_use_negative:
- The input is already a cleaned, dereplicated, FragHub-integrated library (602,744
  spectra for 32,193 unique chemicals) — use integration-level validation instead
  of per-library raw characterization.
- The goal is to standardize or filter spectra (e.g. check_minimum_peak_requiered,
  remove_peak_above_precursormz) rather than characterize library composition.
- Compounds lack InChIKeys or chemical identifiers, making the InChIKey grouping and
  NPClassifier annotation steps infeasible.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: FragHub
  role: Source workflow whose raw OMSL inputs (GNPS .MGF, MassBank, MoNA, MS-Dial-VS17)
    are characterized; the tool's parsing and SPLASH-key deduplication logic demonstrates
    how library extents are established before median computation.
  repo: https://github.com/eMetaboHUB/FragHub
- name: NPClassifier API
  role: Queried per unique compound InChIKey to retrieve PathNP and superclassNP annotations
    used to tabulate chemical-class distributions per library.
- name: Python
  role: Used to group spectra by compound identifier, compute medians, query NPClassifier,
    and aggregate per-library statistics into the comparison CSV.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:04c7d36a93298701e9fd6ee3f8b32f05e98b372e6cb9331f6636ec56a37eb1bc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  repo_url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - NPClassifier API
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# per-library-median-spectra-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A characterization skill for integrated open mass spectral libraries (OMSLs) that groups spectra by compound identifier per source library, computes the median number of spectra each compound contributes, and tabulates chemical-class composition via NPClassifier annotations to expose coverage differences across libraries like GNPS, MassBank, MoNA, and MS-Dial-VS17.

## When to use

Use when a newly integrated or existing mass spectral library collection must be characterized before integration — e.g., reporting how many spectra each compound contributes and which chemical classes dominate across GNPS, MassBank, MoNA, and MS-Dial-VS17.

## When NOT to use

- The input is already a cleaned, dereplicated, FragHub-integrated library (602,744 spectra for 32,193 unique chemicals) — use integration-level validation instead of per-library raw characterization.
- The goal is to standardize or filter spectra (e.g. check_minimum_peak_requiered, remove_peak_above_precursormz) rather than characterize library composition.
- Compounds lack InChIKeys or chemical identifiers, making the InChIKey grouping and NPClassifier annotation steps infeasible.

## Inputs

- Raw OMSL library files: GNPS .MGF, MassBank records, MoNA .MSP/.SDF/.JSON, MS-Dial-VS17
- Compound identifiers per spectrum (InChIKey fields extracted from library metadata)

## Outputs

- CSV table with one row per library/chemical-class combination (NPClassifier PathwayNP and superclassNP counts)
- Companion summary of median spectra per compound and total compound/spectrum counts per library

## How to apply

Load each raw OMSL file (GNPS in .MGF, MoNA in .MSP/.SDF/.JSON, MassBank records, MS-Dial-VS17) and group spectra by the compound identifier, using the InChIKey as the grouping key so that multiple spectra of the same compound collapse to one entry. Count the number of spectra per compound within each library and compute the median over compounds (e.g. GNPS median 2 spectra/compound, MassBank median 12) to quantify how spectrally redundant each library is. For every unique compound InChIKey, query the NPClassifier API to retrieve PathwayNP and superclassNP annotations, then tabulate the counts of compounds (and optionally spectra) per pathway and superclass per library. Aggregate the per-library statistics — total compounds, total spectra, median spectra/compound, and class distributions — into one comparison table and save it as CSV with one row per library/chemical-class combination plus a companion summary of median spectra per compound. Validate against the published reference sizes (GNPS 13,507 compounds/63,935 spectra; MassBank 8,358/164,261; MoNA 21,839/190,359; MS-Dial-VS17 22,282/376,430) to confirm the recomputed library extents match.

## Related tools

- **FragHub** (Source workflow whose raw OMSL inputs (GNPS .MGF, MassBank, MoNA, MS-Dial-VS17) are characterized; the tool's parsing and SPLASH-key deduplication logic demonstrates how library extents are established before median computation.) — https://github.com/eMetaboHUB/FragHub
- **NPClassifier API** (Queried per unique compound InChIKey to retrieve PathNP and superclassNP annotations used to tabulate chemical-class distributions per library.)
- **Python** (Used to group spectra by compound identifier, compute medians, query NPClassifier, and aggregate per-library statistics into the comparison CSV.)

## Examples

```
python scripts/characterize_omsls.py --gnps GNPS.mgf --massbank massbank.msp --mona mona.msp --msdial msdial_vs17.msp --output library_stats.csv
```

## Evaluation signals

- Recomputed library extents must match published values: GNPS 13,507 compounds/63,935 spectra; MassBank 8,358/164,261; MoNA 21,839/190,359; MS-Dial-VS17 22,282/376,430.
- Recomputed medians and dominant chemical classes must reproduce the published characterization (e.g. GNPS median 2 spectra/compound, MassBank median 12).
- Chemical-class distributions must show fatty acids predominating in GNPS and MassBank, alkaloids prominent in GNPS and MS-Dial-VS17, and MoNA rich in carbohydrates, amino acids, and peptides.
- Output CSV must have one row per library/chemical-class combination and include both compound and spectrum counts per class.
- Every unique compound InChIKey must resolve to a PathwayNP and superclassNP annotation set from NPClassifier before class tabulation.

## Limitations

- Chemical-class disparity between OMSLs and natural product catalogues means the characterization alone cannot fill coverage gaps — orthogonal strategies are needed because OMSL coverage (~32,193 compounds) is low versus the estimated millions of natural products.
- Grouping and medians depend on compounds carrying InChIKeys; spectra lacking SMILES/InChI, precursor m/z, and adduct information are removed during FragHub processing, so class coverage computed on raw libraries may exceed the integrated set.
- Counts per library reflect the state of each OMSL at the time of the analysis (GNPS 63,935 spectra, MassBank 164,261, MoNA 190,359, MS-Dial-VS17 376,430) and will drift as libraries update.

## Evidence
