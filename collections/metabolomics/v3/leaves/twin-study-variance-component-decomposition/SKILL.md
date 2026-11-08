---
name: twin-study-variance-component-decomposition
description: Use when SWATH-MS/DIA LC-MS maps from related individuals (e.g. co-twins
  sampled longitudinally) must be quantified and per-protein variance apportioned
  into genetic, shared-environment and time effects.
when_to_use_negative:
- Input is labelled or fractionated shotgun proteomics rather than SWATH-MS DIA plasma
  maps requiring targeted extraction
- No relatedness structure (e.g. unrelated individuals, single time point) — variance
  decomposition into heritable/environmental/longitudinal components is unidentifiable
- Only a small number of samples or proteins — component estimates will be unstable
  relative to the twin-cohort scale
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0121
tools:
- name: OpenMS 2.0
  role: Runs the upstream DIA processing chain (file conversion, centroiding, database
    searching, feature detection) and hosts the OpenSWATH targeted-analysis module
    used for the twin-cohort plasma maps
- name: OpenMS
  role: Provides the processing chain and OpenSWATH module used to extract peptide
    transitions and align/link features across twin-pair plasma maps
  repo: https://github.com/OpenMS/OpenMS
- name: Python
  role: Fits the per-protein linear mixed-effects variance-component model partitioning
    abundance into heritable, environmental, and longitudinal components
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:20b0a047c44f49fceee4235c9e79b6b11a2563ce2f438484e5cf0dc632b98358
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - OpenMS
  - OpenMS 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# twin-study-variance-component-decomposition

## Summary

Quantifies plasma proteins from SWATH-MS DIA maps of a twin cohort using the OpenMS/OpenSWATH pipeline, then decomposes each protein's abundance variance into heritable, environmental (common and individual), and longitudinal components via a Python mixed-effects model. Produces a per-protein variance partition table (variance_components.csv) validated so the three components sum to ~1.

## When to use

Use when SWATH-MS/DIA LC-MS maps from related individuals (e.g. co-twins sampled longitudinally) must be quantified and per-protein variance apportioned into genetic, shared-environment and time effects.

## When NOT to use

- Input is labelled or fractionated shotgun proteomics rather than SWATH-MS DIA plasma maps requiring targeted extraction
- No relatedness structure (e.g. unrelated individuals, single time point) — variance decomposition into heritable/environmental/longitudinal components is unidentifiable
- Only a small number of samples or proteins — component estimates will be unstable relative to the twin-cohort scale

## Inputs

- Twin-cohort blood plasma SWATH-MS (DIA) mass spectrometry maps
- Assay/transition library for targeted OpenSWATH extraction
- OpenMS 2.0 installation with OpenSWATH module
- Decoy transition library for FDR estimation
- Python environment with linear mixed-effects modelling capability

## Outputs

- Per-protein plasma abundance matrix across all twin-cohort runs
- Variance-component table (heritable, environmental, longitudinal fractions per protein) saved as variance_components.csv
- List of quantified plasma proteins (>300 reported by Liu et al.)

## How to apply

Run the OpenMS 2.0 processing chain (file conversion, centroiding, database searching, feature detection) on the twin-cohort blood plasma DIA maps, then use OpenSWATH targeted extraction of peptide transitions from each plasma map against an assay/transition library with decoy-based scoring. Perform retention time alignment and cross-run feature linking across all twin-pair maps so peptide quantities are consistent between runs, aggregate peptides into proteins, and apply statistical error control to obtain a per-protein abundance matrix. Finally, fit a per-protein linear mixed-effects variance-component model in Python that partitions each protein's variance into a heritable component (shared between co-twins), an environmental component (common and individual), and a longitudinal (time) component; Liu et al. quantified over 300 plasma proteins this way. Export the per-protein variance fractions as variance_components.csv and verify the three components sum to approximately 1 per protein.

## Related tools

- **OpenMS 2.0** (Runs the upstream DIA processing chain (file conversion, centroiding, database searching, feature detection) and hosts the OpenSWATH targeted-analysis module used for the twin-cohort plasma maps)
- **OpenMS** (Provides the processing chain and OpenSWATH module used to extract peptide transitions and align/link features across twin-pair plasma maps) — https://github.com/OpenMS/OpenMS
- **Python** (Fits the per-protein linear mixed-effects variance-component model partitioning abundance into heritable, environmental, and longitudinal components)

## Examples

```
OpenSwathWorkflow -in twin_plasma_dia.mzML -tr library.tsv -tr_irtsv iRT.tsv -out osm.mzML -outFeatures feature.tsv; then fit lme4-style mixed-effects model per protein in Python to produce variance_components.csv
```

## Evaluation signals

- The three variance components (heritable, environmental, longitudinal) sum to approximately 1 for each protein in variance_components.csv
- Per-protein variance fractions are consistent with the decomposition reported in the twin-cohort paper (Liu et al.); residual discrepancies resolved by expert review
- Over 300 plasma proteins quantified, matching the reported scale of OpenSWATH analysis
- Peptide quantities are consistent between runs after retention-time alignment and cross-run feature linking
- Statistical error control was applied at the protein level before variance decomposition

## Limitations

- Validation of the decomposition depends on the paper's reported per-protein partitioning; discrepancies require expert review rather than automated resolution
- Variance components must sum to ~1; if not, the mixed-effects model or upstream quantification must be re-examined
- OpenMS 2.0's toolset and workflow coverage may differ from later versions; the OpenMS 3 citation is the current reference (Nat Methods 21, 365–367, 2024)
- Variance decomposition is only meaningful with twin/cohort relatedness structure and repeated (longitudinal) sampling

## Evidence

- [results] OpenSWATH quantified >300 plasma proteins with variance decomposition into heritable, environmental, longitudinal components: [evidence span withheld — anchor status: not_found]
- [methods] OpenMS 2.0 chain: conversion, centroiding, searching, feature detection: [evidence span withheld — anchor status: not_found]
- [methods] Targeted OpenSWATH extraction with decoy scoring: [evidence span withheld — anchor status: not_found]
- [methods] Cross-run alignment and linking for consistent quantities: [evidence span withheld — anchor status: not_found]
- [methods] Python mixed-effects variance decomposition: [evidence span withheld — anchor status: not_found]
- [methods] Validation: components sum to ~1: [evidence span withheld — anchor status: not_found]
- [readme] README: OpenMS supports SWATH quantification: [evidence span withheld — anchor status: external_doc]
- [readme] README: OpenMS C++ library for LC-MS analyses: [evidence span withheld — anchor status: external_doc]
