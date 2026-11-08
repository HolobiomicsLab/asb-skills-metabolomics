---
name: lc-msms-annotation-accuracy-assessment
description: Use when a processed LC-MS/MS feature table with SIRIUS/CSI:FingerID
  annotations must be checked against known expected compounds (e.g. actinomycete
  secondary metabolites or commercial standards) to report detection and annotation
  accuracy fractions.
when_to_use_negative:
- The goal is benchmarking feature detection, quantification or marker selection on
  public datasets (MTBLS733, MTBLS736, MTBLS1129, MTBLS1130) rather than annotation
  accuracy on compounds with known expected identities
- Input is raw vendor data (Thermo raw, Agilent, Bruker) or mzML that still needs
  UmetaFlow pre-processing, re-quantification and annotation steps before accuracy
  can be assessed
- No ground-truth list of expected compounds is available, so detection/annotation
  correctness cannot be scored
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: UmetaFlow
  role: Workflow that produced the validated feature tables and annotations being
    assessed
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: SIRIUS
  role: Source of the molecular formula predictions whose accuracy is scored (76%)
- name: CSI:FingerID
  role: Source of the structural predictions whose accuracy is scored (~62%, MSI level
    3)
- name: Python
  role: pandas-based loading and computation of validation fractions from supplementary
    tables
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:110a608c0f15dc13a240a45c812fe28d99df8e442398ccf77182bcec56e7cd3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - CSI:FingerID
  - Python
  - SIRIUS
  - UmetaFlow
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lc-msms-annotation-accuracy-assessment

## Summary

A validation procedure that quantifies how accurately an untargeted LC-MS/MS workflow (UmetaFlow) detects expected features and annotates molecular formulas and structures, using in-house datasets of actinomycete extracts producing known secondary metabolites plus commercial standards as ground truth.

## When to use

Use when a processed LC-MS/MS feature table with SIRIUS/CSI:FingerID annotations must be checked against known expected compounds (e.g. actinomycete secondary metabolites or commercial standards) to report detection and annotation accuracy fractions.

## When NOT to use

- The goal is benchmarking feature detection, quantification or marker selection on public datasets (MTBLS733, MTBLS736, MTBLS1129, MTBLS1130) rather than annotation accuracy on compounds with known expected identities
- Input is raw vendor data (Thermo raw, Agilent, Bruker) or mzML that still needs UmetaFlow pre-processing, re-quantification and annotation steps before accuracy can be assessed
- No ground-truth list of expected compounds is available, so detection/annotation correctness cannot be scored

## Inputs

- Additional file 2 supplementary tables (Tables S9–S11) with per-compound validation of in-house LC-MS/MS data (actinomycete extracts and commercial standards)
- UmetaFlow-processed feature tables with SIRIUS/CSI:FingerID #1-ranked predictions and spectral-library matches
- List of expected features per sample (known secondary metabolites such as germicidins, kirromycin, desferrioxamine-pathway siderophores; commercial standards)

## Outputs

- validation_summary.csv with computed fractions: 100% expected-feature detection, 76% accurate formula annotations, ~62% accurate CSI:FingerID structures, 65% total accurate structural annotations
- Per-metabolite-class accuracy breakdowns

## How to apply

Obtain the per-compound validation tables (Additional file 2, Tables S9–S11 of the UmetaFlow article, DOI 10.1186/s13321-023-00724-w) covering in-house LC-MS/MS data from actinomycete extracts and commercial standards, load them with pandas, and parse columns for expected-feature detection, SIRIUS formula annotations, CSI:FingerID structural predictions, and spectral-library matches. Compute three fractions: expected features detected (paper reports 100%), accurate SIRIUS molecular formula annotations (76%), and accurate CSI:FingerID structural predictions (~62%). For total structural annotations, combine CSI:FingerID predictions (scored as MSI level 3) with spectral matches whose score is above 60 (MSI level 2, per the UmetaFlow README), yielding the reported 65% total accurate structural annotations. Optionally break results down per metabolite class (e.g. germicidins, kirromycin, desferrioxamine-pathway siderophores) and write the metrics to validation_summary.csv.

## Related tools

- **UmetaFlow** (Workflow that produced the validated feature tables and annotations being assessed) — https://github.com/biosustain/snakemake_UmetaFlow
- **SIRIUS** (Source of the molecular formula predictions whose accuracy is scored (76%))
- **CSI:FingerID** (Source of the structural predictions whose accuracy is scored (~62%, MSI level 3))
- **Python** (pandas-based loading and computation of validation fractions from supplementary tables)

## Examples

```
import pandas as pd; df = pd.read_excel('Additional_file2_TableS9.xlsx'); acc_formula = df['formula_correct'].mean(); acc_struct = df['structure_correct'].mean()
```

## Evaluation signals

- Computed expected-feature detection fraction equals 100% on the in-house actinomycete and commercial standards datasets
- Computed accurate molecular formula annotation fraction equals 76%
- Computed total accurate structural annotation fraction equals 65% (CSI:FingerID ~62% plus spectral matches scored above 60 as MSI level 2)
- Annotation confidence tiers are assigned correctly: spectral matches above the 60-threshold as MSI level 2, SIRIUS/CSI:FingerID predictions as MSI level 3
- Per-metabolite-class breakdown rows (germicidins, kirromycin, desferrioxamine-pathway siderophores) sum consistently with the overall fractions

## Limitations

- Validation metrics (Tables S9–S11) are only in supplementary files, not the main text, so the assessment depends on downloading Additional files 2–4
- Ground truth is limited to datasets of actinomycetes producing known secondary metabolites and commercial standards; accuracy may not generalize to novel compound classes
- No software version numbers are stated for UmetaFlow, OpenMS, SIRIUS, CSI:FingerID, or Snakemake in the article, complicating exact reproduction

## Evidence

- [abstract] in-house LC-MS/MS datasets of actinomycetes producing known secondary metabolites, as well as commercial standards, and it detected all expected features: "in-house LC-MS/MS datasets of actinomycetes producing known secondary metabolites, as well as commercial standards, and it detected all expected features"
- [abstract] accurately annotated 76% of the molecular formulas and 65% of the structures: "accurately annotated 76% of the molecular formulas and 65% of the structures"
- [readme] Spectral matching with score threshold and MSI level assignment: "Spectral matching with in-house or a publicly available library (MGF/MSP/mzML format) and annotation of the feature matrix with matches that have a score above 60 (MSI level 2)"
- [readme] SIRIUS and CSI:FingerID annotation tiers: [evidence span withheld — anchor status: external_doc]
- [discussion] UmetaFlow validated with in-house data and benchmarked on public datasets: "validated UmetaFlow with in-house data, and benchmarked it using the publicly available datasets MTBLS733 and MTBLS736"
