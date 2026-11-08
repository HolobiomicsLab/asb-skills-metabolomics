---
name: dda-vs-dadia-workflow-comparison
description: Decision support for choosing between DDA and DaDIA acquisition when
  you have paired LC-MS runs of individual biological samples and pooled QC samples
  and need to compare significant-feature detection, MS2 coverage, and metabolite
  annotation counts.
when_to_use_negative:
- If you lack paired DDA and DaDIA runs on the same sample set — the comparison is
  only meaningful for matched workflows
- If the goal is to compare DDA vs DIA(SWATH) rather than DDA vs DaDIA — a different
  set of metrics (e.g., total detected features 1591 vs 6680 vs 7040) applies
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DaDIA.R
  role: R pipeline used to extract metabolic features and annotate metabolites from
    DaDIA data that feed the comparison
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Feature extraction (centWave) underlying the DDA and DaDIA peak tables compared
    in the benchmark
  repo: https://github.com/Huan Lab/DaDIA.git
- name: R
  role: Runtime environment (>= 4.0) for DaDIA.R and downstream comparison table construction
  repo: https://github.com/Huan Lab/DaDIA.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:bc9910df6ab3d2fc4d8cd1c30722b59dc6296f31ad80a2f27debd8010f57cfe9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# dda-vs-dadia-workflow-comparison

## Summary

A quantitative benchmarking skill for comparing the traditional DDA workflow against the DaDIA hybrid workflow (DIA on biological samples + DDA on pooled QC) in terms of feature counts, MS2 coverage, and annotated significant metabolites.

## When to use

Decision support for choosing between DDA and DaDIA acquisition when you have paired LC-MS runs of individual biological samples and pooled QC samples and need to compare significant-feature detection, MS2 coverage, and metabolite annotation counts.

## When NOT to use

- If you lack paired DDA and DaDIA runs on the same sample set — the comparison is only meaningful for matched workflows
- If the goal is to compare DDA vs DIA(SWATH) rather than DDA vs DaDIA — a different set of metrics (e.g., total detected features 1591 vs 6680 vs 7040) applies

## Inputs

- Reported per-workflow significant-feature counts from DDA and DaDIA LC-MS analyses (e.g., leukemia plasma data)
- Per-workflow MS2 feature counts and annotated metabolite counts
- Manual EIC quality assessment results for DaDIA-unique features

## Outputs

- Structured comparison table (e.g., leukemia_dda_dadia_comparison.csv) with one row per metric and DDA/DaDIA columns
- Validated counts matching reference values (766, 993, 292, 809, 24, 85, 486, 0.88, 0.05)

## How to apply

Compile per-workflow metrics from the leukemia plasma study: total significant features (DDA 766 vs DaDIA 993), significant features with MS2 spectra (DDA 292 vs DaDIA 809), and annotated significant endogenous metabolites (DDA 24 vs DaDIA 85). Significant features were defined by fold-change > 1.5 or < 0.67 with paired two-tailed t-test p < 0.05; DaDIA-unique features (486) were manually inspected via extracted-ion chromatograms with 88% true-positive and 5% false-positive rates. Build a one-row-per-metric comparison table with DDA and DaDIA columns, validating each entry against reported reference values. Rationale: DaDIA's hybridization of DIA sample runs with DDA pooled-QC runs yields ~30% more significant features and ~4× more annotated metabolites due to better metabolome coverage.

## Related tools

- **DaDIA.R** (R pipeline used to extract metabolic features and annotate metabolites from DaDIA data that feed the comparison) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Feature extraction (centWave) underlying the DDA and DaDIA peak tables compared in the benchmark) — https://github.com/Huan Lab/DaDIA.git
- **R** (Runtime environment (>= 4.0) for DaDIA.R and downstream comparison table construction) — https://github.com/Huan Lab/DaDIA.git

## Examples

```
Rscript DaDIA.R
```

## Evaluation signals

- Each table entry must exactly match the reported reference values: 766, 993, 292, 809, 24, 85, 486, 0.88, 0.05
- DaDIA's significant-feature count should exceed DDA's by ~30%
- Annotated significant metabolites in DaDIA (~85) should be roughly 3–4× the DDA count (~24)
- MS2-attached significant features should cover ≥ 60% of DaDIA significant features (809/993)
- Manual EIC quality of DaDIA-unique features: true-positive rate ≈ 88% and false-positive rate ≈ 5%

## Limitations

- Comparison is grounded in a single biological dataset (leukemia plasma); transferability to other matrices must be assessed
- Manual EIC quality checks (88% true-positive / 5% false-positive) were applied only to DaDIA-unique features; no equivalent manual QC of DDA-unique features was reported
- The counts are produced by DaDIA.R with XCMS centWave; different feature-extraction settings or software may shift the absolute numbers

## Evidence

- [results] DDA and DaDIA analyses detected 766 and 993 significant metabolic features: [evidence span withheld — anchor status: not_found]
- [discussion] DaDIA detecting 30% more significant features due to better metabolome coverage: [evidence span withheld — anchor status: not_found]
- [results] significant features with MS2 spectra (DDA 292 vs DaDIA 809): [evidence span withheld — anchor status: not_found]
- [discussion] annotated significant endogenous metabolites (DDA 24 vs DaDIA 85): [evidence span withheld — anchor status: not_found]
- [results] 88% true-positive and 5% false-positive: [evidence span withheld — anchor status: not_found]
