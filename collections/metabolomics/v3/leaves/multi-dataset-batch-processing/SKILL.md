---
name: multi-dataset-batch-processing
description: Use when a collection of raw LC-MS metabolomics datasets (e.g., 11 validation
  datasets spanning different sample types, data acquisition modes, and MS vendors)
  must be processed with automatically tuned, dataset-specific feature-extraction
  parameters rather than one-size-fits-all defaults.
when_to_use_negative:
- The input is already a processed feature table rather than raw LC-MS data — Paramounter
  measures parameters directly from raw LC-MS-based metabolomics data prior to feature
  extraction.
- Only a single dataset with known-good manually tuned parameters is available and
  no cross-dataset robustness or parameter automation is needed.
- The target feature-extraction program is not among the programs Paramounter translates
  parameters for (e.g., MS-DIAL-based or XCMS CentWave pipelines).
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Paramounter
  role: R program that automatically measures universal parameters from raw LC-MS
    data and translates them into software-specific parameters for data processing
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: Language in which Paramounter is implemented and executed per dataset
- name: MS-DIAL
  role: Feature-extraction program whose default-parameter output is compared against;
    Paramounter-derived parameters yield a threefold increase in extracted metabolites
- name: XCMS CentWave
  role: Feature extraction target whose mzdiff parameter acts as the mass tolerance
    for dereplicating features with similar m/z values and retention times
- name: AutoTuner
  role: Baseline parameter-tuning tool against which Paramounter is compared for true
    positive feature yield
- name: IPO
  role: Baseline parameter-tuning tool against which Paramounter is compared for true
    positive feature yield
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:57bce9a89691bff334ef367a8cfb0c2c69601dd9674d36072297dc9d8f50a878
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - AutoTuner
  - IPO
  - MS-DIAL
  - Paramounter
  - R
  - XCMS CentWave
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multi-dataset-batch-processing

## Summary

Batch-process multiple LC-MS metabolomics datasets through Paramounter, which automatically measures universal parameters (mass tolerance, peak height, peak width, instrumental shift) from raw data and translates them into software-specific parameters for feature extraction, validating that the tuning is robust across sample types, acquisition modes, and MS vendors.

## When to use

Use when a collection of raw LC-MS metabolomics datasets (e.g., 11 validation datasets spanning different sample types, data acquisition modes, and MS vendors) must be processed with automatically tuned, dataset-specific feature-extraction parameters rather than one-size-fits-all defaults.

## When NOT to use

- The input is already a processed feature table rather than raw LC-MS data — Paramounter measures parameters directly from raw LC-MS-based metabolomics data prior to feature extraction.
- Only a single dataset with known-good manually tuned parameters is available and no cross-dataset robustness or parameter automation is needed.
- The target feature-extraction program is not among the programs Paramounter translates parameters for (e.g., MS-DIAL-based or XCMS CentWave pipelines).

## Inputs

- Raw LC-MS-based metabolomics data files (one or more datasets)
- Dataset metadata per dataset: sample type, data acquisition mode, MS vendor
- Choice of target feature-extraction program (e.g., MS-DIAL-based or XCMS CentWave)

## Outputs

- Per-dataset measured universal parameters: mass tolerance, peak height, peak width, instrumental shift
- Software-specific feature-extraction parameters derived from the universal parameters
- Per-dataset extracted metabolite/feature lists
- A summary table (CSV) assembling dataset metadata and extracted feature counts across all datasets

## How to apply

For each raw LC-MS-based metabolomics dataset, run Paramounter (an R program) to automatically measure the universal parameters — mass tolerance, peak height, peak width, and instrumental shift — directly from the raw data prior to metabolic feature extraction. Paramounter then translates these universal parameters into software-specific parameters (e.g., for MS-DIAL-based or XCMS CentWave feature extraction). Apply the derived parameters to drive feature extraction per dataset, recording dataset metadata (sample type, acquisition mode, MS vendor) alongside the number of extracted metabolites/features. Apply Paramounter's optimization that tunes an optimized peak height to maximize true positive features; if false positive rates are problematic, users can raise the peak height threshold (e.g., 2X the optimized value). mzdiff serves as the mass tolerance for dereplication of features with similar m/z and retention times; set mzdiff to a negative value to disable dereplication. Assemble per-dataset results (metadata + extracted feature counts) into a single summary table and validate coverage across all datasets.

## Related tools

- **Paramounter** (R program that automatically measures universal parameters from raw LC-MS data and translates them into software-specific parameters for data processing) — https://github.com/HuanLab/Paramounter
- **R** (Language in which Paramounter is implemented and executed per dataset)
- **MS-DIAL** (Feature-extraction program whose default-parameter output is compared against; Paramounter-derived parameters yield a threefold increase in extracted metabolites)
- **XCMS CentWave** (Feature extraction target whose mzdiff parameter acts as the mass tolerance for dereplicating features with similar m/z values and retention times)
- **AutoTuner** (Baseline parameter-tuning tool against which Paramounter is compared for true positive feature yield)
- **IPO** (Baseline parameter-tuning tool against which Paramounter is compared for true positive feature yield)

## Examples

```
Rscript Paramounter.R --input raw.mzML --output params.csv
```

## Evaluation signals

- The summary table must cover all datasets in the batch (e.g., all 11 validation datasets) with recorded sample type, acquisition mode, and MS vendor.
- Feature extraction driven by Paramounter-derived parameters should yield approximately a threefold increase in extracted metabolites versus default-parameter MS-DIAL-based feature extraction.
- Compared with AutoTuner and IPO, Paramounter-generated parameter sets should produce ~3.7-fold and ~1.6-fold more true positive features, respectively.
-  mzdiff dereplication setting should be confirmed: a positive value (0.001 or 0.01) applies dereplication; any negative value disables it.
- If false positive rates are high or software crashes occur, verify the peak height threshold was raised relative to Paramounter's optimized value (e.g., 2X).

## Limitations

- Paramounter's optimized peak height maximizes true positive features but incurs a higher rate of false positive features and likelihood of software crash; mitigated by using a higher threshold (e.g., 2X the optimized peak height).
- Default mzdiff dereplication (0.001 or 0.01) may remove true positive metabolic features whose mass differences are smaller than that value; set mzdiff negative to disable dereplication.
- No discussion-section limitations were extracted from the provided article content.

## Evidence
