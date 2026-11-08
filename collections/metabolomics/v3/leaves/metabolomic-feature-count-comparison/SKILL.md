---
name: metabolomic-feature-count-comparison
description: Use when evaluating a hybrid DDA/DIA acquisition strategy against conventional
  DDA and DIA(SWATH) runs and you need reproducible feature/MS2-count tables with
  recomputed relative gains.
when_to_use_negative:
- Input is already a processed feature table where counts can be computed directly
  from the data rather than verified against a publication
- The analysis goal is metabolite annotation or spectral matching quality rather than
  acquisition-mode feature-count comparison
- Sample counts come from different sample sets or LC-MS modes (e.g. RP(−) or HILIC)
  that are not directly comparable to the RP(+) urine study figures
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DaDIA.R
  role: R program used to automatically extract metabolic features and perform metabolite
    annotation of the DaDIA dataset, producing the feature counts that feed the comparison
    table
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Feature-detection dependency (version 3.11.4 or above) required to run DaDIA.R
    and generate the detected-feature counts being compared
- name: metaMS
  role: Package dependency (version 1.25.1) required alongside XCMS and DaDIA.R for
    the metabolomic feature extraction workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:1340cb3bdbf9283592ae0979f494b9a5c22c849d2ecab9d4fce92720d874a4e5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - Skyline
  - XCMS
  - metaMS
  - statTarget
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolomics-feature-count-comparison
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomic-feature-count-comparison

## Summary

Compares detected-feature and MS2-feature counts across LC-MS acquisition workflows (DDA, DIA/SWATH, DaDIA) on the same samples, verifying relative percentage increases against the article's reported figures.

## When to use

Use when evaluating a hybrid DDA/DIA acquisition strategy against conventional DDA and DIA(SWATH) runs and you need reproducible feature/MS2-count tables with recomputed relative gains.

## When NOT to use

- Input is already a processed feature table where counts can be computed directly from the data rather than verified against a publication
- The analysis goal is metabolite annotation or spectral matching quality rather than acquisition-mode feature-count comparison
- Sample counts come from different sample sets or LC-MS modes (e.g. RP(−) or HILIC) that are not directly comparable to the RP(+) urine study figures

## Inputs

- Reported average detected-feature counts per acquisition workflow (e.g. 1591 DDA, 6680 DIA(SWATH), 7040 DaDIA)
- Reported average MS2-feature counts per workflow (e.g. 956 DDA, 4024 DIA(SWATH), 4901 DaDIA)
- Reported relative-increase percentages to verify (e.g. 342%, 413%, 5%, 22%)
- Source article Results section text containing the counts

## Outputs

- Structured CSV comparison table with one row per workflow (DDA, DIA(SWATH), DaDIA) and columns: detected features, MS2 features, relative increases
- Recomputed percentage increases (total and MS2) cross-checked against reported values

## How to apply

Extract the reported average detected-feature counts (DDA, DIA(SWATH), DaDIA) and average MS2-feature counts for the same sample set from the article's Results section — in the urine RP(+) study these are 1591/6680/7040 total features and 956/4024/4901 MS2 features. Independently recompute the relative increases (DaDIA vs DDA: 342% total, 413% MS2; DaDIA vs DIA(SWATH): 5% total, 22% MS2) from the raw counts and check they agree with the reported percentages. Assemble one row per workflow with columns for detected features, MS2 features, and the relative increases, then write the table to a CSV such as urine_dadia_comparison_table.csv. Rationale: feature-count differences quantify whether the hybrid acquisition recovers more metabolite signal than either single mode, and recomputation guards against transcription errors.

## Related tools

- **DaDIA.R** (R program used to automatically extract metabolic features and perform metabolite annotation of the DaDIA dataset, producing the feature counts that feed the comparison table) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Feature-detection dependency (version 3.11.4 or above) required to run DaDIA.R and generate the detected-feature counts being compared)
- **metaMS** (Package dependency (version 1.25.1) required alongside XCMS and DaDIA.R for the metabolomic feature extraction workflow)

## Examples

```
python scripts/compare_feature_counts.py --dda 1591 --dia 6680 --dadia 7040 --dda_ms2 956 --dia_ms2 4024 --dadia_ms2 4901 --out urine_dadia_comparison_table.csv
```

## Evaluation signals

- Every cell value in the output table must match the article's reported figures: 1591/6680/7040 total features and 956/4024/4901 MS2 features for DDA, DIA(SWATH), and DaDIA
- Recomputed relative increases must agree with the reported values: 342% (DaDIA vs DDA total), 413% (DaDIA vs DDA MS2), 5% (DaDIA vs DIA(SWATH) total), 22% (DaDIA vs DIA(SWATH) MS2)
- Output CSV must contain exactly one row per acquisition workflow (DDA, DIA(SWATH), DaDIA) with columns for detected features, MS2 features, and relative increases
- Statistical significance for the count differences should be consistent with the article's reported p ≤ 0.001 (total features) and p < 0.001 (MS2 features)
- Counts must originate from the same sample set (e.g. human urine RP(+) mode) and comparable chromatographic/MS conditions

## Limitations

- Counts are averaged per workflow and therefore mask sample-level variability; the article reports averages over replicate urine samples
- The comparison depends on consistent blank filtering and peak-shape criteria across workflows; differences in filtering (e.g. 3× method-blank intensity cutoff) can change counts
- MS2 feature counts depend on the DDA spectral quality of pooled QC samples; poor QC spectra can understate the DaDIA advantage
- The README notes the number of samples in the script must agree with the real number of DIA mzXML files, so miscounted inputs will invalidate downstream counts

## Evidence

- [results] average number of detected features are 1591, 6680, and 7040 in DDA, DIA(SWATH), and DaDIA work flows respectively: [evidence span withheld — anchor status: not_found]
- [results] average number of MS2 features are 956, 4024, and 4901 in DDA, DIA(SWATH), and DaDIA work flows: [evidence span withheld — anchor status: not_found]
