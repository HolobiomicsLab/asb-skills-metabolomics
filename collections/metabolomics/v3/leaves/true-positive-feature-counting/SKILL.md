---
name: true-positive-feature-counting
description: Use when comparing feature-extraction parameter autotuning tools on raw
  LC-MS metabolomics data and reporting performance as counts (or fold differences)
  of true positive features against a ground-truth annotation standard.
when_to_use_negative:
- The input is already an extracted feature table — this skill requires raw LC-MS
  data so each autotuner's peak-picking parameters can be applied end-to-end.
- No ground-truth annotation standard exists — true positive classification cannot
  be performed, so fold-advantage comparisons are not defensible.
- The goal is total extracted metabolite counts against default parameters (e.g.,
  the threefold MS-DIAL result) rather than true-positive-classified benchmarking.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Paramounter
  role: R program that measures universal parameters from raw LC-MS data and translates
    them to software-specific parameters; its tuned peak height is optimized to maximize
    true positive features
  repo: https://github.com/HuanLab/Paramounter
- name: AutoTuner
  role: competing auto-tuning baseline whose parameter set yields true-positive counts
    to compare against Paramounter
- name: IPO
  role: competing auto-optimization baseline for XCMS CentWave parameters used in
    the true-positive comparison
- name: XCMS CentWave
  role: feature extraction engine applied to each method's parameter set before dereplication
    and true-positive classification
- name: R
  role: implementation language for Paramounter and the benchmark workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:812bc64d146b325e754883769faa5729757b8f9fe7cd322ab8b81edd4f869625
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - AutoTuner
  - IPO
  - Paramounter
  - R
  - XCMS CentWave
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# true-positive-feature-counting

## Summary

Benchmarks autotuning methods (Paramounter, AutoTuner, IPO) for LC-MS metabolomics by counting true positive metabolic features after XCMS CentWave extraction, then quantifying the fold advantage of Paramounter over competing auto-tuners.

## When to use

Use when comparing feature-extraction parameter autotuning tools on raw LC-MS metabolomics data and reporting performance as counts (or fold differences) of true positive features against a ground-truth annotation standard.

## When NOT to use

- The input is already an extracted feature table — this skill requires raw LC-MS data so each autotuner's peak-picking parameters can be applied end-to-end.
- No ground-truth annotation standard exists — true positive classification cannot be performed, so fold-advantage comparisons are not defensible.
- The goal is total extracted metabolite counts against default parameters (e.g., the threefold MS-DIAL result) rather than true-positive-classified benchmarking.

## Inputs

- Raw LC-MS-based metabolomics data (unprocessed vendor/MS vendor output) shared across all compared methods
- Ground-truth annotation standard for classifying true positive metabolic features
- Parameter sets produced by Paramounter, AutoTuner, and IPO

## Outputs

- Per-method true-positive feature counts (Paramounter, AutoTuner, IPO)
- Fold-advantage metrics of Paramounter over AutoTuner and IPO
- benchmark_tp_features.csv

## How to apply

Run each auto-tuning tool on the same raw LC-MS-based metabolomics data: Paramounter measures universal parameters (mass tolerance, peak height, peak width, instrumental shift) and translates them to XCMS CentWave parameters, while AutoTuner and IPO auto-tune/optimize their own peak-picking parameters. Perform feature extraction with XCMS CentWave for each parameter set and apply the mzdiff-based dereplication filter (suggested default 0.001 or 0.01) to remove features with similar m/z values and retention times. Classify the extracted features as true positives using the same ground-truth annotation standard across all methods. Tabulate true-positive feature counts per method and compute the fold advantage (reported as 3.7-fold vs AutoTuner and 1.6-fold vs IPO). Write the benchmark to benchmark_tp_features.csv.

## Related tools

- **Paramounter** (R program that measures universal parameters from raw LC-MS data and translates them to software-specific parameters; its tuned peak height is optimized to maximize true positive features) — https://github.com/HuanLab/Paramounter
- **AutoTuner** (competing auto-tuning baseline whose parameter set yields true-positive counts to compare against Paramounter)
- **IPO** (competing auto-optimization baseline for XCMS CentWave parameters used in the true-positive comparison)
- **XCMS CentWave** (feature extraction engine applied to each method's parameter set before dereplication and true-positive classification)
- **R** (implementation language for Paramounter and the benchmark workflow)

## Examples

```
Rscript run_tp_benchmark.R --raw raw_lcms/ --methods Paramounter,AutoTuner,IPO --mzdiff 0.001 --out benchmark_tp_features.csv
```

## Evaluation signals

- benchmark_tp_features.csv exists and contains true-positive counts for exactly the three methods: Paramounter, AutoTuner, IPO
- Fold advantage computed as Paramounter_tp / AutoTuner_tp and Paramounter_tp / IPO_tp; expected order-of-magnitude agreement with reported 3.7-fold and 1.6-fold values
- Identical raw LC-MS input data and identical ground-truth annotation standard are used for all three methods (apples-to-apples comparison)
- The mzdiff dereplication filter was applied consistently (same mzdiff value, e.g., 0.001 or 0.01) across all feature sets before counting
- Feature counts after dereplication are consistent across methods (no method skipped the dereplication step, e.g., by setting mzdiff negative)

## Limitations

- Paramounter's optimized peak height maximizes true positive features but incurs a higher rate of false positive features and likelihood of software crash; a higher threshold (e.g., 2X the optimized peak height) can reduce false positives.
- The mzdiff dereplication default (0.001 or 0.01) can remove true positive metabolic features with mass differences smaller than that value; setting mzdiff negative disables dereplication.
- The comparison result (3.7-fold vs AutoTuner, 1.6-fold vs IPO) is specific to the datasets and ground-truth standard used in the study; Paramounter was validated on 11 datasets covering different sample types, data acquisition modes, and MS vendors.

## Evidence
