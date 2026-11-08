---
name: cross-omsl-comparative-statistics
description: Reference for comparing chemical-class coverage, spectral redundancy
  (median spectra per compound), and compound counts across multiple open mass spectral
  libraries before integrating them or choosing which OMSL to use.
when_to_use_negative:
- You only need to integrate/harmonize library files — that is FragHub's parsing,
  SPLASH-key deduplication and filtering job, not comparative statistics.
- The input is already a merged FragHub output table with class annotations; re-deriving
  class distributions from raw files would duplicate work.
- You need collision-energy comparison across libraries — the paper explicitly states
  collision energies were not harmonized due to ~70 nonstandardized formats.
edam_operation: http://edamontology.org/operation_2943
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0089
tools:
- name: FragHub
  role: Source of the integrated library statistics and the harmonized compound/spectrum
    counts that the comparison validates against
  repo: github.com/eMetaboHUB/FragHub
- name: NPClassifier API
  role: Chemical-class (PathwayNP/superclassNP) annotation service queried per compound
    InChIKey to build class distributions
- name: Python
  role: Implementation language for grouping spectra by InChIKey, computing medians,
    and assembling the CSV table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:601545bc0ecfd47903ac14c9e0e1fbbe88a837ea9cfb47542bb9b5aff7668d67
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - NPClassifier API
  - Python
  edam_operation: http://edamontology.org/operation_2943
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-omsl-comparative-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A comparative-statistics skill that characterizes the chemical-class composition and spectral redundancy of multiple Open Mass Spectra Libraries (GNPS, MassBank, MoNA, MS-Dial-VS17) by counting spectra per compound InChIKey and annotating compounds with NPClassifier pathways/superclasses. It quantifies how unevenly chemical space (fatty acids, alkaloids, carbohydrates/peptides) and spectra-per-compound medians (e.g. GNPS 2 vs MassBank 12) are distributed across OMSLs.

## When to use

Reference for comparing chemical-class coverage, spectral redundancy (median spectra per compound), and compound counts across multiple open mass spectral libraries before integrating them or choosing which OMSL to use.

## When NOT to use

- You only need to integrate/harmonize library files — that is FragHub's parsing, SPLASH-key deduplication and filtering job, not comparative statistics.
- The input is already a merged FragHub output table with class annotations; re-deriving class distributions from raw files would duplicate work.
- You need collision-energy comparison across libraries — the paper explicitly states collision energies were not harmonized due to ~70 nonstandardized formats.

## Inputs

- Raw OMSL library files in .MGF (GNPS), .MSP, .SDF, .JSON (MoNA), MassBank records, and MS-Dial-VS17 format
- Unique compound InChIKeys extracted from each library

## Outputs

- Per-library chemical-class distribution table (compound and spectrum counts per NPClassifier pathway and superclass)
- Median spectra-per-compound statistic for each OMSL
- CSV comparison table with one row per library/chemical-class combination
- Per-library totals of compounds and spectra for validation

## How to apply

Load each raw OMSL file (GNPS .MGF, MassBank records, MoNA .MSP/.SDF/.JSON, MS-Dial-VS17) and group spectra by compound InChIKey to count spectra contributed per compound. Compute the median spectra-per-compound per library (published values: GNPS 2, MassBank 12) to quantify redundancy that would otherwise bias dot-product and reverse dot-product scoring. For every unique compound InChIKey, query the NPClassifier API to retrieve PathwayNP and superclassNP annotations, then tabulate compound (and optionally spectrum) counts per pathway/superclass per library. Aggregate per-library totals and class distributions into one comparison table (CSV, one row per library/class combination) plus a median-spectra-per-compound summary. Validate against the article's reported library sizes: GNPS 13,507 compounds/63,935 spectra; MassBank 8,358/164,261; MoNA 21,839/190,359; MS-Dial-VS17 22,282/376,430; integrated FragHub output 602,744 spectra/32,193 chemicals.

## Related tools

- **FragHub** (Source of the integrated library statistics and the harmonized compound/spectrum counts that the comparison validates against) — github.com/eMetaboHUB/FragHub
- **NPClassifier API** (Chemical-class (PathwayNP/superclassNP) annotation service queried per compound InChIKey to build class distributions)
- **Python** (Implementation language for grouping spectra by InChIKey, computing medians, and assembling the CSV table)

## Examples

```
df.groupby('inchikey').size().median()  # per OMSL: median spectra per compound, then NPClassifier query per unique InChIKey
```

## Evaluation signals

- Recomputed per-library totals match the published counts: GNPS 13,507 compounds/63,935 spectra, MassBank 8,358/164,261, MoNA 21,839/190,359, MS-Dial-VS17 22,282/376,430.
- Computed median spectra-per-compound reproduces published values (GNPS 2, MassBank 12).
- Dominant chemical classes per library reproduce published characterization: fatty acids predominate in GNPS and MassBank, alkaloids in GNPS and MS-Dial-VS17, carbohydrates/amino acids/peptides in MoNA.
- Output CSV has one row per library/chemical-class combination and a companion median summary.
- Integrated totals should converge toward FragHub's refined 602,744 spectra for 32,193 unique chemicals after deduplication.

## Limitations

- Chemical-class disparity between OMSLs and natural-product catalogues necessitates orthogonal strategies — 32,193 covered compounds is 'rather low compared to the diversity of natural products estimated to be several million molecules'.
- Collision energies were not harmonized, so cross-library spectral comparison cannot account for differing fragmentation conditions (around 70 different formats).
- Redundancy across OMSLs 'might also lead to inconsistencies, particularly when using dot product and reverse dot product scoring systems that are highly sensitive to fragment number and intensity'.

## Evidence

- [methods] Median spectra per compound (GNPS 2, MassBank 12, etc.) computed by grouping spectra by InChIKey: [evidence span withheld — anchor status: not_found]
- [methods] NPClassifier-based class annotation: [evidence span withheld — anchor status: not_found]
- [methods] Spectra-per-compound medians (GNPS 2, MassBank 12): [evidence span withheld — anchor status: not_found]
- [methods] Class distributions per library: [evidence span withheld — anchor status: not_found]
- [results] Fatty-acid/alkaloid dominance: [evidence span withheld — anchor status: not_found]
