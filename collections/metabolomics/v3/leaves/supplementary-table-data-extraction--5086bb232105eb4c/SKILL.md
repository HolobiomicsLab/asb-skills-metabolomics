---
name: supplementary-table-data-extraction
description: Use when reproducing or auditing validation numbers (expected-feature
  detection rate, formula/structure annotation accuracy) that appear only in supplementary
  tables rather than the main text, e.g. Tables S9–S11 of the UmetaFlow paper.
when_to_use_negative:
- Input is raw LC-MS/MS data (mzML/Thermo raw files) that still needs feature detection
  and annotation — run the UmetaFlow pipeline itself instead of parsing its published
  validation tables
- Goal is benchmarking feature detection, quantification or marker selection against
  ground truth on MTBLS733/MTBLS736 — those performance numbers are in Tables S5 and
  S7, a different supplementary evaluation
- The supplementary tables have already been parsed and the fractions verified — recomputation
  adds no information
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Python
  role: Used with pandas to load and parse the supplementary validation tables and
    compute the detection/annotation accuracy fractions
- name: UmetaFlow
  role: The workflow whose in-house validation results (expected-feature detection,
    SIRIUS/CSI:FingerID and spectral-match annotations) are tabulated in Additional
    files 2–4
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: SIRIUS
  role: Source of the molecular formula predictions whose 76% accuracy is recomputed
    from the tables
- name: CSI:FingerID
  role: Source of the structural predictions whose ~62% accuracy contributes to the
    65% total structural annotation rate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:9cad1f1de77f3975b2d1b1cc740486b166a6f95360b0df7f972f56a9f22f0f16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - CSI:FingerID
  - Python
  - SIRIUS
  - UmetaFlow
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# supplementary-table-data-extraction

## Summary

Extracts and recomputes validation metrics from a paper's supplementary tables — here, the per-compound validation of UmetaFlow on in-house actinomycete-extract and commercial-standard LC-MS/MS data (Additional files 2–4, Tables S9–S11) — so reported accuracies can be independently reproduced.

## When to use

Use when reproducing or auditing validation numbers (expected-feature detection rate, formula/structure annotation accuracy) that appear only in supplementary tables rather than the main text, e.g. Tables S9–S11 of the UmetaFlow paper.

## When NOT to use

- Input is raw LC-MS/MS data (mzML/Thermo raw files) that still needs feature detection and annotation — run the UmetaFlow pipeline itself instead of parsing its published validation tables
- Goal is benchmarking feature detection, quantification or marker selection against ground truth on MTBLS733/MTBLS736 — those performance numbers are in Tables S5 and S7, a different supplementary evaluation
- The supplementary tables have already been parsed and the fractions verified — recomputation adds no information

## Inputs

- Additional files 2–4 of the UmetaFlow article (DOI 10.1186/s13321-023-00724-w), containing Tables S9–S11 with per-compound in-house validation results
- Per-compound columns: expected-feature detection flags, molecular formula annotations, CSI:FingerID structural predictions, spectral-library match scores
- In-house LC-MS/MS datasets of actinomycetes producing known secondary metabolites and commercial standards (context for the tables)

## Outputs

- validation_summary.csv with computed validation fractions and per-metabolite-class breakdowns (e.g. germicidins, kirromycin, desferrioxamine-pathway siderophores)
- Reproduced headline metrics: 100% expected-feature detection, 76% accurate molecular formula annotations, ~62% accurate CSI:FingerID structures, 65% total accurate structural annotations

## How to apply

Download the supplementary files accompanying the article (Additional files 2–4, containing Tables S9–S11 with per-compound validation of in-house LC-MS/MS data from actinomycete extracts and commercial standards). Load each table into Python with pandas and parse the columns covering expected-feature detection, molecular formula annotations, CSI:FingerID structural predictions, and spectral-library matches. Compute the validation fractions: expected features detected, accurate SIRIUS molecular formula annotations, accurate CSI:FingerID structural predictions, and total accurate structural annotations — counting spectral matches scored above the 60 threshold as MSI level 2 and SIRIUS/CSI:FingerID #1-ranked predictions as MSI level 3. Write the computed metrics plus per-metabolite-class breakdowns (e.g. germicidins, kirromycin, desferrioxamine-pathway siderophores) to validation_summary.csv. The reproduction is correct only if the computed fractions match the paper-reported values: 100% of expected features detected, 76% accurate molecular formulas, ~62% accurate CSI:FingerID structures, and 65% total accurate structural annotations.

## Related tools

- **Python** (Used with pandas to load and parse the supplementary validation tables and compute the detection/annotation accuracy fractions)
- **UmetaFlow** (The workflow whose in-house validation results (expected-feature detection, SIRIUS/CSI:FingerID and spectral-match annotations) are tabulated in Additional files 2–4) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **SIRIUS** (Source of the molecular formula predictions whose 76% accuracy is recomputed from the tables)
- **CSI:FingerID** (Source of the structural predictions whose ~62% accuracy contributes to the 65% total structural annotation rate)

## Examples

```
import pandas as pd; t9 = pd.read_excel('Additional_file_2.xlsx', sheet_name='Table_S9'); print(t9['expected_feature_detected'].mean(), t9['formula_correct'].mean(), t9['csi_fingerid_correct'].mean())
```

## Evaluation signals

- Computed fraction of detected expected features equals 1.00 (100%), matching the abstract-level claim
- Computed accurate molecular formula annotation fraction equals 0.76 (76%)
- Computed accurate CSI:FingerID structural prediction fraction is approximately 0.62 (~62%)
- Total accurate structural annotations, combining spectral matches scored above 60 (MSI level 2) with SIRIUS/CSI:FingerID predictions (MSI level 3), equals 0.65 (65%)
- validation_summary.csv contains per-metabolite-class rows (germicidins, kirromycin, desferrioxamine-pathway siderophores) consistent with the per-compound table rows

## Limitations

- Benchmark performance numbers (e.g. Tables S5, S7) are only in supplementary files, not in the main text, so main-text values alone cannot cross-check the benchmark side of the validation
- No software version numbers are stated for UmetaFlow, OpenMS, SIRIUS, CSI:FingerID, or Snakemake, so exact reproducibility of the underlying annotations may vary
- Structural annotations are tiered by MSI level — spectral matches above 60 are only MSI level 2 and SIRIUS/CSI:FingerID predictions are MSI level 3 — so the 65% total mixes annotation confidence tiers

## Evidence

- [abstract] in-house LC-MS/MS datasets of actinomycetes producing known secondary metabolites, as well as commercial standards, and it detected all expected features: "in-house LC-MS/MS datasets of actinomycetes producing known secondary metabolites, as well as commercial standards, and it detected all expected features"
- [abstract] accurately annotated 76% of the molecular formulas and 65% of the structures: "accurately annotated 76% of the molecular formulas and 65% of the structures"
- [discussion] we have successfully processed 1245 raw files, validated UmetaFlow with in-house data: "we have successfully processed 1245 raw files, validated UmetaFlow with in-house data"
- [readme] Spectral matching with in-house or a publicly available library (MGF/MSP/mzML format) and annotation of the feature matrix with matches that have a score above 60 (MSI level 2).: "Spectral matching with in-house or a publicly available library (MGF/MSP/mzML format) and annotation of the feature matrix with matches that have a score above 60 (MSI level 2)."
