---
name: per-dataset-summary-table-assembly
description: Use when you have run parameter tuning plus feature extraction on multiple
  LC-MS datasets (different sample types, acquisition modes, or MS vendors) and need
  one consolidated table recording each dataset's metadata and extracted-feature count
  to judge robustness across conditions.
when_to_use_negative:
- A single dataset is being analyzed and no cross-dataset robustness comparison is
  intended — a per-dataset table adds no value.
- Feature extraction has not yet been run; the feature-count column would be empty
  and the table would be premature.
- The input is already a consolidated multi-dataset feature table; re-assembling a
  per-dataset summary is redundant.
tools:
- name: Paramounter
  role: R program that automatically measures the universal parameters (mass tolerance,
    peak height, peak width, instrumental shift) from raw LC-MS-based metabolomics
    data, producing the tuned parameters used to drive feature extraction whose per-dataset
    feature counts are then collated into the summary table.
  repo: https://github.com/HuanLab/Paramounter
- name: MS-DIAL
  role: Feature extraction program whose default-parameter extracted metabolite counts
    provide the baseline that the Paramounter-derived counts are compared against
    when validating the summary table's robustness signal.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:838201e71ec18d2d46ad10f50eedc33e33bc0abcade5a33ca814ee996fa14561
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - MS-DIAL
  - Paramounter
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# per-dataset-summary-table-assembly

## Summary

Assembly of a single CSV summary table that collates per-dataset records — dataset metadata (sample type, acquisition mode, MS vendor) alongside the number of extracted metabolites/features obtained with Paramounter-derived parameters — across all validation datasets, so that the robustness of parameter tuning can be compared dataset by dataset.

## When to use

Use when you have run parameter tuning plus feature extraction on multiple LC-MS datasets (different sample types, acquisition modes, or MS vendors) and need one consolidated table recording each dataset's metadata and extracted-feature count to judge robustness across conditions.

## When NOT to use

- A single dataset is being analyzed and no cross-dataset robustness comparison is intended — a per-dataset table adds no value.
- Feature extraction has not yet been run; the feature-count column would be empty and the table would be premature.
- The input is already a consolidated multi-dataset feature table; re-assembling a per-dataset summary is redundant.

## Inputs

- LC-MS-based metabolomics raw data (per dataset) processed with parameter tuning and feature extraction (e.g. Paramounter-derived parameters driving MS-DIAL)
- Per-dataset metadata: sample type, data acquisition mode, MS vendor
- Per-dataset count of extracted metabolites/features

## Outputs

- A single CSV summary table with one record per dataset, containing sample type, acquisition mode, MS vendor, and the number of extracted metabolites/features

## How to apply

For each dataset, first run the parameter-tuning/feature-extraction pipeline (e.g. Paramounter → MS-DIAL) and record the number of extracted metabolites/features. Alongside the feature count, capture per-dataset metadata: sample type, data acquisition mode, and MS vendor. Collect one record per dataset, then merge all records into a single summary table and write it as a CSV file. Validate by checking that the table covers all datasets in the batch and that the feature counts are consistent with the expected behavior — e.g. a threefold increase in extracted metabolites relative to default-parameter MS-DIAL-based feature extraction.

## Related tools

- **Paramounter** (R program that automatically measures the universal parameters (mass tolerance, peak height, peak width, instrumental shift) from raw LC-MS-based metabolomics data, producing the tuned parameters used to drive feature extraction whose per-dataset feature counts are then collated into the summary table.) — https://github.com/HuanLab/Paramounter
- **MS-DIAL** (Feature extraction program whose default-parameter extracted metabolite counts provide the baseline that the Paramounter-derived counts are compared against when validating the summary table's robustness signal.)

## Examples

```
Rscript Paramounter.R --input raw_data/ --output summary_table.csv
```

## Evaluation signals

- The assembled table contains one row for each dataset in the batch (all 11 datasets in the reported validation, covering different sample types, data acquisition modes, and MS vendors).
- Each row's metadata columns (sample type, acquisition mode, MS vendor) are populated and reflect the actual dataset conditions.
- Per-dataset feature counts are present and correspond to the number of extracted metabolites/features obtained with the tuned (e.g. Paramounter-derived) parameters.
- Extracted-feature counts reflect the expected validation behavior: a threefold increase relative to default-parameter MS-DIAL-based feature extraction.
- The output is a single CSV file rather than scattered per-dataset files.

## Limitations

- The table's value depends entirely on having comparable feature-extraction runs across datasets; if the extraction parameters differ in kind (not just value) between datasets, the collated counts are not directly comparable.
- Feature counts alone do not distinguish true positive from false positive features; the article notes Paramounter's optimized peak height trades more extracted metabolites for a higher rate of false positives, so a row can look strong without reflecting feature quality.
- Metadata must be recorded per dataset at the time of processing; if sample type, acquisition mode, or MS vendor is not captured, the robustness claim the table is meant to support cannot be evaluated.

## Evidence
