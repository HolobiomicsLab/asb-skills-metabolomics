---
name: metabolomic-feature-recovery-quantification
description: Use when comparing LC-MS acquisition workflows (DDA vs DIA vs hybrid
  DaDIA) and you need to measure how many DDA-derived metabolic features are recovered
  in the combined data and assess their peak-shape reliability.
when_to_use_negative:
- Input is already a final metabolite annotation table rather than raw mzXML feature
  extraction output — the skill requires re-running feature extraction to compare
  DDA vs DaDIA feature lists.
- Data were acquired without a pooled QC sample run in DDA mode alongside individual
  samples run in DIA mode; the rescue step depends on having both acquisition modes
  represented.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DaDIA.R
  role: R script that extracts metabolic features from combined DDA/DIA data and identifies
    DDA features rescued in the DaDIA result
  repo: https://github.com/HuanLab/DaDIA.git
- name: XCMS
  role: R package (version 3.11.4+) underlying feature detection in DaDIA.R for LC-MS
    mzXML processing
  repo: https://github.com/HuanLab/DaDIA.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:d383152d01b3f0572ecbbe7c3352141f4b8961d27fcd75221d6dbbb0971f59aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolomic-feature-recovery-quantification

## Summary

Quantifies how many metabolic features detected in DDA mode are rescued in the combined DaDIA (DDA+DIA) dataset and validates the rescued features by manual extracted-ion chromatogram (EIC) inspection, yielding recovered-feature counts plus true-positive and false-positive rates.

## When to use

Use when comparing LC-MS acquisition workflows (DDA vs DIA vs hybrid DaDIA) and you need to measure how many DDA-derived metabolic features are recovered in the combined data and assess their peak-shape reliability.

## When NOT to use

- Input is already a final metabolite annotation table rather than raw mzXML feature extraction output — the skill requires re-running feature extraction to compare DDA vs DaDIA feature lists.
- Data were acquired without a pooled QC sample run in DDA mode alongside individual samples run in DIA mode; the rescue step depends on having both acquisition modes represented.

## Inputs

- DaDIA mzXML files (DDA and DIA-(SWATH)) from the repository demo data
- DDA feature list (metabolic features detected in DDA mode)
- Combined DaDIA feature table produced by DaDIA.R

## Outputs

- JSON record of rescue metrics: average rescued features from DDA = 344, true-positive rate = 0.71, false-positive rate = 0.17, undetermined = 0.12
- Per-feature EIC inspection classifications (good / poor / undetermined peak shape)

## How to apply

Process the DaDIA mzXML files (DDA and DIA-(SWATH)) with DaDIA.R in R using XCMS version 3.11.4 or higher to extract metabolic features from each acquisition mode. Identify features present in the DDA feature list that also appear (are rescued) in the combined DaDIA feature table, and compute the average number of rescued features per sample. For the urine study described here this yields 344 rescued features, approximately 22% of the 1591 features detected in DDA data. Validate the rescued features by manually inspecting extracted ion chromatograms (EICs): classify each EIC as good (true positive), poor/noisy (false positive), or undetermined, and compute rates — the reported values are 71% true-positive, 17% false-positive, and 12% undetermined.

## Related tools

- **DaDIA.R** (R script that extracts metabolic features from combined DDA/DIA data and identifies DDA features rescued in the DaDIA result) — https://github.com/HuanLab/DaDIA.git
- **XCMS** (R package (version 3.11.4+) underlying feature detection in DaDIA.R for LC-MS mzXML processing) — https://github.com/HuanLab/DaDIA.git

## Examples

```
Rscript DaDIA.R  # process demo mzXML files and compute rescued DDA feature count
```

## Evaluation signals

- Rescued-feature count matches the reported average of 344 features per sample for the urine study.
- True-positive rate equals 0.71 (features with good chromatographic peak shapes on EIC inspection).
- False-positive rate equals 0.17 (features with poor or noisy peak shapes).
- Rescued features represent approximately 22% of the 1591 average DDA features.
- Sum of true-positive (0.71), false-positive (0.17), and undetermined (0.12) fractions equals 1.

## Limitations

- Rescue is defined only relative to features present in the DDA feature list; features detected only in DIA mode are not counted by this metric.
- True/false-positive classification relies on manual EIC inspection, which is subjective; the article reports a separate undetermined fraction (12%) for ambiguous peak shapes.

## Evidence

- [results] Manual EIC inspection rates of rescued features: [evidence span withheld — anchor status: not_found]
